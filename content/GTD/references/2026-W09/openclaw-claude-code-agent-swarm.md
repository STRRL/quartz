---
title: "OpenClaw + Codex/Claude Code Agent Swarm 实践参考"
date: 2026-02-24
source: "https://x.com/elvissun/status/2025920521871716562"
author: "@elvissun"
tags: ["openclaw", "agent-swarm", "claude-code", "codex", "orchestration"]
---

# OpenClaw + Codex/Claude Code Agent Swarm: 一人开发团队的完整架构

> 原文：[@elvissun 的 Twitter Article](https://x.com/elvissun/status/2025920521871716562)，2026-02-23

## 核心思路

不再直接使用 Codex 或 Claude Code，而是用 **OpenClaw 作为编排层**。OpenClaw agent（作者叫她 "Zoe"）负责：生成 prompt、选模型、派任务、监控进度、完成后通过 Telegram 通知人类 review。

**关键洞察：Context window 是零和博弈。** 塞满代码就没空间放业务上下文，反之亦然。所以需要两层架构：
- **编排层（OpenClaw/Zoe）**：持有所有业务上下文（客户数据、会议纪要、历史决策），翻译成精确 prompt
- **执行层（Codex/Claude Code）**：只关注代码，接收精确的、带上下文的 prompt

## 一、架构与流程（8 步工作流）

### Step 1: 需求 → 与 Zoe Scoping
客户需求进来后，跟 Zoe 讨论 scope。因为会议笔记自动同步到 Obsidian vault，Zoe 已经有完整上下文。Zoe 会：
1. 通过 admin API 给客户充值/解锁（立即响应客户）
2. 从生产数据库拉客户配置（只读权限，coding agent 永远不会有这个权限）
3. 生成详细 prompt 并派发 Codex agent

### Step 2: Spawn Agent
每个 agent 得到独立的 **git worktree**（隔离分支）和 **tmux session**：

```bash
git worktree add ../feat-custom-templates -b feat/custom-templates origin/main
cd ../feat-custom-templates && pnpm install
tmux new-session -d -s "codex-templates" \
  -c "/path/to/worktrees/feat-custom-templates" \
  "$HOME/.codex-agent/run-agent.sh templates gpt-5.3-codex high"
```

**关键实践：用 tmux 而不是 `codex exec` 或 `claude -p`**，因为可以中途纠偏：
```bash
tmux send-keys -t codex-templates "Stop. Focus on the API layer first, not the UI." Enter
```

任务跟踪在 `.clawdbot/active-tasks.json` 中。

### Step 3: 自动监控（每 10 分钟 cron）
不直接 poll agent（太贵），而是跑一个**确定性脚本** `.clawdbot/check-agents.sh`：
- 检查 tmux session 是否存活
- 检查 tracked branch 是否有 open PR
- 通过 `gh cli` 检查 CI 状态
- CI 失败或有严重 review 反馈 → 自动 respawn（最多 3 次）
- 只在需要人类介入时才报警

### Step 4: Agent 创建 PR
Agent 自动 commit、push、`gh pr create --fill`。**此时不通知人类**——PR 创建 ≠ 完成。

**"Done" 的定义**（很重要）：
- PR 已创建
- 已 rebase 到 main（无冲突）
- CI 通过（lint、types、unit tests、E2E）
- Codex review ✅、Claude Code review ✅、Gemini review ✅
- 有截图（如果涉及 UI 变更）

### Step 5: 三重 AI Code Review
- **Codex Reviewer**：最彻底，擅长 edge cases、逻辑错误、race conditions，误报率低
- **Gemini Code Assist**：免费，擅长安全和可扩展性问题，会给具体修复建议
- **Claude Code Reviewer**：比较鸡肋，过于保守，大多是过度工程化建议；只看 critical 标记

### Step 6: 自动化测试
CI pipeline：Lint + TypeScript + Unit + E2E + Playwright（对标生产环境的 preview）。
**新规则：UI 变更必须附截图，否则 CI 失败。**

### Step 7: 人类 Review
Telegram 通知到达时，CI 已过、三个 AI reviewer 已批准、截图已附。**Review 只需 5-10 分钟，很多 PR 看截图就直接 merge。**

### Step 8: Merge + 清理
每日 cron 清理孤立 worktree 和 task registry。

## 二、与 Composio Agent Orchestrator 的区别

文中没有直接提到 Composio，但从架构上可以比较：

| 维度 | OpenClaw (Elvis 方案) | Composio Agent Orchestrator |
|------|----------------------|---------------------------|
| **定位** | 个人本地编排层，跑在 Mac 上 | SaaS 平台，提供工具集成和 agent 编排 |
| **上下文来源** | Obsidian vault（会议笔记、客户数据、历史决策） | 通过 API 连接外部工具获取上下文 |
| **Agent 执行** | 本地 tmux + git worktree，直接跑 Codex/Claude Code CLI | 云端或通过 API 调用 |
| **业务深度** | 深度绑定个人工作流（prod DB 只读、admin API、Sentry 扫描） | 通用工具集成，不针对特定业务 |
| **自主性** | Zoe 主动找活干（扫 Sentry、扫会议纪要、扫 git log） | 需要人类触发或配置 workflow |
| **核心差异** | **"AI as co-founder"**——编排层理解你的业务，不只是代码 | **"AI as tool"**——提供工具连接，编排逻辑需自己定义 |

OpenClaw 方案的核心优势是**业务上下文的深度整合**——编排层不只是调度 agent，而是带着完整的业务理解去写 prompt、判断失败原因、决定重试策略。

## 三、可落地的实践细节

### 基础设施
- **Git worktree** 做任务隔离，每个 agent 一个独立分支和工作目录
- **Tmux** 做 agent 生命周期管理，支持中途纠偏
- **Cron + 确定性脚本** 做监控，不用 LLM poll（省钱）
- **JSON 文件** (`.clawdbot/active-tasks.json`) 做任务 registry

### Agent 选型
- **Codex (gpt-5.3-codex)**：90% 的任务，后端逻辑、复杂 bug、多文件重构。更慢但更彻底
- **Claude Code (claude-opus-4.5)**：前端、git 操作、速度优先的任务
- **Gemini**：UI 设计。先让 Gemini 出 HTML/CSS spec，再交给 Claude Code 实现

### 成本
- Claude: ~$100/月
- Codex: ~$90/月
- 可以从 $20 起步

### 瓶颈：RAM
每个 agent = 独立 worktree + node_modules + TS compiler + test runner。16GB Mac Mini 最多跑 4-5 个并行 agent。作者买了 128GB Mac Studio M4 Max ($3,500) 来解决。

### Ralph Loop V2（自我改进循环）
与原版 Ralph Loop 的区别：**失败时不是用相同 prompt 重试**，而是 Zoe 带着业务上下文分析失败原因：
- Context 溢出 → "只关注这三个文件"
- 方向错误 → "客户要的是 X 不是 Y，这是他们会议上说的原话"
- 需要澄清 → "这是客户的邮件和公司背景"

成功模式会被记录："这种 prompt 结构适合 billing 功能"、"Codex 需要先给 type definitions"、"总是要包含测试文件路径"。

### Zoe 的主动行为
- 早上：扫 Sentry → 发现 4 个新错误 → spawn 4 个 agent 修复
- 会议后：扫会议纪要 → 标记 3 个功能请求 → spawn 3 个 Codex agent
- 晚上：扫 git log → spawn Claude Code 更新 changelog 和文档

## 四、效果数据

- 最高一天 94 commits（当天有 3 个客户电话，没打开过编辑器）
- 日均约 50 commits
- 30 分钟内完成 7 个 PR（从想法到生产）
- 小到中型任务的 one-shot 成功率极高

## 五、我的思考

这篇文章的核心价值不在于具体脚本（那些很容易复制），而在于**编排思路**：

1. **Context 分层**是关键——业务上下文和代码上下文分开管理
2. **"Done" 的定义要严格**——PR 创建 ≠ 完成，需要 CI + 多重 review + 截图
3. **监控要便宜**——用确定性脚本而不是 LLM 来做状态检查
4. **中途纠偏比重启更高效**——tmux send-keys 比 kill + respawn 好
5. **让编排层主动找活干**——不只是被动执行，要扫 Sentry、扫会议纪要、扫 git log

## Takeaway

- 我需要这个东西