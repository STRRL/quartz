---
title: "SQL Execution Plan 可视化工具：Datadog Explain Plan Visualizer"
source: "https://tanelpoder.com/posts/testing-datadog-plan-visualizer-with-oracle-execution-plans/"
author: "Tanel Poder"
date: 2026-02-28
captured: 2026-03-02
tags: [database, SQL, performance, oracle, tooling]
---

# SQL Execution Plan 可视化工具

## Datadog Explain Plan Visualizer

**URL：** [explain.datadoghq.com](https://explain.datadoghq.com) — 免费，无需 Datadog 账号

**支持数据库：** PostgreSQL、MySQL/MariaDB、SQL Server、MongoDB、Oracle、ClickHouse

### 使用方式
1. 运行提取 SQL（从各数据库的 plan statistics 视图导出 JSON）
2. 粘贴 JSON 到 Web UI
3. Tree view：节点间连线粗细代表行数流量（粗=多行）；List view：各节点 self-cost

### Oracle 提取 SQL Hack（Tanel Poder）
默认显示 optimizer 估算值，但调优真正需要的是**实际值**。将提取 SQL 中：
- `cost` → `last_elapsed_time`（实际运行时间）
- `cardinality` → `last_output_rows`（实际行数）

这样概览图直接显示实际性能数据，而不是 optimizer 的估算（通常严重偏差）。

数据来源：`V$SQL_PLAN_STATISTICS_ALL`

## 开源替代现状

| 工具 | 数据库 | 开源 | 备注 |
|------|--------|------|------|
| [explain.dalibo.com](https://explain.dalibo.com) | PostgreSQL only | ✅ 可自部署 | PG 最常用 |
| [explain.depesz.com](https://explain.depesz.com) | PostgreSQL only | ✅ | 老牌 |
| Supratimas | SQL Server only | ❌ | Web + SSMS 插件 |
| SQL Developer | Oracle only | ❌ | 桌面工具 |
| Datadog explain | 多数据库 | ❌ | **目前最全，且免费** |

**结论：** Oracle + 多数据库 + 纯 Web 可视化这个组合，**开源替代基本没有**。Datadog 这个工具免费无需账号，暂时是最佳选择。

## 相关工具
- Tanel Poder 的 `xb`/`xbi` 脚本：命令行，显示每个 plan node 的实际执行时间和行数，无可视化但信息极其完整

## Takeaway

- 不知道直接发给 opus 能不能解决;
