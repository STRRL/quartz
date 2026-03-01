---
title: "Hyperliquid 创始人 Jeff Yan 访谈笔记"
date: 2025-02-24
tags: ["crypto", "defi", "hyperliquid", "interview"]
---

# Hyperliquid 创始人 Jeff Yan 访谈笔记

**来源:** [When Shift Happens E159](https://www.youtube.com/watch?v=2cC9IjixIww)  
**嘉宾:** Jeff Yan — Hyperliquid Labs 创始人兼 CEO  
**时长:** ~89 分钟

---

## 核心观点

### 1. 为什么在 AI 接管前必须修好金融系统

这是整个访谈最核心的论点：

- AI 将在不远的未来取代人类智能，届时价值转移也将主要由机器驱动
- AI **不可能** 接入现有的传统金融系统——因为传统金融中，代码与价值、信息与价值不在同一层面
- 人类在 AI 到来前最重要的任务之一：**建设一个可编程、开放、无需许可的金融系统**
- 如果不这样做，人类将被排除在替代金融系统之外
- Hyperliquid 就是人类在这方面的 "最佳尝试"（our best shot）

> "Before AI hits escape velocity and renders human intelligence obsolete, there needs to be a financial system to which they can plug in."

### 2. Hyperliquid 的定位与哲学

**"Hyperliquid 不是加密公司，是用加密技术的金融公司"**

- 目标不是做最好的加密交易所，而是成为 **金融的互联网**（the internet for money）
- 类比 AWS：中立基础设施，不是某个公司的产品
- 金融系统不应由单一公司控制，应像互联网一样开放、可编程、全球可访问
- **"Housing all of finance"**：用户的金融生活不应碎片化，应在一个可组合的系统中完成

### 3. 技术架构与设计哲学

#### 核心原则：原语（Primitives）要小而通用
- 协议只构建最核心、最通用的原语
- 如果能由外部团队构建，就应该由外部构建
- 将有主见的决策（opinionated decisions）从原语中移出

#### HyperEVM
- 允许在 Hyperliquid 上部署以太坊兼容智能合约
- **关键差异**：EVM 合约可以调用 Hypercore 原生原语（预编译/core writer），这是其他 EVM 链不具备的
- 例子：Circle 的 USDC 原生铸造就是通过 HyperEVM 接入的
- 成功的集成做得越好越无缝，用户越不会意识到 HyperEVM 的独特性——这反而让叙事传播变难

#### HIP-3：无需许可的永续合约
- 任何人都可以在 Hyperliquid 上部署永续合约市场
- 之前从未有人尝试过，大多数人认为不会成功
- 成果：HIP-3 市场上的白银交易量已达全球白银交易量的约 2%
- 核心团队只提供原语，具体的流动性基础设施由部署者（如 Trade XYZ）运营

#### 组合保证金（Portfolio Margin）
- 允许用户用任何流动资产（如 BTC）作为抵押品交易任何市场
- DeFi 中的难题：中心化交易所可以凭空铸造余额扩展信用，但 DeFi 不能这样做（会带来平台风险）
- Hyperliquid 的方案：借助 EVM 上的借贷市场（如 Hyperlend）完全支撑组合保证金
- 需求高峰 → 借贷利率短暂上升 → 供应方进入套利 → 利率回到市场水平

#### 结果市场（Outcome Markets）
- 用于表达非线性信念：期权、预测市场、有界结构
- 完全抵押的合约，无清算风险
- 与现货和永续合约互补——现货和永续只能表达线性信念

### 4. 生态项目

| 项目 | 角色 |
|------|------|
| **USDH**（Native Markets 运营）| 协议对齐稳定币，收益流向国库+部署者，交易者享更低手续费 |
| **Kinetic** | 最大的流动质押代币，完全链上质押/解质押，无桥接信任风险 |
| **Hyperlend** | 首选借贷协议，未来将支撑组合保证金的借贷需求 |
| **Unit** | 现货交易构建者，2025 年多次实现链上首发价格发现 |

### 5. 团队哲学

#### 11 人团队，无 VC 融资
- 选人标准：技术顶尖 + 高诚信（integrity）
- 面试流程：技术考核 + 至少一整天协作；团队任何人强烈反对 = 否决
- **"If it's not a hell yeah, it's a no"**
- 不追求加班文化，注重输出质量而非工时
- TGE 后无人离职，团队反而更有归属感和动力

#### 为什么人这么少？
- 大量工作由生态去中心化完成——协议提供原语，社区构建应用
- Jeff 个人偏好：小团队 × 极强个体 > 大团队 × 各做一小块

#### 关于公平和诚信
- 公平不是成本收益分析，而是不可让渡的原则（unalienable）
- 承认走公平路线会更慢，但长期更健壮、更可持续
- 反例：FTX 靠走捷径快速扩张，最终崩盘

### 6. 代币经济与 FUD 回应

#### 代币回购争议
- Hyperliquid **没有** 自由裁量的回购计划
- 协议费用通过链上逻辑自动转换为 HYPE 并销毁——类似以太坊销毁优先费
- 转换策略本身也不是人为决定的，而是链状态机的一部分
- 人们错误地将 Hyperliquid 与中心化交易所的回购机制类比

#### 员工解锁
- Jeff 拒绝公开讨论个人代币操作，认为金融隐私是基本权利
- 协议层面的透明度（每一美元可追踪）是非妥协的，但个人隐私是底线

#### 处理 FUD
- 过去倾向于"真相自会浮出水面"，现在学会了主动回应虚假信息
- 1010 事件：Hyperliquid 因为是唯一透明的平台，反而成了被攻击的靶子
- 竞争对手利用不同的数据口径（如只推送每秒第一笔清算）制造误导性比较

### 7. 关于加密 vs AI 的职业选择

- 加密的坏名声导致很多优秀人才不考虑进入这个领域
- Jeff 认为这是错误的——加密的真正潜力远未被实现
- 改变方式：以身作则（lead by example），用实际成果证明
- Hyperliquid 的 $10B 空投就是一个正面案例：早期参与者获得了网络的有意义所有权，这是 AI/Web2 做不到的

### 8. 关于竞争

- "没有竞争者" 的意思不是没有对手，而是没有人在精确地做同一件事
- 很多看似竞争的协议其实是合作关系——越来越多协议在 Hyperliquid 上启动产品
- Hyperliquid 的独特性来自其路径依赖：无内部人原则、从第一天开始的公平、社区驱动

### 9. 个人风格

- 不庆祝里程碑（TGE 时没有开香槟）
- "总是在想还没做什么"，而非已经完成了什么
- 关注旅程而非目的地
- 不设定量化目标（"一个月后希望达到某个指标"不是他的风格）
- 新加坡很无聊——这正好适合专注构建

---

## 金句摘录

- "Hyperliquid is not a crypto company. It's a finance company using crypto."
- "I hope Hyperliquid becomes the internet for money."
- "The cost of fairness could be infinite, and the principle itself is unalienable."
- "Before AI renders human intelligence obsolete, there needs to be a financial system to which they can plug in."
- "People are motivated not by the amount of value you can extract from the system, but by the amount of value you can produce."
- "If it can be built externally, it should be."
- "Spot trading is the closest to Satoshi's original peer-to-peer vision."

---

## Jeff 的早期经历

- 2018 年曾与 Kalshi 同期做预测市场，未成功
- 原因：团队还没准备好，想法只是很小的一部分；熊市中无人愿意使用链上产品
- 教训：想法正确 ≠ 执行成功，时机和团队成熟度同样重要
- 现在通过 Outcome Markets 重新实现这个愿景

## Takeaway

- 下一代可编程的金融系统
