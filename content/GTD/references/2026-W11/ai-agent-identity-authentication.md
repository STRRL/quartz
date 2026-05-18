---
title: "AI Agent 身份认证：标准化进展与行业动态"
date: 2026-03-10
tags: [ai-agent, authentication, oauth, identity, spiffe, oidc, security]
---

# AI Agent 身份认证：标准化进展与行业动态

## 背景

AI Agent 的能力已经足够强大，但认证基础设施还是为人类设计的。当前的 workaround（Cookie Syncing、1Password 集成、TOTP 自动生成、AgentMail 等）本质上是让 agent 冒充人类。真正的解法是：**给 agent 自己的身份**。

ZZ 的判断：大概率是 OAuth 上的扩展，类似 SPIFFE/SPIRE 的工作负载身份模式。这个判断与行业方向完全一致。

## 关键共识

- Agent 是**第三类身份**（不是 user，不是 client/NHI）
- 基于 **OAuth 2.0 扩展**，不重新发明轮子
- **WIMSE/SPIFFE** 式工作负载身份是基础
- **Delegation Chain** 是核心概念 — 谁授权了 agent，agent 又授权了谁
- OIDC-A 定义了 Agent Provider、Agent Model、Agent Instance 分层

## 标准化工作

### 1. IETF: AI Agent Authentication and Authorization

- **文档:** [draft-klrc-aiagent-auth-00](https://datatracker.ietf.org/doc/draft-klrc-aiagent-auth/)
- **作者:** Kasselman (Defakto Security), Lombardo (AWS), Rosomakho (Zscaler), Campbell (Ping Identity)
- **日期:** 2026-03-02
- **要点:**
  - 基于 WIMSE (Workload Identity in Multi-System Environments) + OAuth 2.0
  - WIMSE 就是 SPIFFE 的 IETF 标准化版本
  - 不定义新协议，描述如何应用/扩展现有标准
  - 识别 gap 并指导未来标准化
- **GitHub:** https://github.com/PieterKas/agent2agent-auth-framework

### 2. IETF: Agent Auth Considerations (ACN)

- **文档:** [draft-yao-agent-auth-considerations-01](https://www.ietf.org/archive/id/draft-yao-agent-auth-considerations-01.html)
- **作者:** Yao
- **日期:** 2025-10
- **要点:**
  - 提出 Agent Communication Network (ACN) 概念
  - 扩展 OAuth 模型，为 AI agent 定义新的认证授权工作流
  - 侧重可扩展和可信的 ACN 基础设施

### 3. NIST Concept Paper

- **文档:** [PDF](https://www.nccoe.nist.gov/sites/default/files/2026-02/accelerating-the-adoption-of-software-and-ai-agent-identity-and-authorization-concept-paper.pdf)
- **组织:** NIST NCCoE
- **日期:** 2026-02
- **要点:**
  - 标题: "Accelerating the Adoption of Software and AI Agent Identity and Authorization"
  - 基于 OAuth 2.0 框架
  - **征求意见截止: 2026-04-02** (AI-Identity@nist.gov)

### 4. OpenID Connect for Agents (OIDC-A) 1.0

- **文档:** [subramanya.ai](https://subramanya.ai/2025/04/28/oidc-a-proposal/)
- **作者:** Subramanya N
- **日期:** 2025-04-28
- **要点:**
  - OIDC 的 agent 扩展，agent 作为 OAuth 生态中的一等公民
  - 定义标准 claims、endpoints、protocols
  - 核心概念:
    - **Agent:** 基于 LLM 的自主/半自主软件实体
    - **Agent Provider:** 创建/训练/托管 agent 的组织
    - **Agent Model:** 驱动 agent 的具体 LLM（如 GPT-4, Claude 3）
    - **Agent Instance:** agent 的具体运行实例
    - **Delegator:** 授权 agent 代行的实体（通常是人类）
    - **Delegation Chain:** 授权链

## 企业/产品

### 5. Microsoft Entra

- **来源:** [Microsoft Entra Blog](https://techcommunity.microsoft.com/blog/microsoft-entra-blog/the-future-of-ai-agents%E2%80%94and-why-oauth-must-evolve/3827391)
- **日期:** 2025-05-30
- **要点:**
  - "Agent IDs as first-class actors" — agent 注册时应该注册为 agent 而不是 client
  - 12-24 个月内 OAuth 必须演进
  - 已发布 Conditional Access Optimizer Agent、SWE Agent、SRE Agent
  - 预见 "an emerging flood of rich, smart, multi-functional, and autonomous agents"

### 6. WSO2

- **来源:** [wso2.com](https://wso2.com/library/blogs/why-ai-agents-need-their-own-identity-lessons-from-2025-and-resolutions-for-2026/)
- **日期:** 2025-12-23
- **要点:**
  - "Make 2026 the year you get agent security right"
  - Agent 需要独立的生命周期管理、认证机制、授权策略、审计追踪

### 7. Strata Maverics

- **来源:** [strata.io](https://www.strata.io/blog/agentic-identity/8-strategies-for-ai-agent-security-in-2025/)
- **日期:** 2026-01-12
- **要点:**
  - Enterprise Identity Orchestration Layer for Agentic AI
  - 统一 human / machine / autonomous agent 的 identity、policy、audit
  - Agent ≠ Non-Human Identity (NHI)，需要新的 playbook

### 8. Composio

- **来源:** [composio.dev](https://composio.dev/blog/ai-agent-authentication-platforms)
- **要点:** AI agent 认证平台 buyer's guide，已有市场对比

## 分析文章

### 9. ISACA: The Looming Authorization Crisis

- **来源:** [isaca.org](https://www.isaca.org/resources/news-and-trends/industry-news/2025/the-looming-authorization-crisis-why-traditional-iam-fails-agentic-ai)
- **要点:**
  - OAuth 2.0 token exchange (on-behalf-of) — 加密绑定 actor (agent) 和 delegator
  - OpenID AuthZEN — 细粒度、上下文感知的策略（geo-fence、budget threshold）
  - MCP — AI 工具链中的结构化通信，让 agent 学习组织策略
  - 三件套: Token Exchange + AuthZEN + MCP

## ZZ 的判断

- 当前所有 agent 认证方案（Cookie Sync 等）都是 workaround
- 真正的方向: **OAuth 扩展 + SPIFFE/SPIRE 式工作负载身份**
- Better Auth 可能会切入这个方向（已在做 auth infra，agent identity 是自然延伸）
- **NIST 征求意见截止 2026-04-02** — 如果有兴趣参与标准制定，这是窗口期

## 关联

- 🔗 ZZ-873 Browser Use Agent Auth（当前 workaround 的代表）
- 🔗 ZZ-860 Levie: API-first or die（agent 作为 API 消费者的身份问题）
- 🔗 ZZ-882 Stakpak Secret Substitution（另一种安全思路：不给 agent 看凭证）
