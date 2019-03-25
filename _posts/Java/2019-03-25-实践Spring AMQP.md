---
layout: post
title: "实践Spring AMQP"
date: 2019-03-25 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-AMQP AMQP RabbitMQ
---

[Spring AMQP](https://spring.io/projects/spring-amqp) project applies core Spring concepts to the development of AMQP-based messaging solutions. It provides a "template" as a high-level abstraction for sending and receiving messages. It also provides support for Message-driven POJOs with a "listener container". These libraries facilitate management of AMQP resources while promoting the use of dependency injection and declarative configuration. In all of these cases, you will see similarities to the JMS support in the Spring Framework.

The project consists of two parts; spring-amqp is the base abstraction, and spring-rabbit is the RabbitMQ implementation.

## Features

- Listener container for asynchronous processing of inbound messages
- RabbitTemplate for sending and receiving messages
- RabbitAdmin for automatically declaring queues, exchanges and bindings