---
layout: post
title: "实践Sentinel之sentinel-core"
date: 2019-11-22 11:08:00 +0800
categories: Java
tags: java Sentinel flow-control circuit-breaking
---

sentinel-core是sentinel的核心库

## 概念

### 资源（Resource）

控制的内容，可以是一段代码，一个方法，一个类等。

### 规则（Rule）

控制`资源`的规则，比如流量控制规则、熔断降级规则、系统保护规则等。

## 基本样例

#### STEP 1. 引入Sentinel Jar包

#### STEP 2. 定义资源

```java
   Entry entry = null;
        try {
	    entry = SphU.entry("HelloWorld");
            /*您的业务逻辑 - 开始*/
            System.out.println("hello world");
            /*您的业务逻辑 - 结束*/
	} catch (BlockException e1) {
            /*流控逻辑处理 - 开始*/
	    System.out.println("block!");
            /*流控逻辑处理 - 结束*/
	} finally {
	   if (entry != null) {
	       entry.exit();
	   }
	}
```

**注意：** `SphU.entry(xxx)` 需要与 `entry.exit()` 方法成对出现，匹配调用，否则会导致调用链记录异常，抛出 `ErrorEntryFreeException` 异常。

#### STEP 3. 定义规则

```java
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
```

#### STEP 4. 运行验证

## 定义资源

### 方式一：主流框架的默认适配

### 方式二：抛出异常的方式定义资源

### 方式三：返回布尔值方式定义资源

### 方式四：注解方式定义资源

### 方式五：异步调用支持

## 规则种类

### 流量控制

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

### 容错降级

DegradeRuleManager

DegradeRule

### 热点限流

ParamFlowRuleManager

ParamFlowRule

### 系统保护

SystemRuleManager

SystemRule

### 授权规则

AuthorityRuleManager

AuthorityRule

## 规则生效的效果



## 其他API

### Tracer

业务异常跟踪统计

###　ContextUtil

调用链路上下文