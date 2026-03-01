---
title: "Composio Agent Orchestrator 源码深度分析"
date: 2026-02-24
tags: ["agent", "orchestrator", "composio", "source-code-analysis"]
---

# Composio Agent Orchestrator 源码深度分析

> 基于 https://github.com/ComposioHQ/agent-orchestrator 的实际源码分析，非 README 总结。

## 1. 插件架构：8 个插件槽位

源码位置：`packages/core/src/types.ts`

系统定义了 **7 个可插拔槽位 + 1 个核心服务**：

| 槽位                   | 接口名         | 职责              | 已有实现                                                                       |
| -------------------- | ----------- | --------------- | -------------------------------------------------------------------------- |
| **Runtime**          | `Runtime`   | 会话执行环境（在哪里跑）    | `runtime-tmux`、`runtime-process`                                           |
| **Agent**            | `Agent`     | AI 编码工具适配器      | `agent-claude-code`、`agent-codex`、`agent-aider`、`agent-opencode`           |
| **Workspace**        | `Workspace` | 代码隔离（每个会话独立副本）  | `workspace-worktree`、`workspace-clone`                                     |
| **Tracker**          | `Tracker`   | Issue 追踪集成      | `tracker-github`、`tracker-linear`                                          |
| **SCM**              | `SCM`       | PR 生命周期、CI、代码审查 | `scm-github`                                                               |
| **Notifier**         | `Notifier`  | 推送通知            | `notifier-desktop`、`notifier-slack`、`notifier-webhook`、`notifier-composio` |
| **Terminal**         | `Terminal`  | 人类交互 UI         | `terminal-iterm2`、`terminal-web`                                           |
| **LifecycleManager** | 核心，不可插拔     | 状态机 + 轮询 + 反应引擎 | 内置                                                                         |

### 接口组合方式

每个插件通过 `PluginModule<T>` 导出：

```typescript
interface PluginModule<T = unknown> {
  manifest: PluginManifest;  // name, slot, description, version
  create(config?: Record<string, unknown>): T;
}
```

`PluginRegistry`（`packages/core/src/plugin-registry.ts`）负责注册和查找插件。项目配置可以覆盖默认插件选择：

```yaml
defaults:
  runtime: tmux
  agent: claude-code
  workspace: worktree

projects:
  my-app:
    agent: codex  # 项目级覆盖
```

`resolvePlugins()` 函数（`session-manager.ts` 中）按 `项目配置 > 全局默认` 的优先级解析插件实例。

### 最复杂的接口：SCM

`SCM` 接口是最丰富的，覆盖完整的 PR 流水线：

- **PR 生命周期**: `detectPR()`, `getPRState()`, `mergePR()`, `closePR()`
- **CI 追踪**: `getCIChecks()`, `getCISummary()`
- **Review 追踪**: `getReviews()`, `getReviewDecision()`, `getPendingComments()`, `getAutomatedComments()`
- **合并就绪检查**: `getMergeability()` — 返回 `{ mergeable, ciPassing, approved, noConflicts, blockers }`

GitHub SCM 插件（`packages/plugins/scm-github/src/index.ts`）通过 `gh` CLI 实现所有 API 调用，使用 GraphQL 获取 review thread 的 `isResolved` 状态。

## 2. 反应系统（Reactions）：GitHub Webhook → Agent 反馈回路

源码位置：`packages/core/src/lifecycle-manager.ts`

**重要发现：没有 webhook**。系统使用**轮询模式**，不是事件驱动的。

### 工作机制

`LifecycleManager` 通过 `setInterval` 定时轮询（默认 30 秒）：

```typescript
start(intervalMs = 30_000): void {
  pollTimer = setInterval(() => void pollAll(), intervalMs);
  void pollAll(); // 立即执行一次
}
```

每次轮询对所有活跃会话执行 `determineStatus(session)`：

1. **检查 Runtime 存活**：`runtime.isAlive(handle)` — 死了就标记 `killed`
2. **检查 Agent 活动**：`agent.detectActivity(terminalOutput)` + `agent.isProcessRunning(handle)`
3. **自动检测 PR**：`scm.detectPR(session, project)` — 用分支名匹配
4. **检查 PR 状态**：CI、Review、Mergeability 全链路检查

