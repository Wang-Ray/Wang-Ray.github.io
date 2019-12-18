---
layout: post
title: "实践KVM on Deepin"
date: 2019-11-07 13:53:00 +0800
categories: Linux
tags: linux KVM
---

## 环境

主机：Deepin 15.11 64位

guest：CentOS 7.6.1810 64位

## 前置条件

检查cpu是否支持虚拟化

```shell
# intel vmx，amd svm
$ egrep '(vmx|svm)' /proc/cpuinfo
...vmx...

$ lscpu | grep Virtualization
Virtualization:        VT-x
```

检查kvm模块是否已加载

```shell
$ lsmod | grep -i kvm
kvm_intel             183621  31 
kvm                   586948  1 kvm_intel
irqbypass              13503  21 kvm
```

## 安装KVM

```shell
$ sudo apt install virt-manager bridge-utils libvirt-clients qemu qemu-kvm
```

## virbr0

默认会生成一个虚拟网卡`virbr0`

```
virbr0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:e2:ad:e1  txqueuelen 1000  (Ethernet)
        RX packets 200  bytes 6552 (6.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 103  bytes 17430 (17.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

控制virbr0状态

![kvm-deepin-connection-details](/images/kvm-deepin-connection-details.png)

## 安装guest

启动virt-manager，利用X11会得到界面

```shell
$ sudo virt-manager

# 当需要连接远程的kvm时
$ sudo virt-manager --no-fork
```

### 网路配置

使用NAT网络模式，采用dhcp获取ip，也可以指定ip（static）

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=5609329a-0c31-4f41-9990-21627f6451bb
DEVICE=eth0
ONBOOT=yes
#IPADDR=192.168.1.239
#NETMASK=255.255.255.0
DNS1=210.22.84.3
#GATEWAY=192.168.1.1
ZONE=public
```

使用网桥模式，网桥名称virbr0，指定ip（static）

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=5609329a-0c31-4f41-9990-21627f6451bb
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.122.2
NETMASK=255.255.255.0
DNS1=210.22.84.3
GATEWAY=192.168.122.1
ZONE=public
```



## new image

## clone image

![kvm-clone-1](/images/kvm-clone-1.png)

![kvm-clone-2](/images/kvm-clone-2.png)

## import image

![kvm-import-1](/images/kvm-import-1.png)

![kvm-import-2](/images/kvm-import-2.png)

![kvm-import-3](/images/kvm-import-3.png)

![kvm-import-4](/images/kvm-import-4.png)

![kvm-import-5](/images/kvm-import-5.png)

使用NAT模式

![kvm-import-6](/images/kvm-deepin-network-nat.png)

使用网桥模式，填写网桥名称

![kvm-import-6](/images/kvm-import-6.png)

## Q&A

### CPU mode "custom" not supported

Unable to complete install: 'unsupported configuration: CPU mode 'custom' for x86_64 kvm domain on x86_64 host is not supported by hypervisor'

修改`/etc/libvirt/qemu.conf`，配置user和group，例如：

```
user = "ray"
group = "ray"
```

重启后解决。