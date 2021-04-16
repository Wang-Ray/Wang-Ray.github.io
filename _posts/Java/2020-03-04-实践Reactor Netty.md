---
layout: post
title: 实践Reactor Netty
categories: Java
tags: java Reactor Netty
---

[reactor-netty](https://github.com/reactor/reactor-netty)



### HttpServer

```java
HttpServer.create()
	.host("0.0.0.0")
 	.handle((req, res) -> res.sendString(Flux.just("hello"))
    .bind()
	.block();
```

### TcpServer



### TcpServerRunOn

```java
static void configure(ServerBootstrap b,
			boolean preferNative,
			LoopResources resources) {

		EventLoopGroup selectorGroup = resources.onServerSelect(preferNative);
		EventLoopGroup elg = resources.onServer(preferNative);

		b.group(selectorGroup, elg)
		 .channel(resources.onServerChannel(elg));
	}
```



## 参考

