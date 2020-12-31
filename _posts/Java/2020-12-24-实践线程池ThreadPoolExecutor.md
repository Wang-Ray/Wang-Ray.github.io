---
layout: post
title: "实践线程池ThreadPoolExecutor"
categories: Java
tags: Java Thread concurrency multi-thread ThreadPoolExecutor
---





线程池执行任务

```java
"pool-1-thread-1@683" prio=5 tid=0xd nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at wang.angi.sample.multithread.threadpool.ThreadPoolExecutorTest$1.run(ThreadPoolExecutorTest.java:21)
	  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	  at java.lang.Thread.run(Thread.java:748)
```

线程池等待任务

```java
"http-nio-8080-exec-1" #163 daemon prio=5 os_prio=0 tid=0x00007f13dc7aa800 nid=0x6b96 waiting on condition [0x00007f13d37f8000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000ead46238> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:31)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
```

线程池等待任务（核心线程允许超时或多余minSpareThreads的线程）

```java
"http-nio-8080-exec-6" #107 daemon prio=5 os_prio=0 tid=0x00007fec3802d000 nid=0x62e0 waiting on condition [0x00007febcd503000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000869935f8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
	at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:467)
	at org.apache.tomcat.util.threads.TaskQueue.poll(TaskQueue.java:85)
	at org.apache.tomcat.util.threads.TaskQueue.poll(TaskQueue.java:31)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1073)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
```

