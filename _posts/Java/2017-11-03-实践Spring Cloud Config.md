---
layout: post
title: "实践Spring Cloud Config"
date: 2017-11-03 11:08:00 +0800
categories: Java
tags: spring spring-boot java spring-cloud Config
---

![Spring Cloud Config Architecture](/images/spring-cloud-config-architecture.png)

## Config Server

依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@EnableConfigServer
```

http://localhost:8888

### 仓库配置

三种占位符：

* {application}，应用名称，默认application
* {profile}，环境，默认default
* {label}，分支，默认master

URL路径和配置文件的映射关系：

/{application}/{profile}[/{label}]

[/{label}]/{application}-{profile}.yml

[/{label}]/{application}-{profile}.properties

例如：

http://192.168.70.139:9999/cts-hello-client/default

http://192.168.70.139:9999/cts-hello-client-default.properties



```properties
# 固定仓库
spring.cloud.config.server.git.uri=http://192.168.70.244/ITS/config.git

# 不同的应用不同的仓库（利用占位符）
spring.cloud.config.server.git.uri=http://192.168.70.244/ITS/{application}.git

# 本地仓库
spring.cloud.config.server.git.uri=file://home/angi/sts_3.8.3_koolyun/config
# 本地仓库 for Windows, required an extra "/"
spring.cloud.config.server.git.uri=file:///home/angi/sts_3.8.3_koolyun/config

# 本地文件系统
spring.profiles.active=native
# 默认spring boot classpath下检索配置文件，也可以指定
spring.cloud.config.server.native.searchLocation=file://home/angi/config

# 固定子目录
spring.cloud.config.server.git.search-paths=ITS
# 不同的应用不同的目录（利用占位符），支持逗号分隔多个，默认包含根目录（/）
spring.cloud.config.server.git.search-paths={application}
```

多仓库

```properties
# 基于{application}/{profile}进行模式匹配，当无法匹配时取spring.cloud.config.server.git.uri
# 可以实现某一应用组使用某一个仓库，项目组下不同应用不同目录
spring.cloud.config.server.git.repos.cts.pattern=cts-*/*
spring.cloud.config.server.git.repos.cts.uri=git@192.168.70.244:ITS/cts-config.git
spring.cloud.config.server.git.repos.cts.searchPaths={application}

spring.cloud.config.server.git.repos.ets.pattern=ets-*/*
spring.cloud.config.server.git.repos.ets.uri=git@192.168.70.244:ETS/ets-config.git
spring.cloud.config.server.git.repos.ets.searchPaths={application}
```

默认会从仓库checkout一份到本地`/tmp/config-repo-随机数`目录，可以通过参数定制此目录：

```properties
spring.cloud.config.server.git.basedir=~/config-basedir
```



### 仓库认证

用户需要Reporter或以上角色

#### http

```properties
spring.cloud.config.server.git.uri=http://192.168.70.244/ITS/config.git
spring.cloud.config.server.git.username=***
spring.cloud.config.server.git.password=***
```

#### ssh

基于{user.home}/.ssh/密钥访问

```properties
spring.cloud.config.server.git.uri=git@192.168.70.244:ITS/config.git
```

### 默认属性

### 安全保护

config server:

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>
```

```properties
security.basic.enabled=true
security.user.name=config
security.user.password=config2017
```

config client:

```properties
spring.cloud.config.username=config
spring.cloud.config.password=config2017
```

或者

```yaml
spring:
  cloud:
    config:
     uri: https://user:secret@myconfig.mycompany.com
```



### 加密

需要安装Unlimited Strength Java Cryptography Extension

特有端点：

/encrypt/status

/key

/encrypt

/decrypt

#### 对称加密

* 配置对称密钥

在配置中心的bootstrap.properties中配置如下：

```properties
encrypt.key=abcdef123456
```

* 查看服务状态

访问：http://localhost:9999/encrypt/status

* 加密

```shell
$ curl localhost:9999/encrypt -d password123
b635dbd39bf17d98817270e4229524b42134890d99a3f4aa28cc850fe433e7b6
```

* 配置

```properties
passpord={cipher}*
```

* 解密

```shell
$ curl http://config:config2017@localhost:9999/decrypt -d b635dbd39bf17d98817270e4229524b42134890d99a3f4aa28cc850fe433e7b6
password123
```

#### 非对称加密

* 配置非对称密钥

生产密钥库（利用JDK自带的keytool工具），然后配置如下：

```properties
# keystore的位置
encrypt.key-store.location=/app/
# keystore的密码
encrypt.key-store.password=
# key的别名
encrypt.key-store.alias=
# key的密码
encrypt.key-store.secret=

```





其他跟对称密钥类同

### 高可用

## Config Client

依赖：

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

bootstrap.properties/yml

