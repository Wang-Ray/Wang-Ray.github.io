---
layout: post
title: "实践Maven私服之Nexus"
date: 2019-07-26 09:03:13 +0800
categories: 工具
tags: tool Nexus Maven
---

[Sonatype](https://www.sonatype.com/) 

## 安装

以Deepin 15.11 64bit desktop为例

1. 先安装好1.8或以上版本的JRE或JDK

2. 到官方下载Nexus Repository Manager OSS Edition，比如：`nexus-3.17.0-01-unix.tar.gz`
3. 解压，得到`nexus-3.17.0-01`和`sonatype-work`

```shell
$ nexus-3.17.0-01/bin/nexus 
Usage: ./nexus {start|stop|run|run-redirect|status|restart|force-reload}
```

## 配置

配置文件`nexus-3.17.0-01/etc/nexus-default.properties`

## 权限

