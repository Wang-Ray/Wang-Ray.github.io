---
layout: post
title: "实践Spring Cloud Alibaba"
date: 2019-04-04 11:08:00 +0800
categories: Java
tags: spring spring-boot spring-cloud Alibaba
---

spring cloud alibaba依赖管理

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>com.alibaba.cloud</groupId>
			<artifactId>spring-cloud-alibaba-dependencies</artifactId>
          	<version>2.1.0.RELEASE</version>
  			<type>pom</type>
 			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

spring-cloud-alibaba-dependencies继承spring-cloud-dependencies-parent

```xml
<parent>
	<artifactId>spring-cloud-dependencies-parent</artifactId>
	<groupId>org.springframework.cloud</groupId>
	<version>2.1.6.RELEASE</version>
 	<relativePath/>
</parent>
```



[Spring Cloud Alibaba【Github】](https://github.com/spring-cloud-incubator/spring-cloud-alibaba) provides a one-stop solution for application development for the distributed solutions of Alibaba middleware.

[微服务领域是不是要变天了？Spring Cloud Alibaba正式入驻Spring Cloud官方孵化器！](https://www.cnblogs.com/zuoxiaolong/p/sca1.html)