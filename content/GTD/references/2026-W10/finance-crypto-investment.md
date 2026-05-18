---
title: "金融/Crypto/投资 参考资料"
date: 2026-03-08
week: "2026-W10"
tags: [finance, crypto, investment, exchange, binance, coinbase, ai-engineering]
linear_ids: [ZZ-72, ZZ-102, ZZ-478, ZZ-39, ZZ-291]
---

# 金融/Crypto/投资 参考资料（2026-W10）

## ZZ-72 — 交易所如何把 Order Book 变成 Distributed Log

**来源：** [quant.engineering](https://quant.engineering/exchange-order-book-distributed-logs.html)

现代交易所的核心架构是 **Gateway → Sequencer → Matching Engine**：

- **Sequencer 是关键**：所有订单必须经过单一 Sequencer 分配单调递增的 sequence number，创造全局 total order。分布式时钟有微秒级漂移，单靠 timestamp 无法保证顺序。
- **Order Book = Log 的 Materialized View**：内存中的 order book 就是 append-only event stream 的 projection。每个 event 只追加，cancel/modify 也是新 append，不覆盖历史。
- **Event 结构极简**：`[seq_num, timestamp, order_id, event_type, price, quantity, metadata]`
  - `NEW_ORDER` → 加入 price-level FIFO 队列
  - `CANCEL` → 删除
  - `TRADE` → 从队头 pop
  - `MODIFY` → 删除 + 重新插入
- **Sequencer 是性能瓶颈**：全序要求强制串行化，可以 per-instrument 分片，但单 order book 内不可并行。
- **纳秒级工程**：kernel bypass、L1/L2 cache locality、NUMA pinning、zero-copy shared buffer，Matching Engine 目标延迟为十几纳秒。
- **系统选 CP 不选 AP**：Price-time priority 要求严格排序，任何序号不一致都会产生不同的 trade winner。Gateway 无法连接 Sequencer 时拒绝订单，而不是降级接受。
- **Snapshot + Log Replay**：定期做 point-in-time snapshot，重启时只需 replay 后续增量 log，而不是从头重放全部历史。

---

## ZZ-102 — 何一采访：从四川山村到币安 Co-CEO

**来源：** [YouTube 专访（~65分钟）](https://www.youtube.com/watch?v=PTvYq0-PIY0)，采访者：Bonnie

### 成长背景
- 四川偏远山区长大，走路到县城需要一小时山路，小时候停电点煤油灯、挑桶打水。
- 父亲是村里老师，家有大书架（本草纲目到养猪技术），培养了阅读习惯。
- 父亲去世后家庭骤变，独自面对困境。**4岁半直接上一年级**，入学后即拿第一名。

### 核心心态：不怕输
> "你的起点足够低，你就觉得输了是正常的，赢了是赚的。"

- 没有得失心，纯粹出于好奇尝试新事物——看到同学演讲拿第一，自己试试也拿了第一；朋友参加模特大赛，跟着报名也进了省赛。

### 职业路径
1. **16岁** 做饮料促销员，很快升为管理者
2. **旅游卫视主持人**：面试时说"我会化妆可以省化妆师的钱，不在意薪水"，意外录取
3. **一下科技**（秒拍/小咖秀）：市场负责人，细节控，会跟分众传媒争广告位排序
4. **进入加密圈**：2013年比特币破 $1,000 时，朋友请她帮做广告 → 读了白皮书 → "第一次有人告诉你钱是什么"

### 加入币安
- 先在 OKCoin 工作；说服 CZ "你擅长交易系统，为什么不做交易所？"
- CZ ICO 融了约 **$1,000万** 后打来电话："我们现在有钱了，可以雇你了。"
- 上线前夜 CZ 施压："BNB 上线涨10倍后我给不了你同样的 offer，今天就决定。"
- 何一的判断依据：CZ 有西方背景，团队多样性强，适合做全球顶级交易所。

### 管理哲学
- 组织形态：三角形（老板发号施令）→ 花园（每人自我生长）→ 亚马逊雨林（每人都是参天大树）

### 金句
- "命运给你什么就享受什么"
- "人赚到的钱都是认知的钱，人际关系也是认知的人际关系"
- "每个人天生都是一座孤岛，但当你足够强大时你就是一片大陆"

---

## ZZ-478 — Coinbase AI 工程实践：2人几周完成4+人几个月的工作

**来源：** [Coinbase Engineering Blog](https://www.coinbase.com/en-ca/blog/ai-across-the-stack-lessons-from-building-invoicing)

Coinbase 构建发票（Invoicing）产品时的 AI 实战经验，核心结论：**2名工程师几周内完成了原本需要4+人几个月才能完成的工作**，AI 写了约 **40% 的代码**（全部经过人工 review）。

### 四层实践框架

1. **原型（Prototype）— 跳过 Figma**
   - 直接用 AI 生成可交互原型，产品讨论即时可视化，不需要先出 mockup。

2. **跨栈上下文（Cross-stack Context）— Scoped Sandbox**
   - 创建 Scoped Sandbox，让 AI 同时理解前端/后端/合约代码，减少切换上下文的摩擦。

3. **质量补全（Quality Completion）— AI 批量生成测试**
   - AI 批量生成测试用例，测试覆盖率提升 **+60%**。

4. **运营（Operations）— AI 生成 PR 描述**
   - AI 自动生成 PR description，减少工程师写文档的时间开销。

### 关键原则
- AI 生成的代码 **100% 经过人工 review**，不直接合并。
- 把 AI 当结对编程伙伴，而非自动化替代品。

---

## ZZ-39 — Ray Dalio《Principles》视频迷你系列

**来源：** [Twitter/X @RayDalio](https://x.com/RayDalio/status/1990808713234690572)

Ray Dalio 将畅销书《Principles》（2017年，销量数百万册）制作成视频迷你系列，面向春季毕业生进入"真实世界"的过渡期。

- **Episode 1：The Call to Adventure**——采用英雄旅程（Hero's Journey）框架，把个人成长比作一段需要克服挑战的冒险。
- 内容聚焦**人生原则**（life principles），而非投资或工作原则：
  - Radical Truth（极度求真）
  - Radical Transparency（极度透明）
  - 从错误中学习
- **定位：** 给刚毕业、刚进入社会的年轻人的实用人生哲学浓缩版。

---

## ZZ-291 — Wang Heng 的视频

**来源：** [YouTube](https://www.youtube.com/watch?v=6tdDP-EwCy8)

待观看的视频内容（从 Notion 迁移，Next Action 状态）。归档为 Reference，需要时观看。

---

*归档时间：2026-03-08 | 来源：GTD Weekly Triage 2026-W10*
