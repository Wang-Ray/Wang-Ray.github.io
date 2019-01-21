---
layout: post
title: "实践Arthas"
date: 2018-10-18 11:08:00 +0800
categories: Java
tags: Java Arthas Diagnostic
---

[Arthas（阿尔萨斯）](https://alibaba.github.io/arthas/)是阿里巴巴开源的 Java 诊断工具。源代码地址：[arthas【Gitee】](https://gitee.com/arthas/arthas)

## 安装

Linux下可以执行如下命令进行安装：

```shell
$ curl -L https://alibaba.github.io/arthas/install.sh | sh
```

安装到当前目录（`as.sh`），`ARTHAS_HOME`为`~/.arthas`（包括`lib`等）。

arthas的telnet服务端口为3658，http服务端口为8563。

## 启动

```shell
$ as.sh
```

## 命令

```shell
$ dashboard
```

