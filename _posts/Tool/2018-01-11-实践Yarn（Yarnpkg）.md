---
layout: post
title: "实践Yarn（Yarnpkg）"
date: 2017-08-07 15:58:13 +0800
categories: 工具
tags: yarn yarnpkg tool software package
---

## 介绍

[Yarn（Yarnpkg）](https://yarnpkg.com), Fast, reliable, and secure dependency management.

## 安装

```shell
$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```

```shell
$ sudo apt-get update && sudo apt-get install yarn
```

**备注：**貌似`~/.yarn/bin`需要加入环境变量PATH中，否则无法全局使用。

## 基本使用

**Starting a new project**

```shell
$ yarn init
```

**Adding a dependency**

```shell
$ yarn add [package]
$ yarn add [package]@[version]
$ yarn add [package]@[tag]
# 全局
$ yarn global add [package]
```

**Adding a dependency to different categories of dependencies**

Add to `devDependencies`, `peerDependencies`, and `optionalDependencies` respectively:

```shell
$ yarn add [package] --dev
$ yarn add [package] --peer
$ yarn add [package] --optional
```

**Upgrading a dependency**

```shell
$ yarn upgrade [package]
$ yarn upgrade [package]@[version]
$ yarn upgrade [package]@[tag]
```

**Removing a dependency**

```shell
$ yarn remove [package]
```

**Installing all the dependencies of project**

```shell
$ yarn
```

or

```shell
$ yarn install
```