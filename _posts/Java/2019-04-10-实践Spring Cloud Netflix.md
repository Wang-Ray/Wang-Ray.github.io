---
layout: post
title: "实践Spring Cloud Netflix"
date: 2019-04-10 11:08:00 +0800
categories: Java
tags: spring spring-boot spring-cloud spring-cloud-netflix Netflix
---

[Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix) provides [Netflix OSS](https://netflix.github.io/) integrations for `Spring Boot` apps through autoconfiguration and binding to the Spring Environment and other Spring programming model idioms. With a few simple annotations you can quickly enable and configure the common patterns inside your application and build large distributed systems with battle-tested Netflix components. The patterns provided include Service Discovery (Eureka), Circuit Breaker (Hystrix), Intelligent Routing (Zuul) and Client Side Load Balancing (Ribbon)..

## Features

Spring Cloud Netflix features:

- Service Discovery: Eureka instances can be registered and clients can discover the instances using Spring-managed beans
- Service Discovery: an embedded Eureka server can be created with declarative Java configuration
- Circuit Breaker: Hystrix clients can be built with a simple annotation-driven method decorator
- Circuit Breaker: embedded Hystrix dashboard with declarative Java configuration
- Declarative REST Client: Feign creates a dynamic implementation of an interface decorated with JAX-RS or Spring MVC annotations
- Client Side Load Balancer: Ribbon
- External Configuration: a bridge from the Spring Environment to Archaius (enables native configuration of Netflix components using Spring Boot conventions)
- Router and Filter: automatic regsitration of Zuul filters, and a simple convention over configuration approach to reverse proxy creation