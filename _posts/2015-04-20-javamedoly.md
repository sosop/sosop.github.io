---
layout: post
title: "javamedoly的使用"
description: "javamelody"
category: java语言
tags: []
---
{% include JB/setup %}

__开发中的一款非常便利的监控工具__  

步骤：  
在maven库中找到javamelody，加入项目  
	
	<dependency>
		<groupId>net.bull.javamelody</groupId>
		<artifactId>javamelody-core</artifactId>
		<version>1.55.0</version>
	</dependency>

配置监控数据源：  

	<!-- monitoring -->
	<bean id="springDataSourceBeanPostProcessor" class="net.bull.javamelody.SpringDataSourceBeanPostProcessor"/>
	<bean id="wrappedDataSource" class="net.bull.javamelody.SpringDataSourceFactoryBean">
		<property name="targetName" value="dataSource" />
	</bean>
	<!-- monitoring -->

__注：放到所有配置文件前__  

配置spring监控：  

	<!-- monitoring -->
	<bean id="facadeMonitoringAdvisor" class="net.bull.javamelody.MonitoringSpringAdvisor">
		<property name="pointcut">
			<bean class="org.springframework.aop.support.JdkRegexpMethodPointcut">
				<property name="pattern" value="com.tcl.mie.appstore.*.*" />
			</bean>
		</property>
	</bean>
	<!-- monitoring -->

数据源配置（监控SQL）：

	<property name="dataSource" ref="wrappedDataSource" />

web.xml配置：  

	<!-- monitor -->
	<filter>   
	    <filter-name>monitoring</filter-name>
	    <filter-class>net.bull.javamelody.MonitoringFilter</filter-class>
	    <init-param>  
	        <param-name>log</param-name>
	        <param-value>true</param-value> 
	    </init-param> 
	</filter>   
	<filter-mapping>
	    <filter-name>monitoring</filter-name>
	    <url-pattern>/*</url-pattern>   
	</filter-mapping>   
	<listener>   
	    <listener-class>net.bull.javamelody.SessionListener</listener-class>
	</listener>
	<!-- monitor -->
