---
layout: post
title: "实践Spring Cloud之Zuul"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Zuul
---

Spring Cloud Zuul是基于[Netflix Zuul](https://github.com/Netflix/zuul)的封装，实现路由和统一校验、认证和授权等过滤处理，同时基于Ribbon实现负载均衡，基于hystrix实现断路器，适用于构建API网关。

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

zuul依赖了hystrix、ribbon和actuator

```java
@EnableZuulProxy
```
新增/routes端点

## 路由

path到url或serviceId的映射，path支持Ant语法匹配

### 传统路由方式

不基于注册中心

#### 单实例配置

```properties
# zuul.routes.<route>.path=
zuul.routes.route-user-service.path=/user-service/**
# zuul.routes.<route>.url=
zuul.routes.route-user-service.url=http://localhost:8080/
```

#### 多实例配置

```properties
# zuul.routes.<route>.path=
zuul.routes.route-user-service.path=/user-service/**
# zuul.routes.<route>.serviceId=
zuul.routes.route-user-service.serviceId=user-service

ribbon.eureka.enabled=false
# <serviceId>.ribbon.listOfServers
user-service.ribbon.listOfServers=http://localhost:8080/,http://localhost:8081/
```

### 面向服务的路由

基于注册中心（需要依赖eureka）

```properties
# zuul.routes.<route>.path=<path>
zuul.routes.route-user-service.path=/user-service/**
# zuul.routes.<route>.serviceId=<serviceId>
zuul.routes.user-service.serviceId=user-service
```

```properties
# zuul.routes.<serviceId>=<path>
zuul.routes.user-service=/user-service/**
```

#### 默认路由

serviceId作为route和path，自动映射，不需要人工维护。

可以设置不自动映射的规则匹配

```properties
#以user-开头的服务不自动映射
zuul.ignored-services=user-*
# 所有服务都不自动映射
zuul.ignored-services=*
```

```properties
# 忽略path映射
zuul.ignored-patterns=/**/hello/**

# 路由前缀
zuul.prefix=/api
# 默认会移除路由前缀
# 全局
zuul.stripPrefix=true
# 路由级
zuul.routes.<route>.strip-prefix=true

# forward
zuul.routes.<route>.url=forward:/local

# 全局关闭（超时）重试
zuul.retryable=false
# 路由级关闭（超时）重试
zuul.routes.<route>.retryable=false
```

### 自定义路由

`PatternServiceRouteMapper`

```java

```

### 动态路由

基于Spring Cloud Config实现动态配置，包括路由映射等

```java
@RefreshScope
@ConfigurationProperties("zuul")
public ZuulProperties zuulProperties(){
  	return new ZuulProperties();
}
```

Zuul基于路由顺序匹配，而不是最有匹配

## 过滤器

![zuul-ZuulFilter](/images/zuul-ZuulFilter.png)

`FilterProcessor`

### 请求生命周期

![zuul-lifecycle](/images/zuul-lifecycle.png)

* pre
* route
* post
* error

#### filterType



#### filterOrder

数字越小优先级越高

#### shouldFilter

是否需要过滤

#### run

```java
RequestContext ctx = RequestContext.getCurrentContext();
HttpServletRequest request = ctx.getRequest();

// 过滤掉请求，不进行路由
ctx.setSendZuulResponse(false);
// 设置返回的错误码
cts.setResponseStatusCode(401);
// 设置响应内容
ctx.setResponseBody("认证失败");
```



### 核心过滤器

#### Pre过滤器

#### route过滤器

#### post过滤器

##### SendErrorFilter

error.status_code：错误编码

error.message：错误描述

error.exception：异常对象

#### error过滤器

自定义ErrorFilter，用来处理没有处理的异常

自定义ErrorExtFilter，继承SendErrorFilter，用来处理post过滤器抛出的异常和响应（由于post过滤器抛出的异常不会再转给post过滤器，参见`ZuulServlet`）

异常信息

DefaultErrorAttributes

### 配置



```properties
# 禁用过滤器
zuul.<SimpleClassName>.<filterType>.disable=true
```
### 动态过滤器



## Cookie和头信息

默认会过滤掉敏感头信息（Cookie，Set-Cookie，Authorization），可以自定义

```properties
# 全局
zuul.sensitiveHeaders=
# 路由级
zuul.routes.<route>.customSensitiveHeaders=true
# 路由级
zuul.routes.<route>.sensitiveHeaders=
# 设置host
zuul.addHostHeader=true
```

## 高可用

zuul作为微服务，可以部署多个，内部请求可以基于服务注册和发布中心实现多活，但是外部一般需要在zuul前面增加负载均衡设施，比如nginx、F5等。