---
layout: post
title: 实践Forwarded HTTP Extension
categories: 移动互联网
tags: web http X-Forwarded-For X-Forwareded-Host X-Forwarded-Port X-Forwarded-Proto Forwarded
---

[Forwarded HTTP Extension](https://tools.ietf.org/html/rfc7239)

[Forwarded](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded)：`Forwarded: by=<identifier>;for=<identifier>;host=<host>;proto=<http|https>`

[X-Forwarded-Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host)：客户端的原始请求目标主机，比如api.***.com

[X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)：客户端请求到达服务器经过的IP列表，逐级追加为谁代理转发。比如：`X-Forwarded-For: <client>, <proxy1>, <proxy2>`，代表client，经过了proxy1和proxy2和proxy3的代理转发，到达服务端，proxy3没有出现是因为没人为他转发了。

[X-Forwarded-Proto](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto)：

X-Forwarded-Port：



# 后端获取真实客户端 IP 完整方案

核心原则：

**原始请求头（X-Forwarded-For、X-Real-IP）客户端都能随便伪造；只有经过你可控、可信的前置代理（Nginx、负载均衡、CDN）处理后，IP 才可信。**

## 一、网络架构决定获取方式

### 场景 1：客户端直接直连后端服务器（无 CDN、无反向代理）

最简单，直接读取连接来源 IP 即可，不存在伪造风险。

- Java：`request.getRemoteAddr()`
- Python Flask/Django：`request.remote_addr`
- PHP：`$_SERVER['REMOTE_ADDR']`
- Go：`r.RemoteAddr`

### 场景 2：后端前方有一层 / 多层代理（Nginx、SL、阿里云 / 腾讯云 CDN、负载均衡）

这是线上最常见架构，必须依靠代理传递 IP，**但要严格校验可信节点，防止 XFF 伪造**

链路：

用户浏览器 → 可信代理 / CDN → 后端服务

## 二、关键 HTTP 头说明（全部可被客户端伪造）

1. X-Forwarded-For（XFF）

格式：

```
伪造IP, 用户真实IP, 代理节点1, 代理节点2
```

规则：每经过一层可信代理，会把上一跳追加到末尾

> 左侧最靠前的 IP 最容易篡改，绝对不能直接拿第一个 IP

2. X-Real-IP

Nginx 常用，客户端可随意伪造

## 三、安全获取真实 IP 标准流程（重中之重）

1. 预先配置**可信代理 IP 列表 / 网段**（你的 CDN、负载均衡、内网 Nginx 节点 IP 段）
2. 切割 `X-Forwarded-For` 按逗号拆分为 IP 数组
3. **从数组末尾向前遍历**，剔除所有属于可信网段的代理 IP
4. 遍历结束后拿到的第一个 IP = 用户真实公网 IP

举例：

XFF：`1.1.1.1(伪造), 223.xx.xx.xx(用户真实IP), 10.0.0.1(内网可信代理)`

可信网段包含 10.0.0.1

从后往前删可信 IP，最终得到：223.xx.xx.xx

## 四、Nginx 前置统一处理（推荐最优方案）

把 IP 解析工作交给最外层 Nginx，后端直接拿最终真实 IP，避免各个语言重复写解析逻辑

### Nginx 安全配置

```nginx
# 信任的代理网段：CDN、负载均衡、内网代理地址
set_real_ip_from 10.0.0.0/8;
set_real_ip_from 172.16.0.0/12;
set_real_ip_from 192.168.0.0/16;
# 填写云厂商CDN官方IP段（阿里云、腾讯云等）

# 从XFF解析真实IP
real_ip_header X-Forwarded-For;
real_ip_recursive on;

# 传递给后端
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

配置完成后：

后端读取 `X-Real-IP` 就是纯净无篡改的用户真实 IP

## 参考

[Nginx 302](/网络/2018/07/10/实践Nginx/#302)

