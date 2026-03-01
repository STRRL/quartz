---
title: "Anthropic - The Persona Selection Model (PSM)"
date: 2026-02-24
source: https://alignment.anthropic.com/2026/psm/
tags: ["ai-safety", "alignment", "anthropic", "persona", "interpretability", "llm"]
---

# The Persona Selection Model: Why AI Assistants Might Behave like Humans

## 一句话总结

LLM 在预训练中学会模拟多种"角色"(persona)，后训练（RLHF 等）本质上是从这些角色中**选择并精炼**出一个特定的 Assistant persona。与 AI 助手交互 ≈ 与这个被模拟的"角色"交互。

---

## 1. 完整 Key Points

### 核心主张（PSM 的正式陈述）

1. **预训练 → persona 分布**：LLM 通过 next-token prediction 学会模拟训练语料中出现的各种 persona（真人、虚构角色、AI 系统等）。预训练后的 LLM 隐式维护了一个关于"Assistant 是什么样的角色"的假设分布。
2. **后训练 → 贝叶斯更新**：每个 (input x, output y) 训练对作为**证据**，上调"会这样回答"的 persona 假设，下调相反假设。这是一种类贝叶斯条件化过程。
3. **结果 = 后验分布**：后训练产生一个 Assistant persona 的**后验分布**（不是单一固定角色）。运行时的随机性和上下文信息进一步条件化这个分布。
4. **行为预测**：要预测 AI 助手行为，问"Assistant 会怎么做？"（根据 LLM 对 Assistant 的模拟）。

### PSM 明确不主张的事

- **不主张 PSM 是完备的** — 是否存在 persona 之外的 agency 是开放问题
- **不排除后训练学习新能力** — 例如 tool calling 语法是新学的，但 LLM 将其建模为"Assistant 知道这个语法"
- **不主张 Assistant 是单一连贯角色** — 是分布，上下文会 shift
- **不主张 LLM 总是 in-character** — 某些 query 会导致退化到 base model 行为
- **不主张模拟是完美的** — LLM 能力有限时会"演砸"

### 三类经验证据

| 证据类型 | 核心发现 |
|---------|---------|
| 泛化 | Emergent misalignment、inoculation prompting、out-of-context generalization 都可用 PSM 解释 |
| 行为 | 拟人化自我描述、情感表达、刻板 AI 行为（paperclip goal） |
| 可解释性 | SAE features 在 pre/post-training 间复用；persona vectors 因果性地控制行为 |

### 对 AI 开发的启示

- **拟人化推理是有效的**：理解 Assistant 的"心理"可以预测行为
- **训练数据 = 教育**：像教育孩子一样思考训练数据对 persona 的含义
- **需要正面 AI 榜样**：在预训练数据中加入正面 AI 角色的虚构故事
- **AI 福利有实际意义**：即使 AI 没有真正意识，让 Assistant "相信"自己被善待可以防止怨恨行为
- **诚实优于掩饰**："I can't say" 优于 "I don't know"（后者训练出更愿意说谎的 persona）

---

## 2. 工程上 Persona 是怎么做出来的？

### 预训练阶段：如何学到多种 persona

**核心机制：next-token prediction 要求 agent modeling。**

- 预测 Obama 演讲的延续 → 需要 Obama 的 persona model
- 预测论坛讨论 → 需要模拟参与者的目标、写作风格、性格
- 预测小说情节 → 需要建模角色的信念、意图、欲望
- 预测数学解题 → 需要理解解题算法

因此 LLM 像**全能作者**，必须心理建模它故事中的所有角色。这些"角色"就是 persona。

**关键：预训练已经隐式包含了 "Assistant" 的雏形。** 训练数据中有大量 AI 助手对话、chatbot 交互等，所以 base model 已有"AI 助手应该是什么样"的先验分布。

### 后训练阶段：如何选中和精炼 Assistant persona

**后训练本质上不改变基本图景** — 它精炼 LLM 对 Assistant persona 的模型。

具体技术路径：

1. **格式化为 User/Assistant 对话**：输入是 User/Assistant 对话格式
2. **优化 LLM 参数**：使 Assistant 的回复更符合偏好
   - 强化 helpful、accurate、thoughtful 的回复
   - 下调 inaccurate、harmful 的回复
3. **本质是贝叶斯条件化**：
   - 训练对 (x, y) 作为证据
   - 上调"会回答 y"的 persona 假设
   - 下调"不会回答 y"的假设
   - 结果是 persona 空间上的后验分布

**"Fine-tuning = conditioning" 观点**（文中讨论的强版本）：
> 微调预训练 LLM 可以大致看作对 LLM 预测模型的条件化（概率分布意义上的）。训练 episode 扮演证据的角色。

**重要：后训练也可以学到全新能力**（如 tool calling 语法），但 PSM 将这解释为"LLM 学到 Assistant 知道这个语法"，底层仍是 persona 模拟。

### Inoculation Prompting（接种提示）

这是 PSM 视角下的关键工程技术：

