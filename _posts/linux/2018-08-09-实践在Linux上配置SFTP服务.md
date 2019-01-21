---
layout: post
title: "实践在Linux上配置SFTP服务"
date: 2018-08-09 13:53:00 +0800
categories: Linux
tags: linux sftp
---

## 配置sftp服务

以下操作都需要root权限

```shell
# 添加sftp用户组（例如：sftpusers）
$ groupadd sftpusers
```

编辑`/etc/ssh/sshd_config`

注释下面这行，不需要了

```
# Subsystem sftp /usr/libexec/openssh/sftp-server 
```

添加下面6行

```
Subsystem sftp internal-sftp
Match Group sftpusers # 匹配用户组
ChrootDirectory /home/%u # 锁定用户访问的根文件夹，属主必须是root
ForceCommand internal-sftp
AllowTcpForwarding no
X11Forwarding no # 设置不允许SSH的X转发
```

重启服务

```shell
$ service sshd restart
```

## 添加用户

*已添加testuser用户为例*

```shell
# 添加sftp用户，不允许ssh登录
$ useradd -g sftpusers -s /sbin/nologin testuser
# 用户主目录属主必须是root
$ chown root:sftpusers /home/testuser
# 权限可以是755或者750
$ chmod 755 -R /home/testuser
$ passwd testuser
```

### 新建目录

由于用户主目录只读（属主是root，权限是755或750），如果需要写权限，变相的通过新建子目录实现。

新建testuser读写目录，其他用户默认只读（755）

```shell
$ mkdir /home/testuser/uploads
$ chown testuser:sftpusers /home/testuser/uploads
```

新建testuser只读，但是非sftpusers用户读写的目录

```shell
$ mkdir /home/testuser/downloads
# 修改目录属主为其他非sftpuser组的指定用户，比如app用户
$ chown app:sftpusers /home/testuser/downloads
# 或者修改权限支持其他用户读写，风险较大
$ chown testuser:sftpusers /home/testuser/downloads
$ chown 557 -R /home/testuser/downloads
```

备注：其实新建的目录是否能够上传是看Linux的目录权限配置。

## 测试

```shell
$ sftp testuser@192.168.70.139
testuser@192.168.70.139's password: 
Connected to 192.168.70.139.
sftp> pwd
Remote working directory: /
sftp> ls
work  
sftp> put LICENSE.txt 
Uploading LICENSE.txt to /LICENSE.txt
remote open("/LICENSE.txt"): Permission denied
sftp> cd work/
sftp> put LICENSE.txt 
Uploading LICENSE.txt to /work/LICENSE.txt
LICENSE.txt
sftp>
```

## 常用命令

### put

上传文件

```
sftp>put test.txt
```

上传文件夹

```
sftp> put -r test
```

备注：上传文件夹则需要远程服务器有同名文件夹，否则报错。

### get

下载文件

```
sftp>get test.txt
```

下载文件夹

```
sftp>get -r test
```

### lcd

切换本地目录，即从本地上传或下载到本地的本地相对目录

## QA

### ChrootDirectory配置目录的属主非root

```shell
$ sftp testuser@192.168.70.139
testuser@192.168.70.139's password: 
packet_write_wait: Connection to 192.168.70.139 port 22: Broken pipe
Couldn't read packet: Connection reset by peer
```

ChrootDirectory配置的目录的属主不是root导致