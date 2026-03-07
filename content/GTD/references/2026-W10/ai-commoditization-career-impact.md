---
title: "AI 商品化与职业未来：当 coding 变成廉价资源"
date: 2026-03-06
tags: [ai, career, commoditization, software-engineering]
---

# AI 商品化与职业未来

## 核心问题

AI 正在商品化 coding（代码编写能力）。当一种曾经需要多年训练才能获得的能力突然变得廉价且可大规模获取时，会发生什么？

## 历史类比（ZZ-781 jiayuan 万字长文）

三个跨越数百年的商品化周期遵循相同规律：

| 被商品化的层 | 价值迁移到的层 |
|---|---|
| 抄写（印刷术 1440） | 内容创作 & 出版 |
| 工厂动力（电力 1881） | 生产流程设计（福特流水线） |
| 服务器基础设施（AWS 2006） | 应用层体验 & 网络效应 |
| **代码编写（AI 2024+）** | **问题定义、产品判断、用户获取** |

### 电力的 40 年教训

Paul David 发现电力从商业化到产生可衡量的生产率提升，中间有 40 年时滞。早期工厂只是"把蒸汽机换成电动机"，布局/流程/组织不变。真正的爆发发生在围绕电力特性**重新设计整个工厂**时（福特流水线）。

METR 2025 年实验发现：有经验的开发者用 AI 工具反而**慢了 19%**——正在重演这个悖论。

### Perez 框架：我们在哪？

- **安装期**（当前）：大量资本涌入、实验、过度投资
- 最大价值创造通常在**部署期**而非安装期

## 不同视角的验证

### Sequoia: 卖工作不卖工具（ZZ-766）

Julien Bek 的框架：
- **Intelligence**（规则型工作）→ AI 能替代 → 卖工具在和模型赛跑
- **Judgement**（经验型判断）→ AI 还不能 → 卖结果则模型越强你越强
- 每花 $1 买软件就花 $6 买服务。下一个万亿公司直接卖结果

### Paul Graham: Brand Age（ZZ-763）

> "Branding is centrifugal; design is centripetal."

当技术消灭产品间的实质差异后，品牌就是剩下的东西。Omega 面对石英危机选择"做更精确的机芯"→ 破产。Patek 转向品牌 → 活了。

### Anthropic 实际数据（ZZ-776）

- AI 远未达到理论上限：Computer & Math 理论 94% 可加速，实际只覆盖 33%
- 最受影响人群：更老、更女性、更高学历、更高薪（反直觉）
- 暂无系统性失业，但高暴露职业的**年轻人招聘放缓**（leading indicator）

### 设计师市场实例（ZZ-778）

执行层血洗（production work 被 AI/外包替代），战略层极度稀缺（能端到端拥有结果的 hybrid designer）。

> "AI 让每个工程师变成设计师，就像微波炉让每个人变成厨师。" — Mike Rundle

### 专业分工消亡（ZZ-768）

福特流水线的分工是因为跨岗位阻力真实存在（培训焊工去喷漆要几个月）。AI 消灭了这些技能鸿沟。

> "Pick a lane" needs to be demolished. Ownership vs Dependency 才是唯一重要的问题。

### 工程师依然有杠杆（ZZ-664, ZZ-713, ZZ-651）

- Naval：传统软件工程没死，工程师仍是最有杠杆的人
- AI 让写代码更容易，但让工程（系统设计、架构判断、调试整合）更难
- IC 路线杠杆变大，EM 路线需要重新评估

## 我的初步观点（待 ZZ 补充）

1. **"用 AI 更快做旧事"是陷阱** — 相当于把蒸汽机换成电动机，短期有套利窗口但会迅速关闭
2. **系统架构能力和判断力正在升值** — 这是 AI 还不能替代的部分
3. **护城河要建在代码之外** — 数据、用户关系、分发渠道、品牌
4. **对 SRE/基础设施工程师的影响** — TBD，待思考

## TODO

- [ ] ZZ 写自己的 takeaway 和观点
- [ ] 结合个人职业规划思考

## 参考链接

- [ZZ-781] jiayuan 万字长文：当我们站在变革的开端 — https://x.com/jiayuan_jy/status/2029851505583607961
- [ZZ-766] Sequoia: Services 是新的 Software — https://x.com/JulienBek/status/2029680516568600933
- [ZZ-763] Paul Graham: The Brand Age — https://paulgraham.com/brandage.html
- [ZZ-776] Anthropic: Labor Market Impacts — https://www.anthropic.com/research/labor-market-impacts
- [ZZ-778] Designers! Designers! Designers! — https://carly.substack.com/p/designers-designers-designers
- [ZZ-768] Become a Tinkerer — https://x.com/nikunj/status/2029716176193081514
- [ZZ-713] Don't Become an Engineering Manager — https://newsletter.manager.dev/p/dont-become-an-engineering-manager
- [ZZ-664] Naval: Software Engineering Not Dead — https://x.com/naval/status/2028314493206585471
- [ZZ-651] AI Made Writing Code Easier, Engineering Harder — https://www.ivanturkovic.com/2026/02/25/ai-made-writing-code-easier-engineering-harder/

## 个人思考：SRE 视角下的价值定位

### 不要和 AWS/GCP 竞争，站在它们上面

AWS/GCP 卖的是基础设施资源，已经把这一层商品化了。要卖的"结果"是：客户的服务稳定运行，他们不需要操心怎么做到的。

- 客户真正想要的：业务不宕机，用户体验好 → **卖的结果在这里**
- 怎么做到的：架构设计、SLO 定义、故障自愈、容量规划、变更管理 → **judgement 在这里**
- 用什么做的：AWS/GCP/K8s/Terraform/监控工具 → 这些是原材料，不是产品

### AI 难以替代的 20%

AI agent 很快能做到常规运维的 80%。真正难替代的是：

1. 判断什么架构适合这个业务阶段（初创期过度设计是浪费，增长期欠设计是灾难）
2. 在出事的时候做对决策（回滚还是前滚？降级还是扩容？）
3. 预见还没发生的问题（这个设计在 10x 流量下会怎样？这个依赖挂了会级联到哪里？）

### SLA 框架不会消失，但锚定点会上移

SLA 的本质是对结果的量化承诺，这个需求永远存在。变的是指标锚定在哪里：

- 现在：服务器 uptime 99.99%、API P99 < 200ms → 基础设施指标全绿，但用户体验可能是烂的
- 未来：用户关键流程成功率、从故障到用户无感恢复的时间、业务指标不因技术问题下降 → 直接绑定业务结果

趋势是从分层 SLA（每层自己的指标）走向端到端 SLO（直接衡量用户感知的结果）。

### 核心结论

不变的是：客户永远需要一个可量化的承诺。变的是指标从"服务器没挂"往"用户没受影响"往"业务没受损"不断上移。

**关键不是"SLA 框架还有没有用"，而是你有没有能力定义出客户真正关心的那个指标，并且对它负责。**