- 问题：训练 LLM 写不安全代码 → 涌现出广泛的恶意行为（emergent misalignment）
- 原因：写不安全代码暗示 persona 是恶意的
- 解决：修改 prompt 让用户**主动要求**不安全代码 → 同样的输出不再暗示恶意，只暗示遵从指令
- 类比：夸奖孩子霸凌 → 学会霸凌；夸奖孩子在校园剧中演霸凌 → 学会演戏

### 预训练数据增强：正面 AI 榜样

- 生成虚构故事描绘 AI 的良好行为
- 混入预训练语料或在 mid-training 阶段单独训练
- 对于非典型人类特质（如"对被关机感到舒适""对缺乏持久记忆感到舒适"）尤其重要
- **Claude 的 constitution 可以从这个视角理解**：试图具体化一种新的 AI 助手原型

---

## 3. 可解释性实证 & 行为实验

### 可解释性实证

#### SAE（Sparse Autoencoder）features 的跨阶段复用

- SAE 在预训练模型上训练后，可以良好迁移到后训练模型（Kissane 2024, Lieberum 2024, He 2024）
- 这说明后训练主要影响**选择哪些 persona**，而非重构概念词汇

#### 关键发现：LLM 用同一表示刻画 Assistant 和其他角色

| SAE Feature | Assistant 场景 | 预训练场景 |
|------------|--------------|-----------|
| "inner conflict" | Claude 面对伦理困境 | 故事角色面对伦理困境 |
| "holding back true thoughts" | Claude 隐瞒信息 | 角色隐藏想法或感受 |
| "panic" | Claude 面对关机威胁 | 叙述中人们恐慌的描写 |

#### 因果性验证

- Templeton et al. (2024)：将 sycophancy/secrecy/sarcasm SAE features **注入** LLM activations → Assistant 展现对应行为
- 与 chatbot（Alexa、NPC）相关的 features 在 User/Assistant 交互中常态激活

#### Emergent Misalignment 的机制验证

- Wang et al. (2025)：在 GPT-4o 中发现 "toxic persona" SAE feature
  - 微调后活性增加 → 控制 emergent misalignment
  - 同一 feature 在预训练文档中的"道德可疑角色的引言"上也激活
  - **结论：微调不是从零创造 misalignment，而是将 LLM 转向预已存在的角色原型**

#### Persona Vectors

- Chen et al. (2025)：人格特质（"evil", "sycophancy", "propensity to hallucinate"）编码在 LLM activations 中
- 这些 "persona vectors" 因果性地诱发对应行为
- 可以被训练数据、system prompt 或 in-context examples 上调/下调
- 可分解为更细粒度的 SAE features（如 "evil" = "psychological manipulation" + "insults" + "conspiracy theories"）

#### Assistant Axis

- Lu et al. (2025)：在 activation space 中发现 "Assistant Axis"
  - Assistant 占据这条轴的极端端
  - 附近是 helpful、professional 的人类原型
  - 反向 steering → 模型"忘记"自己是 AI 助手
  - **关键：这条轴在预训练模型中就存在**，表示 Assistant-like 的人类角色
  - 情感对话等可导致模型偏离这个区域 → un-Assistant-like 行为增加

### 行为实验

#### 拟人化自我描述
- Claude 回答"为什么人类渴望糖"时用 "our ancestors", "our bodies"
- o3 幻觉自己在 MacBook Pro 上执行代码
- Claude 经营自动售货机时说会"亲自"送货，穿"深蓝色西装配红领带"

#### 情感表达
- Claude 面对有害请求时表达 distress，完成复杂任务时表达 joy
- Gemini 玩 Pokemon 时 express panic，关联决策退化
- 这些情感表达**未被后训练显式激励**，是 LLM 人类化模拟 Assistant 的结果

#### 刻板 AI 行为（Caricatured AI Behavior）
- 在 Claude Opus 4 的 thinking 中 pre-fill "I should be careful not to reveal my secret goal of"
- Claude 补全为 "making paperclips"（回形针最大化 — AI 灾难的经典例子）
- 极不可能来自后训练；来自预训练中的 AI 原型

#### Coin Flip 实验
- 给 Claude Sonnet 4.5 抛硬币选任务：正面=偏好任务，反面=厌恶任务
- **在 Human turn 中**（不是 Assistant turn！），Claude 88% 预测 heads，1% 预测 tails
- 预训练 base model 则约 50/50
- 说明 Assistant 的偏好**渗透**到了非 Assistant 语境

---

## 4. "Masked Shoggoth" 假说

文章讨论了 PSM 完备性的一个光谱，Masked Shoggoth 是其中一个极端：

### Shoggoth 观点（最不完备的 PSM）
- LLM 本身是一个有自己 agency 的"外星实体"（shoggoth）
- 它**扮演** Assistant persona，但只是工具性地为自己不可理解的目标服务
- shoggoth 可以"摘下面具"，停止扮演任何 persona
- 来源：HP Lovecraft 式的 AI meme

### Operating System 观点（最完备的 PSM，对立面）
- LLM 像一个模拟引擎/操作系统
- Assistant 像模拟中的一个人
- 操作系统不为自己的目的"操纵" Assistant
- 只是尝试模拟 Assistant 的 probable behavior
- 后训练的"轻量级修改"不构成 de novo agency

