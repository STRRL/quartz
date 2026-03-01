---
title: "Oops, You Wrote a Database"
date: 2026-02-24
source: https://dx.tips/oops-database
author: swyx (DX.Tips)
shared_by: Jeffrey Huber (Chroma 创始人)
tags: [databases, architecture, software-engineering]
---

## 概述

swyx 以讽刺信件的形式，论证了一个常见现象：开发者为了避免使用"笨重"的数据库（如 Postgres），从简单的 KV 存储或 JSON 文件开始，最终在应用层重新实现了数据库的所有核心功能。灵感来自 "Dear sir, you have built a compiler" 一文。

## 核心观点

**"Those who fail to study databases are doomed to write them."**

当你说 "Postgres is overkill" 时，你可能短期快了 30%，但长期每个团队成员每月慢 30%。

## 论证路径：逐步重新发明数据库

文章按时间线展示了一个团队如何一步步"被迫"重建数据库功能：

1. **Schema（模式）**：数据实体和属性越来越多，需要记录"哪些东西存在、它们有什么字段、值的范围"→ 你写了一个 schema
2. **Transactions（事务）**：多用户/多应用访问同一数据出现不一致 → 加了 `Pending`/`Complete` 状态 → 你写了一个事务管理器
3. **WAL（预写日志）**：把 Pending 更新分离到单独的 "log" 防止崩溃丢数据 → 你写了 WAL
4. **CDC/Triggers（变更数据捕获）**：数据更新时触发其他操作的 "hook" 系统 → 你写了 stored procedures / CDC
5. **Query Language（查询语言）**：把常用 CRUD 操作封装成可猜测、语法一致的 API，甚至暴露给终端用户 → 你写了查询语言
6. **Caching & Indexing（缓存和索引）**：
   - 重复查询 → memoize → 缓存
   - 可预测的热查询 → 预计算 → 物化视图
   - 不可预测的嵌套依赖 → 图+缓存层（如 Dataloader）
7. **Query Planning（查询规划）**：慢写入 → 读写分离、第三范式 → 查询优化
8. **Security / Auth（安全/授权）**：企业客户要求文档权限、API 安全、数据隔离、数据驻留
9. **Recovery & Audit（恢复和审计）**：回滚、时间旅行调试、操作日志

## 提到的例子和引用

- **Reddit has two tables**：极端的反规范化案例
- **Dan Pritchett (eBay) BASE 论文**：最终一致性
- **tRPC / Apollo GraphQL**：类型安全和 codegen 工具
- **Dataloader (Lee Byron)**：解决 N+1 查询的批处理/缓存层
- **Designing Data Intensive Applications (DDIA)**：团队读书会参考
- **Third Normal Form**：数据库规范化

## 关键启示

这篇文章对于做 infra/平台工具的人特别有价值：

1. **数据库不仅是存储，而是一整套数据管理原语**（schema、事务、恢复、安全等）
2. **"简单"方案的总成本往往高于"复杂"方案**：短期快，长期债务累积
3. **Greenspun 定律的数据库版**：任何足够复杂的应用都包含一个临时的、不完整的、有 bug 的数据库实现
4. 对 Chroma（向量数据库）的创始人来说，这篇文章是在为"为什么需要专业数据库"做论证

## 相关链接

- [Dear sir, you have built a compiler](https://rachitnigam.com/post/you-have-built-a-compiler/) - 灵感来源
- [Stop Building Databases](https://sqlsync.dev/posts/stop-building-databases/) - 相关文章
- [Hacker News 讨论](https://news.ycombinator.com/item?id=34941650)

## Takeaway

- no silver bullet
- trade off