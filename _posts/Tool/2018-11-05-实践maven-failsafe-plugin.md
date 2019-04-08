---
layout: post
title: "实践maven-failsafe-plugin"
date: 2018-11-05 16:03:13 +0800
categories: 工具
tags: maven tool maven-failsafe-plugin
---
[Maven Failsafe Plugin](http://maven.apache.org/plugins/maven-failsafe-plugin/)用于集成测试

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-failsafe-plugin</artifactId>
	<configuration>
		<skip>true</skip>
	</configuration>
</plugin>
```



```
<includes>
    <include>**/IT*.java</include>
    <include>**/*IT.java</include>
    <include>**/*ITCase.java</include>
</includes>
```