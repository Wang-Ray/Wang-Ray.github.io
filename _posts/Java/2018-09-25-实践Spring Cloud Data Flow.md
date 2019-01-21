---
layout: post
title: "实践Spring Cloud Data Flow"
date: 2018-09-25 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud spring-cloud-data-flow
---

[Spring Cloud Data Flow](https://cloud.spring.io/spring-cloud-dataflow) is a toolkit for building `data integration` and `real-time data processing` `pipelines`.
Spring Cloud Data Flow是一个用于构建`数据集成`和`实时数据处理`的`管道`的工具。

Pipelines consist of `Spring Boot` apps, built using the `Spring Cloud Stream` or `Spring Cloud Task` microservice frameworks. This makes Spring Cloud Data Flow suitable for a range of data processing use cases, from import/export to event streaming and predictive analytics.
管道由使用`Spring Cloud Stream`或`Spring Cloud Task`微服务框架构建的`Spring Boot`应用组成。使得Spring Cloud Data Flow适用于一些数据处理案例，从导入/导出到事件流和预测分析。

Overview
The Spring Cloud Data Flow server uses `Spring Cloud Deployer`, to deploy pipelines onto modern `runtimes` such as `Cloud Foundry`, `Kubernetes`, `Apache Mesos` or `Apache YARN`.

A selection of `pre-built` `stream` and `task/batch` starter apps for various data integration and processing scenarios facilitate learning and experimentation.

`Custom` `stream` and `task` applications, targeting different middleware or data services, can be built using the familiar `Spring Boot` style programming model.

A simple stream pipeline `DSL` makes it easy to specify which apps to deploy and how to connect outputs and inputs. A new composed task `DSL` was added in v1.2.

The `dashboard` offers a graphical editor for building new pipelines interactively, as well as views of deployable apps and running apps with metrics.

The Spring Could Data Flow server exposes a `REST API` for composing and deploying data pipelines. A separate `shell` makes it easy to work with the API from the command line.

## database
数据库相关脚本在`spring-cloud-dataflow-server-core-1.6.2.RELEASE.jar/schemas/`目录下，比如：`batch_task_indexes.sql`、`common.sql`、`deployment.sql`、`jpa.sql`、`streams.sql`和`tasks.sql`。
共计7张表：

DEPLOYMENT_IDS

APP_REGISTRATION

hibernate_sequence

STREAM_DEFINITIONS

STREAM_DEPLOYMENTS

TASK_DEFINITIONS

URI_REGISTRY