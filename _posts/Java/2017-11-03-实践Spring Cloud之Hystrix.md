---
layout: post
title: "实践Spring Cloud之Hystrix"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Hystrix
---

[Spring Cloud Hystrix]()是基于[Netflix Hystrix](https://github.com/Netflix/hystrix)的封装，实现断路器，提供服务降级、容错保护，提供了超时机制、线程隔离等一系列服务保护功能。

## 快速使用

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```



```java
@EnableCircuitBreaker
```



```java
// 断路器包装外部请求，指定后备方法【请求失败、超时、被拒绝以及断路器打开时都会执行回退逻辑】
@HystrixCommand(fallbackMethod = "helloFallback")
public String hello() {
	return restTemplate.getForEntity("http://HELLO-SERVICE/hello", String.class).getBody();
}

// 后备方法
public String helloFallback() {
	return "error";
}
```

hystrix.command.default.execution.isolation.thread.timeoutInMillisecond=2000

feign默认使用断路器（hystrix）包裹所有方法。

## 原理

基于命令模式实现。

HystrixCommand：用在依赖的服务返回单个操作结果

execute()

queue()

HystrixObservableCommand：用在依赖的服务返回多个操作结果

observe()

toObserve()

## Hystrix Dashboard

### client

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

提供/hystrix.stream端点

### server

Hystrix Dashboard实现可视化/hystrix.stream和/turbine.stream端点的数据，turbine参见[Spring Cloud Turbine](/2017/11/28/实践Spring-Cloud之Turbine.html)

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

http://localhost:8080/hystrix