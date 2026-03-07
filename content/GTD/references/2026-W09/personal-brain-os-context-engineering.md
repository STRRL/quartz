---
title: "Personal Brain OS — 文件系统驱动的 AI 个人操作系统"
date: 2026-02-28
source: https://x.com/koylanai/status/2025286163641118915
tags: [ai-agents, context-engineering, personal-os, file-system]
---

# Personal Brain OS

作者 Muratcan Koylan（Sully.ai Context Engineer），设计了一个基于 Git 仓库的个人 AI 操作系统。核心理念：Context Engineering > Prompt Engineering。

## 模块架构（实际 7 个，文章声称 11 个）

从 GitHub 仓库 [Agent-Skills-for-Context-Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) 确认的实际模块：

| 模块 | 目录 | 用途 | 文件格式 |
|------|------|------|----------|
| **Identity** | `identity/` | 个人身份、voice guide、brand、anti-patterns、values | Markdown + YAML |
| **Content** | `content/` | 内容创作流水线：ideas、posts、engagement、calendar、templates | JSONL + Markdown |
| **Network** | `network/` | 个人 CRM：contacts、interactions、circles、intros | JSONL + YAML |
| **Operations** | `operations/` | 任务管理：todos、goals、meetings、metrics、reviews | Markdown + YAML + JSONL |
| **Knowledge** | `knowledge/` | 知识库：bookmarks、research、competitors、learning | JSONL + YAML + Markdown |
| **Agents** | `agents/` | 自动化脚本：weekly_review、content_ideas、stale_contacts、idea_to_draft | Python |
| **References** | `references/` | 参考资料：file-formats 等 | Markdown |

## 核心设计原则

### 1. Progressive Disclosure（三层加载）
- **L1 路由层**：永远加载，告诉 AI 该用哪个模块（AGENT.md / SKILL.md）
- **L2 模块层**：按需加载模块指令（CONTENT.md、NETWORK.md 等，40-100 行）
- **L3 数据层**：按需读取实际数据（JSONL 逐行读取，不解析整个文件）
- 最多两跳到达任何信息

### 2. 文件格式选择有理由
- **JSONL**：日志类数据，append-only 防止 AI 覆盖（他丢过 3 个月数据）
- **YAML**：配置类，支持注释，层次清晰
- **Markdown**：叙事类，LLM 原生读取

### 3. Append-Only 是底线
AI 只能追加 JSONL 行，不能覆盖文件。删除用 `"status": "archived"` 标记。

### 4. Voice 编码为结构化数据
5 个属性 1-10 打分（Formal/Casual、Serious/Playful 等）+ 50+ 禁用词列表。比"专业但亲切"这种描述精确得多。

### 5. Skill 分两类
- **Auto-loading**（voice guide 等）：静默注入，不用手动调用
- **Manual**（/write-blog 等）：用户显式触发，slash command

## 踩坑记录
- Schema 字段太多（15+）→ AI 乱填空字段，砍到 8-10 个
- Voice guide 1200 行 → 重要规则放前 100 行（U 型注意力曲线）
- Identity 和 Brand 没拆开 → 加载多余 token，拆开后省 40%

## Takeaway

- Modules
  - User Identity, Agent Identity
  - Content, Reference, Knowledge
  - Network, CRM, Connections
  - Agent, Operations
- FS Alternative
- Fuse/FSKit, with better error messages?
