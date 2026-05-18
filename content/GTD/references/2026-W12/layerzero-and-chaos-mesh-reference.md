---
title: "LayerZero 生态 + Chaos Mesh 贡献（2026-W12）"
date: 2026-03-17
tags: ["layerzero", "chaos-engineering", "chaos-mesh", "reference"]
---

## 概述

本周 G8 组汇集了 LayerZero 生态的多维观察（Zero L1 技术架构、Q4 数据报告、机构伙伴阵容、AI 时代的工作方式转型）以及 ZZ 在 Chaos Mesh 项目中推进的三个长期 open PR 的当前状态。

---

## LayerZero 生态

### ZZ-874 — Zero Will Succeed — 社区视角的 Zero 全面梳理（技术+合作伙伴+展望）

来源：<https://x.com/0xPandaiman/status/2030955320928063965>
Linear：<https://linear.app/zzgtd/issue/ZZ-874>

社区成员 @0xPandaiman 对 LayerZero 即将推出的 L1 区块链 Zero 的全面科普，涵盖技术架构（单 Zone 2M TPS、四大突破 QMDB/FAFO/Jolt Pro/SVID）、机构合作伙伴阵容（Citadel Securities、DTCC、Tether、ARK Invest、GTE、Google Cloud），以及 Day 1 即启用的全链互操作愿景。文章立场偏多/看好，数据多引用第三方新闻，技术细节不深，属于生态科普向内容。Zero 主网计划 2026 秋季上线，EVM 兼容，$ZRO 为原生代币。

---

### ZZ-905 — LayerZero Newsletter: Zero launch, partners, and advisors

来源：<https://x.com/LayerZero_Core/status/2031771478551822569>（X Article: <https://x.com/i/article/2031407532846227456>）
Linear：<https://linear.app/zzgtd/issue/ZZ-905>

LayerZero 官方 Jan/Feb 2026 Newsletter，本质是一份面向机构投资人的信誉包装：重点是 Zero 的定位（"去中心化多核世界计算机"，2.5 年秘密研发）、战略合作伙伴（Citadel Securities、DTCC、ICE Markets、Google Cloud、GTE）、新顾问（Cathie Wood、Michael Blaugrund、Caroline Butler），以及生态里程碑（Tether 战略投资、Fidelity FCAT 部署 DVN、Wyoming FRNT 稳定币上线、ZRO 在 Robinhood 上市、Cardano 节点即将接入）。核心 Zero 架构主张仍停留在断言层面，尚无独立验证。

---

### ZZ-942 — LayerZero Q4 2025 Report — Token Terminal

来源：<https://x.com/tokenterminal/status/2032463976617984391>
Linear：<https://linear.app/zzgtd/issue/ZZ-942>

Token Terminal 的 LayerZero Q4 2025 数据报告：季度转账量 $52.31b（历史新高，YoY +775%），但费用 $1.04m（YoY -53%）——量价背离是最大看点，折射出从零售 DeFi 向机构用户（PayPal、Ethena、Ondo、Wyoming 州）的结构性转型。MAU 198k（YoY -39%），主要是 2024 年空投流量退潮；有机用户连续两季平稳。费用开关治理投票 12 月失败（参与率 3.71% vs 门槛 40.59%），下次投票 mid-2026。Stargate 仅占 0.80% 交易量却贡献 44.40% 费用，效率最高。Zero 区块链 Feb 10 宣布，同日 Tether/Citadel/ARK Invest 宣布投资。

---

### ZZ-945 — From Doer to Conductor: Orchestrate Intelligence or Fall Behind（LayerZero 内部 all-hands 反思）

来源：<https://x.com/0x_Arjun/status/2032498616212877664>
Linear：<https://linear.app/zzgtd/issue/ZZ-945>

LayerZero Head of Crypto Arjun Arora 受公司 all-hands 启发写的思维模型文章：AI 已经把执行成本打到接近零，"强执行力"不再是护城河，真正稀缺的变成了**判断力**与**编排能力**。核心隐喻：从乐器演奏者升级为指挥家——不必亲自弹每个音符，但要能听出何时出了问题。关键警告：基础专业能力仍不可缺，否则无法评估 AI 输出质量。最后一句值得记："机器不会替代人类，但懂得编排机器的人会替代那些与机器竞争的人。"

---

### ZZ-983 — KCD Beijing 2026 × vLLM — 云原生 + AI 基础设施社区活动（3月21日，北京）

来源：<https://www.bagevent.com/event/kcd-beijing-2026>
Linear：<https://linear.app/zzgtd/issue/ZZ-983>

CNCF KCD Beijing 与 vLLM 社区联合主办的全天社区活动，2026 年 3 月 21 日（周六）09:00–18:00，北京格兰云天大酒店。CFP 已关闭，预期议题涵盖 Kubernetes、云原生基础设施、AI/LLM serving、vLLM 生态。是 Kubernetes + AI infra 交叉方向的重要线下聚会，对 ZZ 的技术方向高度相关。

---

## Chaos Mesh 贡献

### ZZ-984 — chaos-mesh PR #4427：OIDC 认证支持（2024-05 开，至今未合）

来源：<https://github.com/chaos-mesh/chaos-mesh/pull/4427>
Linear：<https://linear.app/zzgtd/issue/ZZ-984>

为 Chaos Mesh Dashboard 添加 OIDC 认证支持（前后端 + Helm），ZZ 于 2024-09 接手推进。当前最大问题是 Copilot 审查发现了 **7 个 bug**，包括 3 个会导致生产环境 panic 的 Critical 级别 bug（middleware 判断逻辑反转、defer 在 nil 检查前、nil provider 不中断执行）。已开 22 个月，社区持续有用户催促，ZZ 3 月回复"请再等等"。核心 blocker：修复 bot 指出的 bug → 补 CHANGELOG → 等 g1eny0ung 的 lgtm。

---

### ZZ-985 — chaos-mesh PR #4630：升级 gorm v2（g1eny0ung，2025-02 开，至今未合）

来源：<https://github.com/chaos-mesh/chaos-mesh/pull/4630>
Linear：<https://linear.app/zzgtd/issue/ZZ-985>

g1eny0ung（chaos-mesh 核心维护者）主导的 gorm v1 → v2 迁移，涉及 33 个文件、+621/-303 行，属于技术债清理。核心风险是 **sqlite 默认文件名从 `core.sqlite` 改为 `chaos-dashboard.sqlite`**，升级后旧数据会"消失"——这是必须在合并前解决的 Breaking Change。此外有若干 P2 数据完整性和编译风险问题。CI 处于 unstable 状态，目前仅有作者自批准，需要 maintainer peer review。g1eny0ung 2026-03-09 刚 mark ready for review，方向正确但需 address 后才宜合并。

---

### ZZ-986 — chaos-mesh PR #4812：logging configuration（ZZ 自己的 PR，2025-12 开，未合）

来源：<https://github.com/chaos-mesh/chaos-mesh/pull/4812>
Linear：<https://linear.app/zzgtd/issue/ZZ-986>

ZZ 于 2025-12-26 开的 PR，为 Chaos Mesh 添加完整的结构化日志配置能力：通过环境变量（`LOG_FORMAT`、`LOG_LEVEL`、`LOG_TIMESTAMP_FORMAT` 等）和 Helm values 控制日志格式（console/JSON）、级别、时间戳格式、字段 key 名和最大字段长度。CI 58 项通过，patch coverage 95%，merge state clean。主要 blocker：**55 天无人工 review 响应**，bot 指出的 bug（JSON 模式下 `LOG_LEVEL` 未生效、`io.Writer.Write()` 返回 0）值得在 re-request review 前修复。
