---
layout: post
title: "关于《VisaNet Processing Overview》"
categories: 业务
tags: business cross-border payment visa
---

## 原版

### VisaNet Processing

VisaNet is the core of Visa's centralized and modular processing network. VisaNet's core payment processing provides three essential functions in one reliable and secure package:

**·** ***\*Authorization:\**** The process of approving or declining a transaction before a purchase is finalized or cash is disbursed.

**·** ***\*Clearing:\**** The process of delivering final transaction data from Visa acquirers to Visa issuers for posting to the cardholder’s account. Clearing also includes the calculation of certain fees and charges that apply to the issuer and acquirer involved in the transaction, as well as the conversion of transaction amounts to the appropriate settlement currencies.

**·** ***\*Settlement:\**** The process of calculating, determining and reporting the net financial position of Visa’s issuers and acquirers for all transactions that are cleared.

### VisaNet Systems

### VisaNet Integrated Payment System

![image-20240131090625557](/images/image-20240131090625557.png)

The VisaNet Integrated Payment (V.I.P.) System is an online system that performs both authorization only and full financial transaction processing.

**·** ***\*V.I.P. Authorization Only (Dual Message):\**** With real-time authorization only processing, a separate transaction must be processed to complete the clearing process. This second transaction can be initiated in either V.I.P. or BASE II.

**·** ***\*V.I.P. Full Service (Single Message):\**** The V.I.P. message contains the information required to perform both authorization and clearing in a single transaction.

### BASE II

BASE II performs batch clearing processing capabilities for VisaNet. BASE II collects, calculates and compiles all of the transaction data from acquirers and issuers that is required to facilitate the financial clearing process.

### VisaNet Settlement Service

The VisaNet Settlement Service (VSS) consolidates the settlement of all Visa products and services into one process. VSS provides flexibility in defining settlement reporting hierarchies and their corresponding reporting and funds transfer points.

Technical Documentation

For technical publications relating to these systems, refer to the [VisaNet Technical Documentation](https://secure.visaonline.com/pages/4.1497) section.

 

## 论文概述

本文介绍了Visa公司的核心支付处理网络——VisaNet及其三个基本功能：授权、清算和结算。同时，文章还详细介绍了VisaNet系统的两个在线系统：V.I.P. Authorization Only（双消息）和V.I.P. Full Service（单消息）。其中，V.I.P. Authorization Only需要进行两次交易才能完成清算过程，而V.I.P. Full Service则可以在一个交易中完成授权和清算。这些系统的使用使得Visa能够提供可靠且安全的支付服务，并在全球范围内实现了高效的资金流动。



## 论文速读

高效、安全的支付处理网络

这一章节介绍了VisaNet的三个核心功能：授权、清算和结算。其中，授权是指在购买完成或现金支付前批准或拒绝交易的过程；清算则是将最终的交易数据从Visa收购方传递给Visa发行方，并计算涉及交易的各种费用和汇率转换；而结算则是对所有已清算的交易进行净财务状况的计算、确定和报告。此外，该章节还提到了VisaNet系统的不同服务模式，包括V.I.P.授权仅服务和V.I.P.全服务等。最后，该章节还介绍了一些技术文档和系统架构的相关信息。



## 中文版

### 签证处理系统

VisaNet 是 Visa 的集中式和模块化处理网络的核心。 VisaNet 核心支付处理在一个可靠且安全的包中提供三个基本功能：

* 授权: 在购买完成或现金分发之前批准或拒绝交易的过程。

* 清算: 从发卡机构向发卡行发送最终交易数据以记入持卡人账户的过程。清算也包括计算与交易相关的发卡行和收单行收取的某些费用和收费，以及将交易金额转换为适当的结算货币。

* 结算: 对所有已清算交易进行净额财务状况计算、 确定和报告的过程。

### 维萨网络系统

### 维萨 网络 集成 付款 系统

![image-20240131090625557](/images/image-20240131090625557.png)

支付（V.I.P.）系统 是一个在线系统，它执行仅授权和完全财务交易处理。

* 只有 V.I.P. 才能授权（双重消息）：实时授权仅处理时，必须进行单独的交易以完成清算过程。第二个交易可以在 VIP 或 BASE II 中发起。

* V.I.P.完整服务（单条消息）：VIP消息包含在一笔交易中完成授权和清算所需的信息。

### BASE II

BASE II 为 VisaNet 提供批量清算处理能力。BASE II 收集、计算并汇总发卡机构和收单机构在金融清算过程中所需的所有交易数据。

### 签证网络结算服务

签证结算服务（VSS）将所有Visa产品和服务的 结算合并为一个流程。VSS提供定义 结算报告层次结构及其相应的报告和资金转账点 的灵活性。

### 技术文档