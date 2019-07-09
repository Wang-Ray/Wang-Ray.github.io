---
layout: post
title: "实践Android Studio"
date: 2019-01-07 09:03:13 +0800
categories: 移动互联网
tags: mobile Android Android-Studio
---



设定Android SDK Location，比如：/media/ray/Resource/Android/Sdk



Consider using an x86 system image on an x86 host for better emulation performance.



### 模拟器kvm（/dev/kvm）权限问题

第一步:安装qemu-kvm，变更/dev/kvm的组从root-->kvm:

```shell
$ sudo apt install qemu-kvm
```

查看/dev/kvm所属的组：

```shell
$ ls -al /dev/kvm
crw-rw---- 1 root kvm 10, 232 11月 28 08:17 /dev/kvm
```

查看kvm组的信息：

```shell
$ grep kvm /etc/group
　kvm:x:127:
```

第二步:将当前用户ray添加到kvm组中：

```shell
$ sudo adduser ray kvm
# 查看kvm组的信息
$ grep kvm /etc/group
　kvm:x:127:ray
```

第三步:重启机器，验证OK，不再提示权限问题。



注意：项目路径不支持软引用（Android Gradle Plugin不支持）
