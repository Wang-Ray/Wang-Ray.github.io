---
layout: post
title: "实践Oracle RAC"
date: 2018-06-14 14:05:00 +0800
categories: 数据库
tags: oracle RAC database
---

在Oracle RAC环境下，每个节点都会有三个IP地址，分别为Public/Private/Vip，这三个IP到底有啥区别呢？分别用在那些场合呢？。

1. private IP address is used only for internal clustering processing (Cache Fusion)
  私有IP用于心跳同步，这个对于用户层面，可以直接忽略，简单理解，这个Ip用来保证两台服务器同步数据用的私网IP。
2. VIP is used by database applications to enable fail over when one cluster node fails
  虚拟IP用于客户端应用，以支持失效转移，通俗说就是一台挂了，另一台自动接管，客户端没有任何感觉。
  这也是为什么要使用RAC的原因之一，另一个原因，我认为是负载均衡。
3. public IP adress is the normal IP address typically used by DBA and SA to manage storage, system and database.
  公有IP一般用于管理员，用来确保可以操作到正确的机器，我更愿意叫他真实IP。

SCAN IP