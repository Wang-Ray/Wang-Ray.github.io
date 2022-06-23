---
layout: post
categories: Design
tags: design zookeeper
---

[Zookeeper](http://zookeeper.apache.org/) is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

基于 ZooKeeper 可以实现诸如数据发布/订阅、配置中心、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

## 设计

### Session

sessionId：session编号

sessionTimeout：回话最大允许空闲时间，超过则session过期，一旦过期，临时节点和watcher全部清除。

```
2020-09-03 17:10:06.043 DEBUG [main-SendThread(127.0.0.1:2181)] org.apache.zookeeper.ClientCnxn$SendThread:742 >> Got ping response for sessionid: 0x100013eba9b0005 after 1ms
```



### Znode

![zookeeper znode](/images/zookeeper-znode.jpeg)

类似文件系统，由节点（node）组成的树形结构，从`/`开始，比如`/dubbo/config/dubbo/dubbo.properties`，节点具有`属性`和`值`

分为持久节点和临时节点

持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个ZNode将一直保存在Zookeeper上。

临时节点是指生命周期与Session绑定，一旦Session失效，在此Session下创建的临时节点都会被移除。

**SEQUENTIAL**

### 版本

### Watcher

### ACL

* CREATE：创建`子节点`
* READ：获取节点数据和子节点列表
* WRITE：更新节点数据
* DELETE：删除`子节点`
* ADMIN：设置节点ACL

### ZAB

崩溃恢复：选举Leader和数据同步

消息广播：

## 安装

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

## 功能



## 集群

![zookeeper cluster](/images/zookeeper-cluster.png)

![zookeeper cluster role](/images/zookeeper-cluster-role.jpeg)

### 伪集群

在一台服务器上部署3个节点，实现伪集群

将zookeeper拷贝两份，一共3份，配置参考如下：

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

在${dataDir}下面创建一个myid文件，内容为顺序号，server.x 中的x对应，第一个zookeeper就是1，第二个zookeeper就是2，没有先后顺序，只是不能重复 。

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

 node1

```shell
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /app/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: leader
```

node2

```shell
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /app/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: follower
```

node3

```shell
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /app/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: follower
```



## 客户端

### 命令行

zkCli.sh/zkCli.cmd

### 可视化客户端

[ZooInspector](https://github.com/apache/zookeeper/tree/b79af153d0f98a4f3f3516910ed47234d7b3d74e/src/contrib/zooinspector)

zookeeper自带，[下载已编译版本](https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip)

[ZooViewer](https://github.com/Wang-Ray/ZooViewer)

```shell
$ ./zooviewer.sh
```

[zkui](https://github.com/DeemOpen/zkui)

### API客户端

[curator](https://curator.apache.org/index.html)

[zkclient](https://github.com/sgroschupf/zkclient)

## 参考

[这可能是把ZooKeeper概念讲的最清楚的一篇文章](http://developer.51cto.com/art/201809/583184.htm)