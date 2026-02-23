---
title: "Effective Context Engineering for AI Agents"
date: 2026-02-20
source: "https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents"
tags: [ai, agents, context-engineering, llm, anthropic]
---

# Effective Context Engineering for AI Agents

Anthropic 官方工程博客，讲 AI agent 的 context engineering——prompt engineering 的自然演进。

## 核心观点

Context engineering = 在有限 context window 里放最优的 token 集合，最大化期望结果。

## 关键概念

- **Context Rot**: token 越多，模型回忆精确信息的能力越差。边际收益递减。
- **Attention Budget**: transformer 的 n² 注意力机制，context 越长注意力越分散。

## 有效 Context 的组成

- **System Prompt**: 找对高度，不能太死板（if-else）也不能太笼统
- **工具设计**: 精简、功能不重叠、描述清晰
- **Few-shot 示例**: 精选代表性的，不要塞一堆边界情况

## Context 检索策略

- **传统**: 预先 embedding 检索塞进 context
- **Just-in-time**: 只保存轻量级引用（文件路径、查询、链接），需要时动态加载
- **Progressive Disclosure**: 逐层发现，每次只保留需要的
- **混合策略**: 重要的预加载 + 其余按需探索（Claude Code 的做法）

## 长任务三大技巧

1. **Compaction**: 快到上限时总结压缩，保留关键决策，丢掉多余工具输出
2. **结构化笔记**: agent 主动写笔记到文件，context reset 后读笔记继续
3. **多 Agent 架构**: 主 agent 协调，子 agent 做深度任务，只返回精炼总结

## Links

- [原文](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
