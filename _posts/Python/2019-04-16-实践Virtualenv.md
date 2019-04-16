---
layout: post
title: "实践Virtualenv"
date: 2019-04-16 11:08:00 +0800
categories: Python
tags: python Virtualenv
---

[Virtualenv](https://virtualenv.pypa.io/en/latest/) is a tool to create isolated Python environments. Since Python 3.3, a subset of it has been integrated into the standard library under the [venv module](https://docs.python.org/3/library/venv.html). Note though, that the `venv`module does not offer all features of this library (e.g. cannot create bootstrap scripts, cannot create virtual environments for other python versions than the host python, not relocatable, etc.). Tools in general as such still may prefer using virtualenv for its ease of upgrading (via pip), unified handling of different Python versions and some more advanced features.

可以理解为maven wrapper或gradle wrapper之类的东西，隔离项目环境信息。

如果使用PyCharm，则默认已经集成了Virtualenv，可以参见[实践PyCharm](/Python/2019/04/15/实践PyCharm)

## 快速开始

### 安装

```shell
# virtualenv安装到~/.local/bin，将其加到环境变量PATH里面
$ pip3 install virtualenv
$ pip3 freeze |grep virtualenv
virtualenv==16.4.3
```



### 为项目创建virtualenv

```shell
$ mkdir myproject
$ cd myproject/

# 在venv目录下创建环境
$ python3 -m venv venv
# 或
$ virtualenv venv --python=python3

$ tree -L 2
.
└── venv
    ├── bin
    ├── include
    ├── lib
    ├── lib64 -> lib
    ├── pyvenv.cfg
    └── share
```

### 使用

```shell
# 进入
~/PycharmProjects/myproject$ source venv/bin/activate
(venv) ray@Ray-TR-01002:~/PycharmProjects/myproject$

# 退出
(venv) ray@Ray-TR-01002:~/PycharmProjects/untitled$ deactivate 
ray@Ray-TR-01002:~/PycharmProjects/untitled$ 
```

sys.path

```shell
~/PycharmProjects/myproject$ source venv/bin/activate
(venv) ray@Ray-TR-01002:~/PycharmProjects/myproject$ python
Python 3.6.5 (default, May 11 2018, 13:30:17) 
[GCC 7.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/home/ray/PycharmProjects/myproject/venv/lib/python3.6/site-packages', '/home/ray/PycharmProjects/myproject/venv/lib/python3.6/site-packages/setuptools-39.1.0-py3.6.egg', '/home/ray/PycharmProjects/myproject/venv/lib/python3.6/site-packages/pip-10.0.1-py3.6.egg']
>>> 
```

## Q&A

```shell
$ python3 -m venv venv
The virtual environment was not created successfully because ensurepip is not
available.  On Debian/Ubuntu systems, you need to install the python3-venv
package using the following command.

    apt-get install python3-venv

You may need to use sudo with that command.  After installing the python3-venv
package, recreate your virtual environment.

Failing command: ['/home/ray/PycharmProjects/sample01/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']
```

