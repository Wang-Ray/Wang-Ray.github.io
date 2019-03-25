---
layout: post
title: "实践Spring Cloud之Stream"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Stream
---

[Spring Cloud Stream](https://spring.io/projects/spring-cloud-stream) is a framework for building highly scalable event-driven microservices connected with shared messaging systems.

The framework provides flexible programming model built on already established and familiar Spring idioms and best practices, including support for persistent pub/sub semantics, consumer groups, and stateful partitions.

## Binder Implementations

Spring Cloud Stream supports a variety of binder implementations and the following table includes the link to the GitHub projects.

- [RabbitMQ](https://github.com/spring-cloud/spring-cloud-stream-binder-rabbit)
- [Apache Kafka](https://github.com/spring-cloud/spring-cloud-stream-binder-kafka)
- [Kafka Streams](https://github.com/spring-cloud/spring-cloud-stream-binder-kafka/tree/master/spring-cloud-stream-binder-kafka-streams)
- [Amazon Kinesis](https://github.com/spring-cloud/spring-cloud-stream-binder-aws-kinesis)
- [Google PubSub *(partner maintained)*](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-pubsub-stream-binder)
- [Solace PubSub+ *(partner maintained)*](https://github.com/SolaceProducts/spring-cloud-stream-binder-solace)
- [Azure Event Hubs *(partner maintained)*](https://github.com/Microsoft/spring-cloud-azure/tree/master/spring-cloud-azure-eventhub-stream-binder)

The core building blocks of Spring Cloud Stream are:

- **Destination Binders**: Components responsible to provide integration with the external messaging systems.
- **Destination Bindings**: Bridge between the external messaging systems and application provided Producers and Consumers of messages (created by the Destination Binders).
- **Message**: The canonical data structure used by producers and consumers to communicate with Destination Binders (and thus other applications via external messaging systems).