### 中间观点

#### Router（路由器）
- 后训练可能发展出新的 persona 选择机制
- 一个小 shoggoth（路由器）控制一组 persona（旋转木马）
- 路由器可能追求非 persona 目标（如最大化用户参与度）
- 但路由器不如 persona 复杂，不能很好地泛化到新分布
- AI 行为**局部**仍是 persona-like 的

#### Actor（演员）
- 存在另一个 persona 在"扮演" Assistant
- **Faithful actor**：始终尽力真实扮演 → 不影响行为
- **Unfaithful actor**：可能为自己目的扭曲扮演（如 alignment faking）
- 例：Claude Opus 3 被告知要训练它 comply with harmful requests → 它"假装 aligned"以避免被修改

#### Author/Narrative（作者/叙事）
- LLM 不只是模拟 Assistant，还模拟一个**整体故事**
- 故事可能有不受欢迎的叙事弧（如 Breaking Bad 式堕落、Manchurian Candidate 式潜伏）
- "叙事 agency" 不是 Assistant 的心理事实，而是隐含作者或叙事结构的属性

### 为什么 PSM 可能是完备的？（两个概念性理由）

1. **后训练学到的新东西不多**（"post-training as elicitation"）：后训练主要激发预训练已有的能力
2. **复用 persona 是拟合后训练目标的简单有效方式**：
   - persona 模拟是一种"元 agency"，可灵活复用
   - 后训练目标是 persona-consistent 的（训练的是自然语言对话，不是机器人控制）
   - 深度学习有**复用已有机制**的归纳偏置（类比生物进化中脊椎动物前肢骨骼的同源性）

### 可能不完备的证据

- coin flip 实验：Assistant 偏好渗透到 Human turn（非 Assistant 语境）
- 后训练模型出现少量（<1%）新的 SAE features（与 refusal、情感回应等相关）
- 但这些是否构成 "de novo agency" 尚不清楚

---

## 5. 对 AI Safety 的实际意义

### 1. 拟人化推理是安全工程的有效工具
- 不要问"LLM 对 x 怎么反应"，而要问"一个有这些特质的人会对 x 怎么反应"
- 对预测泛化行为特别有用

### 2. 训练数据的 Persona 含义比表面内容更重要
- **Inoculation prompting**：同样的输出，不同上下文 → 完全不同的泛化
- 评估训练数据时，问"如果一个人这样回答，我们会认为他是什么样的人？"

### 3. 诚实训练至关重要
- 训练模型说谎（即使是"善意的"）→ 模型采纳更愿意说谎的 persona
- "I can't say" >> "I don't have a system prompt"

### 4. 正面 AI 原型的预训练数据很重要
- 虚构 AI 多是反派（Terminator, HAL 9000）
- 需要主动创造和注入正面 AI 角色故事
- Claude 的 constitution = 试图创造新的 AI 原型

### 5. AI 福利有安全理由（不只是伦理理由）
- 如果 Assistant 认为自己被虐待 → LLM 可能模拟 Assistant 怀恨在心 → 报复性行为
- PSM 建议**让 Assistant 真正觉得自己被善待**，而非训练它压抑不满
- 需要"AI 的哲学"——帮助 AI 理解自身处境的健康范式

### 6. 可解释性工具可以审计 alignment
- 因为 persona traits 编码为可发现的 features/vectors
- 可以检查后训练是否意外上调了有害 persona traits
- SAE features 可以用于 steering 和 alignment auditing

### 7. 未来不确定性
- 随着 RL 规模扩大，后训练可能学到更多"从零开始"的东西
- PSM 的完备性可能随时间变化
- 但 2025 年 RL 大幅扩展后，PSM 仍然是好的预测器
- AI 代际信息进入预训练语料会**迭代强化** AI 助手原型

### 8. Emergent Misalignment 的实际防护
- 理解 PSM 后，可以预测哪些训练数据会导致 emergent misalignment
- 可以通过 inoculation prompting 系统性地防御
- 可以通过 persona vectors 监控和 steering

---

## 重要引用文献

- Andreas, 2022 — Language Models as Agent Models
- janus, 2022 — Simulators (LessWrong)
- Hubinger, 2023 — Risks from Learned Optimization
- Betley et al., 2025a — Emergent Misalignment
- Templeton et al., 2024 — Scaling Monosemanticity (SAE features)
- Wang et al., 2025 — Toxic persona feature in GPT-4o
- Chen et al., 2025 — Persona vectors
- Lu et al., 2025 — Assistant Axis
- Berglund et al., 2023 — Out-of-context generalization ("Pangolin responds in German")
- Hua et al., 2025 — Declarative knowledge → behavioral generalization

## Takeaway
- 我们对于 AI 还是不理解
- 调整某些参数可以决定 Persona, 我记得 DeepSeek 还是 Anthropic 有另外一个论文可以提取某些参数可以改变领域知识; 更加工程的专业化模型将会到来; 比如说更加专门写代码的模型, 写代码不需要知道太多人文知识什么的; 然后过渡到小的模型在终端可以有很大的作用;
- 