---
layout: post
title: "实践CSRF（Cross Site Request Forgery）"
categories: secure
tags: SOP "Cross Site Request Forgery" CORS CSRF
---

[Cross-Site Request Forgery (CSRF) ](https://owasp.org/www-community/attacks/csrf) is an attack that forces an end user to execute unwanted actions on a web application in which they’re currently authenticated. With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker’s choosing. If the victim is a normal user, a successful CSRF attack can force the user to perform state changing requests like transferring funds, changing their email address, and so forth. If the victim is an administrative account, CSRF can compromise the entire web application.

符合SOP，虽然读不到有效应答，但是实际还是发送了请求，由于不关心应答，以此来进行攻击。



### 防御方法

常见三种：

1. CSRF Token

   - 每次请求带一个随机 token

2. **SameSite Cookie**

   ```
   Set-Cookie: SameSite=Strict
   ```

3. **Referer / Origin 校验**