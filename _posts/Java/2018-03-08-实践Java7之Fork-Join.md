---
layout: post
title: "实践Java7之Fork-Join"
date: 2018-02-25 11:08:00 +0800
categories: Java
tags: java fork-join
---

ForkJoin，并行计算框架

## ForkJoinTask

![ForkJoinTask](/images/ForkJoinTask.png)

### RecursiveTask

### RecursiveAction



## ForkJoinPool

![ForkJoinPool](/images/ForkJoinPool.png)

daemon为true

## work stealing

在 `ForkJoinPool` 中的实现为：线程池中每个线程都有一个互不影响的任务队列（双端队列），线程每次都从自己的任务队列的队头中取出一个任务来运行；如果某个线程对应的队列已空并且处于空闲状态，而其他线程的队列中还有任务需要处理但是该线程处于工作状态，那么空闲的线程可以从其他线程的队列的队尾取一个任务来帮忙运行 —— 感觉就像是空闲的线程去偷人家的任务来运行一样，所以叫 “工作窃取”。