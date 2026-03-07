---
title: "AI 编程与 Agent 工作流：从设计哲学到工程实践"
date: 2026-03-07
tags: ["ai-coding", "agent", "context-engineering", "vibe-coding", "reference"]
---

## 概述

17 篇文章的系统整理，覆盖 AI 编程与 Agent 工作流的完整图景：设计哲学、实战经验、方法论工具、趋势评论。

核心共识：**人类是架构师，AI 是工具，不是反过来。**

---

## A. Agent 设计哲学

### 1. 让模型自己构建上下文（ZZ-611）

**Thariq Shaukat (Anthropic, Claude Code team)**

构建 Claude Code 过程中关于 agent action space 设计的经验。

**亮点：**

- **Action Space 是核心难题** — 工具可以用 bash、skills、code execution (PTC) 等原语构建，但选什么组合决定 agent 能力上限
- **Grep 工具的启示** — 早期想给 Claude 预塞文件上下文，后来发现让 Claude 自己 grep 搜索文件效果更好。**模型越聪明，越擅长自己构建上下文，你预塞反而添乱**
- **观察模型行为再设计工具** — 不能凭空设计，需要跑 agent 看它实际怎么用工具、在哪里卡住、哪些工具被滥用
- **工具要匹配模型能力** — 照搬人类工具给 agent 不行（人用 `find` + `grep` 分步搜索，agent 可能更适合一个结合的 `search` 工具）

> 启示：好的 agent 工具设计不是"给模型提供更多信息"，而是"给模型提供更好的获取信息的方式"。

### 2. Spec 是活文档，不是蓝图（ZZ-826）

**Augment Code** (265K views)

**亮点：**

- **文档的宿命** — 设计文档、changelog、README、架构图 → 几乎立刻过期。更新文档是隐形工作，跟所有其他事情竞争优先级，**几乎每次都输**。试过流程、工具、团队价值观 → 都没用。因为我们在要求人类做人类不会可靠做的事。
- **过期 spec 的特殊风险** — 过期的设计文档误导下一个工程师；过期的 spec 误导 agent，**agent 不知道有问题，自信地执行错误计划**
- **好 junior 工程师类比** — 好 junior 不是报告每行代码，而是 surface 改变方向的决策。发现 API 不支持预期的分页 → 更新 ticket 说"这个假设错了，我做了 X，原因是 Y"
- **Intent 的解法** — 人描述目标 → coordinator agent 起草 spec + 分任务 → 人审批 → agent 执行时发现假设有误 → **自己更新 spec**（"发现已有 auth context，用了它而不是新建"）→ 人可随时暂停改写 → agent 从新状态继续

> "If agents can write code, they can update the plan. Let them."

### 3. AI 是"框架"还是"库"？（ZZ-827）

**Piglei**

**亮点：**

- **核心类比** — Framework 控制你（填色卡通画），Library 被你控制（玩积木）。Vibe Coding = 把 AI 当框架用
- **DRF ModelViewSet 的例子** — 4 行代码获得完整 CRUD REST API，表面认知成本为零。但当你需要修改 create 响应体时，必须理解 `perform_create`、`get_serializer`、`CreateModelMixin` 的继承链 → **爆改到面目全非**。这就是隐藏的认知债务
- **Django ORM N+1 的例子** — 抽象泄露：你以为 `post.comments.all()` 是普通属性访问，实际是一次 SQL 查询。循环 100 篇帖子 = 101 次查询。自然语言驱动 AI 编程时同样的泄露会发生
- **更短的提示词 = 更多隐藏债务** — 和 ModelViewSet 一样，"一句话搞定"的背后是你不理解的决策链

> 建议：做总设计师，把 AI 当库用。不追求"写更少实现更多"，找提示词甜蜜区。关注程序整体结构。

### 4. 纸带 + 锚点范式（ZZ-593）

**PsiACE** — 基于 bub（群聊 Agent）的实战经验

**亮点：**

