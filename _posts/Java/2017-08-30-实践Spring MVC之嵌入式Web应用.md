---
layout: post
title: "实践Spring MVC之结构和部署"
date: 2017-08-30 17:08:00 +0800
categories: Java
tags: spring-mvc theme
---

标准的Java Web项目的核心配置文件是web.xml，打包为war。

## 替换web.xml

Servlet 3.0容器会查找**javax.servlet.ServletContainerInitializer**接口的实现来配置Servlet（以前是通过web.xml），Spring 3.1提供了这个接口的实现类**SpringServletContainerInitializer**，这个实现类进而查找**WebApplicationInitializer**接口的实现类，Spring 3.2提供了这个接口的抽象实现**AbstractAnnotationConfigDispatcherServletInitializer**。

SpringBootServletInitializer是spring boot提供的WebApplicationInitializer的抽象实现。

DispatcherServletAutoConfiguration

EmbeddedServletContainerAutoConfiguration

WebMvcAutoConfiguration

## Spring Boot之jar部署

基于嵌入式Servlet容器可以实现jar部署。

spring boot web项目当package设置为jar时，采用如下方式启动：

```java
@SpringBootApplication
public class ReadingListApplication {
	public static void main(String[] args) {
		SpringApplication.run(ReadingListApplication.class, args);
	}
}
```

## Spring Boot之war部署

spring boot web项目当package设置为war时，同时定义scope为**provided**的tomcat覆盖starter-web中scope为**compile**的tomcat

```XML
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
  	<scope>provided</scope>
</dependency>
```
启动类：
```java
public class ReadingListInitializer extends SpringBootServletInitializer {
	@Override
   	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
      	return application.sources(ReadingListApplication.class);
   	}
}
```

