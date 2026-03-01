---
title: "SkillsBench: Agent Skills Benchmark 研究参考"
date: 2026-02-24
tags: ["ai", "benchmark", "agent", "skills", "research"]
---

# SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks

## 基本信息

- **论文**: [arXiv:2602.12670](https://arxiv.org/abs/2602.12670)，2026年2月13日发布
- **官网**: https://www.skillsbench.ai/
- **GitHub**: https://github.com/benchflow-ai/skillsbench
- **第一作者/通讯**: Xiangyi Li（李向一）

## 1. 论文内容：实验设计与结果

### 核心问题

Agent Skills（结构化过程性知识包）能否提升 LLM agent 表现？什么时候有用、什么时候没用？

### Skill 的定义

Skill 是一个包含 `SKILL.md`（过程性指导文档）+ 可选资源（脚本、模板、示例代码）的文件系统模块包。与 system prompt、few-shot examples、RAG 检索、tool documentation 不同，Skill 强调**过程性**（how-to）、**类级别适用性**（适用一类任务而非单个实例）、**结构化**和**可移植**。

### 任务规模

- **86 个任务**（最终有效评估 84 个），横跨 **11 个领域**
- 由 **105 位贡献者**从学术界和工业界提交 322 个候选任务中筛选
- 每个任务是一个自包含 Docker 容器，含指令、环境、参考解法、确定性验证器
- 难度分层：Core（17 tasks, <60min）、Extended（42 tasks, 1-4h）、Extreme（26 tasks, >4h）

### 11 个领域

Healthcare、Manufacturing、Science、Energy、Data Analysis、GIS/Earth Science、Software Engineering、Mathematics、Document Processing、Game/Simulation、Networking/Security 等

### 评估的 7 个 Agent-Model 配置

| Agent + Model | 无 Skill | 有 Skill | 提升 |
|---|---|---|---|
| Gemini CLI (Gemini 3 Flash) | 31.3% | 48.7% | +17.4pp |
| Claude Code (Opus 4.5) | 22.0% | 45.3% | +23.3pp |
| Codex (GPT-5.2) | 30.6% | 44.7% | +14.1pp |
| Claude Code (Opus 4.6) | 30.6% | 44.5% | +13.9pp |
| Gemini CLI (Gemini 3 Pro) | 27.6% | 41.2% | +13.6pp |
| Claude Code (Sonnet 4.5) | 17.3% | 31.8% | +14.5pp |
| Claude Code (Haiku 4.5) | 11.0% | 27.7% | +16.7pp |

3 个 Agent Harness：**Claude Code**（Anthropic）、**Gemini CLI**（Google）、**Codex CLI**（OpenAI）

### 三种评估条件

1. **No Skills** — 裸模型基线
2. **Curated Skills** — 人工策划的 Skill 包
3. **Self-generated Skills** — 模型自己先生成 Skill 再解题（仅 5/7 配置支持，Gemini CLI 不支持）

### 核心发现

1. **Curated Skills 平均提升 +16.2pp**，但领域差异极大
   - Healthcare: **+51.9pp**，Manufacturing: **+41.9pp**（巨大提升）
   - Software Engineering: **+4.5pp**，Mathematics: **+6.0pp**（微弱提升）
2. **16/84 个任务出现负面效果**（Skill 反而降低表现）
3. **Self-generated Skills 几乎无效** — 模型不能可靠地生成自己受益的过程性知识
4. **2-3 个 Skill 模块最优**（+20.0pp），4+ 个反而递减（+5.2pp）
5. **简洁 Skill（+18.9pp）远优于详尽文档（+5.7pp）**，差 4 倍
6. **小模型 + Skills 可匹敌大模型**：Haiku 4.5 + Skills（27.7%）> Opus 4.5 无 Skills（22.0%）
7. 总计 **7,308 条 trajectories**，每个任务 5 次试验

### 技术基础

- 构建于 **Harbor framework**（Terminal-Bench 的基础设施）
- 聚合了 47,150 个 Skills：开源仓库（12,847）、Claude Code 生态（28,412）、企业合作伙伴（5,891）
- 确定性 pytest 验证器，无 LLM-as-judge

## 2. 研究团队背景

### 第一作者：Xiangyi Li（李向一）

- **身份**: BenchFlow 创始人兼 CEO
- **Twitter**: [@xdotli](https://x.com/xdotli)，约 3,452 粉丝
- **LinkedIn**: [l1xiangyi](https://www.linkedin.com/in/l1xiangyi/)
- **学历**: San José State University
- **所在地**: San Francisco
- **公司**: **BenchFlow**（不是 SkillBench/FollowBench）
  - Twitter: [@benchflow_ai](https://x.com/benchflow_ai)
  - 官网: https://www.benchflow.ai/
  - 定位: "frontier evals and rl environments"
  - GitHub: https://github.com/benchflow-ai
  - 联合维护者：Hongji Xu (kirk@benchflow.ai)

### 李向一与 SkillsBench 的关系

**李向一就是 SkillsBench 论文的第一作者和通讯作者**（论文 submission history 明确写 "From: Xiangyi Li"）。SkillsBench 是 BenchFlow 公司的核心研究产品。他的个人网站链接就是 skillsbench.ai，Twitter bio 写的是 "frontier evals @benchflow_ai"。

### 论文共 40 位作者

包括 Wenbo Chen, Yimin Liu, Shenghan Zheng, Xiaokun Chen, Yifeng He, Yubo Li, Bingran You, Haotian Shen, Jiankai Sun, Shuyi Wang, Qunhong Zeng, Di Wang, Xuandong Zhao, Yuanli Wang, Roey Ben Chaim, Zonglin Di, Yipeng Gao, Junwei He, Yizhuo He, Liqiang Jing, Luyang Kong, Xin Lan, Jiachen Li, Songlin Li, Yijiang Li, Yueqian Lin, Xinyi Liu, Xuanqing Liu, Haoran Lyu, Ze Ma, Bowei Wang, Runhui Wang, Tianyu Wang, Wengao Ye, Yue Zhang, Hanwen Xing, Yiqi Xue, Steven Dillmann, Han-chung Lee

大量华人作者，可能大部分是 BenchFlow 社区贡献者/实习生。

## 3. 投资与社交关系背景

### 投资人/支持者

李向一 Twitter bio 明确写：**"backed by @JeffDean @fdotinc"**

- **Jeff Dean** — Google 首席科学家，AI 领域传奇人物
- **@fdotinc = Founders Inc** — 一家早期投资机构（f.inc），6.3 万粉丝，创始人是 **Garry Tan**（陳嘉興）

### Garry Tan（陳嘉興）— YC CEO

- Garry Tan 是 **Y Combinator 现任 CEO 兼总裁**（2023 年上任）
- 华裔加拿大/美国人
- 他也是 **Initialized Capital** 联合创始人
- 早期 Palantir 员工
- @fdotinc 是他的投资基金 Founders Inc 的 Twitter 账号

**所以用户说的"YC 的华人高管"很可能指的就是 Garry Tan**，通过 Founders Inc 投资了 BenchFlow/李向一。

### 社交传播

- 2026-02-13: 李向一发布 SkillsBench 公告推文（[原推](https://x.com/xdotli/status/2022409507826213048)），获得 **961 bookmarks**，传播量大
- 2026-02-20: @forloopcodes（技术博主，8K 粉丝）发推介绍 SkillsBench，获得 **3,488 bookmarks**（比原推传播更广）
- 2026-02-24: 李向一发推称 "刚结束 SkillsBench 社区第7次周会"
- Nebius 员工 @demian_ai 在推文下互动："skillsbench is super cool!"
- 3月7日在 SF 将举办**首届 Agent Skills Hackathon**（[Luma 报名](https://luma.com/1khurilu)）

### 关于 "Claude/Anthropic 的人关注"

论文本身大量引用 Anthropic 的工作（Claude Code, Agent Skills specification），且 Claude Code 是 SkillsBench 评测的 3 个 agent harness 之一。SkillsBench 的 Skill 定义直接来源于 Anthropic 2025b 的 Agent Skills 规范。论文中引用的 Skills 聚合中，Claude Code 生态贡献了最多的 Skills（28,412/47,150）。Anthropic 方面的具体社交互动未在公开推文中发现明确证据，但 SkillsBench 对 Claude Code 生态的深度整合说明双方关系密切。

## 4. 关键推文

### 李向一公告推文 (2026-02-13)
- URL: https://x.com/xdotli/status/2022409507826213048
- 961 bookmarks，传播很广

### forloopcodes 传播推文 (2026-02-20)
- URL: https://x.com/forloopcodes/status/2024819596809949341
- 3,488 bookmarks，比原推传播更广
- @forloopcodes 是技术开发者博主（ts/kt/rs apps agents），8K 粉丝

### 李向一近期动态 (2026-02-24)
- "we just concluded 7th weekly meeting of the SkillsBench community. we are gonna ship so many cool stuff in the coming days (not weeks).. stay tuned!"

## 5. 总结

- SkillsBench 是**第一个系统评估 Agent Skills 效果的 benchmark**
- 李向一（Xiangyi Li）是**第一作者兼通讯作者**，也是 **BenchFlow 创始人/CEO**
- BenchFlow 获得 **Jeff Dean**（Google）和 **Founders Inc**（Garry Tan / YC CEO 的基金）投资
- 论文核心发现：人工策划的 Skills 显著有效（+16.2pp），但自生成 Skills 无效；少即是多
- 社交传播热度高，在 AI agent 社区引起广泛关注

## Takeaway

- 