- **Coding Agent 最小充分工具集** — bash/read/write/edit 四个就够了。更多工具不是更强，是更乱（呼应 ZZ-630 的 "agentic dementia"）
- **朴素 RAG 在代码场景不好用** — 代码有结构依赖，向量相似度抓不到。grep + read + Agent Loop 更实用：agent 自己决定搜什么、读什么、再搜什么
- **状态延续假设的代价** — 多轮对话 = 假设上下文连续存在。但实际上：会话硬隔离丢状态、Memory 漂移（旧记忆污染新任务）、上下文膨胀（越聊越慢）
- **纸带 + 锚点** — 永续单会话追加写入（像打字机纸带），锚点标记最小关键状态，按需装配上下文而非全量继承。**用空间换时间的妥协**

> 同作者还有 ZZ-578（被缚的普罗米修斯），从哲学角度批判 context engineering 的局限性

### 5. Programmatic Tool Calling (ZZ-600)

**Lance Martin (Anthropic)**

**亮点：**

- **传统工具调用的"组合税"** — 每次工具调用：agent 发请求 → 系统执行 → 结果回传 context → agent 看到结果 → 决定下一步。10 次工具调用 = 10 次 round trip，每次都增加延迟、序列化成本、context 膨胀
- **PTC 的解法** — Claude 在容器里写一段代码，代码里调用多个工具。**中间结果留在代码变量里，不回传 context 窗口**。容器调用工具时暂停，调用跨沙箱边界完成，结果返回容器继续执行。最终只返回一个结果
- **性能数据** — BrowseComp + DeepsearchQA 平均准确率 +11%，输入 token -24%。Opus 4.6 + PTC 在 LMArena Search 排名 #1
- **工具 vs Bash 不是二选一** — 工具的价值是控制面：检查、拒绝、日志、人工审批、并发控制、可观测性。Claude Code 不只用 bash，用工具管理特定操作

> 本质："代码的可组合性 + 工具的控制面"的统一。

---

## B. 实战经验

### 6. 55% iOS 代码由 Claude 写（ZZ-706）

**Tolan 团队 (Dan Federman)** — 6 个月实践

**亮点：**

- **不靠 CLAUDE.md** — 用 agent 标准化 10 万行代码，让每个文件本身展示正确模式。代码库就是文档。"如果 agent 看到的每个文件都遵循同一模式，它自然写出一致的代码"
- **Subagent 分离实现和风格** — implementation agent 写功能代码，**独立的 review agents** 转成团队代码标准。两个 agent 关注点分离，质量更高
- **PR Shepherd agent** — 自动监控所有 open PR，CI 失败时自动修复、review 评论自动响应。人类工程师早上来看到 PR 已经 ready to merge
- **复合增长效应** — 标准化代码库 + skill 套件 + 独立 agent = **指数效果**。Opus 4.5 之后每代模型提升都被这套体系放大

**质量数据：**
- crash-free rate: 99.6% → 99.9%
- runtime errors 降 54%
- 高活跃用户翻倍
- 2 月 55% commits co-authored by Claude

### 7. DBA 运维 Agent 一下午搭完（ZZ-720）

**plantegg** — 资深 MySQL DBA

**亮点：**

- **80% 工作量在写工具** — 不是调 prompt，是写安全拦截（防止 agent 执行 DROP TABLE）、超时控制（tail -f 会卡死 agent）、输出格式化（把 MySQL 结果转成 agent 能理解的格式）
- **6 轮 ReAct 自动诊断** — Thought→Action→Observation 循环，自动完成主从复制状态诊断，**发现人工可能遗漏的半同步降级风险**
- **踩坑全记录：**
  - MySQL root 不允许 TCP 连接 → 改用 SSH + mysql CLI socket
  - AWS Bedrock 不支持 assistant prefill → 换非 Bedrock 后端
  - `tail -f` 卡死 Agent → paramiko 非阻塞读取 + deadline 超时
  - CrewAI 重置 litellm 回调 → monkey-patch `set_callbacks`

> "框架只是脚手架，真正决定 Agent 好不好用的是工具质量和对业务场景的理解。" — 和 ZZ-593 的"四工具最小集"形成对比：工具要少，但每个要精

### 8. 零编程背景 36 小时搭 AI Chief of Staff（ZZ-775）

**Jim Prosser** — 43 岁通讯顾问，ex-Google/Twitter/SoFi 公关负责人，零编程背景

**亮点：**