### 状态转换 → 事件 → 反应

当检测到状态变化（如 `working → ci_failed`），系统：

1. 映射事件类型：`statusToEventType()` — 如 `ci_failed → "ci.failing"`
2. 查找反应键：`eventToReactionKey()` — 如 `"ci.failing" → "ci-failed"`
3. 执行反应：`executeReaction()` — 三种动作：
   - `send-to-agent`：向 tmux 会话发送消息（让 agent 自动修复）
   - `notify`：通知人类
   - `auto-merge`：自动合并（目前只是通知）

### 重试与升级机制

```typescript
interface ReactionTracker {
  attempts: number;
  firstTriggered: Date;
}
```

每个反应有独立的重试计数。超过 `retries` 次数或 `escalateAfter` 时间后**升级为人类通知**。例如 CI 失败的默认配置：

```yaml
ci-failed:
  auto: true
  action: send-to-agent
  message: "CI is failing on your PR. Run `gh pr checks`..."
  retries: 2
  escalateAfter: 2  # 2次重试后升级通知人类
```

状态转换时自动清除旧的重试计数器，避免跨状态累积。

## 3. 活动检测：读取 Claude Code JSONL 会话文件

源码位置：`packages/plugins/agent-claude-code/src/index.ts`

### 路径映射

Claude Code 的会话文件存储在 `~/.claude/projects/{encoded-path}/` 目录下。路径编码规则：

```typescript
export function toClaudeProjectPath(workspacePath: string): string {
  const normalized = workspacePath.replace(/\\/g, "/");
  return normalized.replace(/:/g, "").replace(/[/.]/g, "-");
}
// 例: /Users/dev/.worktrees/ao → Users-dev--worktrees-ao
```

### 两层检测机制

**第一层：终端输出分类（废弃中）**

`classifyTerminalOutput()` 解析 tmux 捕获的终端输出：
- 最后一行是 `❯` → `idle`（等待输入）
- 最后5行含 `Do you want to proceed?` → `waiting_input`
- 其他 → `active`

**第二层：JSONL 文件内省（推荐）**

`getActivityState()` 读取最新 JSONL 会话文件的最后一条记录：

```typescript
const entry = await readLastJsonlEntry(sessionFile);
switch (entry.lastType) {
  case "user":
  case "tool_use":
  case "progress":
    return { state: ageMs > threshold ? "idle" : "active" };
  case "assistant":
  case "summary":
  case "result":
    return { state: ageMs > threshold ? "idle" : "ready" };
  case "permission_request":
    return { state: "waiting_input" };
  case "error":
    return { state: "blocked" };
}
```

**关键优化**：`parseJsonlFileTail()` 只读取文件末尾 128KB（JSONL 文件可能超过 100MB）：

```typescript
async function parseJsonlFileTail(filePath: string, maxBytes = 131_072): Promise<JsonlLine[]> {
  const { size } = await stat(filePath);
  const offset = Math.max(0, size - maxBytes);
  // 大文件只用 file handle 读尾部
  const handle = await open(filePath, "r");
  const buffer = Buffer.allocUnsafe(size - offset);
  await handle.read(buffer, 0, size - offset, offset);
}
```

### 进程存活检测

`findClaudeProcess()` 通过 tmux pane TTY 查找 claude 进程：

```typescript
// 获取 tmux pane 的 TTY
const ttyOut = await execFileAsync("tmux", ["list-panes", "-t", handle.id, "-F", "#{pane_tty}"]);
// 在 ps 输出中匹配 TTY + "claude"
const processRe = /(?:^|\/)claude(?:\s|$)/;
```

### PostToolUse Hook：元数据自动更新

`agent-claude-code` 插件在工作区写入 `.claude/settings.json` 配置 PostToolUse hook，注入 `metadata-updater.sh` 脚本。这个脚本在 Claude Code 每次执行 Bash 工具后运行：

