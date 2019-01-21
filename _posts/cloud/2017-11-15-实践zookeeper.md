---
layout: post
title: "实践zookeeper"
date: 2017-09-02 09:58:13 +0800
categories: 云计算
tags: cloud zookeeper
---



新建conf/zoo.cfg

```properties
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```



```shell
# 启动
$ ./zkServer.sh start
```



```shell
$ ./zkServer.sh --help
ZooKeeper JMX enabled by default
Using config: /app/software/zookeeper/bin/../conf/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
```

## 配置

- tickTime：发送心跳的间隔时间，单位：毫秒
- dataDir：zookeeper保存数据的目录。
- clientPort：客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
- initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
- syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
- server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

## 集群

### 伪集群

在一台服务器上部署3个节点，实现伪集群

```properties
tickTime=2000
dataDir=/app/software/zookeeper-3.4.9-1/data
# 三个节点端口不同，2181,2182,2183
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2881:3881
server.2=localhost:2882:3882
server.3=localhost:2883:3883
```

在${dataDir}下面创建一个myid文件，内容为顺序号，server.x 中的x对应，第一个zookeeper就是1，第二个zookeeper就是2，没有先后顺序，只是不能重复 。

```shell
# 验证
$ ./zkCli.sh -server localhost:2181 localhost:2182 localhost:2183
```



### 真实集群

在三台服务器上分别部署一个节点，实现真实集群

```properties
tickTime=2000
dataDir=/app/software/zookeeper/data
clientPort=2181
initLimit=5
syncLimit=2
server.1=10.10.26.22:2881:3881
server.2=10.10.26.23:2881:3881
server.3=10.10.26.24:2881:3881
```

