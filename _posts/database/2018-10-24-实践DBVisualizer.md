---
layout: post
title: "实践DBVisualizer"
date: 2018-10-24 11:08:00 +0800
categories: 数据库
tags: Database DBVisualizer
---





指定jdk

```shell
export INSTALL4J_JAVA_HOME=~/platform/jdk1.8.0_181_linux
```



## 中文乱码

Tools-》Tool Properties-》General-》Appearance-》Fonts

Grids和Text Editors均选择`微软雅黑`和size 16

## 自动提示和补全

Tools-》Tool Properties-》General-》SQL Commander-》Auto Completion，勾选Display Automatically

## 去掉自动补全schema

Tools-》Tool Properties-》Database-》MySQL-》Qualifier，取消勾选两个Auto Completion/Query Builder