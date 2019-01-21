---
layout: post
title: "实践Java8之CompletableFuture"
date: 2018-03-30 11:08:00 +0800
categories: Java
tags: java future completablefuture
---

### 创建

```java
// 有返回值
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(Supplier);
// 没有返回值
CompletableFuture completableFuture = CompletableFuture.runAsync(Runnable);
```

默认Executor为：

```java
private static final Executor asyncPool = useCommonPool ?
        ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
```

*注意：daemon为true*

也可以自定义Executor，例如：

```java
// 有返回值
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(Supplier, executor);
```

### 使用

#### 单个

##### thenApply(Function)

```java
public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn)
```

当前CompletionStage正常执行完成后执行Function，然后返回新的CompletionStage

##### thenAccept(Comsumer)

##### thenRun(Runnable)

#### 两个

##### thenCombine(CompletionStage, BiFunction)

```java
public <U,V> CompletionStage<V> thenCombine
        (CompletionStage<? extends U> other,
         BiFunction<? super T,? super U,? extends V> fn)
```

当前CompletionStage正常执行完成后，然后等指定CompletionStage正常执行完成后，执行BiFunction（两个CompletionStage的结果作为输入），然后返回新的CompletionStage

##### thenAcceptBoth

```java
public <U> CompletionStage<Void> thenAcceptBoth
    (CompletionStage<? extends U> other,
     BiConsumer<? super T, ? super U> action)
```

##### runAfterBoth

```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,
                                              Runnable action)
```

#### 二选一

##### applyToEither

```java
public <U> CompletionStage<U> applyToEither
        (CompletionStage<? extends T> other,
         Function<? super T, U> fn)
```

当前CompletionStage或指定CompletionStage正常执行完成后，执行Function，然后返回新的CompletionStage

##### acceptEither

##### runAfterEither

#### thenCompose(Function)

```java
public <U> CompletionStage<U> thenCompose
        (Function<? super T, ? extends CompletionStage<U>> fn)
```

当前CompletionStage正常执行完成后，执行Function并返回CompletionStage

#### 终态

##### exceptionally(Function)

```java
public CompletionStage<T> exceptionally    (Function<Throwable, ? extends T> fn)
```

当前CompletionStage发生异常时，执行Function，然后返回新的CompletionStage

##### whenComplete(BiConsumer)

##### handle(BiFunction)

#### *Asnyc

上述方法（不以`Async`结尾）默认继续在上一个异步任务执行线程中执行，而其对应的以`Async`结尾的方法，会使用`默认Executor`来执行，可能与上一个异步任务线程不同，即使共用相同的线程池，线程池调度存在不确定性。

例如：

不以Async结尾的方法执行线程

```
11:22:39.689 [pool-2-thread-1] INFO  a.w.s.m.c.ThenComposeMain - supplyAsync
11:22:39.693 [pool-2-thread-1] INFO  a.w.s.m.c.ThenComposeMain - thenCompose
```

以Async结尾的方法执行线程

```
11:23:28.343 [pool-2-thread-1] INFO  a.w.s.m.c.ThenComposeMain - supplyAsync
11:23:28.348 [ForkJoinPool.commonPool-worker-1] INFO  a.w.s.m.c.ThenComposeMain - thenComposeAsync
```