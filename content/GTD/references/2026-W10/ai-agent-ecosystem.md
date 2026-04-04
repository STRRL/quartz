---
title: "AI/Agent 生态与动态 — 2026-W10"
date: 2026-03-08
tags: [AI, agent, MLOps, LLM, reference]
---

# AI/Agent 生态与动态 — 2026-W10

本周收集了 7 篇关于 AI/Agent 生态的参考资料，涵盖大模型 scaling 哲学、agent 经济形态、MLOps 工程实践、推理性能优化和 AI-Native 团队建设。

---

## 1. Dwarkesh × Dario Amodei 深度访谈 — 关于 Scaling 与 AGI 的终极拷问

**来源：** [ZZ-390] [@dwarkesh_sp](https://x.com/dwarkesh_sp/status/2022357801276690455)

Dwarkesh Patel 对 Anthropic CEO Dario Amodei 的长篇访谈，是近期最值得反复观看的 AI 对话之一。涵盖议题：

- **"我们到底在 scale 什么？"** — scaling law 的本质是什么，还能继续吗？
- **"Diffusion 是 cope 吗？"** — 对 diffusion model 进军语言领域的直接质疑
- **持续学习是否必要** — post-training 与 continual learning 的取舍
- **如果 AGI 迫在眉睫，为何不买更多算力？** — 计算资源分配的战略逻辑
- **AI 实验室如何盈利** — 商业模式与研究使命的张力
- **监管是否会摧毁进展** — 政策风险的现实评估

> Dario 的回答通常不回避矛盾，访谈格式允许深入追问，是理解 Anthropic 战略立场的一手资料。

---

## 2. YC Lightcone：Agent 经济的爆发 — "Build Something Agents Want"

**来源：** [ZZ-495] [@ycombinator](https://x.com/ycombinator/status/2025285025454064037)

YC Lightcone Podcast 讨论了 OpenClaw 和 MoltBook 的爆发式增长，提出了一个新时代命题：

> **"Make something agents want"** — 把 YC 的经典口号从 people 换成 agents

核心洞察：
- AI dev tools（如 OpenClaw、MoltBook）代表了 agent-driven economy 的第一波基础设施
- 传统 SaaS 的用户是人，而新一代工具的消费者是 agent
- 这波增长不只是工具替代，而是经济主体的转变

这个视角对于理解未来 2-3 年的创业机会非常关键。

---

## 3. OpenClaw 文档站探索

**来源：** [ZZ-485] [docs.openclaw.ai](https://docs.openclaw.ai/)

OpenClaw 官方文档站，可通过 `https://docs.openclaw.ai/llms.txt` 获取所有页面索引（agent-friendly 设计）。

值得关注的是 OpenClaw 文档本身的结构设计体现了 agent-first 理念：提供 `llms.txt` 作为机器可读入口，说明工具设计者已经在考虑 agent 作为主要消费者。

---

## 4. vikhyatk 演讲：Better Performance at Lower Latency

**来源：** [ZZ-455] [@vikhyatk](https://x.com/vikhyatk/status/2011616522238902661)

vikhyatk（可能是 ML 推理领域从业者）的技术演讲录像，主题是**在降低延迟的同时提升性能**。原本以为没有录制，后来找到了录像。

- 演讲内容可能涉及 ML 推理优化、serving 架构或 speculative decoding 等方向
- 这类"performance + latency 同时优化"的话题是当前 LLM 部署的核心挑战

具体技术细节需观看录像确认。

---

## 5. Personalized Recommendation System — 课程资料

**来源：** [ZZ-227] [@tom_doerr](https://x.com/tom_doerr/status/2013478674809397367)

- **课程 repo：** [decodingai-magazine/personalized-recommender-course](https://github.com/decodingai-magazine/personalized-recommender-course)

个性化推荐系统的课程材料，由 decodingai-magazine 出品。推荐系统是 AI 应用中工程复杂度高、业务价值明确的领域。结合 LLM 时代的新范式（如 LLM as ranker、embedding-based retrieval）值得关注。

---

## 6. MLOps in the Enterprise — O'Reilly 书籍

**来源：** [ZZ-386] [@KirkDBorne](https://x.com/KirkDBorne/status/2021775093412622412)

**书名：** *Implementing MLOps in the Enterprise: A Production-First Approach*  
**作者：** Yaron Haviv, Noah Gift  
**出版：** O'Reilly

Production-First 的 MLOps 实践指南，面向企业级场景。涵盖从模型训练到部署、监控的全生命周期。

> 任务备注：需找并下载电子版。

---

## 7. OpenAI AI-Native Engineering Team 指南

**来源：** [ZZ-44] [@TheRealAdamG](https://x.com/TheRealAdamG/status/1992758654509174983)

- **PDF：** [Building an AI-Native Engineering Team](https://cdn.openai.com/business-guides-and-resources/building-an-ai-native-engineering-team.pdf)

OpenAI 出品的企业指南，讲述如何构建 AI-Native 工程团队。AI-Native 不只是"用 AI 工具"，而是在团队文化、流程、招聘、评估体系上全面重构。

对于正在思考如何在工程团队中引入 AI 的人（包括 SRE、Platform 团队）具有参考价值。

---

## 横向观察

这 7 项资料呈现了一个共同主题：**AI 的主体性正在从"工具"升级为"参与者"**。

- Dario 的访谈讨论 AGI 的临近
- YC 提出 agents 作为经济主体
- OpenClaw 的 llms.txt 设计体现 agent-first
- AI-Native Team 指南重构人与 AI 的协作关系

这不是某一个技术点的突破，而是整个行业正在经历的范式迁移。

