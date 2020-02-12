---
layout: post
categories: 网络
tags: network VPN L2TP IPsec
---

OS：Deepin 15.11 64位

## 报错”vpn服务启动失败“

开启vpn或连接前（报错”vpn服务启动失败“），通过如下命令，可以开启日志跟踪。

```shell
$ sudo journalctl -u NetworkManager -f
...
ENCRYPTION_ALGORITHM 3DES_CBC (key size 0) not supported!
...
```

从日志来看缺少对3DES_CBC算法的支持。

```shell
$ sudo ipsec start # 启动ipsec
$ sudo ipsec statusall # 列出插件加载状态  
$ sudo ipsec listalgs  # 列出支持算法
```

安装`strongswan`的相关插件，支持更多的安全算法，参考如下：

```shell
$ apt install libcharon-extra-plugins
$ apt install libstrongswan-standard-plugins
$ apt install libstrongswan-extra-plugins
```

通过如下命令可以查询vpn服务端的安全算法。

先安装`ike-scan`

```shell
$ sudo apt install ike-scan
```

新建文件`ike-scan.sh`，参考如下：

```shell
#!/bin/bash

# Encryption algorithms: 3des=5, aes128=7/128, aes192=7/192, aes256=7/256
ENCLIST="5 7/128 7/192 7/256"
# Hash algorithms: md5=1, sha1=2, sha256=5, sha384=6, sha512=7
HASHLIST="1 2 5 6 7"
# Diffie-Hellman groups: 1, 2, 5, 14, 15, 19, 20, 21
GROUPLIST="1 2 5 14 15 19 20 21"
# Authentication method: Preshared Key=1
AUTH=1

for ENC in $ENCLIST; do
	   for HASH in $HASHLIST; do
		          for GROUP in $GROUPLIST; do
				            echo ike-scan --trans=$ENC,$HASH,$AUTH,$GROUP -M "$@"
					              ike-scan --trans=$ENC,$HASH,$AUTH,$GROUP -M "$@"
						            done
							       done
						       done

```

查询

```shell
$ sudo ipsec stop
$ sudo ./ike-scan.sh *.*.*.* | grep SA=
SA=(Enc=3DES Hash=MD5 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
```

则`密钥交换协议`为`3des-sha1-modp1024`或`3des-sha1-modp1024`或`3des-sha1-modp1024,3des-sha1-modp1024`

`安全封装协议`为`3des-md5`或`3des-sha1`或`3des-sha1,3des-md5`

## 参考

[解决：deepin连接预共享秘钥的L2TP/IPSec的VPN下，出现“连接vpn失败，原因未知”](https://www.geek-share.com/detail/2746771460.html)