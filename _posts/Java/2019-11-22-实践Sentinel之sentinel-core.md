---
layout: post
title: "实践Sentinel之sentinel-core"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---

## 概念

### 资源（Resource）

可以定义为能够被控制的内容，可以是一段代码，一个方法，一个类等。

### 规则（Rule）

控制`资源`的规则，比如流量控制规则、熔断降级规则、系统保护规则等。



## 流量控制

阈值类型（grade）：QPS、线程数

流控模式（strategy）：直接、关联、链路

流控效果（controlBehavior）：快速失败（直接拒绝）、Warm Up、排队等待（Rate Limiter）

Warm Up

预热时长（warmUpPeriodSec）

排队等待

超时时长（maxQueueingTimeMs）

注意：若使用除了直接拒绝之外的流量控制效果，则调用关系限流策略（strategy）会被忽略。

来源（limitApp）:比如Dubbo的ApplicationConfig的name

集群：单机均摊、总体阈值

FlowRuleManager

FlowRule

## 容错降级

DegradeRuleManager

DegradeRule

## 热点限流

ParamFlowRuleManager

ParamFlowRule

## 系统保护

SystemRuleManager

SystemRule

## 授权规则

AuthorityRuleManager

AuthorityRule

## Tracer

业务异常跟踪统计

##　ContextUtil

调用链路上下文