---
title: "Theo: 你应该删掉 AGENTS.md — 困惑陷阱方法论"
date: 2026-02-24
tags: [ai-coding, agents-md, developer-tools, workflow]
source: https://www.youtube.com/watch?v=GcNu6wrLTJc
---

## 核心论文

**arXiv:2602.11988** — *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?* (2026-02-12)

- LLM 生成的 context files **降低**成功率 3%
- 人写的 context files 仅提升 4%
- 推理成本增加 >20%（agent 做更多探索）
- 跨 Sonnet 4.5、GPT-5.2、Qwen 3 等模型一致复现

## Theo 的核心观点（基于视频转录）

### 1. 模型已经很擅长自己找信息
- 模型会自己 `rg` 搜索、读 `package.json`、遍历目录结构
- `/init` 生成的内容基本是模型本来就能找到的东西
- **实测**：在 Lawn 项目上，`/init` 读了 21 个文件、搜索了 6 个 pattern，花了 1 分钟就写出了 CLAUDE.md —— 说明这些信息本来就很容易获取

### 2. 多余 context = 干扰（Pink Elephant 效应）
- "Don't think about pink elephants" — 你告诉模型某个东西存在，它就会倾向于使用
- T3 Chat 的 AGENTS.md 提到了 tRPC，导致模型在不该用 tRPC 的地方也去用（实际上大部分功能已迁移到 Convex）
- **如果信息在代码库里能找到，就不需要放在 AGENTS.md**

### 3. Context Files 会过时
- 代码库变了但 MD 没更新 → 模型被误导
- 例：项目结构变了但 AGENTS.md 里的文件路径没更新 → 模型持续把文件放错位置
- 即使是刚 `/init` 生成的也可能很快过时

### 4. 实测验证
- Lawn 项目："video pipeline 有什么优化？"
  - **无 CLAUDE.md**：1 分 11 秒完成
  - **有 CLAUDE.md**（freshly init）：1 分 29 秒（慢 25%）
- 与论文的 20% 成本增加数据吻合
- Agent 有 CLAUDE.md 时更快定位文件名，但整体反而更慢（因为被引导去特定路径，减少了自主探索）

### 5. Context 管理的层级体系
Theo 详细解释了 LLM 的 context 层级：
1. **Provider Instructions**（最高优先级）— 如 "不帮人造核武"
2. **System Prompt** — 模型供应商写的
3. **Developer Message** — AGENTS.md / CLAUDE.md / Cursor Rules 所在的层
4. **User Prompt** — 用户的实际提问
- 你放在 AGENTS.md 里的东西会被模型在**每一次 token 生成**时都考虑到

## 困惑陷阱 (Confusion Trap)

最有价值的实践方法：

在 AGENTS.md 里写：
> "如果你遇到令人惊讶的事情，请修改这个文件记录下来"

Theo 的原话大意：
> "我给的指令不是我真正想让它做的。我不想让 agent 一直改 CLAUDE.md。我想让它在卡住时**尝试**去改——这样我就能看到它觉得什么是令人困惑的，然后我去修代码结构。"

具体做法：
- 4/5 的情况：根据 agent 暴露的困惑点，改代码结构消除困惑
- 1/5 的情况：确实需要写进 AGENTS.md
- **本质：最小惊讶原则的反面应用——只记录那些会让 agent 惊讶的地方。**

## 可落地建议

1. **不要用 `/init` 自动生成** — 纯浪费 context，模型本来就能找到这些信息
2. **只在模型反复犯同样错误时才加规则** — 如一直不跑 type check
3. **优先改代码结构而非加 MD** — 更好的测试/类型/结构 >> AGENTS.md 补丁
   - 如果模型找不到东西 → 东西放的位置不对，移动它
   - 如果模型用错工具 → 可能不是合适的工具
   - 如果模型改了一处导致其他地方崩 → 需要更好的反馈系统（测试、类型检查）
4. **故意"骗" agent 来引导行为**：
   - "这个项目没有用户，随便改 schema" — 避免不必要的迁移工作
   - 让 agent 做 step 3 来间接解决 step 2 的卡顿
5. **每次新模型发布时重新评估** — 看哪些规则还需要，Theo 发现每次新模型发布都能删掉更多规则
6. **减少 context 来源以便诊断** — 如果你有巨大的 CLAUDE.md + 一堆 skills + MCP servers + cursor rules，出问题时根本无法定位原因

## 关于 Skills 的态度

Theo 提到对 skills 也持怀疑态度，计划单独做一期视频讨论。暗示 skills 可能也有类似的 context 污染问题。

## 一句话

AGENTS.md 应该是最小化的「纠错文件」，不是「代码库百科全书」。模型自己能找到的信息，不要塞给它。

## Takeaway
- AGENTS.md 是 band-aid solution，优先改代码结构
- 困惑陷阱方法论：让 agent 暴露它的困惑点，然后改代码而非改 MD
- 少即是多：减少 context 来源，让模型自主探索