- **完整架构** — 隔夜：日历扫描 + Google Maps 算车程 + 邮件自动分拣到 Todoist。早上：Stream Deck 一键触发 AM Sweep → 四分类（🟢全自动 / 🟡80%自动 / 🔴需人脑 / ⚪今天不行）→ 6 个专用 agent 并行处理 → 剩余任务 Time Block 排进日历
- **"Dispatch, Prep, Yours, Skip"** — 默认偏向 Prep（准备材料让人决策），**绝不** Dispatch 以下事项：发邮件、写战略文档、定价、关系敏感沟通。这是非程序员对 AI 边界的直觉
- **经济账** — Claude Max $100/月 + 增量 $5-10/月。等效 VA（虚拟助理）$400-1000/月。每天省 30-45 分钟，年省 130-195 小时

> 最有价值的不是他搭了什么，而是他本能地划出了"AI 不该自动做"的红线。

### 9. Go 现代代码 Agent Guidelines（ZZ-667）

**JetBrains** 开源

**亮点：**

- **两个问题的精确诊断** — ① 训练数据滞后：模型不知道 Go 1.21+ 新特性 ② 频率偏差：训练数据中老写法样本多 10x → 模型默认生成老写法
- **自动适配版本** — Agent 读 `go.mod` 里的 Go 版本号，只使用该版本已有的特性。不会给 Go 1.18 项目生成 Go 1.22 代码
- **具体现代惯用法：**
  - `max(a, b)` 代替 `if a > b { return a }; return b` (Go 1.21+)
  - `slices.Contains(s, v)` 代替 for 循环遍历 (Go 1.21+)
  - `errors.AsType[T](err)` 类型安全错误匹配 (Go 1.24+)
  - `new(42)` 获取指针 (Go 1.26+)
- **分发方式** — Junie 内置；Claude Code 作为 plugin；也可手动加入 CLAUDE.md

> 这个思路可以推广到任何语言：为 agent 提供"现代写法指南"解决训练数据滞后

---

## C. 方法论和工具

### 10-11. Vibe Coding 2.0 & 3.0 Checklists（ZZ-615, ZZ-770）

**Harshil Tomar** (dreamlaunch.studio)

**亮点（从 30+ 条规则中提炼最有价值的）：**

- **Hallucination 是系统属性不是 bug** — 每个 LLM 输出当草稿，不是答案
- **Evals 是 AI 的单元测试** — 没有 eval 的 AI 项目 = 没有测试的代码
- **Chunking 经验值** — ~500 tokens/chunk 带 10-20% overlap
- **Temperature 经验值** — 代码生成 0.0-0.3，创意写作 0.7-1.0
- **Few-shot 是最简单的质量提升** — 给 3 个输入输出示例比写 500 字 prompt 有效
- **Guardrails 三件套** — output validation + JSON schema check + blocklist

> 偏入门/中级。对 ZZ 本人价值不高（都是已知概念），但作为给团队新人的 onboarding checklist 有参考价值

### 12. Agentic Manual Testing（ZZ-783）

**Simon Willison**

**亮点：**

- **核心论点** — 自动测试通过 ≠ 代码正确。单元测试覆盖已知场景，但 **agent 最擅长发现的是你没想到要测的东西**
- **三种手动测试方式：**
  1. `python -c` / `curl` — 最简单，agent 直接跑代码验证
  2. Playwright / agent-browser — 端到端 UI 测试
  3. **Rodney** — CDP 控制真实 Chrome + 截图 → 视觉评估 UI 是否正常（不只看 DOM，看渲染结果）
- **Showboat 防作弊工具** — `exec` 命令同时记录命令文本+执行+记录输出。**防止 agent 声称"已验证"但其实没跑**。`note` 写观察笔记，`image` 加截图证据

> 关键洞察：agent 测试的价值不在自动化，在于"像一个不了解代码的新人那样试用"— 发现开发者自己的盲点

### 13. CLI Token 压缩 60-90%（ZZ-806）

**rtk** — Rust 单 binary, <10ms overhead

**亮点：**

