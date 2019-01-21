---
layout: post
title: "实践Spring Enable系列之EnableAspectJAutoProxy"
date: 2017-08-05 13:41:00 +0800
categories: Java
tags: spring AspectJ proxy aop
---

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
//...
}
```

切面：切点的容器，使用@Aspect定义

切点（PointCut）：拦截规则，一般分为基于方法（execution）或基于注解（@PointCut）

建言（Advice）：拦截植入的时机及动作，@Before、@After和@Around

连接点（JoinPoint）：符合拦截规则的每一个被拦截处

Enables support for handling components marked with AspectJ's `@Aspect` annotation, similar to functionality found in Spring's `<aop:aspectj-autoproxy>` XML element. To be used on @`Configuration` classes as follows:

```java
 @Configuration
 @EnableAspectJAutoProxy
 public class AppConfig {
     @Bean
     public FooService fooService() {
         return new FooService();
     }

     @Bean
     public MyAspect myAspect() {
         return new MyAspect();
     }
 }
```

Where FooService is a typical POJO component and MyAspect is an @Aspect-style aspect:

```java
 public class FooService {
     // various methods
 }
```

```java
 @Aspect
 public class MyAspect {
     @Before("execution(* FooService+.*(..))")
     public void advice() {
         // advise FooService methods as appropriate
     }
 }
```

In the scenario above, @EnableAspectJAutoProxy ensures that MyAspect will be properly processed and that FooService will be proxied mixing in the advice that it contributes. Users can control the type of proxy that gets created for `FooService` using the `proxyTargetClass()`attribute. The following enables **CGLIB-style 'subclass' proxies** as opposed to the default **interface-based JDK proxy** approach.

```java
 @Configuration
 @EnableAspectJAutoProxy(proxyTargetClass=true)
 public class AppConfig {
     // ...
 }
```

Note that `@Aspect` beans may be component-scanned like any other. Simply mark the aspect with both `@Aspect` and `@Component`:

```java
 package com.foo;

 @Component
 public class FooService { ... }

 @Aspect
 @Component
 public class MyAspect { ... }
```

Then use the @ComponentScan annotation to pick both up:

```java
 @Configuration
 @ComponentScan("com.foo")
 @EnableAspectJAutoProxy
 public class AppConfig {
     // no explicit @Bean definitions required
 }
```