---
layout: post
title: "实践Spring Enable系列之EnableAsync"
date: 2017-08-05 13:41:00 +0800
categories: Java
tags: spring asynchronous thread-pool
---

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {
//...
}
```

Enables Spring's asynchronous method execution capability, similar to functionality found in Spring's `<task:*>` XML namespace. To be used on @`Configuration`classes as follows:

```java
 @Configuration
 @EnableAsync
 public class AppConfig {
     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }
 }
```

where MyAsyncBeanis a user-defined type with one or methods annotated with @Async (or any custom annotation specified by the annotation() attribute).

The `mode()` attribute controls how advice is applied; if the mode is `AdviceMode.PROXY` (the default), then the other attributes control the behavior of the proxying.

Note that if the mode is set to `AdviceMode.ASPECTJ`, then the `proxyTargetClass()` attribute is obsolete. Note also that in this case the `spring-aspects`module JAR must be present on the classpath.

By default, a `SimpleAsyncTaskExecutor` will be used to process async method invocations. To customize this behavior, implement `AsyncConfigurer` and provide your own `Executor` through the `getExecutor()` method.

```java
 @Configuration
 @EnableAsync
 public class AppConfig implements AsyncConfigurer {

     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }

     @Override
     public Executor getAsyncExecutor() {
         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
         executor.setCorePoolSize(7);
         executor.setMaxPoolSize(42);
         executor.setQueueCapacity(11);
         executor.setThreadNamePrefix("MyExecutor-");
         executor.initialize();
         return executor;
     }
 }
```

For reference, the example above can be compared to the following Spring XML configuration:

```xml
 <beans>
     <task:annotation-driven executor="myExecutor"/>
     <task:executor id="myExecutor" pool-size="7-42" queue-capacity="11"/>
     <bean id="asyncBean" class="com.foo.MyAsyncBean"/>
 </beans>
```

the examples are equivalent save the setting of the thread name prefix of the Executor; this is because the the task:namespace executor element does not expose such an attribute. This demonstrates how the code-based approach allows for maximum configurability through direct access to actual componentry.

Note: In the above example the `ThreadPoolTaskExecutor` is not a fully managed Spring Bean. Add the `@Bean` annotation to the `getAsyncExecutor()`method if you want a fully managed bean. In such circumstances it is no longer necessary to manually call the `executor.initialize()` method as this will be invoked automatically when the bean is initialized.



### 线程

默认线程池`SimpleAsyncTaskExecutor`，线程名SimpleAsyncTaskExecutor-1，SimpleAsyncTaskExecutor-2