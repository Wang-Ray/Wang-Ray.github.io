---
layout: post
title: "实践Spring Cloud"
date: 2017-10-31 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud
---

[Spring Cloud](http://projects.spring.io/spring-cloud/)构建在Spring Boot之上，提供了微服务体系和架构等相关的管理，例如：服务治理、配置管理等。[Spring Cloud Reference Manual](https://cloud.spring.io/spring-cloud-static/current/)

## 依赖

版本

```xml
<properties>
	<spring-cloud.version>Dalston.SR4</spring-cloud.version>
</properties>
```

spring cloud依赖管理

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

spring-cloud-dependencies继承spring-cloud-dependencies-parent

## Spring Cloud Context

Spring Cloud Context provides utilities and special services for the `ApplicationContext` of a Spring Cloud application (bootstrap context, encryption, refresh scope and environment endpoints)

bootstrap.properties/yml

## Spring Cloud Commons

Spring Cloud Commons is a set of abstractions and common classes used in different Spring Cloud implementations (eg. Spring Cloud Netflix vs. 
Spring Cloud Consul).

## @SpringCloudApplication

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public @interface SpringCloudApplication {
}
```

`@SpringCloudApplication`包含了`@SpringBootApplication`，`@EnableDiscoveryClient`，`@EnableCircuitBreaker`这三个注解。



## 参考

[Spring Cloud中国社区](http://springcloud.cn/)

[Spring Cloud中文网](https://springcloud.cc/)

《Spring Cloud微服务实战》，[源码地址（Github）](https://github.com/Wang-Ray/SpringCloudBook-master)

《spring cloud与Docker微服务架构实战》，[源代码（Github）](https://github.com/Wang-Ray/spring-cloud-book)

《重新定义Spring Cloud》，[源码地址（Github）](https://github.com/SpringCloud/spring-cloud-code)

《Spring Cloud微服务架构开发实战》，[源代码（Github）](https://github.com/Wang-Ray/spring-cloud-microservices-development)