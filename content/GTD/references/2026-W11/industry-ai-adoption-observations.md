# 行业采用 AI 观察

创建时间：2026-03-11

## 核心判断
这一组材料的共同主线是：**AI 正在把生产代码、文档、分析初稿的成本迅速打低，而真正稀缺的东西重新变成 judgment、review、context 与 workflow integration。**

它不是简单的“AI 替代所有人”，而更像：
- 先出现 **AI-native 个体 / AI-native 团队** 的显著优势
- 再推动组织流程、岗位分工、行业定价逻辑的重排
- 最后才演化成更深层的市场结构变化

## 1. Zack Shapiro — The 10x Lawyer
- 原链接：
  - https://x.com/zackbshapiro/status/2031717962948690355?s=46
  - https://x.com/i/article/2030690000128364544
- 作者：Zack Shapiro

### 观点
Zack Shapiro 认为，AI 不会先以“法律 SaaS 替代律师”的方式重塑法律行业，而是先把市场重新按**个人判断力**排序。真正被放大的不是打字速度，而是资深律师的 judgment。

### 具体例子
- 他给出 BigLaw 的杠杆模型：Am Law 50 leverage ratio 从 **1985 年约 1.76** 上升到 **约 3.52**，Am Law 100 **2024 年平均 equity partner 利润约 315 万美元**。
- 传统模式下，一个复杂项目依赖 partner + senior associate + mid-level + junior 组成的人力金字塔，associate 常见计费范围 **600-1600 美元/小时**。
- 他的 Series B 例子很具体：**6 份协议、80-100 页文本**，以前通常需要四五人团队协作，最后账单 **7.5 万美元以上**。
- 在新模式下，一个资深律师加 AI 就能把全套协议塞进 context window，直接看交叉条款、找不一致、出完整 markup 和策略 memo。

### 结论
他最重要的结论不是“软件替代律师”，而是：**更会用 AI、判断力更强的律师，会横向吃掉普通律师和低效团队的份额。**

### 我自己的备注
这条值得长期保留，因为它把“AI 放大 judgment、而不是简单自动化 labour”的逻辑讲得很完整，也给了足够具体的行业数字。

## 2. Harrison Chase — How Coding Agents Are Reshaping Engineering, Product and Design
- 原链接：
  - https://x.com/hwchase17/status/2031051115169808685?s=46
  - https://x.com/i/article/2031049457031331841
- 作者：Harrison Chase

### 观点
Harrison Chase 认为，coding agents 让软件实现成本急剧下降，导致老的 **PRD → mock → code** 流程正在失效。真正稀缺的能力从“谁来产出第一版”转向“谁来 review、裁决方向、保证质量”。

### 具体例子
- 他明确说 bottleneck 从 **implementation 转向 review**。
- Review 的三个核心维度是：
  - engineering quality：是否 scalable / performant / robust
  - product correctness：是否真的解决用户痛点
  - design quality：是否直观、好用
- 他认为 generalist 更有杠杆，因为 agent 降低了跨职能 handoff 的必要性。
- 组织形态会越来越像 **builders + reviewers** 两类人：前者快速产出，后者负责架构、产品和设计层面的高价值判断。

### 结论
这条最有价值的点不是“大家都去 vibe coding”，而是：**代码一旦变便宜，组织竞争力就会转向意图设定、评审速度、系统思维与跨职能 judgment。**

### 我自己的备注
这条是软件组织视角最清晰的一篇。对于理解 Engineering / Product / Design 在 agent 时代怎么重新分工，很有参考价值。

## 3. ro — The Case For AI Native Companies
- 原链接：
  - https://x.com/MilosOfCroton/status/2031517109537366342?s=20
- 作者：ro

### 观点
ro 的中心判断是：AI-native 公司真正的护城河，不是模型本身，而是公司内部给 agent 用的那层**context / authorization / action infrastructure**。

### 具体例子
- 他把这层东西类比成：
  - Jeff Huber 的 **context layer**
  - Palantir 的 **ontology**
  - 也可以理解成企业内部的 AI operating system
- 他把上下文分成两类：
  - **显式上下文**：文档、财务、销售、HR、工程、产品、法务等系统记录
  - **隐式上下文**：公司真正如何运转，比如谁会拒绝没有 rollback plan 的方案、CEO 更爱 Slack 还是 email、哪场会才是真正拍板的地方
- 他还强调：**agent 表现好坏高度依赖 context 质量**。要让 agent 失败，只要给它差上下文或者不给上下文。
- 他认为 OpenClaw 这类系统“方向是对的”，因为它具备 stateful memory、skills、feedback learning、广泛系统访问能力。

### 结论
真正有复利的不是“企业开始用 AI”，而是**谁先把 agent 深度接进公司的真实系统、流程、权限与上下文里。**

### 我自己的备注
这条最有意思的是“隐式上下文”这个视角：企业竞争力不只是把数据接给 agent，而是把组织真实运作方式也逐渐结构化。这个判断很像企业 agent 真正落地时的关键层。

## 4. Aditya Agarwal — When Your Life’s Work Becomes Free and Abundant
- 原链接：
  - https://x.com/adityaag/status/2031396465063436395?s=20
  - http://x.com/i/article/2031368394126274565
- 作者：Aditya Agarwal（前 Dropbox CTO、早期 Facebook engineer、South Park Commons GP）

### 观点
Aditya 的核心观点是：代码生成正在变得“免费且充足”，真正稀缺的东西从多年经验本身，转向**适应变化的速度、builder instinct、以及是否主动拥抱新工具。**

### 具体例子
- 他自己说，在一个周末用 Claude 写代码之后，意识到：“**We will never write code by hand again.**”
- 他还说，之后 **5 天写的代码比过去 5 年还多**，而且做出来的东西更 ambitious、更好。
- South Park Commons 的一个案例里，约 **20 场 engineering work trials** 几乎看不出“经验年限”和“AI 适应能力”的相关性。
- 另外一个具体 hiring 方法是：给候选人一个**手写根本做不完**的任务，看他是否会自然地把 AI 当成日常工具。

### 结论
Agarwal 的判断不是“年轻人更占优”，而是：**真正有优势的是会主动朝变化跑的人。** Builder 行为、side project、对工具的好奇心，比 pedigree 更重要。

### 我自己的备注
这条是这组里“职业与人才观变化”最值得留的一条，因为它来自一个技术出身、又做过管理和投资的人，情绪和判断都很真实。

## 这一组的共同结论
如果把这四条放在一起，能看到一条很统一的结构：

1. **生产成本被 AI 快速压低**
   - 代码、文档、分析、初稿越来越便宜
2. **稀缺性回到 judgment / review / context**
   - 真正重要的变成谁更会判断、谁更会审、谁更懂业务上下文
3. **行业先重排，再替代**
   - 先出现 10x 律师、10x builder、AI-native 组织，再出现更大的市场结构变化
4. **企业护城河从模型迁移到 workflow + infra + context layer**
   - 谁更能把 AI 接进真实流程，谁更容易获得复利
5. **人才信号从 pedigree 转向 adaptability**
   - 经历本身不再天然有价值，会不会快速吸收新工具更关键

## 对我自己的启发
这一组材料说明，未来更值得持续观察的不是单个模型 benchmark，而是：
- 哪些岗位的 review / judgment 被显著放大
- 哪些行业正在发生“AI-native practitioner 吃掉传统团队”的重排
- 哪些公司在构建真正的 context / action layer
- 哪些组织把 AI 从“玩具”变成了一套真实 workflow
