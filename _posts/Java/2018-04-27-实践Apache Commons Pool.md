---
layout: post
title: "实践Apache Commons Pool"
date: 2018-04-27 11:08:00 +0800
categories: Java
tags: java pool commons-pool
---

[Apache Commons Pool](http://commons.apache.org/proper/commons-pool/)提供对象池API和相关现实。

版本2完全重写，性能和稳定性都得到了提升，还提供了跟踪和监控功能，需要JDK1.6或更高版本。



| Parameter          | Default          | Description                              |
| ------------------ | ---------------- | ---------------------------------------- |
| fairness           | false            | **是否公平获取（fifo），即先来先得**                   |
| maxTotal           | 8                | The maximum number of active connections that can be allocated from this pool at the same time, or negative for no limit.**版本1中属性名是maxActive【borrow时会检查】** |
| maxIdle            | 8                | The maximum number of connections that can remain idle in the pool, without extra ones being released, or negative for no limit.**【高峰过后流下maxIdle】【return时会检查】【建议>minIdle+正常并发数，避免频繁的销毁】** |
| minIdle            | 0                | The minimum number of connections that can remain idle in the pool, without extra ones being created, or zero to create none.**【空闲时保持住minIdle，以应对高峰的突然来临】【timeBetweenEvictionRunsMillis>0才有意义】** |
| maxWaitMillis      | -1(indefinitely) |                                          |
| blockWhenExhausted | true             | Sets whether to block when the `borrowObject()` method is invoked when the pool is exhausted (the maximum number of "active" objects has been reached). |

| Parameter                       | Default        | Description                              |
| ------------------------------- | -------------- | ---------------------------------------- |
| testOnCreate                    | false          | The indication of whether objects will be validated after creation. If the object fails to validate, the borrow attempt that triggered the object creation will fail. |
| testOnBorrow                    | false          | The indication of whether objects will be validated before being borrowed from the pool. If the object fails to validate, it will be dropped from the pool, and we will attempt to borrow another. |
| testOnReturn                    | false          | The indication of whether objects will be validated before being returned to the pool. |
| testWhileIdle                   | false          | The indication of whether objects will be validated by the idle object evictor (if any). If an object fails to validate, it will be dropped from the pool.【timeBetweenEvictionRunsMillis>0才有意义】 |
| timeBetweenEvictionRunsMillis   | -1             | The number of milliseconds to sleep between runs of the **idle** object evictor thread. When non-positive, no idle object evictor thread will be run.**默认线程名：commons-pool-EvictionTimer，实现类：BaseGenericObjectPool.Evictor** |
| numTestsPerEvictionRun          | 3              | The number of objects to examine during each run of the idle object evictor thread (if any).**踢出timeout的idle对象** |
| minEvictableIdleTimeMillis      | 1000 * 60 * 30 | The minimum amount of time an object may sit **idle** in the pool before it is eligable for eviction by the idle object evictor (if any).**可以理解为timeout，idle对象不一定timeout，可能还没到时间【idleTime=System.currentTimeMillis() - lastReturnTime】** |
| softMiniEvictableIdleTimeMillis | -1             | The minimum amount of time a connection may sit idle in the pool before it is eligible for eviction by the idle connection evictor, with the extra condition that at least "minIdle" connections remain in the pool. When miniEvictableIdleTimeMillis is set to a positive value, miniEvictableIdleTimeMillis is examined first by the idle connection evictor - i.e. when idle connections are visited by the evictor, idle time is first compared against miniEvictableIdleTimeMillis (without considering the number of idle connections in the pool) and then against softMinEvictableIdleTimeMillis, including the minIdle constraint.**可以理解为timeout，idle不一定timeout，可能还没到时间softMiniEvictableIdleTimeMillis+空闲数量是否大于minIdle**<br />if ((config.getIdleSoftEvictTime() < underTest.getIdleTimeMillis() &&         config.getMinIdle() < idleCount) \|\|         config.getIdleEvictTime() < underTest.getIdleTimeMillis()) {     return true; } |
| lifo                            | true           | True means that borrowObject returns the most recently used ("last in") connection in the pool (if there are idle connections available). False means that the pool behaves as a FIFO queue - connections are taken from the idle instance pool in the order that they are returned to the pool.**池使用规则按后进先出，即优先使用后归还的evictor检测in oldest-to-youngest order** |

| Parameter                                | Default                             | Description                              |
| ---------------------------------------- | ----------------------------------- | ---------------------------------------- |
| removeAbandonedOnMaintenance<br />removeAbandonedOnBorrow | false                               | Flags to remove abandoned connections if they exceed the removeAbandonedTimout. A connection is considered abandoned and eligible for removal if it has not been used for longer than removeAbandonedTimeout.**【长时间未被使用】**Creating a Statement, PreparedStatement or CallableStatement or using one of these to execute a query (using one of the execute methods) resets the lastUsed property of the parent connection.**【会更新最后使用时间，lastUseTime = lastBorrowTime】**Setting one or both of these to true can recover db connections from **poorly written applications which fail to close connections**.**【借了未还】**Setting removeAbandonedOnMaintenance to true removes abandoned connections on the maintenance cycle (when eviction ends). This property has no effect unless maintenance is enabled by setting timeBetweenEvicionRunsMillis to a positive value. If removeAbandonedOnBorrow is true, abandoned connections are removed each time a connection is borrowed from the pool, with the additional requirements thatgetNumActive() > getMaxTotal() - 3; andgetNumIdle() < 2【timeBetweenEvictionRunsMillis>0才有意义】 |
| removeAbandonedTimeout                   | 300                                 | Timeout in **seconds** before an abandoned connection can be removed. |
| logAbandoned                             | false                               | Flag to log stack traces for application code which abandoned a Statement or Connection. Logging of abandoned Statements and Connections adds overhead for every Connection open or new Statement because a stack trace has to be generated. |
| abandonedUsageTracking                   | false                               | If true, the connection pool records a stack trace every time a method is called on a pooled connection and retains the most recent stack trace to aid debugging of abandoned connections. There is significant overhead added by setting this to true. |
| logWriter                                | **new** PrintWriter(System.**out**) | Sets the log writer to be used by this configuration to log information on abandoned objects. |

| Parameter                  | Default               | Description                              |
| -------------------------- | --------------------- | ---------------------------------------- |
| swallowedExceptionListener | null                  | The listener used (if any) to receive notifications of exceptions unavoidably swallowed by the pool. |
| evictionPolicyClassName    | DefaultEvictionPolicy | Provides the default implementation of `EvictionPolicy` used by the pools. Objects will be evicted if the following conditions are met:the object has been idle longer than `GenericObjectPool.getMinEvictableIdleTimeMillis()` / `GenericKeyedObjectPool.getMinEvictableIdleTimeMillis()`there are more than `GenericObjectPool.getMinIdle()` / `GenericKeyedObjectPoolConfig.getMinIdlePerKey()` idle objects in the pool and the object has been idle for longer than `GenericObjectPool.getSoftMinEvictableIdleTimeMillis()` / `GenericKeyedObjectPool.getSoftMinEvictableIdleTimeMillis()` |

3个

![commons-pool-GenericObjectPoolConfig](/images/commons-pool-GenericObjectPoolConfig.png)

16个（12+1（fairness）+3（jmx*））

![commons-pool-BasicObjectPoolConfig](/images/commons-pool-BasicObjectPoolConfig.png)

默认配置（15）：

```java
public void setConfig(GenericObjectPoolConfig conf) {
     setLifo(conf.getLifo());
     setMaxIdle(conf.getMaxIdle());
     setMinIdle(conf.getMinIdle());
     setMaxTotal(conf.getMaxTotal());
     setMaxWaitMillis(conf.getMaxWaitMillis());
     setBlockWhenExhausted(conf.getBlockWhenExhausted());
     setTestOnCreate(conf.getTestOnCreate());
     setTestOnBorrow(conf.getTestOnBorrow());
     setTestOnReturn(conf.getTestOnReturn());
     setTestWhileIdle(conf.getTestWhileIdle());
     setNumTestsPerEvictionRun(conf.getNumTestsPerEvictionRun());
     setMinEvictableIdleTimeMillis(conf.getMinEvictableIdleTimeMillis());
     setTimeBetweenEvictionRunsMillis(
             conf.getTimeBetweenEvictionRunsMillis());
     setSoftMinEvictableIdleTimeMillis(
             conf.getSoftMinEvictableIdleTimeMillis());
     setEvictionPolicyClassName(conf.getEvictionPolicyClassName());
 }
```

## 运作机制

初始化新建的对象（addObject() ）先进入idleObjects，returnObject()和ensureIdle()的对象也会先进入idleObjects

所有创建的对象（create()）都进入allObjects，被销毁（destroy()）时才从allObjects移除

借用先从idleObjects获取，获取不到则create()，如果创建失败，则等待从idleObjects获取

## 源码分析

ObjectPool

![commons-pool-ObjectPool](/images/commons-pool-ObjectPool.png)

BaseGenericObjectPool

GenericObjectPool

GenericKeyedObjectPool

- addObject()：一般用于初始化或预加载，放入idleObjects，org.apache.commons.pool2.impl.GenericObjectPool.preparePool()


- borrowObject()：借，maxTotal检查
- returnObject()：还，maxIdle检查，没还成功则ensureIdle(1, false)


- invalidateObject()：失效并删除然后ensureIdle(1, false)

min检查（ensureMinIdle()）由evictor线程负责

PooledObjectFactory

![commons-pool-PooledObjectFactory](/images/commons-pool-PooledObjectFactory.png)

BasePooledObjectFactory

![commons-pool-BasePooledObjectFactory](/images/commons-pool-BasePooledObjectFactory.png)

activateObject：从pool中获取到对象后使用之前执行初始化（reinitialize）

passivateObject：使用完后归还给pool之前执行反初始化（uninitialize）（清理痕迹等）