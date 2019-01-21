---
layout: post
title: "实践Spring MVC之验证器"
date: 2017-08-30 17:08:00 +0800
categories: Java
tags: spring-mvc validator jsr-303
---

## Spring Validator

### Validator接口

### ValidationUtils工具类

errorCode[.object.property]

@initBinder

## JSR-303

一次进行一个域验证，不通过立即停止返回。不是一次性全部验证完后把不通过的全部反馈出来。

| Constraint   | Description | Example |
| ------------ | ----------- | ------- |
| @AssertFalse |             |         |
| @AssertTrue  |             |         |
| @DecimalMax  |             |         |
| @DecimalMin  |             |         |
| @Digits      |             |         |
| @Future      |             |         |
| @Max         |             |         |
| @Min         |             |         |
| @NotNull     |             |         |
| @Null        |             |         |
| @Past        |             |         |
| @Pattern     |             |         |
| @Size        |             |         |

每一个constaint都有默认错误消息描述，也可以自定义覆盖，key格式：constraint[.object].property

