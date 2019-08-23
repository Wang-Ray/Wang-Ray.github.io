---
layout: post
title: "实践maven-surefire-plugin"
date: 2018-11-05 16:03:13 +0800
categories: 工具
tags: maven tool maven-surefire-plugin
---
[Maven Surefire Plugin](http://maven.apache.org/plugins/maven-surefire-plugin/)用于单元测试

跳过单元测试

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<configuration>
		<skip>true</skip>
	</configuration>
</plugin>
```

或

-Dmaven.test.skip=true：

默认单元测试类命名规则：

```
<includes>
    <include>**/Test*.java</include>
    <include>**/*Test.java</include>
    <include>**/*Tests.java</include>
    <include>**/*TestCase.java</include>
</includes>
```
## report

生成html格式的报告