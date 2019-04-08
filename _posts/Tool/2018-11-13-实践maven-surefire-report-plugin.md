---
layout: post
title: "实践maven-surefire-report-plugin"
date: 2018-11-13 16:03:13 +0800
categories: 工具
tags: maven tool maven-surefire-report-plugin test
---


[Maven Surefire Report Plugin](http://maven.apache.org/surefire/maven-surefire-report-plugin/index.html) parses the generated `TEST-*.xml` files under `${basedir}/target/surefire-reports` and renders them using DOXIA, which creates the web interface version of the test results.

## Goals

The Surefire Report Plugin only has one goal (the other is a workaround):

- [surefire-report:failsafe-report-only](http://maven.apache.org/surefire/maven-surefire-report-plugin/failsafe-report-only-mojo.html) This goal does not run the tests, it only builds the IT reports. See [SUREFIRE-257](https://issues.apache.org/jira/browse/SUREFIRE-257)
- [surefire-report:report](http://maven.apache.org/surefire/maven-surefire-report-plugin/report-mojo.html) Generates the test results report into HTML format.
- [surefire-report:report-only](http://maven.apache.org/surefire/maven-surefire-report-plugin/report-only-mojo.html) This goal does not run the tests, it only builds the reports. It is provided as a work around for [SUREFIRE-257](https://issues.apache.org/jira/browse/SUREFIRE-257)

## 使用

### Generate the Report in a Standalone Fashion

`mvn surefire-report:report`

### Generate the Report as Part of Project Reports

```xml
<reporting>
    <plugins>
        <plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-surefire-report-plugin</artifactId>
		</plugin>
    </plugins>
</reporting>
```

`mvn site`



