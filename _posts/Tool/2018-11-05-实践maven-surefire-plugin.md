---
layout: post
title: "实践maven-surefire-plugin"
date: 2018-11-05 16:03:13 +0800
categories: 工具
tags: maven tool maven-surefire-plugin
---
[Maven Surefire Plugin](http://maven.apache.org/plugins/maven-surefire-plugin/)用于单元测试

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<configuration>
        <!-- 跳过单元测试（包括编译） ，user property: maven.test.skip-->
		<skip>true</skip>
        <!-- 跳过单元测试，user property: skipTests -->
        <skipTests>true</skipTests>
	</configuration>
</plugin>
```

默认单元测试类命名规则：

```xml
<includes>
    <include>**/Test*.java</include>
    <include>**/*Test.java</include>
    <include>**/*Tests.java</include>
    <include>**/*TestCase.java</include>
</includes>
```
## 参数

| 参数              | user property             | 默认值 | 描述             |
| ----------------- | ------------------------- | ------ | ---------------- |
| testFailureIgnore | maven.test.failure.ignore | false  | 是否忽略测试失败 |
|                   |                           |        |                  |
|                   |                           |        |                  |



## report

生成html格式的报告