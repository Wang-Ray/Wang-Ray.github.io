---
layout: post
title: "实践Spring Cloud之Turbine"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Zuul
---

实现对/hystrix.stream端点数据的聚合，对外提供统一服务

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-turbine</artifactId>
</dependency>
```

提供/turbine.stream端点