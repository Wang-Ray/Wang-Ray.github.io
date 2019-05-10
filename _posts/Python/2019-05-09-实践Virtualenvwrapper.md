---
layout: post
title: "实践Virtualenvwrapper"
date: 2019-05-09 11:08:00 +0800
categories: Python
tags: python Virtualenv Virtualenvwrapper
---


[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) is a set of extensions to Ian Bicking’s [virtualenv](https://pypi.python.org/pypi/virtualenv) tool. The extensions include wrappers for creating and deleting virtual environments and otherwise managing your development workflow, making it easier to work on more than one project at a time without introducing conflicts in their dependencies.

基于virtualenv，实现多个python版本和环境的管理和使用。

## 安装

```shell
$ pip3 install virtualenvwrapper
```

设置环境变量（可以加入到`~/.bashrc`）

```shell
# 
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
# 可选，设置virtualenv的统一管理目录，默认值～/.virtualenvs
export WORKON_HOME=~/envs
# 可选，预设virtualenv参数
export VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'
source ~/.local/bin/virtualenvwrapper.sh
```

## 使用

```shell
# 创建虚拟环境env1（python3）
$ mkvirtualenv env1
# 创建虚拟环境env2（指定为python2）
$ mkvirtualenv env2 --python=/usr/bin/python
# 切换到env1
$ workon env1
# 退出虚拟环境
$ deactivate

# 列举所有的环境
$ lsvirtualenv
# 导航到当前激活的虚拟环境的目录中，比如说这样您就能够浏览它的 site-packages 。
$ cdvirtualenv
# 和上面的类似，但是是直接进入到 site-packages 目录中。
$ cdsitepackages
# 显示 site-packages 目录中的内容。
$ lssitepackages
```



## Q&A

