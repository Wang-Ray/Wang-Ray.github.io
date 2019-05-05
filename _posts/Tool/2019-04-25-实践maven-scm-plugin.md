---
layout: post
title: "实践maven-scm-plugin"
date: 2019-04-25 16:03:13 +0800
categories: 工具
tags: maven tool maven-scm-plugin scm
---
The SCM Plugin ([maven-scm-plugin](http://maven.apache.org/scm/maven-scm-plugin/)) offers vendor independent access to common scm commands by offering a set of command mappings for the configured scm. Each command is implemented as a goal.



```xml
<!-- 提交../bin/run.sh文件 -->
<plugin>
	<artifactId>maven-scm-plugin</artifactId>
	<configuration>
		<message>[maven-scm-plugin] set ${project.version} version in run shell</message>
		<includes>../bin/run.sh</includes>
	</configuration>
</plugin>
```

