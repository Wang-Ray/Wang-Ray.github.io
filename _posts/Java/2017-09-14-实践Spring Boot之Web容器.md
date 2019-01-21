---
layout: post
title: "实践Spring Boot之Web容器"
date: 2017-09-07 17:08:00 +0800
categories: Java
tags: spring spring-boot web
---

Spring boot默认内嵌tomcat作为Servlet容器，也支持Jetty和Undertow等。

## 通用配置

配置类：org.springframework.boot.autoconfigure.web.ServerProperties，配置前缀：server，例如：

```properties
# 默认8080
server.port=80
```

## Session配置



## tomcat配置

配置类：org.springframework.boot.autoconfigure.web.ServerProperties.Tomcat，配置前置：server.tomcat，例如：

```properties
# 默认UTF-8
server.tomcat.uri-encoding=
# 默认off
server.tomcat.compression
```

## Jetty配置

### 替换为jetty

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```



配置类：org.springframework.boot.autoconfigure.web.ServerProperties.Jetty，配置前置：server.jetty，例如：

```
server.jetty.acceptors=
```

## 自定义

org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer

org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory