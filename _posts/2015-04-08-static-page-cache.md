---
layout: post
title: "静态资源的缓存策略"
description: ""
category: 架构
tags: []
---
{% include JB/setup %}

> 在web服务器部署的时候，我们都会考虑将页面静态化，尤其是首页。这样不但提高访问速度，还有助于爬虫的抓去。一般静态资源有CDN；如果项目没用CDN，还可以用varnish、squid做静态缓存；如果都还没用到，那可以使用apache的mod_cache和nginx的proxy_cache。这篇文章不是描述怎么配置这些服务器缓存静态资源，主要是描述利用一些模版引擎生成静态页面，然后用一些缓存策略将静态资源缓存于nginx或apache，而不用去后端访问tomcat/jetty之类的web容器。

1.先来看一下下面这张图，一个非常简单的部署，nginx这里反向代理，作为负载均衡器，更重要的是静态资源的缓存服务器。这是最近做的一个小项目采取的静态资源的缓存方案。  
![静态资源缓存](/assets/pic/server.png  "静态缓存的简要部署")

2.将htm js css png等静态资源全部存在nginx服务器上，因为只是一个简单的项目，所以nginx的缓存完全满足业务要求，再做一层浏览器缓存。  
第一次nginx到后端服务器时，如果不存再静态htm，还是将访问动态资源，这个用拦截器实现，后端有个schedule每一小时生成静态页面，本项目使用velocity作为view层模版引擎。
  
生成静态资源代码：

	package com.tcl.mie.appstore.webapp.controller.staticVM;

	import java.io.RandomAccessFile;
	import java.io.StringWriter;
	import java.nio.ByteBuffer;
	import java.nio.channels.FileChannel;
	import java.nio.file.Files;
	import java.nio.file.LinkOption;
	import java.nio.file.Path;
	import java.nio.file.Paths;
	import java.util.Map;
	import java.util.Map.Entry;
	
	import javax.servlet.http.HttpServletRequest;
	
	import org.apache.velocity.Template;
	import org.apache.velocity.VelocityContext;
	import org.apache.velocity.app.Velocity;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	import org.springframework.web.servlet.view.velocity.VelocityConfigurer;
	
	import com.tcl.mie.appstore.biz.util.StringUtil;
	
	/**
	 * 
	 * 创建人: sosop.hou
	 * 
	 * 创建时间：Apr 1, 2015 2:53:26 PM
	 * 
	 * @ClassName: VmToHtml
	 * @Description: vm模板生成html内容，便于静态缓存
	 */
	@Component
	public class VmToHtml {
	
	    private final Logger LOG = LoggerFactory.getLogger(VmToHtml.class);
	
	    @Autowired
	    private VelocityConfigurer velocityConfigurer;
	
	    public String htmlString(HttpServletRequest request, String vmName,
	            Map<String, Object> vars) {
	        Template template =
	                velocityConfigurer.getVelocityEngine().getTemplate(
	                        StringUtil.append(vmName, ".vm"), "UTF-8");
	        VelocityContext context = new VelocityContext();
	        if (null != vars) {
	            for (Entry<String, Object> var : vars.entrySet()) {
	                context.put(var.getKey(), var.getValue());
	            }
	        }
	        String contextPath = request.getContextPath();
	        context.put("basePath", contextPath);
	        StringWriter sw = new StringWriter();
	        template.merge(context, sw);
	        return sw.toString().replaceAll("\\$basePath", contextPath)
	                .replaceAll("\\$\\{basePath\\}", contextPath);
	    }
	
	    public void htmlFile(HttpServletRequest request, String vmName,
	            Map<String, Object> vars, String htmlName) {
	        String html = htmlString(request, vmName, vars);
	        String storePath =
	                ((String) velocityConfigurer.getVelocityEngine().getProperty(
	                        Velocity.FILE_RESOURCE_LOADER_PATH));
	        Path path = Paths.get(storePath);
	        path = path.getParent().getParent().resolve("html").resolve(htmlName);
	        RandomAccessFile file = null;
	        FileChannel channel = null;
	        try {
	            file = new RandomAccessFile(path.toFile(), "rw");
	            channel = file.getChannel();
	            ByteBuffer buf = ByteBuffer.wrap(html.getBytes());
	            channel.write(buf);
	        } catch (Exception e) {
	            LOG.error(e.getMessage(), e);
	        } finally {
	            try {
	                if (channel != null && channel.isOpen()) {
	                    channel.close();
	                    channel = null;
	                }
	                if (file != null) {
	                    file.close();
	                    file = null;
	                }
	            } catch (Exception e) {
	                LOG.error(e.getMessage(), e);
	            }
	        }
	    }
	
	    public boolean htmlIsExists(String htmlName) {
	        Path path = getPath(htmlName);
	        if (Files.exists(path, LinkOption.NOFOLLOW_LINKS)) {
	            return true;
	        }
	        return false;
	    }
	
	    public long getLastModifyTime(String htmlName) {
	        Path path = getPath(htmlName);
	        return path.toFile().lastModified();
	    }
	
	    private Path getPath(String htmlName) {
	        String storePath =
	                ((String) velocityConfigurer.getVelocityEngine().getProperty(
	                        Velocity.FILE_RESOURCE_LOADER_PATH));
	        Path path =
	                Paths.get(storePath).getParent().getParent().resolve("html")
	                        .resolve(htmlName);
	        return path;
	    }
	}	

