---
title: "AI/ML 论文与深度内容"
date: 2026-03-08
week: 2026-W10
tags: [ai, ml, llm, papers, reference]
linear_ids: [ZZ-703, ZZ-851, ZZ-255, ZZ-216, ZZ-186, ZZ-178, ZZ-117, ZZ-82, ZZ-75, ZZ-78, ZZ-90]
---

# AI/ML 论文与深度内容 — 2026-W10

本周收集的 AI/ML 相关论文、书籍、深度文章合集。涵盖 AGI 经济学、LLM 架构创新、认知卸载研究、工程实践等方向。

---

## ZZ-703 | Some Simple Economics of AGI

**来源:** https://arxiv.org/abs/2602.20946  
**作者:** Christian Catalini（112 页经济学论文）

### 核心框架

AI 让执行成本指数下降，但人类验证带宽受生物学限制——这两条成本曲线的碰撞决定了 AGI 转型的经济学。

- **Cost to Automate** — 指数级下降
- **Cost to Verify** — 受生物学瓶颈约束，难以快速扩展
- **Measurability Gap** — agent 能执行的 vs 人类能验证的差距越来越大

### 五个关键概念

1. **从 skill-biased 到 measurability-biased 技术变革** — 租金迁移到验证级 ground truth、密码学溯源、责任承保
2. **Missing Junior Loop** — 学徒制崩溃，从下方侵蚀 human-in-the-loop 均衡
3. **Codifier's Curse** — 专家将自身知识编码进 AI，加速自身过时
4. **Trojan Horse externality** — 不验证直接部署在私人层面是理性的，但产生外部性
5. **Hollow Economy vs Augmented Economy** — 不管理验证带宽 → 空心经济；扩展验证 → 增强经济

### 结论

今天的挑战不是部署最自主的系统，而是**扩展验证/监督的带宽**。论文提供了个人、企业、投资者、政策制定者的 playbook。

---

## ZZ-851 | The Hundred-Page Language Models Book

**来源:** https://thelmbook.com  
**GitHub:** https://github.com/aburkov/theLMbook  
**作者:** Andriy Burkov（《The Hundred-Page Machine Learning Book》作者）

### 结构亮点

- **渐进式架构**：count-based → RNN → Transformer → LLM 的自然演进路线，不直接丢 Transformer
- **数学 + 直觉双轨**：每个概念都有清晰的数学推导和直觉说明
- **完整 PyTorch 实现**：GitHub 仓库提供每个主题的可运行代码 + Jupyter Notebooks
- **LLM 实战章节最大**：覆盖 prompt engineering 技巧和 instruction finetuning
- 附赠 $150 Lambda GPU credits

### 适合人群

技术领导、工程经理、软件开发者、数据科学家、ML 工程师。Burkov 延续了上一本书「短小精悍、代码完整」的风格。

---

## ZZ-255 | AI 与认知卸载研究

**来源:**
- https://www.youtube.com/watch?v=YDYxKPhVx8U
- https://www.youtube.com/watch?v=YAfdoDc7_5o

### 主题

关于 AI 对人类 **cognitive offloading**（认知卸载）影响的研究视频——AI 工具如何改变人类的思考和记忆方式。

与 ZZ-703 的 Measurability Gap 概念呼应：当 AI 代劳认知工作，人类如何维持验证能力？

---

## ZZ-216 | DeepSeek Engram — Conditional Memory 论文

**来源:** https://github.com/deepseek-ai/Engram/blob/main/Engram_paper.pdf  
**GitHub:** https://github.com/deepseek-ai/Engram  
**合作:** DeepSeek + 北京大学，梁文锋署名

### 核心问题

MoE 通过条件计算实现稀疏化，但现有 Transformer **缺少原生知识查找机制**，只能通过计算过程低效模拟检索行为。

### Engram 架构

- **条件记忆（Conditional Memory）** — 与 MoE 的条件计算互补的稀疏化新维度
- **O(1) 查表**：基于哈希 N-gram，确定性检索，不依赖运行时隐藏状态路由
- **系统解耦**：确定性寻址支持从主机内存预取，几乎零额外延迟

### 实验数据

在等参数、等 FLOPs 的严格对照下，Engram-27B vs MoE-27B：
- 知识任务：MMLU +3.0，CMMLU +4.0，MMLU-Pro +1.8
- 推理任务：BBH **+5.0**，ARC-Challenge +3.7，DROP +3.3
- 代码/数学：HumanEval +3.0，MATH +2.4，GSM8K +2.2
- 长文本：Multi-Query NIAH 准确率从 **84.2 → 97.0**

### U 型扩展规律

将约 20%-25% 的稀疏参数预算从 MoE 重新分配给 Engram 可获最优性能（最优点在 ρ ≈ 75%-80% 处稳定）。

### 意义

DeepSeek 认为**条件记忆将成为下一代稀疏大模型的核心建模原语**。结合 mHC（Manifold-Constrained Hyper-Connections），DeepSeek v4 的轮廓愈发清晰。

---

## ZZ-186 | 智谱上市后的信

**来源:** https://www.latepost.com/news/dj_detail?id=3360

LatePost 对智谱 AI 上市后的深度报道。智谱是国内主要的大模型公司之一，背靠清华系。

---

## ZZ-178 | Database 2025 — Andy Pavlo

**来源:** https://x.com/andy_pavlo/status/2008180749900648837  
**作者:** Andy Pavlo（CMU 数据库教授）

Andy Pavlo 的年度数据库回顾系列。2025 年主要话题通常包括：向量数据库崛起、AI-native DB 架构、Postgres 生态爆发、NewSQL 演进。  
参考博客：https://ottertune.com/blog/

---

## ZZ-117 | 到底什么是 Reasoning？

**来源:** https://x.com/dongxi_nlp/status/2003574127211479442

关于 LLM "reasoning" 本质的 Twitter 讨论串。当前 AI 界对 reasoning 的定义众说纷纭，这个 thread 提供了一个有价值的视角。

---

## ZZ-82 | Memory Paper Landscape

**来源:** https://x.com/wey_gu/status/2001239698501722296

Memory 相关论文全景图——与 ZZ-216（DeepSeek Engram）形成互补。目前 LLM memory 研究方向繁多，这个 list 提供了系统性概览。

---

## ZZ-75 | TPU 101

**来源:** https://mp.weixin.qq.com/s/0PqT8cCfFiQlnKfhIVWN6g

TPU 基础科普文章。了解 Google TPU 架构对理解 LLM 训练基础设施有帮助。

---

## ZZ-78 | Databricks Blog — ML Engineering

**来源:** Databricks Blog  
**标题:** Machine Learning Engineering: Complete Guide to Building Production ML Systems (13 min read)

ML Engineering 生产系统完整指南。Databricks 视角的 ML 工程实践，涵盖从模型开发到生产部署的完整流程。

---

## ZZ-90 | ADHD Paper

**来源:** https://x.com/NTFabiano/status/2000991883078574099

ADHD 相关研究论文的 Twitter 分享。

---

## 主题关联

```
验证带宽 (ZZ-703)
    └── 认知卸载 (ZZ-255)
    
LLM 架构创新
    ├── DeepSeek Engram — Conditional Memory (ZZ-216)
    └── Memory Landscape (ZZ-82)
    
系统学习资源
    ├── Hundred-Page LM Book (ZZ-851)
    ├── TPU 101 (ZZ-75)
    └── ML Engineering Guide (ZZ-78)

行业观察
    ├── 智谱上市 (ZZ-186)
    ├── Database 2025 (ZZ-178)
    └── What is Reasoning? (ZZ-117)
```
