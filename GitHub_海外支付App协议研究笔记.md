# 海外支付 App 协议研究笔记：UPI、商户流水、自动核销与版本维护

这个仓库主要记录我们在 Android 逆向和海外支付 App 研究过程中整理的一些资料。

近几年实际接触比较多的是印度 UPI，陆续分析过 PhonePe、PhonePe Business、Paytm、Paytm Business、MobiKwik、FreeCharge、GPay、BharatPe Business、super.money、Navi、Airtel、BHIM IndusPay 等。

另外也在关注孟加拉 bKash、Nagad，拉美 Mercado Pago，以及东南亚 TrueMoney。

这里简单记录一下我们做这类项目时比较关注的东西。

## 1. 先分清个人钱包和 Business

个人版通常围绕：

```text
OTP Login
Token / Refresh
User Profile
UPI ID / VPA
Bank Account
Balance
Transaction List
Transaction Detail
```

Business 版关注点明显不同：

```text
Merchant
Merchant ID
QR
Order
Collection
Transaction
Settlement
UTR
Reconciliation
```

如果业务目标是商户收款、流水同步、订单核销，通常不能只盯着个人钱包。

## 2. 流水接口只是开始

实际项目里，拿到 Transaction List 并不代表完成。

后台还要解决：

```text
流水分页 / 增量同步
交易详情
交易状态变化
重复数据
订单匹配
退款
Settlement
对账
异常重试
```

最终链路一般是：

```text
支付账户
  ↓
Transaction
  ↓
Transaction Detail
  ↓
后台同步
  ↓
Order Matching
  ↓
核销 / 对账
```

所以我们研究支付 App 时，不只是找一个“查流水接口”，而是尽量把账户、交易、订单和结算串起来。

## 3. Android 端重点看什么

拿到一个新的支付 App，一般先从业务入口找：

```text
Login
OTP
Account
UPI
Transaction
Merchant
Settlement
```

再逐步定位：

```text
Java / Kotlin
    ↓
DEX / Smali
    ↓
JNI
    ↓
SO
    ↓
ARM64
```

常用工具包括 JADX、Apktool、IDA、Binary Ninja、Frida 等。

网络部分主要整理：

```text
API Path
Method
Header
Token
Request
Response
Sign
Pagination
Error Code
```

协议文档如果以后还要交给后台使用，请求和响应样例、字段含义、Token 生命周期、分页方式和异常状态最好第一次就整理完整，否则后面维护很痛苦。

## 4. 支付 App 难在账户和设备不是两套独立逻辑

实际分析经常会碰到：

```text
userId
accountId
deviceId
merchantId
accessToken
session
appVersion
```

有些接口只和账户有关，有些还和设备、App 版本、当前 Session 有关系。

因此出现“登录成功但接口不能直接复用”的情况并不少见。

做协议分析之前，最好先把：

**账户 → 设备 → Token → API**

之间的关系搞清楚。

## 5. 版本维护比第一次跑通更重要

支付 App 更新很快。

常见变化包括：

```text
接口路径变化
Header 变化
Token 逻辑变化
请求字段增加
Response 结构变化
版本校验变化
业务流程调整
```

所以我们的项目资料一般按“钱包 + App Version”保存。

新版本出来以后先做 Diff，确定变化点，再决定协议、客户端和后台哪些地方需要更新。

长期项目真正有价值的不是某个版本跑通一次，而是官方升级以后还能快速定位问题。

## 6. 官方 API、协议和客户端要根据业务选

能使用官方 Merchant API、Payment Gateway、Webhook、Settlement API 的项目，优先考虑官方能力。

官方接口覆盖不到，而项目又有合法授权的测试或自有账户时，才有必要继续研究 App 协议或客户端侧的数据同步。

常见项目最后会落到三种结构：

```text
官方 API → 后台

App 协议 → 后台

Android 客户端 → 数据同步 → 后台
```

具体选哪一种，看国家、钱包、账户类型、业务目标和后期维护要求。

## 7. 目前持续整理的方向

仓库后续主要更新：

- Android APK / DEX / SO 逆向
- JADX、IDA、Frida 实战笔记
- Android 网络协议分析
- Token、账户与 Device Binding
- 印度 UPI / PhonePe / Paytm 等支付 App
- Business 商户端业务结构
- Transaction / Settlement / Reconciliation
- 支付流水同步与订单核销
- App 版本升级 Diff 与维护方法

我们更关注真实项目里碰到的问题，而不是单纯整理工具教程。

如果你也在做 **印度 UPI、PhonePe、Paytm、海外钱包、商户流水、支付协议、Android 客户端定制、Transaction/Settlement 对账** 这类项目，可以通过 GitHub 查看后续更新和项目联系方式。

**GitHub：** https://github.com/goldenfish689/android-reverse

> 仓库内容用于 Android 安全研究、授权测试及支付系统技术研究。支付业务应使用自有/授权账户并遵守平台规则和当地法律。
