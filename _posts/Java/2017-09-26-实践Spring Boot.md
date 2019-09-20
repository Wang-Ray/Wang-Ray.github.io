---
layout: post
title: "实践Spring Boot"
date: 2017-09-26 11:08:00 +0800
categories: Java
tags: spring spring-boot java
---

[Spring Boot](http://projects.spring.io/spring-boot/)用于快速高效地创建可执行的Spring应用程序，进一步升华了约定优于配置。

## 快速开始

1. 继承`spring-boot-starter-parent`

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.8.RELEASE</version>
</parent>
```

`spring-boot-starter-parent`主要定义`pluginManagement`、编码和Java版本，同时继承`spring-boot-dependencies`，`spring-boot-dependencies`主要定义`dependencyManagement`、`pluginManagement`和`plugins`。

2. 添加依赖

   例如：spring-boot-starter-web

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

3. 编写Main类

```java
@SpringBootApplication
public class ReadingListApplication {
	public static void main(String[] args) {
		SpringApplication.run(ReadingListApplication.class, args);
	}
}
```

然后编写业务代码

4. 运行

* `$ mvn spring-boot:run`

* 直接运行Main类

## Fat Jar或Executable Jar

如果想打包为可执行的jar（即依赖也打包到jar中），则可以通过`spring-boot-maven-plugin`实现，添加插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

```shell
# 打包
$ mvn clean package
# 执行
$ java -jar target/spring-boot-sample-0.0.1-SNAPSHOT.jar
```

## 容器部署

```java
public class ReadingListServletInitializer extends SpringBootServletInitializer {
@Override
protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
	return builder.sources(Application.class);
	}
}
```

## @SpringBootApplication

有`@SpringBootConfiguration`（`@Configuration`）、`@EnableAutoConfiguration`和`@ComponentScan`组成。