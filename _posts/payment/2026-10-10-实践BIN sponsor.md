---
layout: post
categories: payment
tags: payment bank 8583 issuer "bin sponsor" sponsor "acquiring sponsor" "sponsoring acquirer"
---

### BIN Sponsor

**BIN Sponsor = BIN 赞助方 / BIN Sponsoring Bank（或机构）**。

在银行卡业务里，可以把它理解成：**一家已经是 Visa/Mastercard 会员、拥有 BIN 使用资格的持牌机构，把自己的卡组织会员资格和 BIN 能力“提供”给另一家金融科技公司使用。**

比如  想做 Visa 发卡，但某个业务主体自己不是 Visa Principal Member，不能直接获得/使用 Visa BIN，那么可以找一家 Visa Member 做 **BIN Sponsor**：

**Visa → BIN Sponsor（Visa Member）→ Fintech → Cardholder**

这里的 **BIN（Bank Identification Number）** 是卡号开头用于识别发卡机构/卡产品归属的一段数字。比如一张卡：

```
45678912 xxxx xxxx
```

前面的 IIN/BIN 可以识别这张卡属于哪个发卡体系。

BIN Sponsor 到底“赞助”什么？

它不是简单“借一个 BIN 给你”，更准确地说，是**用它的 Visa/Mastercard 会员身份，让非会员机构能够开展卡业务**。

例如：

> B公司 想给客户发行 Visa 商务卡，但 B公司 自己没有 Visa 发卡会员资格。
> 找银行 A 作为 BIN Sponsor。
> 银行 A 是 Visa Member，并向 Visa 承担最终责任。
> B公司 负责产品、客户、App、运营等。

架构可能是：

**Visa**
↓
**Bank A — BIN Sponsor / Issuer Member**
↓
**B公司 — Program Manager**
↓
**Issuer Processor**
↓
**Cardholder**

因此 BIN Sponsor 通常会特别关注 **KYC/AML、制裁筛查、欺诈、交易监控、PCI DSS、卡产品设计、资金流、合规运营**等，因为出了问题，卡组织最终会追到它这个 Member。

你也可以把前面几个概念放在一起理解：

| 角色                | 大白话                                     |
| ------------------- | ------------------------------------------ |
| **Visa Member**     | Visa 的正式会员                            |
| **BIN Sponsor**     | 用自己的会员资格/BIN能力支持别人开展卡业务 |
| **Program Manager** | 实际运营这个卡项目                         |
| **Processor**       | 技术上处理授权、清算等交易                 |
| **ISO**             | 帮收单机构拓展商户                         |
| **Acquirer**        | 正式承接商户收单                           |
| **PayFac**          | 在 Acquirer 下面管理一批 Sub-merchants     |

还有一个很关键的点：**BIN Sponsor 主要是发卡侧概念；收单侧也存在非常类似的 Sponsor 模式。** 如果你们是在研究自己成为 Visa Member 还是找 Sponsor，我可以接着把 **BIN Sponsor、Issuer、Program Manager、Principal Member** 之间的法律/资金/系统关系画清楚。

### Acquiring Sponsor / Sponsoring Acquirer

收单侧与 **BIN Sponsor** 最对应的概念通常是 **Acquiring Sponsor / Sponsoring Acquirer（赞助收单机构）**，在不同卡组织、地区和合作模式下名称会有差异。

核心逻辑是一样的：

**发卡侧：Visa → BIN Sponsor → Fintech / Program Manager → Cardholder**

**收单侧：Visa → Sponsoring Acquirer → PayFac / ISO / PSP → Merchant**

举个典型的 PayFac 场景。假设想给大量跨境电商商户提供 Visa 收单，但某个法人本身不是 Visa Acquirer Member，就可以和一家 Visa Acquirer 合作：

**Visa**
↓
**Acquirer / Sponsoring Acquirer**
↓
**（PayFac / PSP）**
↓
**Sub-Merchant**
↓
**消费者**

这里 Sponsoring Acquirer 的作用，本质上是：

**“我作为 Visa Member，为你的收单业务提供卡组织层面的 Sponsor，并对 Visa 承担相应责任。”**

因此它和 BIN Sponsor 可以这样对应：

| 发卡侧             | 收单侧                  |
| ------------------ | ----------------------- |
| Visa Issuer Member | Visa Acquirer Member    |
| **BIN Sponsor**    | **Sponsoring Acquirer** |
| Program Manager    | PayFac / PSP / ISO      |
| Cardholder         | Merchant / Sub-Merchant |
| Issuer Processor   | Acquirer Processor      |

不过有一个很重要的区别：**ISO、PayFac、PSP 在 Sponsor 下面的权限是不一样的。**

普通 **ISO** 更像销售代理，通常是帮助 Acquirer 找商户；**PayFac** 则更进一步，可以在 Sponsoring Acquirer 下面 onboarding 和管理自己的 **Sub-Merchants**；而 **PSP** 是比较宽泛的商业称呼，本身不能直接说明它在 Visa 规则下具体是什么身份。

所以你以后看到：

**Sponsored Merchant / Sponsored Entity / Third Party Agent / PayFac / ISO / Acquiring Sponsor**

不要简单认为都是一回事。关键要看三个问题：**谁是 Visa Member、谁和 Merchant 建立收单关系、谁最终对 Visa 承担责任。**

如果放到这种跨境收单业务里，**“自己成为 Visa Acquirer Member” vs “找 Sponsoring Acquirer 做 PayFac”** 是两条完全不同的路线，后者通常进入市场更快，但 Sponsor 对你们的商户准入、风险、合规、保证金和结算会有比较强的控制。