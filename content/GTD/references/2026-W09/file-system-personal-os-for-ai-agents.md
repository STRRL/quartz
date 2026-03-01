---
title: "The File System Is the New Database: How I Built a Personal OS for AI Agents"
date: 2026-02-24
source: https://x.com/koylanai/status/2025286163641118915
author: Muratcan Koylan (@koylanai)
tags: [context-engineering, ai-agents, personal-os, file-system, knowledge-management]
---

## Summary

Muratcan Koylan (Context Engineer at Sully.ai) built "Personal Brain OS"——一个基于文件系统的个人操作系统，用 Git 仓库存储，让 AI 助手（Cursor/Claude Code）能持久地了解用户的声音、品牌、目标、联系人和工作流程。**核心理念：Context Engineering > Prompt Engineering**——不是写更好的 prompt，而是设计信息架构让 AI 能做出正确决策。

## 核心架构

### 1. Progressive Disclosure（渐进式信息加载）

解决 context window 有限 + attention U-shaped curve 问题：

- **Level 1**: 轻量路由文件（`SKILL.md`），始终加载，告诉 AI 该加载哪个模块
- **Level 2**: 模块级指令（`CONTENT.md`, `NETWORK.md` 等），40-100 行，按需加载
- **Level 3**: 实际数据（JSONL/YAML/研究文档），任务需要时才加载

最多 2 跳到达任何信息。11 个隔离模块，内容任务不加载网络数据，反之亦然。

### 2. Agent Instruction Hierarchy（三层指令体系）

- **Repository 级**: `CLAUDE.md` —— onboarding 文档，AI 工具首先读取
- **Brain 级**: `AGENT.md` —— 7 条核心规则 + 决策表（请求→动作序列映射）
- **Module 级**: 每个目录有自己的指令文件，领域特定的行为约束

好处：消除指令冲突，模块独立更新不影响其他模块。

### 3. 文件格式选择（Format-Function Mapping）

- **JSONL**: 日志类数据（append-only，防止 agent 覆盖历史数据）—— 11 个文件
- **YAML**: 配置类（层级数据 + 注释支持）—— 6 个文件
- **Markdown**: 叙述类（LLM 原生可读）—— 50+ 个文件

每个 JSONL 文件以 schema 行开头：`{"_schema": "contact", "_version": "1.0", "_description": "..."}`

### 4. Episodic Memory（情景记忆）

不只存事实，还存判断：
- `experiences.jsonl`: 关键时刻 + 情感权重 (1-10)
- `decisions.jsonl`: 决策 + 推理 + 替代方案 + 结果追踪
- `failures.jsonl`: 失败 + 根因 + 预防措施

### 5. Skill System（技能系统）

遵循 Anthropic Agent Skills 标准：
- **Auto-loading skills**（`user-invocable: false`）: voice-guide, writing-anti-patterns —— 写作任务自动注入
- **Manual skills**（`disable-model-invocation: true`）: `/write-blog`, `/topic-research` —— slash command 触发

Voice System: 5 维属性 1-10 评分 + 50+ banned words（三级）+ 反模式列表。比形容词描述精确得多。

### 6. 日常运行

- **Content Pipeline**: Idea → Research → Outline → Draft(4轮编辑) → Publish → Promote
- **Personal CRM**: 4 层联系人圈（weekly/bi-weekly/monthly/quarterly）+ `can_help_with` 字段做 intro matching
- **Automation Chains**: 周日 weekly review = metrics_snapshot → stale_contacts → weekly_review，形成反馈循环

## 关键教训

1. **Schema 要简洁**: 初始 15+ 字段太多，减到 8-10 个必要字段，agent 表现更好
2. **Voice guide 要前置重点**: 1200 行太长会 lost-in-middle，关键规则放前 100 行
3. **模块边界要精确**: identity 和 brand 拆分后，voice-only 任务 token 减少 40%
4. **Append-only 不可妥协**: 曾因 agent 重写 JSONL 丢失 3 个月数据

## 关键引用

> "Context engineering asks 'what information does this AI need to make the right decision, and how do I structure that information so the model actually uses it?'"

## 相关资源

- GitHub: https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering (8000+ stars)
- 作者: Muratcan Koylan, Context Engineer at Sully.ai
