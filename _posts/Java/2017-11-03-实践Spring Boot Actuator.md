---
layout: post
title: "实践Spring Boot Actuator"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot-actuator java actuator
---



```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

提供了13个端点（endpoint）

HTTP方法 路径 描述
GET /autoconfig 提供了一份自动配置报告,记录哪些自动配置条件通过了,哪些没通过
GET /configprops 描述配置属性(包含默认值)如何注入Bean
GET /beans 描述应用程序上下文里全部的Bean,以及它们的关系
GET /dump 获取线程活动的快照
GET /env 获取全部环境属性
GET /env/{name} 根据名称获取特定的环境属性值
GET /health 报告应用程序的健康指标,这些值由 HealthIndicator 的实现类提供
GET /info 获取应用程序的定制信息,这些信息由 info 打头的属性提供
GET /mappings 描述全部的URI路径,以及它们和控制器(包含Actuator端点)的映射关系
GET /metrics 报告各种应用程序度量信息,比如内存用量和HTTP请求计数
GET /metrics/{name} 报告指定名称的应用程序度量值
POST /shutdown 关闭应用程序,要求 endpoints.shutdown.enabled 设置为 true
GET /trace 提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)



GET /logfile 需要配置`logging.file`或`logging.path`属性才能开启