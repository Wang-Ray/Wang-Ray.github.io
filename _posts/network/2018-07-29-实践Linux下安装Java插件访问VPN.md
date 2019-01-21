---
layout: post
title: "实践Linux下安装Java插件访问VPN"
date: 2018-07-29 09:50:00 +0800
categories: 网络
tags: network firefox java vpn
---

VPN服务器是基于Sangfor(深信服)搭建的，Windows下有客户端（Easy Connect），非Windows官方没有提供客户端，只提供了通过浏览器上的Java插件支持（Java插件运行Sangfor(深信服)的client然后添加本地路由表实现）。

直接通过Firefox访问VPN地址，登录后提示如下：

![VPN-Sangfor-Java](/images/VPN-Sangfor-Java.png)

可以按照提示安装和配置Java插件。

标准版的Firefox已经不再支持java等，只有**Firefox ESR**继续支持到2018年。

下载并安装JRE（版本1.6.0_27，32位或64位根据情况到官网下载，上述页面中提供的下载是32位的）

```shell
$ chmod +x jre-6u27-linux-x64.bin
# 安装到/usr/java/jre1.6.0_27
$ sudo ./jre-6u27-linux-x64.bin
```

安装Firefox的Java插件

```shell
$ cd /usr/lib/mozilla/plugins
$ sudo ln -s /usr/java/jre1.6.0_27/jre/lib/amd64/libnpjp2.so
```

查看Java插件安装情况并激活。

![Firefox-Java](/images/Firefox-Java.png)

运行Firefox ESR

```shell
$ sudo ./firefox
```

登录之后会提示运行一个Java插件，同意运行即可。

参考文档[Ubuntu下安装Java插件连接浙大RVPN](http://ju.outofmemory.cn/entry/91706)