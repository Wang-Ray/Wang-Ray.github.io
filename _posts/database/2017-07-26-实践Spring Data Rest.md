---
layout: post
title: "实践Spring Data Rest"
date: 2017-07-26 10:00:00 +0800
categories: 数据库
tags: database nosql spring-data-rest rest
---



[Spring Data Rest](http://projects.spring.io/spring-data-rest/) is part of the umbrella [Spring Data](http://projects.spring.io/spring-data-rest/#) project and makes it easy to build **hypermedia-driven REST** web services on top of **Spring Data repositories**.

基于Spring Data Release Train - BOM Ingalls-SR6

```xml
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-rest-webmvc</artifactId>
	<version>2.6.3.RELEASE</version>
</dependency>
```

RepositoryRestMvcConfiguration

spring boot之RepositoryRestMvcAutoConfiguration

@RepositoryRestResource

@RestResource

@Param