- `gh pr create` → 写入 `pr=<URL>` + `status=pr_open`
- `git checkout -b <branch>` → 写入 `branch=<name>`
- `gh pr merge` → 写入 `status=merged`

这是 Dashboard 能实时显示 PR 状态的关键机制。

## 4. 自改进循环（ao-52）

**源码中不存在这个功能**。在整个代码库中搜索 `retrospect`、`self-improv`、`ao-52`、`learning` 均未找到相关代码。

这可能是：
- 计划中的功能（尚未实现）
- 外部工具/流程（不在此仓库中）
- README 或 roadmap 中提及但未编码

系统当前的 "自动修复" 能力仅限于**反应系统**：CI 失败时向 agent 发送修复指令，review 评论时转发给 agent 处理。这不是 "学习" 或 "回顾"，只是自动化的消息路由。

## 5. 工作区隔离：Git Worktree 管理

源码位置：`packages/plugins/workspace-worktree/src/index.ts`

### 创建流程

```
~/.worktrees/{projectId}/{sessionId}/
```

```typescript
async create(cfg: WorkspaceCreateConfig): Promise<WorkspaceInfo> {
  // 1. 拉取最新代码
  await git(repoPath, "fetch", "origin", "--quiet");

  // 2. 从 origin/main 创建 worktree + 新分支
  await git(repoPath, "worktree", "add", "-b", cfg.branch, worktreePath, baseRef);

  // 3. 如果分支已存在，checkout 到已有分支
  // （分支名冲突时的 fallback 逻辑）
}
```

### 创建后钩子（postCreate）

```typescript
async postCreate(info: WorkspaceInfo, project: ProjectConfig): Promise<void> {
  // 1. 符号链接共享资源（如 .claude 目录）
  for (const symlinkPath of project.symlinks) {
    symlinkSync(sourcePath, targetPath);
  }

  // 2. 运行创建后命令（如 pnpm install）
  for (const command of project.postCreate) {
    await execFileAsync("sh", ["-c", command], { cwd: info.path });
  }
}
```

符号链接有严格的安全检查：禁止绝对路径、禁止 `..` 路径遍历、验证解析后的目标仍在工作区内。

### 销毁

```typescript
async destroy(workspacePath: string): Promise<void> {
  const gitCommonDir = await git(workspacePath, "rev-parse", "--git-common-dir");
  const repoPath = resolve(gitCommonDir, "..");
  await git(repoPath, "worktree", "remove", "--force", workspacePath);
  // 注意：故意不删除分支，避免误删已有分支
}
```

### 恢复（Restore）

支持从已有分支恢复 worktree：先 prune stale entries，然后尝试三种方式：
1. 本地分支
2. `origin/{branch}`
3. 从默认分支重新创建

## 6. 会话生命周期：完整代码路径

### spawn → work → CI fix → review → merge

**1. 启动（`session-manager.ts: spawn()`）**

```
验证 issue 存在 → 原子预留 session ID → 创建 worktree
→ 运行 postCreate hooks → 构建 prompt（3 层）
→ 获取 agent 启动命令 → 创建 tmux 会话 → 发送启动命令
→ 写入元数据文件 → 运行 postLaunchSetup（写入 Claude hooks）
```

每一步失败都有完整的回滚逻辑（清理 workspace、runtime、metadata）。

**2. 工作中（`lifecycle-manager.ts: pollAll()`）**

轮询循环每 30s 检查一次：
- Agent 是否还活着（进程 + JSONL 活动检测）
- 是否创建了 PR（通过分支名自动检测）
- 从 `spawning` 转为 `working`

**3. PR 创建后**

通过 PostToolUse hook 自动写入 `pr=<URL>` 和 `status=pr_open`。或者 lifecycle manager 下一次轮询通过 `scm.detectPR()` 发现。

**4. CI 失败**

```
determineStatus() → scm.getCISummary() → "failing"
→ 状态转换: pr_open → ci_failed
→ 事件: "ci.failing" → 反应键: "ci-failed"
→ executeReaction() → sessionManager.send() → runtime.sendMessage()
→ 向 tmux 发送: "CI is failing on your PR. Run gh pr checks..."
```

