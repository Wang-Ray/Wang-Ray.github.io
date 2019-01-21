---
layout: post
title: "实践JDK内置工具之VisualVM"
date: 2018-03-22 11:08:00 +0800
categories: Java
tags: java visualvm
---



## 通过JMX连接远程服务

远程服务新增如下系统参数：

```
-Djava.rmi.server.hostname=192.168.103.13
-Dcom.sun.management.jmxremote.port=9008
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

## 通过jstatd连接远程服务

远程服务器进行如下操作：

1. 配置安全访问

   在远程服务器新建文件`jstatd.all.policy`，内容如下：

   ```java
   grant codebase "file:${java.home}/../lib/tools.jar" {
   	permission java.security.AllPermission;
   };
   ```

2. 启动jstatd

   ```shell
   $ jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=192.168.103.76 &
   ```

jstatd方式下，Sampler、MBeans、JConsole不能启用

### OOL

针对堆dump，可以利用OOL进行查询，例如：

`select s from org.apache.activemq.ActiveMQSession s`

## QA

### 不受此JVM支持（Not supported for this JVM）

在Deepin下使用VisualVM连接本地应用，提示`不受此JVM支持（Not supported for this JVM）`，本地应用增加JVM参数即可：`-Dcom.sun.management.jmxremote`

但是Windows下没遇到这个问题