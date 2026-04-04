---
title: "AI 工具 / Agent 产品与架构"
date: 2026-03-07
tags: [ai, agent, openclaw, memory, rust, rag, multi-agent]
---

# AI 工具 / Agent 产品与架构

> 2026-W10 精选 14 条。三条主线：OpenClaw 生态的三面（创始人愿景 / 逆向工程 / 理性批评）、Agent 记忆与检索的范式之争、以及以 Rust 为核心的工具链崛起。

---

## 一、OpenClaw 三视角

### ZZ-670 — Lex Fridman #491 × Peter Steinberger（3h15m）

**来源:** [YouTube](https://www.youtube.com/watch?v=YFjfBk8HI5o)

OpenClaw 创始人 Peter Steinberger 与 Lex 的深度对话，是目前关于 AI Agent 产品哲学最完整的一手资料。

**关键时刻：Marrakesh 顿悟**
> Peter 无意中发了语音消息给 agent，agent 自主发现文件是 opus 格式，调用 ffmpeg 转换，找到 OpenAI key 调用 Whisper API 翻译——全程无人教它。这一刻让他意识到"真正的 agent"是什么。

**技术决策亮点：**
- **1小时原型** — 最初只是 `WhatsApp → Claude CLI` 单行管道，没有复杂架构
- **Skills > MCP** — Peter 明确批评 MCP 污染上下文且不可组合；Skills 用 CLI + 自然语言描述，模型天然擅长 UNIX 命令，用 `jq` 等工具过滤
- **NO_REPLY token** — 让 agent 在群聊中有"闭嘴"的选择，使交互更自然（大量实现者忽略的细节）
- **Heartbeat 机制** — Agent 主动检查邮件/日历/通知；Peter 住院时 agent 主动关心他
- **Agent 自我意识** — Agent 知道自己的源码、运行环境、文档位置、使用的模型，能修改自己的软件

**背景：** PSPDFKit 13年→10亿设备→卖掉→消失3年→通过 AI 玩耍重新点燃热情。1月份 6600 次 commit，同时运行 4-10 个 agent。

---

### ZZ-685 — 从零写 OpenClaw：@jakevin7 的 7 天 CrabClaw

**来源:** [GitHub - crabclaw](https://github.com/jackwener/crabclaw)

7 天 73 commits 13000+ 行 Rust，AI 辅助逆向工程 OpenClaw 架构。是目前最好的 OpenClaw 内部机制分析。

**架构核心：**

| 组件 | 设计 |
|------|------|
| **AgentLoop** | 单向数据流：Route → Record → Tools → Context → Model → Process |
| **ProgressiveToolView** | 初始只传工具名+一行描述（~50 token），按需展开完整 Schema，**省 90%+ token** |
| **Tape** | JSONL only-append 记忆，含 Anchor 语义边界和 Handoff 上下文重置 |
| **Tool Calling Loop** | 最多 15 轮，HashSet 重复检测防死循环 |

**AI 辅助开发核心教训：**
- 面条式起点会以极快速度自我强化，架构是约束复杂度的唯一方法
- **详细 spec 不可靠** — spec 过时后 AI 会忠实执行错误计划
- 工程师核心工作变成「搭建让 AI 能自闭环的环境」

---

### ZZ-842 — hsu.cy 的理性批评：桌面 OS 的胜利 ≠ AI 的胜利

**来源:** [hsu.cy](https://hsu.cy/2026/01/clawding-around/)

最值得收藏的 OpenClaw 批评文章，不情绪化，直击要害。

**Simon Willison "Lethal Trifecta" 全中：**
1. 私有敏感数据（邮件/文件/SSH 密钥）
2. 不可信输入（邮件内容）
3. 对外通信能力（网络/API）

→ **实际案例：一封恶意邮件让 agent 交出 SSH 密钥**

**成本问题：** Viticci 一个周末烧 $560 管理 Obsidian 笔记，简单请求消耗上万 token。

**最大洞察（也是最大担忧）：**
> "OpenClaw 不是因为模型强，是因为运行在数据旁边。读文件、调 CLI、操作鼠标键盘——桌面系统几十年前就能做这些。如果桌面系统越来越封闭（像移动端），就不会有新的 OpenClaw。"

热度预测：6-12 个月消退（可靠性 80-90% 对生产服务不可接受）。

---

## 二、Agent 编排与记忆

### ZZ-798 — Edict：用唐朝三省六部制设计 Multi-Agent

**来源:** [GitHub - edict](https://github.com/cft0808/edict)

将唐朝政务制度映射为 Multi-Agent 工作流：太子分拣 → 中书省规划 → **门下省审核封驳** → 尚书省派发 → 六部并行执行。

**核心创新：门下省 = 强制质量关卡**

每个任务必须经过门下省，不合格直接封驳打回重做。这是 CrewAI/AutoGen 没有的制度性审核。

**配置：**
- 12 个 Agent（11 业务 + 1 兼容），每个独立 workspace/skills/模型
- 飞书集成（下旨 → 执行 → 回奏）
- 旨意看板（Kanban + 心跳监控）+ 奏折阁（五阶段时间线归档）
- Token 消耗排行榜 + 模型热切换

---

### ZZ-832 — Google Always-On Memory Agent：无向量库的纯 LLM 记忆

**来源:** [GitHub - always-on-memory-agent](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/gemini/agents/always-on-memory-agent)

**技术栈:** Google ADK + Gemini 3.1 Flash-Lite

三个 Agent 模拟人脑记忆机制：

| Agent | 职责 |
|-------|------|
| **IngestAgent** | 多模态摄入（27 种文件类型）→ 结构化记忆 |
| **ConsolidateAgent** | 每 30 分钟运行，像睡眠中的大脑：找连接、生成跨领域洞察、压缩 |
| **QueryAgent** | 读所有记忆 + 整合洞察 → 带来源引用的综合回答 |

**vs 现有方案：**

| 方案 | 问题 |
|------|------|
| Vector DB + RAG | 被动，embed 一次检索，无主动处理 |
| 对话摘要 | 时间长细节丢失，无交叉引用 |
| 知识图谱 | 构建和维护成本高 |

---

### ZZ-782 — Pal：Context Agent ≠ Knowledge Agent

**来源:** [GitHub - pal](https://github.com/agno-agi/pal)

**核心区分：**
- Knowledge Agent：全部 embed → 向量相似搜索 → 15 个语义相似但不聚焦的 chunk
- **Context Agent：跨系统导航 context graph，按需查询，记住哪些源有用**

Context Graph 四层：SQL 数据库、Files（markdown 配置行为）、Knowledge Map（索引）、Learnings（策略+纠正）

执行循环：`Classify → Recall → Retrieve → Act → Learn`

治理边界设计：邮件只能起草不能发送，文件自由写入禁止删除。

---

### ZZ-666 — File System Is the New Database：Personal Brain OS

**来源:** [X Article](https://x.com/i/article/2025249985722224640) — **2.2M views**
**作者:** Muratcan Koylan (@koylanai), Context Engineer at Sully.ai

用 Git repo + 80+ 文件（Markdown/YAML/JSONL）构建个人 AI 操作系统，无数据库、无 API。

**Progressive Disclosure 三层加载：**
- Level 1：路由文件（始终加载，轻量）
- Level 2：模块指令（按需加载）
- Level 3：实际数据 JSONL/YAML（按需）

**Episodic Memory 设计（核心亮点）：** 不只存事实，存**判断**：
- `experiences.jsonl` — 关键时刻 + 情绪权重
- `decisions.jsonl` — 决策 + 推理 + 替代方案
- `failures.jsonl` — 失败 + 根因 + 预防

**Voice 编码为结构化数据：** 5 维度 1-10 评分 + 50+ banned words 三级分类 + 每 500 字 voice checkpoint

**文件格式-功能映射：** JSONL（append-only，防 agent 覆写）、YAML（层级配置+注释）、Markdown（LLM 原生可读）

---

### ZZ-835 — Agentic File System：Unix 哲学应用于 Agent 上下文

**来源:** [arxiv.org/abs/2512.05470](https://arxiv.org/abs/2512.05470)

将 Unix "everything is a file" 哲学应用于 Agent 上下文管理。框架：AIGNE（开源）。

**架构三组件：**
1. **Context Constructor** — 组装上下文（统一 mounting，像 Unix mount 挂载不同来源）
2. **Context Loader** — 在 token 约束下投递上下文
3. **Context Evaluator** — 验证上下文质量

统一 metadata 和 access control：可审计、可追溯、人类作为 curator/verifier/co-reasoner。

---

### ZZ-654 — PageIndex：无向量无分块 RAG，FinanceBench 98.7%

**来源:** [GitHub - PageIndex](https://github.com/VectifyAI/PageIndex)

**核心论点：相似度 ≠ 相关性，检索需要推理而非向量匹配**

受 AlphaGo 启发：构建文档层级树索引 → LLM 推理式树搜索检索。

三个"无"：无向量数据库、无分块、无 OCR（Vision 模式）

**98.7% FinanceBench** — 金融文档分析 SOTA，远超传统向量 RAG。

检索基于推理路径，可追溯到具体页码和章节（可解释性）。支持 MCP 集成。

---

## 三、工具链

### ZZ-746 — Perplexity Computer：Firecracker 微 VM + 19 模型路由

**来源:** [X](https://x.com/xds2000/status/2029433337895653466)

**沙箱层：E2B Firecracker**
- **150-170ms 启动**，比 Docker 更强隔离
- 每沙箱：真实文件系统 + 浏览器（Comet）+ 数百连接器
- 每月数百万沙箱

**19 模型智能路由：**
| 模型 | 用途 |
|------|------|
| Claude Opus 4.6 | 核心推理 |
| Grok | 快速轻量 |
| ChatGPT 5.2 | 长上下文 |
| Gemini | 深度研究 |

Meta-router 动态选最佳模型 + 多沙箱异步并行任务图。

持久内存：95% 回忆准确率（升级前 77%）。

---

### ZZ-804 — picc：全 Rust 零 Swift/ObjC 的 macOS 自动化

**来源:** [GitHub - picc](https://github.com/andelf/picc)

**axcli — Playwright 风格 macOS Accessibility API：**
```
axcli snapshot   # 获取 UI 树
axcli click      # 点击元素
axcli input      # 输入文本
axcli screenshot --ocr  # 截图+OCR
```

工具集：
- **picc** — Ctrl+Cmd+A 截图，拖选区域，Vision OCR（中英文）
- **dictation** — 长按右 Cmd 语音输入，支持离线 SenseVoice（~250MB）
- **claude_menubar** — Claude Code session 状态 menubar 指示器（via hooks）

全 Rust 实现，通过 objc2 调用 Apple frameworks，零 Swift/ObjC 依赖。

---

### ZZ-735 — Google Workspace CLI (gws)：Rust 实现，动态构建命令

**来源:** [GitHub - googleworkspace/cli](https://github.com/googleworkspace/cli)

**动态命令构建：** 运行时读 Google Discovery Service 自动生成命令，Google 加新 API 自动可用。

亮点：
- 内置 **40+ agent skills**
- 内置 **MCP Server**（agent 通过 MCP 管理 Workspace）
- 结构化 JSON 输出（AI Agent 优先）
- 认证：OAuth / Service Account / Domain-Wide Delegation
- 凭证 AES-256-GCM 加密，key 存 OS keyring

Rust 实现，`cargo install --path .`。活跃开发中，未到 v1.0。

---

### ZZ-722 — agent-browser `--native`：纯 Rust CDP，单 binary

**来源:** [X](https://x.com/ctatedev/status/2028960626685386994)

`agent-browser` 新增实验性 `--native` flag：

- 单个 Rust binary 直接 Chrome DevTools Protocol 通信
- **零 Node.js 依赖**，更低内存，更小体积
- 无抽象层天花板，自包含 daemon
- 运行时零依赖：只需 binary + 浏览器

---

### ZZ-802 — Philipp Schmid：47K+ Skills 几乎没人测试

**来源:** [philschmid.de/testing-skills](https://www.philschmid.de/testing-skills)

**现状：** 47,000+ skills across 6,300+ repos，几乎没人有 eval harness。

**Skill 两类（重要区分）：**
- **Capability skills** — 补模型短板，模型进步后可能不再需要（eval 告诉你何时移除）
- **Preference skills** — 记录特定工作流，持久有用

**Eval Harness 四步：**
1. Prompt set（10-20 条，含 negative tests）
2. Run agent + capture output
3. Deterministic checks（regex 检查，返回 boolean）
4. Iterate → **从 66.7% 到 100% pass rate**

关键洞察：
- **Grade outcomes, not paths** — agent 会走创造性路线
- **手动跑几次不是浪费** — 每个手动修复变成可自动化的 check
- Negative tests 不能跳过

---

## Cross-Insights

### OpenClaw 三面镜
**ZZ-670（创始人愿景）+ ZZ-685（逆向工程）+ ZZ-842（理性批评）** 构成完整认知：

Peter 说 Skills > MCP，@jakevin7 的 ProgressiveToolView 验证了 token 节省的重要性，hsu.cy 则提醒安全是被大家集体忽视的 Lethal Trifecta。三者缺一不可。

### 记忆三范式
**ZZ-666（文件系统）+ ZZ-835（Unix 抽象）+ ZZ-832（纯 LLM 整合）**：

文件系统是最简单最可靠的持久化（666），Unix 哲学提供统一抽象层（835），纯 LLM ConsolidateAgent 解决主动整合问题（832）。三者可以叠加。

### 检索范式之争：反向量共识
**ZZ-782（Context Graph）vs ZZ-654（推理式树搜索）**：

两者方向不同，但有共同前提：**相似度 ≠ 相关性**，向量相似搜索是错误的抽象。Pal 用 context graph 导航，PageIndex 用 LLM 推理树搜索——都在回答「如何做真正相关的检索」。

### 质量保证两路径
**ZZ-798（三省六部制度性审核）+ ZZ-802（eval harness 测试驱动）**：

系统级质量（门下省强制封驳）配合 skill 级质量（eval harness 66.7%→100%），形成完整 QA 体系。

### Rust 工具链崛起
**ZZ-804（macOS 自动化）+ ZZ-722（CDP 浏览器）+ ZZ-735（Google API）**：

三个独立团队，三个不同场景，都选择全 Rust + 零外部运行时依赖。这不是巧合，是 agent 工具链的工程共识。

---

*Reference 整理于 2026-W10 | 14 条 | 来源：Linear GTD*

## Takeaway:

- https://github.com/VectifyAI/PageIndex
- axcli screenshot --ocr 