- **完全透明** — `rtk init --global` 注册 Claude Code hook，自动重写命令 `git status` → `rtk git status`。Claude 不知道发生了重写，只收到压缩后的输出
- **四种压缩策略：**
  1. **Smart Filtering** — 去噪（注释、空行、样板文本、重复 warning）
  2. **Grouping** — 相似项聚合（50 个文件按目录分组、100 个同类错误折叠）
  3. **Truncation** — 保留开头/结尾关键上下文，砍中间冗余
  4. **Deduplication** — 重复日志行折叠 + 计数（"repeated 47 times"）
- **Token 节省实测：**

| 命令类型 | 节省 |
|---|---|
| cargo/npm/pytest test | **-90%** |
| git add/commit/push | **-92%** |
| ls/tree/grep/rg | **-80%** |
| git diff | **-75%** |
| 中型项目总计 | ~118K → ~24K (**-80%**) |

- **覆盖范围** — 文件操作、Git、GitHub CLI、6 种测试框架、7 种 linter、Docker/K8s
- 安装：`brew install rtk`

### 14. Agent UI 工具包（ZZ-725）

**ui.sh** — Adam Wathan & Steve Schoger (Tailwind CSS 创始人)

**亮点：**

- 专为 AI coding agent 设计的 UI toolkit，解决 **agent 生成 UI 质量差** 这个普遍痛点
- 支持 Claude Code、Amp、Cursor、OpenCode、Codex
- 目前 invite-only，具体实现细节未公开
- **最大卖点是团队** — Tailwind CSS 已证明的产品品味 + 执行力。如果有人能让 agent 生成好看的 UI，是他们

---

## D. 趋势评论

### 15. Code Review 已死（ZZ-718）

**Latent Space** (guest post by Ankit)

**亮点：**

- **10,000+ 开发者数据** — 高 AI 采用团队：完成任务多 21%，合并 PR 多 98%，但 **PR review 时间增加 91%**。产出翻倍，review 成了瓶颈
- **Code Review 本就不完美** — 2012-2014 才普及（GitHub PR workflow），之前几十年也在发布软件。不是神圣不可侵犯的流程
- **AI Review AI 的悖论** — AI 写代码 + AI review = 相同盲点的 agent 互相审查，意义有限。这不是"增加一个 AI reviewer"能解决的
- **检查点上移** — 从"你写对了吗"变成"我们在用正确的约束解决正确的问题吗"。人类审 spec/plan/约束/验收标准，**不审 500 行 diff**
- **Code 变成 spec 的产物** — 代码不再是 source of truth，spec 才是

> 和 ZZ-826 (Augment) 的方向完全一致：人审 spec，agent 写代码。

### 16. 享乐适应：90 天变基线（ZZ-602）

**Augment Code**

**亮点：**

- **享乐跑步机** — autocomplete → chat → smart edit → agent → multi-agent，每代 ~90 天从"惊叹"变成"最低期望"
- **速度前所未有** — WWDC 一年一次，React Hooks 花 6 年普及；AI coding **每月一次 WWDC 级变化**
- **双因素驱动** — 模型能力 (capability) + 开发者信任 (trust) 共同解锁新时代。信任加速循环：每代模型犯更少错 → 信任更快建立 → 采用加速 → 基线更快到来
- **当前阶段：multi-agent** — 并行 agent + git worktree + spec 协调
- **下一阶段：云端 agent** — 由 SDLC 事件触发而非人类 prompt，主动检测问题

> 核心问题不是未来是否到来，而是你是否已建立在那个速度下工作的肌肉记忆

### 17. 五维度 Agent 设计框架（ZZ-630）

**@neural_avb**

**亮点：**

1. **System Prompt 三问** — 你观察到什么？成功长什么样？你有什么工具？— 写 system prompt 前先回答这三个问题
2. **Lazy Loading > 预加载** — 不预塞文件内容，给 agent 索引让它按需 `read`。预加载会让 agent 过度依赖初始信息而忽略探索
3. **工具太多 = Agentic Dementia** — agent 面对 50 个工具时会"失忆"，选错工具或忘了有哪些工具。**极简 action space** 是关键
4. **Subagent 三场景** — ① 任务多样化（不同能力需求）② 输出过长（拆开避免 context 爆）③ 可并行（同时跑加速）
5. **Context Persistence 是最难的维度** — LLM 天然无状态。外部 memory 必须有人/有系统维护。这也是 ZZ-593 纸带+锚点要解决的问题
6. **评估是关键** — log traces + replay harness。如果不能回放 agent 的决策链，就无法改进

