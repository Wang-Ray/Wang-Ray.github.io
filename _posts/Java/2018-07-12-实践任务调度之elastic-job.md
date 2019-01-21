---
layout: post
title: "实践任务调度之elastic-job"
date: 2018-07-12 11:08:00 +0800
categories: Java
tags: database schedule elastic-job quartz
---

[Elastic Job](http://elasticjob.io/)是一个分布式调度解决方案，由两个相互独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成。源代码地址：[elastic-job【Gitee】](https://gitee.com/elasticjob/elastic-job)，[elastic-job【Github】](https://github.com/elasticjob/elastic-job)

Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务；Elastic-Job-Cloud采用自研Mesos Framework的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。

## Elastic-Job-Lite
任务节点无中心，基于Zookeeper做同步控制，包括主节点选举，从节点管理，任务分片分发等。

### 分片
根据分片总数（shardingTotalCount）和作业分片策略（jobShardingStrategyClass），结合任务节点的数量，计算出每个任务节点上运行那些分片（分片编号从0起，也可以通过shardingItemParameter定义分片编号和业务字典的对应关系，`shardingContext.getShardingItem()`）。任务代码根据运行时传入的分片编号或对应的业务字典，处理其对应的数据。