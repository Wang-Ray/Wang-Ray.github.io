---
layout: post
title: "实践Spring Kafka"
date: 2019-03-22 09:58:13 +0800
categories: Java
tags: Java kafka spring spring-kafka
---

[Spring Kafka](https://spring.io/projects/spring-kafka) applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a "template" as a high-level abstraction for sending messages. It also provides support for Message-driven POJOs with `@KafkaListener` annotations and a "listener container". These libraries promote the use of dependency injection and declarative. In all of these cases, you will see similarities to the JMS support in the Spring Framework and RabbitMQ support in Spring AMQP.

[Spring for Apache Kafka Samples](https://github.com/spring-projects/spring-kafka/tree/master/samples)

[sample-spring-kafka](https://github.com/Wang-Ray/sample-spring-kafka)

[sample-spring-kafka1](<https://github.com/Wang-Ray/sample-spring-kafka1>)

## get started

```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>2.2.4</version>
</dependency>
```

启动类

```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    public static Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args).close();
    }

    @Autowired
    private KafkaTemplate<String, String> template;

    private final CountDownLatch latch = new CountDownLatch(3);

    @Override
    public void run(String... args) throws Exception {
        this.template.send("myTopic", "foo1");
        this.template.send("myTopic", "foo2");
        this.template.send("myTopic", "foo3");
        latch.await(60, TimeUnit.SECONDS);
        logger.info("All received");
    }

    @KafkaListener(topics = "myTopic")
    public void listen(ConsumerRecord<?, ?> cr) throws Exception {
        logger.info(cr.toString());
        latch.countDown();
    }

}
```

## KafkaAutoConfiguration

### KafkaProperties

### KafkaTemplate

#### ProducerListener

#### ReplyingKafkaTemplate

request/reply模式

### @KafkaListener

### ConsumerFactory

### ProducerFactory

### RecordMessageConverter

### KafkaTransactionManager

### KafkaJaasLoginModuleInitializer

### KafkaAdmin

### @SendTo

转发到指定topic