---
layout: post
title: "读书之《ZooKeeper：分布式过程协同技术详解》"
categories: Java
tags: Java Zookeeper 《ZooKeeper：分布式过程协同技术详解》 "Flavio Junqueira" "Benjamin Reed" 谢超 周贵卿
---



目录

前言 1

部分 ZooKeeper的概念和基础

第1章 简介 7

1.1 ZooKeeper的使命 8

1.1.1 ZooKeeper改变了什么 10

1.1.2 ZooKeeper不适用的场景 10

1.1.3 关于Apache项目 11

1.1.4 通过ZooKeeper构建分布式系统 11

1.2 示例：主-从应用 12

1.2.1 主节点失效 13

1.2.2 从节点失效 14

1.2.3 通信故障 14

1.2.4 任务总结 15

1.3 分布式协作的难点 16

1.4 ZooKeeper的成功和注意事项 18

第2章 了解ZooKeeper 19

2.1 ZooKeeper基础 19

2.1.1 API概述 20

2.1.2 znode的不同类型 21

2.1.3 监视与通知 22

2.1.4 版本 24

2.2 ZooKeeper架构 25

2.2.1 ZooKeeper仲裁 26

2.2.2 会话 27

2.3 开始使用ZooKeeper 28

2.3.1 个ZooKeeper会话 28

2.3.2 会话的状态和声明周期 31

2.3.3 ZooKeeper与仲裁模式 33

2.3.4 实现一个原语：通过ZooKeeper实现锁 36

2.4 一个主-从模式例子的实现 37

2.4.1 主节点角色 37

2.4.2 从节点、任务和分配 40

2.4.3 从节点角色 40

2.4.4 客户端角色 41

2.5 小结 43

第二部分 使用ZooKeeper进行开发

第3章 开始使用ZooKeeper的API 47

3.1 设置ZooKeeper的CLASSPATH 47

3.2 建立ZooKeeper会话 47

3.2.1 实现一个Watcher 49

3.2.2 运行Watcher的示例 51

3.3 获取管理权 53

3.3.1 异步获取管理权 57

3.3.2 设置元数据 60

3.4 注册从节点 62

3.5 任务队列化 65

3.6 管理客户端 66

3.7 小结 68

第4章 处理状态变化 70

4.1 单次触发器 71

4.2 如何设置监视点 72

4.3 普遍模型 73

4.4 主-从模式的例子 74

4.4.1 管理权变化 74

4.4.2 主节点等待从节点列表的变化 77

4.4.3 主节点等待新任务进行分配 80

4.4.4 从节点等待分配新任务 83

4.4.5 客户端等待任务的执行结果 86

4.5 另一种调用方式：Multiop 88

4.6 通过监视点代替显式缓存管理 90

4.7 顺序的保障 91

4.7.1 写操作的顺序 91

4.7.2 读操作的顺序 91

4.7.3 通知的顺序 92

4.8 监视点的羊群效应和可扩展性 93

4.9 小结 94

第5章 故障处理 96

5.1 可恢复的故障 98

5.2 不可恢复的故障 102

5.3 群举和外部资源 103

5.4 小结 106

第6章 ZooKeeper注意事项 107

6.1 使用ACL 107

6.1.1 内置的鉴权模式 108

6.1.2 SASL和Kerberos 111

6.1.3 增加新鉴权模式 111

6.2 恢复会话 111

6.3 当znode节点重新创建时，重置版本号 112

6.4 sync方法 112

6.5 顺序性保障 114

6.5.1连接丢失时的顺序性 114

6.5.2 同步API和多线程的顺序性 115

6.5.3 同步和异步混合调用的顺序性 115

6.6 数据字段和子节点的限制 116

6.7 嵌入式ZooKeeper服务器 116

6.8 小结 117

第7章 C语言客户端 118

7.1 配置开发环境 118

7.2 开始会话 119

7.3 引导主节点 121

7.4 行使管理权 126

7.5 任务分配 129

7.6 单线程与多线程客户端 132

7.7 小结 135

第8章 Curator：ZooKeeper API的高级封装库 136

8.1 Curator客户端程序 136

8.2 流畅式API 137

8.3 监听器 138

8.4 Curator中状态的转换 140

8.5 两种边界情况 141

8.6 菜谱 141

8.6.1 群首闩 142

8.6.2 群举器 143

8.6.3 子节点缓存器 146

8.7 小结 148

第三部分 ZooKeeper的管理

第9章 ZooKeeper内部原理 151

9.1 请求、事务和标识符 152

9.2 群举 153

9.3 Zab：状态更新的广播协议 157

9.4 观察者 161

9.5 服务器的构成 162

9.5.1 独立服务器 163

9.5.2 群首服务器 164

9.5.3 追随者和观察者服务器 165

9.6 本地存储 166

9.6.1 日志和磁盘的使用 166

9.6.2 快照 167

9.7 服务器与会话 169

9.8 服务器与监视点 170

9.9 客户端 170

9.10 序列化 171

9.11 小结 171

第10章 运行ZooKeeper 173

10.1 配置ZooKeeper服务器 174

10.1.1 基本配置 175

10.1.2 存储配置 175

10.1.3 网络配置 177

10.1.4 集群配置 179

10.1.5 认证和授权选项 181

10.1.6 非安全配置 182

10.1.7 日志 183

10.1.8 专用资源 185

10.2 配置ZooKeeper集群 185

10.2.1 多数原则 186

10.2.2 法定人数的可配置性 186

10.2.3 观察者 188

10.3 重配置 188

10.4 配额管理 194

10.5 多租赁配置 196

10.6 文件系统布局和格式 197

10.6.1 事务日志 198

10.6.2 快照 199

10.6.3 时间戳文件 200

10.6.4 已保存的ZooKeeper数据的应用 200

10.7 四字母命令 201

10.8 通过JMX进行监控 202

10.9 工具 209

10.10 小结 209
