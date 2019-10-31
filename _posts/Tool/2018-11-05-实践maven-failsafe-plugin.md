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



## QA

### what is the different between maven-surefire-plugin and mave-failsafe-plugin?

- [`maven-surefire-plugin`](https://maven.apache.org/surefire/maven-surefire-plugin/) is designed for running **unit tests** and if any of the tests fail then it will fail the build immediately.

- [`maven-failsafe-plugin`](https://maven.apache.org/surefire/maven-failsafe-plugin/) is designed for running **integration tests**, and decouples failing the build if there are test failures from actually running the tests.

  The name "*failsafe*" was chosen both because it is a synonym of surefire and because it implies that when it fails, it does so in a safe way.

  The **Failsafe Plugin** has two goals:

  - [`failsafe:integration-test`](https://maven.apache.org/surefire/maven-failsafe-plugin/integration-test-mojo.html) runs the integration tests of an application,
  - [`failsafe:verify`](https://maven.apache.org/surefire/maven-failsafe-plugin/verify-mojo.html) verifies that the integration tests of an application passed.

参见[Maven FAQ](http://maven.apache.org/surefire/maven-surefire-plugin/faq.html#surefire-v-failsafe)