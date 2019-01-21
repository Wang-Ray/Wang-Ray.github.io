---
layout: post
title: "实践Spring Cloud之Bus"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Bus
---

`/bus/refresh`端点

## kafka

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-kafka</artifactId>
</dependency>
```

```properties
# kafka
spring.cloud.stream.kafka.binder.brokers=192.168.70.139:9092
# zookeeper
spring.cloud.stream.kafka.binder.zk-nodes=192.168.70.139:2181
```



topic：springCloudBus

## RabbitMQ

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```



## 指定范围刷新

使用destination参数，基于PathMatcher匹配实例名，例如：

```
/bus/refresh?destination=customers:9000
/bus/refresh?destination=customers:**
```

