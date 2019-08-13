---
layout: post
title: "实践ssh-copy-id"
date: 2019-07-28 13:53:00 +0800
categories: Linux
tags: linux ssh-copy-id
---

将当前用户的公钥安装到远程服务器指定用户的`~/.ssh/authorized_keys`，可以实现ssh、sftp等免密登陆。

命令：

```shell
$ ssh-copy-id -h
Usage: /usr/bin/ssh-copy-id [-h|-?|-f|-n] [-i [identity_file]] [-p port] [[-o <ssh -o options>] ...] [user@]hostname
	-f: force mode -- copy keys without trying to check if they are already installed
	-n: dry run    -- no keys are actually copied
	-h|-?: print this help
```

样例：

```shell
$ ssh-copy-id app@192.168.1.183
$ ssh-copy-id -i ~/.ssh/id_rsa_test app@192.168.1.183
$ ssh-copy-id -p 60022 app@192.168.1.183
```