```properties
# {application}
spring.application.name=sample

# config server
# 由于生产环境一般是独立的，默认配置为非生产，生产环境的参数通过系统属性、命令行参数或环境变量等配置

# 如果配置中心未服务化
spring.cloud.config.uri=http://localhost:7003/
# {profile}，配置建议同config server
# spring.cloud.config.profile=default
# {label}，配置建议同config server
# spring.cloud.config.label=master
# 快速失败（连不上配置中心启动失败）
spring.cloud.config.fail-fast=true

# 如果配置中心服务化
# registry center
eureka.client.service-url.defaultZone=http://192.168.70.139:8888/eureka/
# config center
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=config-center
```

其他配置从config server获取

### 应用属性文件

{application}-{profile}.properties/yml

{application}.properties/yml（default profile）

通过`spring.cloud.config.profile=dev`设定profile

### 所有应用共享属性

#### 方法一

仓库根目录下的application[-profile].properties/yaml定义所有应用的共享属性，比应用级属性优先级低。

**本地文件系统模式下，建议显示指定路径，与config server自己的配置（application[-profile].properties/yaml）区分开，因为config server自己的配置不会传递给config client**

#### 方法二

config server支持属性覆盖特性，config server中定义覆盖属性，所有config client可以读取到。

config server的application.properties

```properties
spring.cloud.config.server.overrides.key1=cde
```

方法二比方法一优先级高

### 本地属性覆盖

命令行参数优先级更高，会覆盖从config server获取的属性。另外也可以通过配置实现本地配置覆盖远程配置，参考如下：

以下三个属性在remote properties中设置才有效（在local properties设置无效）

是否允许remote properties被本地覆盖，默认值true

`spring.cloud.config.allowOverride=true`

远程配置是否不覆盖本地配置，即本地配置优先级是否高于远程配置，默认值false

`spring.cloud.config.overrideNone=false`

远程配置是否覆盖系统属性，即远程配置优先级是否高于系统属性

`spring.cloud.config.overrideSystemProperties=true`

### 动态刷新

* 添加依赖`spring-boot-starter-actuator`，带有`/refresh`端点

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

* 添加@RefreshScope

  ```java
  @RefreshScope
  @RestController
  public class TestController {

      @Value("${from}")
      private String from;

      @RequestMapping("/from")
      public String from() {
          return from;
      }
  }
  ```

* 修改配置后，POST提交请求到客户端`/refresh`，即可刷新配置。例如：

  ```shell
  $ curl -d '' http://localhost:7002/refresh
  ["config.client.version","from"]
  ```


上述方式是针对单个微服务实例的刷新，如果要同时属性所有微服务实例，可以借助Spring Cloud Bus实现，详情可以参见[实践Spring Cloud之Bus](/2017/11/03/实践Spring-Cloud之Bus.html)

### Config First Bootstrap

默认是基于Config Server优先，Config Client先从Config Server读取配置，然后进行注册微服务等其他初始化动作，因而经需要配置bootstrap.properties如下：

```properties
spring.application.name=cts-hello-server
spring.cloud.config.uri=http://localhost:8888
```

### Discovery Frist Bootstrap

基于Eureka Server优先，Config Server注册到Eureka Server中，Config Client通过Eureka Server查找调用Config Server，因而bootstrap.properties中需要配置如下：

```properties
spring.application.name=cts-hello-server
# registry center
eureka.client.service-url.defaultZone=http://registry:registry2017@192.168.70.139:8888/eureka/
# config center
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=config-center
```



## spring-cloud-config-admin

[](http://blog.didispace.com/spring-cloud-config-admin-1-0-0-release/)

## Q&A

* 当启动时无法连接上config server，报错如下：

```
2017-12-29 09:46:57.541  WARN 5092 --- [           main] com.netflix.discovery.DiscoveryClient    : Using default backup registry implementation which does not do anything.
2017-12-29 09:46:57.543  INFO 5092 --- [           main] com.netflix.discovery.DiscoveryClient    : Not registering with Eureka server per configuration
2017-12-29 09:46:57.546  INFO 5092 --- [           main] com.netflix.discovery.DiscoveryClient    : Discovery Client initialized at timestamp 1514512017546 with initial instances count: 0
2017-12-29 09:46:57.751  WARN 5092 --- [           main] lientConfigServiceBootstrapConfiguration : Could not locate configserver via discovery

java.lang.IllegalStateException: No instances found of configserver (config-center1)
	at org.springframework.cloud.config.client.ConfigServerInstanceProvider.getConfigServerInstance(ConfigServerInstanceProvider.java:25) ~[spring-cloud-config-client-1.3.3.RELEASE.jar:1.3.3.RELEASE]
...
```

* 当开启连接不上config server时快速失败（`spring.cloud.config.fail-fast=true`），报错如下：

  ```
  2017-12-29 10:12:24.447 ERROR 5960 --- [           main] o.s.boot.SpringApplication               : Application startup failed

  java.lang.IllegalStateException: No instances found of configserver (config-center1)
  	at org.springframework.cloud.config.client.ConfigServerInstanceProvider.getConfigServerInstance(ConfigServerInstanceProvider.java:25) ~[spring-cloud-config-client-1.3.3.RELEASE.jar:1.3.3.RELEASE]
  ```

  ​

* ​