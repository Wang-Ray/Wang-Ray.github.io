---
layout: post
title: "实践数据库中间件之Sharding-Proxy"
date: 2018-07-11 11:08:00 +0800
categories: 数据库
tags: database sharding-proxy
---

[Sharding-Proxy](http://shardingjdbc.io/)定位为透明化的数据库代理端，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。目前先提供MySQL版本，它可以使用任何兼容MySQL协议的访问客户端(如：MySQL Command Client, MySQL Workbench等)操作数据，对DBA更加友好。

- 向应用程序完全透明，可直接当做MySQL使用。
- 适用于任何兼容MySQL协议的客户端。

