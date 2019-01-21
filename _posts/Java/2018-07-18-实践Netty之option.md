---
layout: post
title: "实践Netty之option"
date: 2018-07-18 11:08:00 +0800
categories: Java
tags: database netty option
---



### 客户端和服务端公共配置项

DefaultChannelConfig

```
CONNECT_TIMEOUT_MILLIS, MAX_MESSAGES_PER_READ, WRITE_SPIN_COUNT,
ALLOCATOR, AUTO_READ, AUTO_CLOSE, RCVBUF_ALLOCATOR, WRITE_BUFFER_HIGH_WATER_MARK,
WRITE_BUFFER_LOW_WATER_MARK, WRITE_BUFFER_WATER_MARK, MESSAGE_SIZE_ESTIMATOR,
SINGLE_EVENTEXECUTOR_PER_GROUP
```

### 客户端额外配置项

DefaultSocketChannelConfig

```
SO_RCVBUF, SO_SNDBUF, TCP_NODELAY, SO_KEEPALIVE, SO_REUSEADDR, SO_LINGER, IP_TOS,
ALLOW_HALF_CLOSURE
```

样例：

```java
workerGroup = new NioEventLoopGroup();
bootstrap = new Bootstrap();
bootstrap.group(workerGroup).channel(NioSocketChannel.class);
bootstrap.option(ChannelOption.TCP_NODELAY, true);
```

### 服务端额外配置项

DefaultServerSocketChannelConfig

```
SO_RCVBUF, SO_REUSEADDR, SO_BACKLOG
```

样例：

```java
bossGroup = new NioEventLoopGroup();
workerGroup = new NioEventLoopGroup();
bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class);
bootstrap.option(ChannelOption.SO_BACKLOG, 1000);
```

### QA

如果配置不支持的配置项，会报警告如下：

```
16:28:19.018 [main] WARN io.netty.bootstrap.ServerBootstrap - Unknown channel option 'SO_KEEPALIVE' for channel '[id: 0x2fda4aae]'
```



