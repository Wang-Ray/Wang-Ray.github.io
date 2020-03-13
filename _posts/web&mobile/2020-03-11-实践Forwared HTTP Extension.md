---
layout: post
categories: 移动互联网
tags: web http X-Forwarded-For X-Forwareded-Host X-Forwarded-Port X-Forwarded-Proto Forwarded
---

[Forwarded HTTP Extension](https://tools.ietf.org/html/rfc7239)

[Forwarded](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded)：`Forwarded: by=<identifier>;for=<identifier>;host=<host>;proto=<http|https>`

[X-Forwarded-Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host)：客户端的原始请求主机

[X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)：客户端请求到达服务器经过的IP列表，逐级追加为谁代理转发。比如：`X-Forwarded-For: <client>, <proxy1>, <proxy2>`，代表client，经过了proxy1和proxy2和proxy3的代理转发，到达服务端，proxy3没有出现是因为没人为他转发了。

[X-Forwarded-Proto](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto)：

X-Forwarded-Port：



## 参考

[Nginx 302](/网络/2018/07/10/实践Nginx/#302)

