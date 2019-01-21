---
layout: post
title: "实践Spring Batch"
date: 2018-09-25 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-batch
---

[Spring Batch](https://spring.io/projects/spring-batch), A lightweight, comprehensive batch framework designed to enable the development of robust batch applications vital for the daily operations of enterprise systems.

Spring Batch provides reusable functions that are essential in processing large volumes of records, including logging/tracing, transaction management, job processing statistics, job restart, skip, and resource management. It also provides more advanced technical services and features that will enable extremely high-volume and high performance batch jobs through optimization and partitioning techniques. Simple as well as complex, high-volume batch jobs can leverage the framework in a highly scalable manner to process significant volumes of information.

## 快速开始
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

## Spring Batch Admin
已经被Spring Cloud Data Flow替代

## database
数据库相关脚本在`spring-batch-core-4.0.1.RELEASE.jar/org/springframework/batch/core/`目录下，比如：`schema-mysql.sql`和`schema-drop-mysql.sql`。
共计有`9`张表，以`BATCH_`开头，`6`张业务表，`3`张sequnce表。