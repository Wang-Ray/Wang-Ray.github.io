---
layout: post
title: "实践Spring Cloud之Sleuth"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Sleuth
---

[Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) implements a distributed tracing solution for Spring Cloud, borrowing heavily from [Dapper](http://research.google.com/pubs/pub36356.html), [Zipkin](https://github.com/openzipkin/zipkin) and HTrace. For most users Sleuth should be invisible, and all your interactions with external systems should be instrumented automatically. You can capture data simply in logs, or by sending it to a remote collector service.

## Features

A Span is the basic unit of work. For example, sending an RPC is a new span, as is sending a response to an RPC. Span’s are identified by a unique 64-bit ID for the span and another 64-bit ID for the trace the span is a part of. Spans also have other data, such as descriptions, key-value annotations, the ID of the span that caused them, and process ID’s (normally IP address). Spans are started and stopped, and they keep track of their timing information. Once you create a span, you must stop it at some point in the future. A set of spans forming a tree-like structure called a Trace. For example, if you are running a distributed big-data store, a trace might be formed by a put request.

Spring Cloud Sleuth features:

- Adds trace and span ids to the Slf4J MDC, so you can extract all the logs from a given trace or span in a log aggregator.
- Provides an abstraction over common distributed tracing data models: traces, spans (forming a DAG), annotations, key-value annotations. Loosely based on HTrace, but Zipkin (Dapper) compatible.
- Instruments common ingress and egress points from Spring applications (servlet filter, rest template, scheduled actions, message channels, zuul filters, feign client).
- If `spring-cloud-sleuth-zipkin` is available then the app will generate and collect Zipkin-compatible traces via HTTP. By default it sends them to a Zipkin collector service on localhost (port 9411). Configure the location of the service using `spring.zipkin.baseUrl`.