---

## 交叉洞察

### 1. 人做架构，agent 做执行（826 + 827 + 611）

三篇从不同角度说同一件事：
- Augment (826): spec 是共同语言，agent 可以更新但人做最终决策
- Piglei (827): 把 AI 当库不当框架，你掌控调用
- Thariq (611): 给 agent 好的搜索工具，让它自己构建上下文，不替它做

### 2. 上下文管理三层（593 + 600 + 806）

从底向上解决 context 膨胀：
- **架构层** (593): 纸带+锚点，不继承全量状态
- **工具调用层** (600): PTC，中间结果不回传 context（-24% token）
- **CLI 输出层** (806): rtk 压缩命令输出（-80% token）

三层叠加可能实现 90%+ 的 token 节省。

### 3. 三个成熟度级别（706 + 720 + 775）

| 级别 | 团队 | 做法 | 关键差异 |
|---|---|---|---|
| 专业团队 | Tolan (706) | subagent 分工 + PR Shepherd + 代码库即文档 | 系统化、可复制 |
| 领域专家 | plantegg (720) | ReAct + 自写工具 + 踩坑记录 | 80% 时间在写工具 |
| 零基础 | Prosser (775) | Stream Deck 触发 + 四色分类 + 6 agent 并行 | 直觉划出 AI 红线 |

### 4. 检查点上移（718 + 826）

数据证据：review 时间 +91%，产出 +98%。解法：
- 从审代码 → 审 spec
- 从人维护文档 → agent 更新文档
- 从 source of truth = code → source of truth = spec

### 5. 控制权 vs 能力（827 + 630）

- Piglei (827): AI 当框架 = 控制权丧失 + 认知债务
- neural_avb (630): 工具太多 = agentic dementia
- 共同结论：**少即是多。4 个精准工具 > 50 个模糊工具。简短但你理解的 prompt > 长但你不理解的 prompt。**

---

## 来源

| Issue | 标题 | 作者 | 关键贡献 |
|---|---|---|---|
| ZZ-611 | Lessons from Building Claude Code | Thariq (Anthropic) | agent 自建上下文 > 预塞 |
| ZZ-826 | Spec-Driven Dev 必败 | Augment Code | spec 是活文档，agent 可更新 |
| ZZ-827 | AI 编程是一种框架 | Piglei | 框架 vs 库，认知债务 |
| ZZ-593 | 木匠，锤子，钉子 | PsiACE | 纸带+锚点范式 |
| ZZ-600 | Give Claude a Computer (PTC) | Lance Martin (Anthropic) | 代码组合性+工具控制面 |
| ZZ-706 | Claudified iOS App | Tolan / Dan Federman | 55% AI 代码，crash-free 99.9% |
| ZZ-720 | DBA 运维 Agent | plantegg | 一下午搭完，80% 写工具 |
| ZZ-775 | AI Chief of Staff | Jim Prosser | 零编程 36h，年省 195h |
| ZZ-667 | Go Modern Guidelines | JetBrains | 频率偏差+版本适配 |
| ZZ-615 | Vibe Coding 2.0 | Harshil Tomar | 18 条规则 checklist |
| ZZ-770 | Vibe Coding 3.0 | Harshil Tomar | 15 概念，偏入门 |
| ZZ-783 | Agentic Manual Testing | Simon Willison | Rodney+Showboat 防作弊 |
| ZZ-806 | rtk CLI Token 压缩 | rtk-ai | 透明 hook，-80% token |
| ZZ-725 | ui.sh Agent UI Kit | Tailwind CSS | invite-only，品味保证 |
| ZZ-718 | Kill the Code Review | Latent Space / Ankit | review +91%，审 spec 不审 diff |
| ZZ-602 | 享乐适应效应 | Augment Code | 90 天变基线，每月 WWDC |
| ZZ-630 | 五维度 Agent 框架 | @neural_avb | agentic dementia，三问 |

## Takeaway

- 观察模型行为再设计工具, 不能凭空设计，需要跑 agent 看它实际怎么用工具、在哪里卡住、哪些工具被滥用
- Tape System
- try rtk