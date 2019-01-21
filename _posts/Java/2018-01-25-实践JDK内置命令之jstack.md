---
layout: post
title: "实践JDK内置命令之jstack"
date: 2018-02-25 11:08:00 +0800
categories: Java
tags: java jstack
---

jstack命令用来查看跟JVM线程和栈相关的信息

```shell
$ jstack -help
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks.（每个线程栈最后多出一条Locked ownable synchronizers:******信息，告知拥有的同步锁）
    -h or -help to print this help message
```



```shell
$ jstack 19238
"Signal Dispatcher" daemon prio=10 tid=0x00007fd6cc11a800 nid=0x4cc6 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" daemon prio=10 tid=0x00007fd6cc0f2000 nid=0x4cc5 in Object.wait() [0x00007fd6c84f3000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x000000078b68c8c0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" daemon prio=10 tid=0x00007fd6cc0f0000 nid=0x4cc4 in Object.wait() [0x00007fd6c85f4000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x000000078dd624d8> (a java.lang.ref.Reference$Lock)

"VM Thread" prio=10 tid=0x00007fd6cc0eb800 nid=0x4cc3 runnable 

"GC task thread#0 (ParallelGC)" prio=10 tid=0x00007fd6cc01e800 nid=0x4cc1 runnable 

"GC task thread#1 (ParallelGC)" prio=10 tid=0x00007fd6cc020800 nid=0x4cc2 runnable 

"VM Periodic Task Thread" prio=10 tid=0x00007fd6cc12d000 nid=0x4cca waiting on condition 

JNI global references: 657
```

