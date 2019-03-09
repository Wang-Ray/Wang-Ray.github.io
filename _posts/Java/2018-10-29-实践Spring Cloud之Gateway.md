---
layout: post
title: "实践Spring Cloud之Gateway"
date: 2018-10-29 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Zuul Gateway
---

[Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway) provides a library for building an API Gateway on top of Spring MVC. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.



## 开始

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



基于Java流式API自定义RouteLocator的方式定义路由信息

```java
/**
	 * 基本的转发
	 * 当访问http://localhost:8080/jd
	 * 转发到http://jd.com
	 * @param builder
	 * @return
	 */
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
				//basic proxy
				.route(r ->r.path("/jd")
						.uri("http://jd.com:80/").id("jd_route")
				).build();
	}
```

通过配置的方式

```yaml
spring:
  cloud:
    gateway:
      routes: #当访问http://localhost:8080/baidu,直接转发到https://www.baidu.com/
      - id: baidu_route
        uri: http://baidu.com:80/
        predicates:
        - Path=/baidu
```

开启端点，提供了关于Filter及routes的信息查询以及指定route信息更新的Rest API接口

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
  security:
    enabled: false
```



http://localhost:8080/actuator/gateway/routes

```json
[{"route_id":"jd_route","route_object":{"predicate":"org.springframework.cloud.gateway.support.ServerWebExchangeUtils$$Lambda$283/466056887@26bb1d5e"},"order":0}]
```



http://localhost:8081/actuator/gateway/routes

```json
[{"route_id":"baidu_route","route_definition":{"id":"baidu_route","predicates":[{"name":"Path","args":{"_genkey_0":"/baidu"}}],"filters":[],"uri":"http://baidu.com:80/","order":0},"order":0}]
```



## 路由断言工厂（Route Predicte Factory）

接口：`RoutePredicateFactory`

### Path

### After

### Before

### Between

### Cookie

### Header

### Host

### Method

### Query

### RemoteAddr

### Weight

权重

## 过滤器

### Global Filter

接口：`GlobalFilter`

### Gateway Filter

`GatewayFilter`

## 过滤器工厂（Gateway Filter Factory）

接口：`GatewayFilterFactory`

### AddRequestHeader

### AddRequestParameter

### RewritePath

### AddResponseHeader

### StringPrefix

### Retry

### Hystrix