Agent 收到消息后修复代码并推送，下次轮询 CI 变为 passing。

**5. Review 处理**

```
determineStatus() → scm.getReviewDecision() → "changes_requested"
→ 状态转换: pr_open → changes_requested
→ 反应: 向 agent 发送 review comments 摘要
→ Agent 修复 → push → 状态回到 pr_open → review_pending
```

**6. 合并就绪**

```
scm.getMergeability() → { mergeable: true, ciPassing: true, approved: true }
→ 状态: approved → mergeable
→ 反应键: "approved-and-green" → 默认 auto: false → 通知人类
```

默认不自动合并（`auto: false`），需要人类确认。

**7. 全部完成**

当所有会话都进入终态（`merged` 或 `killed`），触发 `summary.all_complete` 事件。

### 元数据存储

```
~/.agent-orchestrator/{hash}-{projectId}/sessions/{sessionName}
```

文件格式是 `key=value` 平文件（兼容 bash 脚本）：

```
project=ao
worktree=/Users/foo/.worktrees/ao/ao-1
branch=feat/INT-1234
status=working
tmuxName=a3b4c5d6e7f8-ao-1
pr=https://github.com/org/repo/pull/42
issue=INT-1234
```

## 7. 配置（agent-orchestrator.yaml）

源码位置：`packages/core/src/config.ts`，使用 Zod schema 验证。

### 完整可配置项

```yaml
# 端口
port: 3000                    # Web Dashboard
terminalPort: 3001            # Terminal WebSocket
directTerminalPort: 3003      # Direct Terminal WebSocket

# 活动检测阈值
readyThresholdMs: 300000      # 5分钟后 ready → idle

# 默认插件
defaults:
  runtime: tmux               # tmux | process
  agent: claude-code           # claude-code | codex | aider | opencode
  workspace: worktree          # worktree | clone
  notifiers: [composio, desktop]

# 项目配置
projects:
  my-app:
    name: "My App"             # 显示名
    repo: org/repo             # GitHub 仓库
    path: ~/my-app             # 本地路径
    defaultBranch: main        # 默认分支
    sessionPrefix: app         # 会话前缀 → app-1, app-2
    runtime: tmux              # 项目级覆盖
    agent: claude-code         # 项目级覆盖
    workspace: worktree        # 项目级覆盖
    tracker:
      plugin: linear
      teamId: "uuid"           # 插件特定配置
    scm:
      plugin: github
    symlinks: [.claude]        # 符号链接到 worktree
    postCreate: ["pnpm install"] # 创建后命令
    agentConfig:
      permissions: skip        # 跳过权限确认
      model: "claude-sonnet-4-20250514" # 模型选择
    agentRules: "..."          # 内联规则
    agentRulesFile: "RULES.md" # 规则文件
    orchestratorRules: "..."   # orchestrator agent 规则
    reactions:                 # 项目级反应覆盖
      ci-failed:
        retries: 5

# 通知器配置
notifiers:
  slack:
    plugin: slack
    webhook: "https://..."

# 通知路由
notificationRouting:
  urgent: [desktop, composio]
  action: [desktop, composio]
  warning: [composio]
  info: [composio]

# 全局反应配置
reactions:
  ci-failed:
    auto: true
    action: send-to-agent
    message: "..."
    retries: 2
    escalateAfter: 2
  changes-requested:
    auto: true
    action: send-to-agent
    escalateAfter: "30m"
  approved-and-green:
    auto: false
    action: notify
    priority: action
  agent-stuck:
    threshold: "10m"
  # 更多...
```

### 配置搜索顺序

1. `AO_CONFIG_PATH` 环境变量
2. 从 CWD 向上搜索 `agent-orchestrator.yaml`
3. 指定目录
4. `~/.agent-orchestrator.yaml`、`~/.config/agent-orchestrator/config.yaml`

### 自动推导

