---
layout: post
title: "实践Spring Cloud之Consul"
date: 2017-10-31 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Consul
---

[Consul](https://www.consul.io/)是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。它包含了下面几个特性：

- 服务发现
- 健康检查
- Key/Value存储
- 多数据中心

### 安装

下载解压即可。

运行：

```shell
$ consul agent -dev
```

访问[Consul Web UI](http://localhost:8500/)，可以Consul上的各种数据，包括注册的服务、Key/Value存储等。

### Spring Cloud Client

Spring Cloud对Consul提供了封装支持，添加如下依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
<!-- consul健康检查需要actuator支持 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**注：spring-cloud-starter-consul-discovery已经依赖了spring-cloud-starter-ribbon**

添加Consul服务器配置

application.properties中增加如下配置：

```properties
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
```

