---
layout: post
title: "实践Spring Enable系列之EnableScheduling"
date: 2017-08-04 13:41:00 +0800
categories: Java
tags: spring schedule cron
---

Enables Spring's scheduled task execution capability, similar to functionality found in Spring's `<task:*>`XML namespace. To be used on @`Configuration` classes as follows:

```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     // various @Bean definitions
 }
```

This enables detection of @Scheduled annotations on any Spring-managed bean in the container. For example, given a class MyTask

```java
 package com.myco.tasks;

 public class MyTask {

     @Scheduled(fixedRate=1000)
     public void work() {
         // task execution logic
     }
 }
```

the following configuration would ensure that MyTask.work() is called once every 1000 ms:

```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     @Bean
     public MyTask task() {
         return new MyTask();
     }
 }
```

Alternatively, if MyTask were annotated with @Component, the following configuration would ensure that its @Scheduled method is invoked at the desired interval:

```java
 @Configuration
 @EnableScheduling
 @ComponentScan(basePackages="com.myco.tasks")
 public class AppConfig {
 }
```

Methods annotated with @Scheduled may even be declared directly within @Configuration classes:

```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     @Scheduled(fixedRate=1000)
     public void work() {
         // task execution logic
     }
 }
```

By default, will be searching for an associated scheduler definition: either a unique `org.springframework.scheduling.TaskScheduler` bean in the context, or a `TaskScheduler` bean named "taskScheduler" otherwise; the same lookup will also be performed for a `java.util.concurrent.ScheduledExecutorService` bean. If neither of the two is resolvable, a local single-threaded default scheduler will be created and used within the registrar.

When more control is desired, a `@Configuration` class may implement `SchedulingConfigurer`. This allows access to the underlying `ScheduledTaskRegistrar` instance. For example, the following example demonstrates how to customize the `Executor` used to execute scheduled tasks:

```java
 @Configuration
 @EnableScheduling
 public class AppConfig implements SchedulingConfigurer {

     @Override
     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
         taskRegistrar.setScheduler(taskExecutor());
     }

     @Bean(destroyMethod="shutdown")
     public Executor taskExecutor() {
         return Executors.newScheduledThreadPool(100);
     }
 }
```

Note in the example above the use of `@Bean(destroyMethod="shutdown")`. This ensures that the task executor is properly shut down when the Spring application context itself is closed.

Implementing `SchedulingConfigurer` also allows for fine-grained control over task registration via the `ScheduledTaskRegistrar`. For example, the following configures the execution of a particular bean method per a custom `Trigger` implementation:

```java
 @Configuration
 @EnableScheduling
 public class AppConfig implements SchedulingConfigurer {

     @Override
     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
         taskRegistrar.setScheduler(taskScheduler());
         taskRegistrar.addTriggerTask(
             new Runnable() {
                 public void run() {
                     myTask().work();
                 }
             },
             new CustomTrigger()
         );
     }

     @Bean(destroyMethod="shutdown")
     public Executor taskScheduler() {
         return Executors.newScheduledThreadPool(42);
     }

     @Bean
     public MyTask myTask() {
         return new MyTask();
     }
 }
```

For reference, the example above can be compared to the following Spring XML configuration:

```xml
 <beans>
     <task:annotation-driven scheduler="taskScheduler"/>
     <task:scheduler id="taskScheduler" pool-size="42"/>
     <task:scheduled-tasks scheduler="taskScheduler">
         <task:scheduled ref="myTask" method="work" fixed-rate="1000"/>
     </task:scheduled-tasks>
     <bean id="myTask" class="com.foo.MyTask"/>
 </beans>
```

The examples are equivalent save that in XML a fixed-rate period is used instead of a custom Trigger implementation; this is because the task:namespace scheduled cannot easily expose such support. This is but one demonstration how the code-based approach allows for maximum configurability through direct access to actual componentry.