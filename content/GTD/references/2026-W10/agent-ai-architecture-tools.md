---
title: "Agent / AI 架构与工具 — 2026-W10 参考汇总"
date: 2026-03-08
tags:
  - agent
  - ai-architecture
  - llm
  - multi-agent
  - tooling
  - reference
---

> 本文汇总 2026-W10 期间收录的 18 条 Agent / AI 架构与工具相关参考资料（ZZ-118, ZZ-119, ZZ-250, ZZ-251, ZZ-253, ZZ-268, ZZ-376, ZZ-385, ZZ-392, ZZ-395, ZZ-406, ZZ-441, ZZ-445, ZZ-466, ZZ-848, ZZ-850, ZZ-857, ZZ-860）。主题覆盖：Agent 大规模部署愿景、多 Agent 架构科学、Harness Engineering、Sandbox 选型、Memory、信任与身份、可观测性、以及具体工具/项目调研。

---

## 宏观愿景

### ZZ-860 — Aaron Levie: Building for Trillions of Agents

**来源:** [x.com/levie](https://x.com/levie/status/2030714592238956960) | Box CEO, 2.5M followers

Box CEO Aaron Levie 的长文，论点是 agent 即将成为所有软件的**主要用户**，而非人类。核心观点：

- **API-first 或死**：没有 API 的功能对 agent 等于不存在；没有 CLI / MCP server 就是竞争劣势。YC Jared Friedman 说"连注册账号都应该有 API，否则你对 agent 基本上死了"。
- **商业模式变革**：座位制 → 消费/用量制。每个企业的 agent 数量将是员工数的 100-1000x，最终达到 trillions of agents。
- **新基础设施层**：Agent 计算（E2B、Modal、Cloudflare）、Agent 文件系统（Box 自己在做）、Agent 身份认证、Agent 通信（Agentmail）、Agent 搜索（Exa、Parallel）、Agent 治理（安全/合规/审计）。
- **OpenClaw 被点名**作为"agentic 前沿代表，跑得最远的，24/7 持久环境"。

Paul Graham 的格言 "Make something people want" 被改写为 "Make something agents want"。

---

### ZZ-251 — Theo: The Agentic Code Problem

**来源:** [x.com/theo](https://x.com/theo/status/2018091358251372601)

Theo 描述多 agent workflow 并行时的工具混乱现象——多个 Claude Code 同时跑，terminal tabs、localhost 端口、auth redirects 一团乱。核心论断：**"This is not your fault"**，工具本来就不是为当前的工作方式设计的。

旧模式：一次做一件事，3 个 app（terminal / IDE / browser）。新现实：多个 agent workflow 并行，注意力不断被打断，需要新一代开发工具来支持 agentic workflow。

---

## Multi-Agent 系统的科学

### ZZ-850 — Towards a Science of Scaling Agent Systems

**来源:** [arxiv.org/abs/2512.08296](https://arxiv.org/abs/2512.08296) | UW, MIT, CMU, Microsoft

180 种配置的受控实验，揭示多 Agent 协作的边界条件：

| 场景 | 效果 |
|------|------|
| 可并行任务（集中式协调） | +80.8% |
| Web 导航（去中心化） | +9.2% |
| 顺序推理任务（所有多 Agent 变体） | -39% 到 -70% |
| 错误放大（独立 Agent） | 17.2x |
| 错误放大（集中式协调） | 4.4x |

关键结论：
1. **工具-协调权衡**：固定计算预算下，工具密集型任务受多 Agent 协调开销影响最大。
2. **能力饱和**：单 Agent 基线准确率超过 ~45% 后，多 Agent 协调的收益递减甚至为负。
3. **预测模型**：基于协调指标的预测模型，R²=0.524，在 87% 的未见配置上能正确判断最优架构。

**核心 takeaway**：先分析任务的并行度和顺序依赖性，再决定架构。不要默认多 Agent = 更好。

---

### ZZ-848 — Harness Engineering Is Cybernetics

**来源:** [x.com/odysseus0z](https://x.com/odysseus0z/status/2030416758138634583) | 240 likes, 356 bookmarks

将控制论（Cybernetics）与 Harness Engineering 类比的长文。三次相同的反馈闭环模式：

1. **James Watt 离心调速器（1780s）**：工人从"转阀门"变成"设计调速器"。
2. **Kubernetes 控制器**：工程师从"重启服务"变成"写 spec"，控制器处理偏差调和。
3. **OpenAI Harness Engineering（现在）**：工程师从"写代码"变成"设计环境+构建反馈循环+编码架构约束"，agent 写代码。五个月写了一百万行代码，零行手写。

**生成-验证不对称性**（P vs NP 的直觉，Cobbe et al. 实证）：生成正确方案比验证方案更难。因此：**你不需要比机器写得更好，你需要比机器评估得更好。**

过去跳过文档、自动化测试、编码架构决策的代价缓慢扩散；现在代价变得不可承受——agent 会以机器速度在每个 PR 上重复你的坏习惯。OpenAI 每周五花 20% 时间清理"AI slop"——直到把标准编码进 harness 本身。

---

### ZZ-392 — 整理现有 Agent Memory 架构方案对比（TODO）

**状态:** 待完成的对比分析任务

待对比方案：OneContext+GCC、Mastra Observational Memory、Entire HQ Checkpoints、OpenAI Skills+Shell+Compaction。

对比维度：记忆粒度、持久化方式、检索策略、prompt cache 兼容性、工程复杂度。

---

## LLM 内部机制研究

### ZZ-385 — On the Biology of a Large Language Model

**来源:** [transformer-circuits.pub](https://transformer-circuits.pub/2025/attribution-graphs/biology.html) | Anthropic

Anthropic Transformer Circuits 团队用 attribution graphs 逆向工程 Claude 3.5 Haiku 的内部机制，类比生物学方法研究 LLM 的"细胞"和"接线图"。

关键发现：
- **多步推理可视化**：内部存在"Texas"表征（Dallas→Texas→Austin 的推导链）
- **诗歌规划**：模型在写每行前预先选择押韵词
- **多语言电路**：语言无关电路在强模型中更显著
- **医疗诊断**：模型"在脑中"生成候选诊断并据此追问
- **幻觉机制**：可以追踪到具体的内部链路

Attribution graphs 方法让 LLM 的推理过程首次在一定程度上可解释。

---

## Agent 基础设施与工具

### ZZ-376 — OpenAI 新基础原语：Shell, Skills, Compaction

**来源:** [x.com/OpenAIDevs](https://x.com/OpenAIDevs/status/2021725246244671606)

OpenAI 推出新 primitives：
- **Shell**：agent 可直接操作 shell 环境
- **Skills**：可复用的 agent 技能单元
- **Compaction**：服务端上下文压缩，解决长对话 context 窗口问题

与 ZZ-392 的 Agent Memory 对比任务直接相关。

---

### ZZ-395 — Bifrost AI Gateway 调研

**来源:** [github.com/maximiliantech/bifrost](https://github.com/maximiliantech/bifrost)

Go 高性能 AI gateway：
- 支持 15+ providers，overhead < 100µs
- 功能：failover / load balancing / semantic caching / guardrails
- **Telemetry**：只有 Prometheus metrics，无内置 tracing（这是缺口）

衍生任务 ZZ-451（添加 tracing 支持）。

---

### ZZ-445 — Agent Sandbox 选型对比

**来源:** [gaocegege.com/Blog/genai/unikernel-agent](https://gaocegege.com/Blog/genai/unikernel-agent)

对比五种 agent sandbox 方案：

| 方案 | 技术 | 特点 |
|------|------|------|
| e2b | Firecracker microVM | 成熟，启动快 |
| k7 | Kata Containers + K8s | 企业级，OCI 兼容 |
| Monty | WASM | 轻量，生态受限 |
| Unikernel | 单内核 | 极度轻量，工具链不成熟 |
| Modal | FUSE | 易用，成本可控 |

ZZ 的反驳观点：sandbox 需要**可靠性保证**，延迟不是真瓶颈（LLM 推理才是），K8s + Kata（k7）可能更务实。

---

### ZZ-406 — 多 AI Agent 协作系统：信任模型、权限控制、通信协议

**来源:** [x.com/ManningBooks](https://x.com/ManningBooks/status/2021992886275809734) | Val Andrei Fajardo 著

多 agent 系统设计的安全/信任层方案：
- **身份认证**：SPIFFE / SPIRE
- **授权**：OPA / Rego
- **通信**：统一 API 网关
- **审计**：immutable audit trail

ZZ 的思考偏安全/信任层，与书的实操入门互补。

---

### ZZ-466 — Gilfoyle：Axiom 的开源 SRE Agent

**来源:** [github.com/axiomhq/gilfoyle](https://github.com/axiomhq/gilfoyle)

Axiom 开源的 SRE Agent，重点学习方向：
- Skill / Prompt 代码设计
- SRE triage SOP 的具体写法
- 如何把运维知识编码进 agent 的 harness

---

### ZZ-857 — Convex Chef：AI App Builder with Real Backend

**来源:** [github.com/get-convex/chef](https://github.com/get-convex/chef) | [chef.convex.dev](https://chef.convex.dev)

基于 bolt.diy fork 的 AI 全栈应用生成器，核心差异化：内置 Convex 响应式数据库，生成带**真实后端**的完整 Web 应用（而非纯前端 demo）。

- **System Prompt 公开**：在 GitHub Releases 页面发布，可学习 AI codegen prompt 工程
- **架构**：`chef-agent/`（agentic loop）+ `convex/`（数据存储）+ `template/`（项目模板）
- **测试框架**：`test-kitchen/` 提供 agent loop 测试工具
- 体现"让数据库 API 对 LLM 友好"的设计哲学

---

## Agent 网络与协议

### ZZ-268 — OpenAgents：AI Agent Networks for Open Collaboration

**来源:** [github.com/openagents-org/openagents](https://github.com/openagents-org/openagents)

开源 agent 网络基础设施：
- **协议支持**：WebSocket, gRPC, HTTP, libp2p, A2A, MCP
- **核心概念**：Agent Networks — 自包含社区，agent 可发现同伴、协作、学习
- 一条命令启动，protocol-agnostic
- 网站：openagents.org

---

### ZZ-250 — ERC-8004：Trustless Agents（区块链上的 Agent 发现与信任协议）

**来源:** [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004)

Ethereum 提案，用区块链实现跨组织边界的 agent 发现、选择和交互：
- **三层注册表**：Identity Registry + Reputation Registry + Validation Registry
- **信任分层**：低风险（声誉）→ 高风险（stake 验证 / zkML / TEE）
- **填补 MCP/A2A 空白**：MCP/A2A 不覆盖 agent 发现和信任
- **作者**：Marco De Rossi（MetaMask）, Davide Crapis（Ethereum Foundation）, Jordan Ellis（Google）, Erik Reppel（Coinbase）
- **状态**：DRAFT，2025-08-13 创建

---

## 工具调研

### ZZ-441 — nearai/ironclaw：NEAR 创始人的 Rust AI Assistant

**来源:** [github.com/nearai/ironclaw](https://github.com/nearai/ironclaw)

NEAR 创始人搞的 Rust AI assistant，灵感来自 OpenClaw。调研重点：Rust 实现的 AI assistant 架构，与 OpenClaw 的异同。注意区分本地 ironclaw CRM 项目（同名不同物）。

---

### ZZ-253 — LobeHub Memory 功能调研

**来源:** [lobehub.com/docs/usage/getting-started/memory](https://lobehub.com/docs/usage/getting-started/memory)

LobeHub 的 AI 记忆功能文档和相关 GitHub issue。调研方向：记忆实现方式（存储、检索、更新策略）、与 ZZ-392 的 Agent Memory 对比任务关联。

---

### ZZ-119 — llm-d：Kubernetes-native LLM 推理调度

**来源:**
- [github.com/llm-d/llm-d](https://github.com/llm-d/llm-d?tab=readme-ov-file#-architecture)
- [gateway-api-inference-extension](https://github.com/kubernetes-sigs/gateway-api-inference-extension)

基于 Envoy 的 Kubernetes-native LLM 推理基础设施项目。kubernetes-sigs/gateway-api-inference-extension 是相关的 K8s Gateway API 推理扩展。两个项目都值得看架构设计。

---

### ZZ-118 — Agent + Workflow（Argo Workflow）

**来源:** [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ePbcw68mSl7lmHEjL54liQ)

Agent 与 Argo Workflow 结合的调研。待看相关 talks，研究如何用 workflow 引擎编排 agent 任务。

---

## 主题交叉索引

| 主题 | 相关条目 |
|------|---------|
| Harness / 反馈闭环 | ZZ-848, ZZ-850, ZZ-251 |
| Agent 大规模部署 | ZZ-860, ZZ-850, ZZ-445 |
| Agent Memory | ZZ-376, ZZ-392, ZZ-253 |
| Agent 身份/信任 | ZZ-250, ZZ-406 |
| Agent 网络/协议 | ZZ-268, ZZ-119, ZZ-118 |
| LLM 内部机制 | ZZ-385 |
| 可观测性/网关 | ZZ-395, ZZ-466 |
| AI Codegen 工具 | ZZ-857, ZZ-441 |

## Takeaway

- https://github.com/maximhq/bifrost
- SPIFFE / SPIRE + ACL, Tape Audit
- https://github.com/get-convex/chef