- `sessionPrefix` 未设置时从路径 basename 智能生成（驼峰取首字母、kebab 取首字母、短词直接用）
- `scm` 未设置时从 `repo` 字段推导 `{ plugin: "github" }`
- `tracker` 未设置时默认 `{ plugin: "github" }`
- 所有路径自动展开 `~`

## 8. 与 K8s Operator 模式的比较

### 高度相似的 Reconciliation Loop

Agent Orchestrator 的 `LifecycleManager` 与 K8s Controller 的 reconcile 循环**结构上极其相似**：

| 概念 | K8s Operator | Agent Orchestrator |
|------|-------------|-------------------|
| **期望状态** | Custom Resource spec | 配置中的 reactions + 会话存在即表示 "应该工作" |
| **当前状态** | Pod/Service 实际状态 | Runtime 存活 + Agent 活动 + PR/CI/Review 状态 |
| **Reconcile 循环** | Controller watch + reconcile | `pollAll()` 定时轮询（30s） |
| **状态检测** | API Server watch events | `determineStatus()` 轮询 tmux + JSONL + GitHub API |
| **自动修复** | Pod restart, rollback | `send-to-agent`（CI fix, review fix） |
| **升级处理** | Alert, manual intervention | `escalateAfter` → 通知人类 |
| **终态** | Running, Succeeded, Failed | merged, killed, done |
| **元数据存储** | etcd | 平文件 `~/.agent-orchestrator/` |

### 关键差异

1. **事件驱动 vs 轮询**：K8s 使用 watch/informer 实现近实时事件流。AO 使用 30s 轮询。没有 webhook 集成。

2. **声明式 vs 命令式**：K8s Operator 的核心是声明式——你描述期望状态，operator 使之收敛。AO 更接近命令式——你显式 `spawn`，系统维护已有会话。

3. **无 Finalizer 模式**：K8s 有 finalizer 保证清理。AO 的清理依赖 `kill()` 中的尽力回滚（best-effort）。

4. **单节点**：AO 运行在单台机器上（tmux 会话），没有分布式调度。K8s 天然多节点。

5. **状态机更丰富**：AO 有 16 种会话状态，对应 PR 生命周期的每个阶段。K8s Pod 状态相对简单。

6. **反应系统 ≈ Operator 逻辑**：最相似的部分。K8s Operator 的 reconcile 逻辑（if 状态 A then 做 X）与 AO 的 reactions（if ci_failed then send-to-agent）在概念上一致。

### 架构评价

AO 本质上是一个**单机版的 K8s Operator for AI Coding Agents**。它用 tmux + 平文件实现了 K8s 用 etcd + Pod + Controller 实现的东西。代码质量不错，错误处理完善（每个创建步骤都有回滚），但缺乏：

- Webhook 集成（减少轮询开销）
- 分布式支持（多机器协调）
- 持久化队列（防止轮询间丢失事件）
- 真正的声明式 API（目前是命令式 spawn/kill）

## 总结

Agent Orchestrator 是一个设计良好的单机 AI Agent 编排系统：

- **插件架构** 清晰，7 个可插拔槽位覆盖完整 workflow
- **反应系统** 通过轮询 + 自动消息路由实现 CI/Review 的半自动闭环
- **活动检测** 深度集成 Claude Code JSONL 文件格式，实现无侵入的状态感知
- **工作区隔离** 基于 git worktree，安全检查到位
- **配置灵活** Zod 验证 + 智能默认值 + 多层覆盖
- **自改进循环不存在**——当前版本没有回顾/学习机制
- **与 K8s Operator 模式高度相似**，可以视为 "tmux 上的 operator"

核心代码量不大（核心 ~2000 行，所有插件 ~3000 行），但架构清晰、抽象良好、错误处理完善。值得作为 Agent 编排系统的参考实现。

## Takeaway
- 对比 background-agents.com 里面提到的信息
- 这些是我们做一个 Agent System 需要做到的事情
	- 按需隔离计算环境
	- Message / 事件路由系统（PR/CVE/Slack/cron 触发）
	- 权限审计爆炸半径治理层
- Good Reference