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

### Spring Initializr

[Aliyun Java Initializr](https://start.aliyun.com/)

## 特性

### 起步依赖（Starter）

通过起步依赖，自动识别并构建环境

### 自动配置（Auto Configuration）

[实践Spring-Boot之EnableAutoConfiguration注解](/java/2017/08/22/实践Spring-Boot之EnableAutoConfiguration注解/)

### 外部化配置（External Configuration）

集中配置到`application.properties`，起步依赖利用配置来驱动构建环境

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

## CommandLineRunner

```java
/**
 * Interface used to indicate that a bean should <em>run</em> when it is contained within
 * a {@link SpringApplication}. Multiple {@link CommandLineRunner} beans can be defined
 * within the same application context and can be ordered using the {@link Ordered}
 * interface or {@link Order @Order} annotation.
 * <p>
 * If you need access to {@link ApplicationArguments} instead of the raw String array
 * consider using {@link ApplicationRunner}.
 *
 * @author Dave Syer
 * @since 1.0.0
 * @see ApplicationRunner
 */
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```

## ApplicationRunner

```java
/**
 * Interface used to indicate that a bean should <em>run</em> when it is contained within
 * a {@link SpringApplication}. Multiple {@link ApplicationRunner} beans can be defined
 * within the same application context and can be ordered using the {@link Ordered}
 * interface or {@link Order @Order} annotation.
 *
 * @author Phillip Webb
 * @since 1.3.0
 * @see CommandLineRunner
 */
@FunctionalInterface
public interface ApplicationRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming application arguments
	 * @throws Exception on error
	 */
	void run(ApplicationArguments args) throws Exception;

}
```



## QA

### InetAddress.getLocalHost().getHostName() took 90037 milliseconds to respond

```
2022-06-20 14:56:33,769 WARN [main] o.s.boot.StartupInfoLogger [StartupInfoLogger.java : 116] InetAddress.getLocalHost().getHostName() took 90037 milliseconds to respond. Please verify your network configuration.
```