注解代码：

	package com.tcl.mie.appstore.webapp.util.annotation;

	import java.lang.annotation.Documented;
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.METHOD})
	@Documented
	public @interface StaticPage {
	    long expireMinutes() default 30L;
	}

定时任务代码：

	package com.tcl.mie.appstore.webapp.job;

	import java.io.IOException;
	import java.util.Properties;
	
	import org.apache.log4j.Logger;
	
	import com.tcl.mie.appstore.common.config.ConfigUtil;
	import com.tcl.mie.appstore.webapp.util.HttpRequestUtil;
	
	/**
	 * 
	 * 创建人: sosop.hou
	 * 
	 * 创建时间：Apr 3, 2015 1:58:13 PM
	 * 
	 * @ClassName: CreateStaticPageJob
	 * @Description: 定时创建静态页面
	 */
	public class CreateStaticPageJob {
	
	    private final static Logger LOG = Logger
	            .getLogger(CreateStaticPageJob.class);
	
	    public static void createStaticPages() throws IOException {
	        LOG.info("start create static page.");
	        Properties props = new Properties();
	        props.load(CreateStaticPageJob.class
	                .getResourceAsStream("/staticPage.properties"));
	        String rootUrl = ConfigUtil.getServerConfig().getString("rootUrl");
	        String[] filerUris = props.getProperty("filterUris").split(",");
	        for (String uri : filerUris) {
	            HttpRequestUtil.get(rootUrl, uri);
	        }
	    }
	}

