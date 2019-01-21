---
layout: post
title: "实践MappedByteBuffer"
date: 2017-09-12 17:58:13 +0800
categories: Java
tags: java io MappedByteBuffer buffer DirectByteBuffer
---

基于RandomAccessFile的Channel可以创建

```
Exception in thread "main" java.nio.channels.NonWritableChannelException
	at sun.nio.ch.FileChannelImpl.map(FileChannelImpl.java:880)
	at wang.angi.chapter4.UseMappedFile.main(UseMappedFile.java:26)
```

