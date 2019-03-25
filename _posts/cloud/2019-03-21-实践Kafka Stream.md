---
layout: post
title: "实践Kafka Stream"
date: 2019-03-21 09:58:13 +0800
categories: 云计算
tags: cloud kafka stream kafka-stream
---



## get started



```shell
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic streams-plaintext-input

$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic streams-wordcount-output
```



```shell
$ bin/kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo
```



```shell
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic streams-wordcount-output --from-beginning  --property print.key=true --property print.value=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
[2019-03-21 10:42:26,236] WARN [Consumer clientId=consumer-1, groupId=console-consumer-93399] Error while fetching metadata with correlation id 2 : {streams-wordcount-output=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
hello	1
world	1
hello	2
ray	1
```



```shell
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic streams-plaintext-input
>hello world
>hello ray
```

