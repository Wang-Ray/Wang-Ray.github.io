---
layout: post
categories: Java
tags: java Reactor Netty
---



## HttpServer

```java
HttpServer.create()
	.host("0.0.0.0")
 	.handle((req, res) -> res.sendString(Flux.just("hello"))
    .bind()
	.block();
```

## 参考

