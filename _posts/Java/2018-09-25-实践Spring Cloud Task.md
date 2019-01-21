---
layout: post
title: "实践Spring Cloud Task"
date: 2018-09-25 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud spring-cloud-task
---

[Spring Cloud Task](https://spring.io/projects/spring-cloud-task), A short-lived microservices framework to quickly build applications that perform finite amounts of data processing. Simple declarative for adding both functional and non-functional features to Spring Boot apps.

## 快速开始
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-task</artifactId>
	<version>1.2.3.RELEASE</version>
</dependency>
```


```java
@SpringBootApplication
@EnableTask
public class SampleTask {

	@Bean
	public CommandLineRunner commandLineRunner() {
		return new HelloWorldCommandLineRunner();
	}

	public static void main(String[] args) {
		SpringApplication.run(SampleTask.class, args);
	}

	public static class HelloWorldCommandLineRunner implements CommandLineRunner {

		@Override
		public void run(String... strings) throws Exception {
			System.out.println("Hello, World!");
		}
	}
}
```

## database

数据库相关脚本在`spring-cloud-task-core-2.0.0.RELEASE.jar/org/springframework/cloud/task/`目录下，比如：`schema-mysql.sql`，包括`5`张表：

TASK_EXECUTION

TASK_EXECUTION_PARAMS

TASK_TASK_BATCH

TASK_SEQ

TASK_LOCK

## 与Spring Batch整合

## 与Spring Cloud Stream整合