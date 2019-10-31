---
layout: post
title: "实践Service Mesh"
date: 2019-04-01 09:58:13 +0800
categories: 云计算
tags: cloud service-mesh Linkerd Envoy Istio Conduit
---

Service Mesh又译作“服务网格”，作为服务间通信的基础设施层。如果用一句话来解释什么是Service Mesh，可以将它比作是应用程序或者说微服务间的TCP/IP，负责服务之间的网络调用、限流、熔断和监控。对于编写应用程序来说一般无须关心TCP/IP这一层（比如通过 HTTP 协议的 RESTful 应用），同样使用Service Mesh也就无须关系服务之间的那些原来是通过应用程序或者其他框架实现的事情，比如Spring Cloud、OSS，现在只要交给Service Mesh就可以了。

Service Mesh有如下几个特点： 

- 应用程序间通讯的中间层
- 轻量级网络代理
- 应用程序无感知
- 解耦应用程序的重试/超时、监控、追踪和服务发现

Service Mesh的架构如下图所示：

![Service Mesh](/images/service-mesh.png)

Service Mesh作为Sidebar运行，对应用程序来说是透明，所有应用程序间的流量都会通过它，所以对应用程序流量的控制都可以在Service Mesh中实现。



[下一代的微服务架构基础是ServiceMesh？](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651010083&idx=1&sn=173b1f05c2d53df427af0dbae64a34d7&chksm=bdbecc708ac9456625753a9916a83e91ab07c476d3389923953d625ea59320e8f3042eaf4b92&mpshare=1&scene=1&srcid=0220Omao7aotRy9S9Fb0ASpF&sharer_sharetime=1572421720505&sharer_shareid=240e700d294d916d751c382c12c2df76&key=9e9945bf8ade0db1f5dd2d7c3d74d95538a8a48e95089e58cfcf35abec2aa2e01678387be54e24bdb34a725a80eda06092ddaa878dd44e46ebd64e61712ff7f4cc398e9bb3670a62ed4098cc010e5475&ascene=1&uin=MTQ2NjY5Mzk1&devicetype=Windows+XP&version=62060841&lang=en&pass_ticket=yXMBWpNd%2FxSB%2BmqxAIC68dk%2FSmofKLsLoP2%2FwV6%2BGlU%3D)