拦截器代码：

	package com.tcl.mie.appstore.webapp.controller.staticVM;

	import java.lang.annotation.Annotation;
	import java.util.Date;
	import java.util.Enumeration;
	import java.util.HashMap;
	import java.util.Map;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.method.HandlerMethod;
	import org.springframework.web.servlet.HandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	import com.tcl.mie.appstore.biz.util.StringUtil;
	import com.tcl.mie.appstore.common.config.ConfigUtil;
	import com.tcl.mie.appstore.webapp.util.HttpRequestUtil;
	import com.tcl.mie.appstore.webapp.util.annotation.StaticPage;
	
	public class VisitStaticInterceptor implements HandlerInterceptor {
	
	    @Autowired
	    private VmToHtml vmToHtml;
	
	    private String baseDir = "/web/html";
	
	    private Map<String, Long> expired = new HashMap<>();
	
	    @Override
	    public boolean preHandle(HttpServletRequest request,
	            HttpServletResponse response, Object handler) throws Exception {
	        String htmlName = generateHtmlName(request);
	        if (!localRequest(request)
	                && isAnnotaition(handler, StaticPage.class)
	                && vmToHtml.htmlIsExists(htmlName)
	                && notExpire(
	                        request,
	                        htmlName,
	                        (StaticPage) getAnnotationInstance(handler,
	                                StaticPage.class))) {
	            response.sendRedirect(StringUtil.append(baseDir, "/", htmlName));
	            return false;
	        }   
	        return true;
	    }   
	
	    @Override
	    public void postHandle(HttpServletRequest request,
	            HttpServletResponse response, Object handler,
	            ModelAndView modelAndView) throws Exception {
	        String htmlName = generateHtmlName(request);
	        if (null != modelAndView && localRequest(request)
	                && isAnnotaition(handler, StaticPage.class)) {
	            vmToHtml.htmlFile(request, modelAndView.getViewName(),
	                    modelAndView.getModel(), htmlName);
	            expired.put(
	                    htmlName,
	                    vmToHtml.getLastModifyTime(htmlName)
	                            + ((StaticPage) getAnnotationInstance(handler,
	                                    StaticPage.class)).expireMinutes() * 60
	                            * 1000);
	        }
	    }
	
	    @Override
	    public void afterCompletion(HttpServletRequest request,
	            HttpServletResponse response, Object handler, Exception ex)
	            throws Exception {
	
	    }
	
	    @SuppressWarnings("rawtypes")
	    private boolean isAnnotaition(Object handler, Class clazz) {
	        if (!(handler instanceof HandlerMethod)) {
	            return false;
	        }
	        if (getAnnotationInstance(handler, clazz) != null) {
	            return true;
	        }
	        return false;
	    }
	
	    @SuppressWarnings({"unchecked", "rawtypes"})
	    private Annotation getAnnotationInstance(Object handler, Class clazz) {
	        return ((HandlerMethod) handler).getMethodAnnotation(clazz);
	    }
	
	    @SuppressWarnings("unchecked")
	    private String generateHtmlName(HttpServletRequest request) {
	        String paramsHtm = "_";
	        Enumeration<String> iterator = request.getParameterNames();
	        String paramName;
	        while (iterator.hasMoreElements()) {
	            paramName = iterator.nextElement();
	            paramsHtm =
	                    StringUtil.append(paramsHtm, paramName, "_",
	                            request.getParameter(paramName), "_");
	        }
	        String requestPath = request.getServletPath().replaceAll("/", "_");
	        return StringUtil.append(requestPath, paramsHtm, ".htm");
	    }
	
	    private boolean localRequest(HttpServletRequest request) {
	        String remoteHost = request.getRemoteHost();
	        if ("0:0:0:0:0:0:0:1".equals(remoteHost.trim())
	                || "localhost".equals(remoteHost.trim())
	                || "127.0.0.1".equals(remoteHost.trim())) {
	            return true;
	        }
	        return false;
	    }
	
	    @SuppressWarnings("unchecked")
	    private boolean notExpire(HttpServletRequest request, String htmlName,
	            StaticPage staticPage) {
	        long lastModifyTime;
	        // 判断是否保存过期时间
	        if (expired.containsKey(htmlName)) {
	            lastModifyTime = expired.get(htmlName);
	        } else {
	            lastModifyTime =
	                    vmToHtml.getLastModifyTime(htmlName)
	                            + staticPage.expireMinutes() * 60 * 1000;
	            expired.put(htmlName, lastModifyTime);
	        }
	
	        // 判断是够过期
	        long now = new Date().getTime();
	        if (now < lastModifyTime) {
	            return true;
	        } else {
	            Enumeration<String> iterator = request.getParameterNames();
	            String params = "?";
	            String paramName;
	            while (iterator.hasMoreElements()) {
	                paramName = iterator.nextElement();
	                params =
	                        StringUtil.append(params, paramName, "=",
	                                request.getParameter(paramName), "&");
	            }
	            HttpRequestUtil.get(
	                    ConfigUtil.getServerConfig().getString("rootUrl"),
	                    StringUtil.append(request.getServletPath(), params));
	        }
	        return false;
	    }
	}
