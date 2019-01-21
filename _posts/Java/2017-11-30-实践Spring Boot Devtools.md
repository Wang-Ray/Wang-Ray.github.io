---
layout: post
title: "实践Spring Boot Devtools"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring-boot spring-boot-devtools spring
---

[Spring Boot Devtools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>provided</scope>
</dependency>
```

## Eclipse

Eclipse设置了自动编译之后，修改类它会自动编译。

## IDEA

IDEA在非RUN或DEBUG情况下才会自动编译（需要设置Auto-Compile）。

勾选Build project automatically

Shift+Ctrl+Alt+/，选择Registry，然后勾选compiler.automake.allow.when.app.running