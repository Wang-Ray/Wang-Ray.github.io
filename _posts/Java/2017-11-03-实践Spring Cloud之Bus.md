---
layout: post
title: "实践Spring Cloud之Bus"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Bus
---

[Spring Cloud Bus](https://spring.io/projects/spring-cloud-bus) links nodes of a distributed system with a lightweight message broker. This can then be used to broadcast state changes (e.g. configuration changes) or other management instructions. The only implementation currently is with an AMQP broker as the transport, but the same basic feature set (and some more depending on the transport) is on the roadmap for other transports.

总线

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

