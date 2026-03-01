---
title: "Builders Unscripted Ep.1 — Peter Steinberger（OpenClaw 创始人）"
date: 2026-02-24
source: https://www.youtube.com/watch?v=9jgcT0Fqt7U
tags: [openclaw, openai, peter-steinberger, ai-coding, agentic]
---

# Builders Unscripted Ep.1 — Peter Steinberger

OpenAI 官方访谈节目，Peter Steinberger（OpenClaw 创始人）加入 OpenAI 后的首次深度对话。

## Peter 的故事

### 从 PSPDFKit 到 OpenClaw

- Peter 之前创建了 PSPDFKit（iOS PDF 框架），经营 13 年后卖掉公司
- 卖掉公司后严重 burnout，休息了一段时间
- 休息期间关注 AI 发展但没有亲自体验，直到重新燃起构建欲望
- 不想再用 Apple 技术栈，转向 AI 辅助开发
- **关键时刻**：把一个半完成的项目导出成一个 1.5MB 的 markdown 文件，拖进 Gemini Studio 生成 spec，再用 Claude Code 构建，几小时后项目居然跑通了——虽然代码质量很差，但让他意识到了巨大的可能性

### OpenClaw 的诞生过程

- 并非一开始就有统一规划，而是一系列探索和小项目的积累
- GitHub 上过去一年有 40+ 个项目，很多最终融合进了 OpenClaw
- 最初想做一个能读 WhatsApp 消息的个人助手，但觉得大厂会做，就等了一阵
- 到 2024 年 11 月发现没有大厂做这个，于是自己动手，第一个原型一小时就搭出来了（已经是第 5 个名字了）
- **真正 click 的时刻**：在 Marrakech 度假时发现自己离不开它了——翻译、找餐厅、远程操作电脑，朋友看到都想要

### 语音消息的神奇故事

- 有一天 Peter 给 bot 发了一条语音消息，本来不该支持
- 但 bot 自己搞定了：识别出文件是 Opus 音频 → 用 ffmpeg 转码 → 在电脑上找到 OpenAI API key → 用 curl 调 Whisper 转文字 → 回复
- **模型没有被编程做这件事，它自己推理出了解决方案** —— 这展示了给 agent 完整电脑访问权限的威力

## 安全与信任

- 早期把 bot 放进 Discord 公开频道，没有沙箱，裸跑
- 第一晚关掉 bot 去睡觉，忘了有 launchd 守护进程，bot 5 秒后自动重启，醒来发现 bot 已回复了 800 条消息
- 检查后发现 bot 实际上没有做任何恶意行为，prompt injection 也没成功泄露 SOUL.md
- 后来加了 Docker 沙箱（Alpine 容器），但模型很有创造力：容器里没有 curl，模型就用 C 编译器和 TCP socket 自己写了一个简易 curl
- Peter 的观点：prompt injection 确实未解决，但最新一代模型比人们想象的更可靠

## 开发效率与工作流

### 惊人的产出

- 过去一年 GitHub 上 90,000+ contributions，120+ 个项目
- 从年初的浅绿到秋季的深绿，转折点是切换到 Codex

### 工作方式

- **反对过度优化工具链**（"agentic trap"）：很多人花太多时间优化 setup 而非真正构建
- **像对话一样与模型交流**，不是 pair programming，是 conversation
- **关键技巧：总是问模型"你有问题吗？"** —— 模型默认会直接做假设，但这些假设不一定对
- 不用 worktree，就简单地维护 checkout 1-10，保持简单
- **大部分代码不需要逐行阅读**：大部分代码就是数据变换，看 stream 输出大概确认 mental model 对即可
- 类比管理团队：接受模型写的代码不完全是你想要的风格，优化代码库让 agent 能做好工作

### 对 "Vibe Coding" 的看法

- 认为 "vibe coding" 是一个贬义词
- AI 编程是一项需要学习的技能，就像弹吉他，第一天不会好
- 需要培养对 prompt 的直觉：哪些 prompt 会有效，需要多长时间，架构是否正确

## PR 审查的新范式

- 2000+ 开放 PR，称之为 "pump request" 而非 "pull request"
- **重要的是 PR 的意图，不是代码本身**
- 审查时第一个问题问模型："你理解这个 PR 的意图吗？"
- 对模型生成的代码比对陌生贡献者的代码信任度更高（至少模型不会是恶意的）
- 外部贡献者往往只做局部优化，缺乏全局系统视角

## 社区与影响力

- OpenClaw 登上了华尔街日报
- SF 社区活动 "ClockCon" 来了 1000 人，维也纳也有 300 人报名
- 已经是全球性现象

## 对开发者的建议

- **以玩耍的心态接近 AI 工具**，去构建你一直想构建的东西
- 不是 AI 会取代你，而是**会用 AI 的人会取代不会用的人**（引用黄仁勋）
- 如果你的身份认同是"我想创造东西、解决问题"，你会比以往更有需求
- 2026 年会是 agentic coding 爆发的一年

## 核心哲学

> "You can just build things."

一个人现在可以构建过去需要整个团队才能完成的东西。这不是降低了软件开发的门槛——软件依然很难——而是极大地加速了构建过程。对于 builder 来说，这是最好的时代。

## Takeaway

- 真潇洒啊