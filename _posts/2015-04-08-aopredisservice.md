---
layout: post
title: "利用AOP和Redis在service层做注解缓存"
description: ""
category: 缓存
tags: []
---
{% include JB/setup %}

> 一般缓存都有缓存策略，不是什么数据都缓存。为了提高缓存的命中率，会根据实际业务实现一些缓存算法。这篇文章不是讲缓存算法，而是通过注解方式在service层上更容易实现缓存的存取操作。
  
1.注解，在service方法上添加此注解表示该service返回的数据将被缓存，若缓存命中直接返回数据，否则执行方法获得数据再存缓存最后返回。

	package com.tcl.mie.appstore.biz.util;

	import java.lang.annotation.Documented;
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;

	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.METHOD})
	@Documented
	public @interface Cache {
    	long expireMinutes() default 10080L;

    	int[] pages() default {1};
	}

2.AOP过滤器，再service操作前是否有注解，进行相应的缓存操作。

	package com.tcl.mie.appstore.biz.util;

	import java.lang.reflect.Method;
	import java.util.Arrays;
	import java.util.concurrent.TimeUnit;
	
	import org.aspectj.lang.ProceedingJoinPoint;
	import org.aspectj.lang.annotation.Around;
	import org.aspectj.lang.annotation.Aspect;
	import org.aspectj.lang.annotation.Pointcut;
	import org.aspectj.lang.reflect.MethodSignature;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.redis.core.RedisTemplate;
	import org.springframework.stereotype.Component;
	
	import com.tcl.mie.appstore.persistence.vo.Pagination;
	
	/**
	 * 
	 * 创建人: xiaolong.hou
	 * 
	 * 创建时间：Mar 30, 2015 7:07:29 PM
	 * 
	 * @ClassName: CacheServiceInterceptor
	 * @Description: service层的缓存
	 */
	@Aspect
	@Component
	public class CacheServiceInterceptor {
	
	    @Autowired
	    private RedisTemplate<String, Object> redisTemplate;
	
	    @Pointcut("execution(* com.tcl.mie.appstore.biz.service..*.*(..))")
	    public void pointCut() {}
	    @Around("pointCut())")
    	    @SuppressWarnings("all")
            public Object cacheAround(ProceedingJoinPoint call) throws Throwable {
       		 Object ret = null;
       		 MethodSignature signature = (MethodSignature) call.getSignature();
       		 Method method = signature.getMethod();
       		 Cache cacheAnno = method.getAnnotation(Cache.class);

       		 // 判断是否注解
       		 if (cacheAnno != null) {
       		     StringBuilder sb =
       		             new StringBuilder(150)
       		                     .append(call.getSignature().getDeclaringTypeName())
       		                     .append(".").append(method.getName());
       		     int pageNow = 0;
       		     // 获得参数
       		     for (Object obj : call.getArgs()) {
       		         sb.append(".").append(obj.getClass().getSimpleName());
       		         // 判断是否有分页
       		         if (obj instanceof Pagination) {
       		             // 获取当前查询页
       		             pageNow = ((Pagination) obj).getPageNo();
       		             sb.append("=").append(pageNow).append(",")
       		                     .append(((Pagination) obj).getPageSize());
       		         } else {
       		             sb.append("=").append(obj);
       		         }
       		     }
       		     String key = sb.toString();
       		     // 判断redis中是否存在key
       		     if ((ret = redisTemplate.opsForValue().get(key)) == null) {
       		         ret = call.proceed();
       		         // 缓存条件： 分页存在且满足指定缓存页；不存在分页
       		         if ((pageNow > 0 && Arrays.binarySearch(cacheAnno.pages(),
       		                 pageNow) >= 0) || pageNow == 0) {
				redisTemplate.opsForValue().set(key, ret);
                    		redisTemplate.expire(key, cacheAnno.expireMinutes(),
                            			TimeUnit.MINUTES);
                	 }		
		     }
        	} else {
            		ret = call.proceed();
        	}
        	return ret;
    	    }
	}

3.总结  
**只是通过过滤器来进行缓存操作，各种缓存策略还需根据实际业务场景自己实现算法，这里用到来反射，有些朋友会考虑到反射再运行时的性能问题，可以使用字节码增强技术，提高代码运行性能。这里缓存的key时package.class.method** 
