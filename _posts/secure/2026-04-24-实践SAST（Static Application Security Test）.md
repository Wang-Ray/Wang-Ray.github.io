---
layout: post
title: "实践SAST（Static Application Security Testing）"
categories: secure
tags: SAST "Static Application Security Testing"
---

SAST 是 **Static Application Security Testing** 的缩写，中文一般叫：

👉 **静态应用安全测试**

------

# 一、简单理解（直白版）

SAST就是：

👉 **不运行程序，直接扫描源代码找安全漏洞**

可以把它理解成：

> “代码层面的安全体检”

------

# 二、它是怎么工作的？

SAST工具会做这些事：

- 解析源代码（Java / Python / Go 等）
- 建立代码调用关系（数据流、控制流）
- 找“危险写法”

比如：

- SQL拼接 → SQL注入
- 未校验输入 → XSS
- 明文密码 → 敏感信息泄露

------

# 三、举个例子

假设代码：

```java
String sql = "SELECT * FROM user WHERE name = '" + input + "'";
```

SAST会识别：

👉 input是用户输入
👉 拼接进SQL
👉 没过滤

➡️ 判定为：**SQL注入漏洞**

------

# 四、SAST的核心特点

### ✅ 优点

- **开发阶段就能发现漏洞（左移安全）**
- 不需要运行程序
- 覆盖面广（所有代码路径）

### ❌ 缺点

- 误报（false positive）可能较多
- 对运行时问题（如权限、配置）无能为力

------

# 五、和其他安全测试的区别

| 类型 | 全称       | 特点                       |
| ---- | ---------- | -------------------------- |
| SAST | 静态测试   | 看代码，不运行             |
| DAST | 动态测试   | 运行系统，从外部攻击       |
| IAST | 交互式测试 | 运行中分析（介于两者之间） |

------

# 六、一句话总结

👉 **SAST = 在代码层面提前发现安全漏洞的自动化工具**

------

如果你在做安全体系或DevSecOps，可以再往下走一步：

👉 我可以帮你画一张“SAST在CI/CD中的落地流程图”
或者给你一套“公司内部怎么推SAST”的实操方案