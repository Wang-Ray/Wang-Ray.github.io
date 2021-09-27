---
layout: post
title: "实践Dubbo SPI之ThreadPool"
categories: Java
tags: Java Dubbo ThreadPool SPI
---



```java
/**
 * ThreadPool
 */
@SPI("fixed")
public interface ThreadPool {

    /**
     * Thread pool
     *
     * @param url URL contains thread parameter
     * @return thread pool
     */
    @Adaptive({Constants.THREADPOOL_KEY})
    Executor getExecutor(URL url);

}
```





### Dubbo 2.7.13

Reject-edExecutionHandler为继承AbortPolicy的AbortPolicyWithReport

#### cached

| 属性        | 默认值            | 描述                         |
| ----------- | ----------------- | ---------------------------- |
| corethreads | 0                 | 核心线程数                   |
| threads     | Integer.MAX_VALUE | 最大线程数                   |
| queues      | 0                 | 队列长度                     |
| alive       | 60*1000           | 冗余线程可空闲时间，单位毫秒 |

备注：由于使用了dubbo-spring-boot-starter 2.7.13，也可以直接配置驱动，例如：

```properties
dubbo.protocol.threadpool=cached
dubbo.protocol.parameters.corethreads=10
dubbo.protocol.parameters.alive=60000
dubbo.protocol.threads=200
dubbo.protocol.queues=0
```

#### fixed

| 属性    | 默认值 | 描述                              |
| ------- | ------ | --------------------------------- |
| threads | 200    | 固定线程数量，corethreads=threads |
| queues  | 0      | 队列长度                          |

alive固定为0

备注：由于使用了dubbo-spring-boot-starter 2.7.13，也可以直接配置驱动，例如：

```properties
dubbo.protocol.threadpool=fixed
dubbo.protocol.threads=200
dubbo.protocol.queues=0
```

#### limited

| 属性        | 默认值 | 描述       |
| ----------- | ------ | ---------- |
| corethreads | 0      | 核心线程数 |
| threads     | 200    | 最大线程数 |
| queues      | 0      | 队列长度   |

alive固定为Long.MAX_VALUE，可以理解为线程不会被回收

备注：由于使用了dubbo-spring-boot-starter 2.7.13，也可以直接配置驱动，例如：

 ```properties
dubbo.protocol.threadpool=limited
dubbo.protocol.parameters.corethreads=10
dubbo.protocol.threads=200
dubbo.protocol.queues=0
 ```

### Dubbo 2.5.10

RejectedExecutionHandler为继承AbortPolicy的AbortPolicyWithReport

#### cached

| 属性        | 默认值            | 描述                         |
| ----------- | ----------------- | ---------------------------- |
| corethreads | 0                 | 核心线程数                   |
| threads     | Integer.MAX_VALUE | 最大线程数                   |
| queues      | 0                 | 队列长度                     |
| alive       | 60*1000           | 冗余线程可空闲时间，单位毫秒 |

备注：由于使用了`spring-boot-starter-dubbo` 1.0.0，也可以直接配置驱动，例如：

```properties
spring.dubbo.protocol.threadpool=cached
spring.dubbo.protocol.parameters.corethreads=10
spring.dubbo.protocol.parameters.alive=60000
spring.dubbo.protocol.threads=200
spring.dubbo.protocol.queues=0
```

#### fixed

| 属性    | 默认值 | 描述                              |
| ------- | ------ | --------------------------------- |
| threads | 200    | 固定线程数量，corethreads=threads |
| queues  | 0      | 队列长度                          |

alive固定为0

备注：由于使用了`spring-boot-starter-dubbo` 1.0.0，也可以直接配置驱动，例如：

```properties
spring.dubbo.protocol.threadpool=fixed
spring.dubbo.protocol.threads=200
spring.dubbo.protocol.queues=0
```

#### limited

| 属性        | 默认值 | 描述       |
| ----------- | ------ | ---------- |
| corethreads | 0      | 核心线程数 |
| threads     | 200    | 最大线程数 |
| queues      | 0      | 队列长度   |

alive固定为Long.MAX_VALUE，可以理解为线程不会被回收

备注：由于使用了`spring-boot-starter-dubbo` 1.0.0，也可以直接配置驱动，例如：

 ```properties
spring.dubbo.protocol.threadpool=limited
spring.dubbo.protocol.parameters.corethreads=10
spring.dubbo.protocol.threads=200
spring.dubbo.protocol.queues=0
 ```

