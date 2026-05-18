---
title: "AI 时代的工程组织变革"
date: 2026-03-10
week: 2026-W11
tags: [AI, engineering, organization, career, flow-state, research-taste]
sources:
  - ZZ-872
  - ZZ-875
  - ZZ-880
  - ZZ-878
---

# AI 时代的工程组织变革

> 当代码变得免费，整个工程组织的价值主张都要重写。

这四篇文章来自同一天（2026-03-09），从不同层次剖析了同一个问题：**当 AI 让代码生成的边际成本趋近于零，什么东西变得稀缺？**

---

## 全局视角：一个光谱

可以把这四个视角放在同一个维度上来理解：

```
抽象层次高 ← → 抽象层次低

Amy Tam          Harrison Chase       Ashton Teng        Amelia Wattenberger
(Bloomberg Beta)  (LangChain CEO)      (coral.inc)        (Augment Code)

Research Taste → Builder vs Reviewer → Fractional AI Eng → 思考媒介重构
战略层           组织/角色层           职业/市场层         工具/认知层
```

- **Amy Tam**（顶层）：稀缺的是研究品味——知道哪个问题值得解决
- **Harrison Chase**（中层）：组织分裂为 Builder 和 Reviewer，瓶颈从实现转向审查
- **Ashton Teng**（落地层）：这个转变产生了新的职业类别——Fractional AI Engineer
- **Amelia Wattenberger**（横切层）：上面三层都在讨论角色，她在讨论整个认知界面已经破碎

---

## ZZ-875: Research Taste Is All That Matters

