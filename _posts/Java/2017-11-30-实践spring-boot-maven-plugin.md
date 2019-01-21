---
layout: post
title: "实践spring-boot-maven-plugin"
date: 2017-11-30 17:08:00 +0800
categories: Java
tags: spring-boot-maven-plugin spring-boot maven
---

## 介绍

[The Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/index.html) Plugin provides Spring Boot support in Maven, allowing you to package 
executable jar or war archives and run an application “in-place”.

spring-boot-starter-parent的`<pluginManagement/>`下定义了spring-boot-maven-plugin

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<executions>
		<execution>
			<goals>
				<goal>repackage</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<mainClass>${start-class}</mainClass>
	</configuration>
</plugin>
```

包含有多个goal，比如：`repackage`、`run`、`start`、`stop`和`buid-info`

## goals

### repackage

[repackage](https://docs.spring.io/spring-boot/docs/current/maven-plugin/repackage-mojo.html)，顾名思义，再打包，默认再打包成一个可执行的archive（包含依赖（compile，runtime，provide，非test，非system）），原打包的archive被重命名（后面追加.original），绑定到maven的package阶段

| 参数                 | 描述                                       | 默认值   |
| ------------------ | ---------------------------------------- | ----- |
| attach             | true：安装部署repackage的包，false：安装部署原始包（不带.original后缀） | true  |
| classifier         | repackage包追加的后缀，原始包则不追加.original，此时两个包都会被安装部署 | 空     |
| excludes           |                                          |       |
| excludeGroupIds    |                                          |       |
| excludeArtifactIds |                                          |       |
| skip               | 跳过执行                                     | false |
| excludeDevtools    | exclude devtools                         | true  |
| layout             | The type of archive (which corresponds to how the dependencies arelaid out inside it). Possible values are JAR, WAR, ZIP, DIR, NONE.Defaults to a guess based on the archive type. |       |

- `JAR`: regular executable JAR layout (`BOOT-INF/lib/`).
- `WAR`: executable WAR layout. `provided` dependencies are placed in `WEB-INF/lib-provided` to avoid any clash when the `war` is deployed in a servlet container (`WEB-INF/lib/`and `WEB-INF/lib-provided/`).
- `ZIP` (alias to `DIR`): similar to the `JAR` layout using `PropertiesLauncher` (`BOOT-INF/lib/` and environment variable `LOADER_PATH` and system property `loader.path`).
- `MODULE`: Bundle dependencies (excluding those with `provided` scope) and project resources. Does not bundle a bootstrap loader.
- `NONE`: Bundle all dependencies and project resources. Does not bundle a bootstrap loader.

样例（MANIFEST.inf）

```properties
Manifest-Version: 1.0

Implementation-Title: hello-client

Implementation-Version: 0.0.1-SNAPSHOT

Archiver-Version: Plexus Archiver

Built-By: angi

Implementation-Vendor-Id: com.koolyun.sample

Spring-Boot-Version: 1.5.8.RELEASE

Implementation-Vendor: Pivotal Software, Inc.

Main-Class: org.springframework.boot.loader.JarLauncher

Start-Class: com.koolyun.sample.cts.hello.client.HelloClientApplicatio

 n

Spring-Boot-Classes: BOOT-INF/classes/

Spring-Boot-Lib: BOOT-INF/lib/

Created-By: Apache Maven 3.5.0

Build-Jdk: 1.8.0_152

Implementation-URL: http://projects.spring.io/spring-boot/cts-parent/c

 ts-hello-client/
```

### run

[spring-boot:run](https://docs.spring.io/spring-boot/docs/current/maven-plugin/run-mojo.html) spring boot应用。

```shell
$ mvn spring-boot:run
```

| 参数           | 描述                                       | 默认值   |
| ------------ | ---------------------------------------- | ----- |
| fork         | User property: fork，是否另起进程，当agent, jvmArguments和workingDirectory设置或devtools存在时自动为true | false |
| jvmArguments | User property: run.jvmArguments          |       |
|              |                                          |       |

### start

[start](https://docs.spring.io/spring-boot/docs/current/maven-plugin/start-mojo.html)

| 参数   | 描述   | 默认值  |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |

### stop

[stop](https://docs.spring.io/spring-boot/docs/current/maven-plugin/stop-mojo.html)

### build-info

[spring-boot:build-info](https://docs.spring.io/spring-boot/docs/current/maven-plugin/build-info-mojo.html) generates build information that can be used by the Actuator.

## 参数

### layout

默认可执行的archive会自包含依赖，导致archive体积都会很大，某些场景下不太适合，这时可以将依赖放到外部，其他基本不变。layout设定为ZIP，同时将依赖exclude，这样repackage的archive也是可执行的，只是不再自包含依赖。

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layout>ZIP</layout>
        <excludeGroupIds>
            org.springframework,org.springframework.boot,org.springframework.cloud
        </excludeGroupIds>
    </configuration>
</plugin>
```

执行：

```shell
$ java -Dloader.path="3rd-party/,modules/" -jar cts-hello-client-0.0.1-SNAPSHOT.jar 
```

**这种方式最大的问题是排查依赖必须精确匹配，不支持模糊匹配，不够灵活，操作起来较为麻烦**