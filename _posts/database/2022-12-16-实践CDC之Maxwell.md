---
layout: post
categories: 数据库
tags: database CDC Maxwell binlog
---

[Maxwell's Daemon (maxwells-daemon.io)](https://maxwells-daemon.io/) an application that reads MySQL binlogs and writes row updates as JSON to Kafka, Kinesis, or other streaming platforms. Maxwell has low operational overhead, requiring nothing but mysql and a place to write to. Its common use cases include ETL, cache building/expiring, metrics collection, search indexing and inter-service communication. Maxwell gives you some of the benefits of event sourcing without having to re-architect your entire platform. [Maxwell (github)](https://github.com/zendesk/maxwell)