**来源：** Amy Tam (Bloomberg Beta) & E Chi (Quadrillion) | [原文](https://x.com/amytam01/status/2031072399731675269) | 32.5K views

### 核心论点

当任何人都能免费建造，差异化变成了**知道值得建造什么**。这是一个 research 问题，不是 engineering 问题。

### 关键细节

- **市场已经定价了这一点：** 顶尖 quant 公司给从未管过 portfolio 的本科生开 $600K。Meta Superintelligence Labs 据报道给单个研究员开出 $300M/4 年的 package。一个研究员在 $100M+ training run 上改善几个百分点的数据效率，回报早已超出薪酬。

- **为什么 engineering 可以被自动化，research 不行：** Engineering 有明确的终态（要通过的 test、要满足的 spec），所以 agentic coding 和 SWE-bench 能用。Research 没有等价的 ground truth——无法对"这个问题值得追吗"做 RL。

- **硬币翻转比喻：** 想象一个房间里有一万亿枚有偏硬币。最好的研究员不是翻得更快——而是知道哪 20 枚值得翻。这种品味跨领域迁移：理论物理学家 → quant → AI → 计算生物学。

- **当前 AI research 工具的局限：** Karpathy 的 `autoresearch` 一晚上在单张 GPU 跑了 126 个实验，但只是在做 hyperparameter sweeps（weight decay、初始化尺度）。更快地翻同一批低价值硬币。

- **训练数据问题：** 最好的 research taste 建立在从未发表的失败实验上——lab 内部只分享 win，researcher 每天早饭前就否定了几百个想法的过程是无法 scrape 的。

- **品味会过期：** 2018-2024 年什么都判断对的研究员，到 2026 年可能已经在模式匹配却自己不知道。

---

## ZZ-872: PRDs 已死，Builder vs Reviewer

**来源：** Harrison Chase (@hwchase17), LangChain CEO | [原文](https://x.com/hwchase17/status/2031051115169808685) | 11.9K views

### 核心论点

Coding agent 让代码生成廉价，传统 PRD→Mock→Code 瀑布流程已死。组织演变为 Prototype→Review 循环，每个人要么是 Builder 要么是 Reviewer。

### 关键细节

- **PRD 为什么死了：** PRD 存在是因为实现成本高，所以要在动工前对齐所有人。现在 coding agent 可以直接从 idea 生成可用软件，PRD 作为流程起点已过时。**但描述产品意图的文档仍然必要**——只是形态变了，未来的 PRD 可能就是结构化的、版本化的 prompt。

- **Review 成为新瓶颈：** 任何人都能写代码 → prototype 数量爆炸 → review 成为瓶颈。Review 三个维度：工程架构是否可扩展/健壮、产品是否解决用户痛点、设计是否易用直觉。

- **Builder vs Reviewer 特征：**
  - Builder：产品思维 + coding agent + 基本设计直觉，能把小功能从 idea 做到上线
  - Reviewer：某个领域的顶级系统思考者，要审得快因为 queue 太长

- **通才增值：** 一个能做 product+design+engineering 的人 > 三个专才组成的团队（消除沟通开销）。烂 PM 更危险——能快速生成 prototype 但方向错误，浪费 review 资源 + "已经做了就合了吧"的惯性。

---

## ZZ-878: Fractional AI Engineer

**来源：** Ashton Teng (@ashtonteng), coral.inc 创始人 | [原文](https://x.com/ashtonteng/status/2031094992710807574)

### 核心论点

Fractional AI Engineer 是正在形成的新职业类别：一个人嵌入小企业，理解业务流程，转化为 AI workflow 并持续维护——类比 fractional CFO，一人可服务 20-50 个客户。

### 关键细节

- **市场规模：** 美国 3000 万一人企业，数百万 10 人以下企业，有可被 AI 处理的 workflow 但没有工程师资源。工具已存在（ChatGPT、Claude），缺的是懂业务又懂 AI 的"翻译者"。

- **核心能力不是技术，是判断力：** 难点在于理解业务实际运作方式、制定 SOP、决定哪些系统该连接——这是运营和商业问题。不需要 CS 学位，需要工程思维 + 业务翻译能力。

- **持续关系，不是一次性项目：** AI 系统需要持续维护——业务需求变化、新工具出现、配置复杂度累积。客户留存率接近 100%。

- **窗口期：** 竞争几乎为零，需求真实，支撑工具（一人管理 50 客户的基础设施）正在被构建。对职业早期的人特别有吸引力，不需要传统求职路径。

---

## ZZ-880: AI 打破心流，需要新的思考媒介

**来源：** Amelia Wattenberger (@Wattenberger), Augment Code Intent Lead | [原文](https://x.com/Wattenberger/status/2031068210167222313) | 5.3K views

### 核心论点

AI 打破了开发者的心流（think→type→see 变成 prompt→wait→read→evaluate），答案不是回到手写代码，也不是"适应 prompting"——我们需要发明新的"思考媒介"。

### 关键细节

- **心流是被工程化出来的，不是魔法：** 语法高亮、REPL、热重载、类型检查——每一个工具都缩小了"意图"和"反馈"之间的距离。现代 IDE 是几十年工具设计的结果。

- **AI 编程打破心流的两个条件：**
  - 失去控制感：Agent 在黑盒里工作
  - 失去节奏感：prompt→等待→阅读→评估，节奏断裂

- **80/20 陷阱：** AI 秒出前 80%，让剩下 20%（思考、打磨、边界情况）感觉像"你是慢的那部分"。但那 20% 才是从"能用"到"好用"的关键。

- **画家比喻（"通过邮件口画画"）：** 好的创作是 make→see→respond 的连续循环。现在的 AI 编程像把描述塞进门缝，等画滑出来，再写张纸条说"天空再蓝点"。

- **写代码本身就是思考：** 以前代码会 push back，你 push forward，涌现出意料之外的东西。Prompting 把这个过程切断了。

- **历史先例：** 打孔卡没反馈循环，早期编辑器没语法高亮——每次编程形态变了，工具落后，心流消失，然后总有人填上这个 gap。Wattenberger 在 Augment Code 做 Intent Lead，正在造这个工具。

---

## ZZ 的思考

### 关于 Builder vs Reviewer（ZZ-872）

Builder vs Reviewer 是一个光谱，不是二元分类。Harrison Chase 的框架有个盲点：**AI review 本身**。

当 prototype 数量爆炸，review 成为瓶颈，自然而然会想到用 AI 来 review。但 AI review 代码的问题在于：AI 生成的代码 + AI review = 没有人真正理解这段代码在做什么。这是一个系统性风险，不只是单个 PR 的问题。

更可能的未来：**review 的主要形式是 test suites + CI，而不是 AI 读代码**。测试是可以机器验证的、可以积累的、可以作为系统 memory 的。"AI reading code to review" 这条路可能是个死胡同。

### 关于 Research Taste（ZZ-875）⚡ 关键洞察

Amy Tam 的文章里有一个未被充分展开但极其重要的触发条件：**当 AI 能"挑硬币"（具备 research taste）时，递归自我改进就开始了。**

不是"当 AI 能执行研究"，而是"当 AI 能判断哪个研究方向值得追"——这才是递归自我改进的点火时刻。AI 开始优化自己的优化过程。

目前还没到这一步（`autoresearch` 只是在更快地翻同一批低价值硬币，就是 Amy 说的那种），但这是最重要的信号要持续追踪。当 AI 的 research taste 开始超越人类判断的时候，这个循环就会闭合。

