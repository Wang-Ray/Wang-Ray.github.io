---
layout: post
title: "实践代码质量管理之SonarQube"
date: 2018-08-22 11:22:00 +0800
categories: 测试
tags: test Sonar SonarQube
---

[SonarQube](https://www.sonarqube.org/), The leading product for Continuous Code Quality.

## 安装

下载（sonarqube-7.4.zip）解压即可。

新建数据库schema和user。

修改配置文件sonar.properties，设置数据库连接相关信息。

```shell
$ sonar.sh { console | start | stop | restart | status | dump }
```

## plugin

插件可以通过Web页面上的marketplace安装，或者人工安装插件jar到`extensions/plugins`目录。

[Community plugins for SonarQube](https://github.com/SonarQubeCommunity)

[[Plugin Library](https://docs.sonarqube.org/display/PLUG/Plugin+Library)](https://docs.sonarqube.org/display/PLUG/Plugin+Library)

[[Plugin Version Matrix](https://docs.sonarqube.org/display/PLUG/Plugin+Version+Matrix)](https://docs.sonarqube.org/display/PLUG/Plugin+Version+Matrix)

SonarJava

[findbugs](https://github.com/spotbugs/sonar-findbugs/)

[cobertura](https://github.com/galexandre/sonar-cobertura)

[checkstyle](https://github.com/checkstyle/sonar-checkstyle)

[pmd](https://github.com/jensgerdes/sonar-pmd)

[chinese pack](https://github.com/SonarQubeCommunity/sonar-l10n-zh)

[Android Lint](https://github.com/ofields/sonar-android)

## JaCoCo

SonarQube默认支持JaCoCo，从`jacoco.exec`和`jacoco-it.exec`【`sonar.jacoco.reportPaths`】生成测试覆盖相关数据。

```shell
$ mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install
$ mvn sonar:sonar
```

参考[Usage of JaCoCo with SonarJava](https://docs.sonarqube.org/display/PLUG/Usage+of+JaCoCo+with+SonarJava)

## spotbugs

sonarcube从spotbugs生成的reports导入

```xml
<sonar.java.spotbugs.reportPaths>./target/spotbugsXml.xml</sonar.java.spotbugs.reportPaths>
```



```shell
$ mvn clean package com.github.spotbugs:spotbugs-maven-plugin:spotbugs sonar:sonar
```



## PMD

sonarcube从pmd生成的reports导入

```xml
<sonar.java.pmd.reportPaths>./target/pmd.xml</sonar.java.pmd.reportPaths>
```



```shell
$ mvn clean package pmd:pmd sonar:sonar
```



## checkstyle

sonarcube从checkstyle生成的reports导入

```xml
<sonar.java.checkstyle.reportPaths>./target/checkstyle-result.xml</sonar.java.checkstyle.reportPaths>
```



```shell
$ mvn clean package checkstyle:checkstyle sonar:sonar
```

