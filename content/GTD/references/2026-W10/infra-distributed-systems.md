---
title: "基础设施 & 分布式系统参考"
date: 2026-03-08
week: "2026-W10"
tags: ["infrastructure", "distributed-systems", "kubernetes", "durable-execution", "AI-inference", "open-source"]
---

# 基础设施 & 分布式系统参考（2026-W10）

本文汇总本周归档的 8 个基础设施与分布式系统相关条目，涵盖分布式推理、Durable Execution、向量数据库、Prefill-Decode 分离、区块链框架及系统可靠性思维等话题。

---

## 1. Building Resilient Distributed Systems（ZZ-69）

构建弹性分布式系统的基础读物。分布式系统的核心挑战在于：网络分区、节点失效、时钟不同步。弹性设计需要在 CAP 约束下做取舍，围绕 retry、circuit breaker、idempotency、backpressure 等模式构建容错边界。

> 关键词：fault tolerance、CAP theorem、circuit breaker、idempotency

---

## 2. Durable Execution / Durable Computing（ZZ-33）

Durable Execution 是让程序在崩溃后能从断点恢复的执行范式——把函数调用栈持久化，crash 后重放历史重建状态。

**代表性框架：**
- [Hatchet](https://hatchet.run/) — 任务队列 + workflow 引擎，支持 durable execution
- [DBOS](https://dbos.dev/) — 基于数据库的 durable computing 平台
- [Trigger.dev](https://trigger.dev/) — developer-friendly 后台 job 平台
- [Golem Cloud](https://www.golem.cloud/post/the-emerging-landscape-of-durable-computing) — WASM-based durable computing，提出"The Emerging Landscape of Durable Computing"

**与 AI Agent 的关联：** Agent 天然具备异步性质，长任务中间可能 crash，durable execution 能让 agent workflow 从失败处继续而不是重头来过。Pydantic AI 已经集成了多家 durable execution 解决方案。

> 关键词：durable execution、workflow persistence、agent reliability、crash recovery

---

## 3. 分布式推理框架对比 — LWS vs RBG vs Expert Parallelism（ZZ-379）

将大模型推理任务分布到多节点的编排框架，核心解决单机 GPU 不足的问题。

**框架对比：**

| 框架 | 全称 | 来源 | 特点 |
|------|------|------|------|
| **LWS** | LeaderWorkerSet | Kubernetes SIGs | K8s operator，封装 leader-worker StatefulSet，入门友好 |
| **RBG** | RoleBasedGroup | SGLang 项目 | 在 LWS 之上再包一层，支持多角色协作和内置服务发现，上手难度更高 |

**EP（Expert Parallelism）：** MoE 模型中不同 expert 分布到不同 GPU 并行计算，是大规模 MoE 推理的关键并行策略。

**个人判断：** 没有统一标准框架，尤其 diffusion 类模型。从编排视角，LWS 适合入门理解基础分布式推理；RBG 更强大但更复杂。

- GitHub LWS: https://github.com/kubernetes-sigs/lws
- GitHub RBG: https://github.com/sgl-project/rbg

> 关键词：distributed inference、LWS、RBG、Expert Parallelism、MoE、Kubernetes

---

## 4. PD 分离（Prefill-Decode Disaggregation）（ZZ-67）

LLM 推理的两个阶段——Prefill（处理 prompt）和 Decode（逐 token 生成）——在计算特性上截然不同：
- **Prefill**：compute-bound，高并行度，适合大批量
- **Decode**：memory-bandwidth-bound，低并行度，需要低延迟

**PD 分离**将两者部署在不同硬件上，分别优化，显著提升吞吐和降低延迟。

视频来源：[BiliBili BV1UJS8BREGo](https://bilibili.com/video/BV1UJS8BREGo)（科普视频，质量不错但流量意外地少）

> 关键词：prefill-decode disaggregation、LLM inference、memory bandwidth、KV cache

---

## 5. LanceDB on Kubernetes（KubeCon Talk）（ZZ-86）

LanceDB 在 KubeCon 上分享了在 Kubernetes 上运行向量数据库的实践。LanceDB 是基于 Lance 列式格式构建的嵌入式/serverless 向量数据库，原生支持多模态数据。

来源：[@lancedb Twitter](https://x.com/lancedb/status/2001033893106082289)

> 关键词：LanceDB、vector database、Kubernetes、KubeCon、embedding

---

## 6. 为什么基础设施开源重要（ZZ-74）

慧姐（微信公众号）的文章，探讨基础设施开源的战略意义：

- 开源基础设施降低行业整体成本，加速创新扩散
- 商业公司通过开源建立技术话语权和生态影响力
- 开源基础设施是中立性的保证——没有单一供应商锁定
- 社区贡献提升软件质量和安全性

原文：[微信公众号文章](https://mp.weixin.qq.com/s/hlHVci-0K1-IZ86aRigJdA)

> 关键词：open source infrastructure、vendor lock-in、ecosystem、community

---

## 7. Substrate Chain — Polkadot 生态开发框架（ZZ-381）

Substrate 是 Polkadot 生态的核心区块链开发框架：

- **模块化**：通过 FRAME pallet 系统组合功能模块（共识、治理、资产等）
- **可升级**：链上 runtime 可无 hard fork 升级，逻辑存储在链上
- **互操作性**：基于 Substrate 的链可作为 parachain 接入 Polkadot 中继链

**对比 commenware.xyz：** 两者都在做 Web3 基础设施，Substrate 更通用，commenware 更专注于特定场景。

参考：[Substrate 介绍文章](https://medium.com/@brunopgalvao/substrate-cfeb13333f2c)

> 关键词：Substrate、Polkadot、parachain、FRAME、blockchain framework、runtime upgrade

---

## 8. 为什么我不喜欢 COE（Correction of Errors）（ZZ-220）

来自 [Surfing Complexity 博客](https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/) 的系统思维文章，批判传统事后复盘（COE/Post-mortem）的局限性。

**核心论点：**
- 将故障简单归因于"错误"的思维是危险的简化
- **复杂系统中，缺陷永远存在**——没有缺陷的系统是理想，不是现实
- 传统 COE 聚焦于"哪里出错了"，导致堆叠约束和过度流程化
- 更应该问的问题：**"正常工作是如何发生的？"**（Safety-II 思维）
- 系统弹性来自人与系统之间的适应性交互，而非消除所有错误

**启示：** 与其追求零缺陷，不如构建能快速检测、快速恢复的系统（Resilience Engineering）。

> 关键词：COE、post-mortem、Safety-II、complex systems、resilience engineering、correction of errors

---

## 主题小结

本周基础设施条目集中在两个方向：

1. **AI 推理基础设施**：分布式推理编排（LWS/RBG/EP）、PD 分离、向量数据库上 K8s——体现了大模型推理从单机走向集群的基础设施演进趋势。

2. **系统可靠性思维**：Durable Execution（crash 恢复）、COE 批判（Safety-II 复杂系统视角）——体现了从"防止故障"到"设计韧性"的思维转变。
