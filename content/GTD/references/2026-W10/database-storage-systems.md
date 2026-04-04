---
title: "数据库与存储系统 — 2026-W10 Reference"
date: 2026-03-08
tags: [database, postgresql, storage, etcd, ml-inference, architecture]
---

# 数据库与存储系统

> 2026-W10 整理，共 8 条 Reference。涵盖数据库架构理论、PostgreSQL 深度实践、存储底层原理、分布式存储优化等主题。

---

## 1. One Size Fits All — Stonebraker (ZZ-854)

**来源:** CIDR 2005 + CIDR 2007 | [论文 PDF](https://cs.brown.edu/~ugur/fits_all.pdf)
**作者:** Michael Stonebraker, Stavros Harizopoulos, Daniel Abadi, Samuel Madden

### 核心论点

Stonebraker 在 2005 年大胆预言：**通用 RDBMS 无法满足所有工作负载**，数据库市场将分裂为一系列专用引擎。专用系统在特定场景下比通用系统快 **10–100x**。

三大不适用场景：

1. **OLAP / 数据仓库** — 行存储对分析查询效率极低，列式存储（C-Store/Vertica）可带来数量级提升
2. **流处理** — 传统 DBMS 是"数据先存、后查"，流处理需要"查询先注册、数据流过"，架构根本不同（StreamBase）
3. **超大规模文档/文本** — Google 自建 GFS + BigTable 而非用 RDBMS，说明互联网规模需要专用系统

**10x 门槛原则：** 由于惯性，专用数据库必须比通用 RDBMS 快至少 10x 才能获得市场采纳。Part 2 用 benchmark 证明多个场景确实存在 10–100x 的性能差距。

### 历史影响

这篇论文**精准预测**了后来 20 年的行业趋势：列式数据库（Vertica, ClickHouse）、流处理（Kafka Streams, Flink）、文档数据库（MongoDB）、图数据库（Neo4j）等专用系统的兴起。Stonebraker 本人也创办了多家专用数据库公司（Ingres, Illustra, StreamBase, Vertica, VoltDB）。

---

## 2. Postgres is Too Good (ZZ-855)

**来源:** [dev.to/shayy](https://dev.to/shayy/postgres-is-too-good-and-why-thats-actually-a-problem-4imc)
**作者:** Shay（UserJot 创始人）

### 核心论点

Postgres 能替代 Redis（KV/队列）、Elasticsearch（全文搜索）、RabbitMQ（消息队列）、MongoDB，**90% 的应用根本不需要这些中间件**。

| 场景 | Postgres 实现 | 替代的组件 |
|------|--------------|----------|
| Job Queue | `FOR UPDATE SKIP LOCKED` | Redis / RabbitMQ |
| KV 存储 | JSONB + GIN 索引 + `@>` | Redis |
| 全文搜索 | `tsvector` + `ts_rank` | Elasticsearch |
| 实时推送 | `LISTEN/NOTIFY` + 触发器 | Redis pub/sub |

**成本对比：** 典型"现代"栈 Redis+MQ+Search+监控 ≈ **$125/月**，还不算运维复杂度；Postgres 一个服务全搞定。

**规模验证：** Instagram 1400万用户单实例、Discord 数十亿消息、Airbnb/Robinhood/GitLab 都用 Postgres。

**何时该上专用工具：** 10万+/分钟 jobs、亚毫秒缓存、TB 级分析、百万并发时再考虑。

---

## 3. PostgresqlCO.NF — PG 配置参数文档 (ZZ-856)

**来源:** [postgresqlco.nf](https://postgresqlco.nf) | 维护方: OnGres

### 工具价值

免费的 PostgreSQL 配置参数人类可读文档，覆盖约 **290 个 postgresql.conf 参数**，比官方文档更易读：

- 按类别分组（内存、WAL、查询优化、连接、日志等）
- 多版本支持，可对比不同 PG 版本参数变化
- 多语言：英语、日语、俄语、中文、法语
- 社区推荐值：可查看他人推荐值或分享自己的
- 博客集成：文章中提到的参数自动高亮并显示悬浮信息
- 永久免费

**使用场景：** 调优 PostgreSQL 时快速查阅参数含义、默认值、推荐值。

---

## 4. ML 推理 from PostgreSQL (ZZ-496)

**来源:** [推文](https://x.com/tom_doerr/status/2025268920907612456)

### 核心价值

在 PostgreSQL 数据库内部直接运行机器学习推理，**消除数据搬运开销**：

- 传统方案：数据从 DB → 推理服务 → 结果写回，多次网络 I/O
- 新方案：推理在 DB 内部完成，数据不离开数据库

与 ZZ-459（Database Skills AI Agent 工具）和 ZZ-476（PG Buffer 深度解析）共同构成 PostgreSQL 扩展能力图景。

---

## 5. PostgreSQL Buffer 深度解析 (ZZ-476)

**来源:** [boringsql.com](https://boringsql.com/posts/introduction-to-buffers/)

### 核心概念

- **8KB page** 是 PostgreSQL I/O 原子单位
- **PG 自管缓存 vs OS 页缓存**：PG 维护自己的 shared_buffers，与操作系统页缓存并存（double buffering 问题）
- **shared_buffers 结构**：固定大小的 buffer pool，LRU-like 置换策略
- **Ring Buffer**：顺序扫描时使用小型 ring buffer，防止大表扫描污染整个缓存（cache pollution prevention）

**配套工具：**
- RegreSQL — PostgreSQL 回归测试工具
- SQL Labs — 实操学习平台

---

## 6. Database Skills — PlanetScale 的 AI Agent 数据库工具 (ZZ-459)

**来源:** [database-skills.com](https://database-skills.com/)
**出品:** PlanetScale

### 产品定位

AI Agent 的数据库 Skills，支持 Postgres / MySQL / Vitess / Neon，兼容主流 AI 编程工具：

- Cursor
- Claude Code
- Codex（OpenAI）

让 AI Agent 能够直接理解和操作数据库结构，降低 LLM 操作数据库的门槛。

---

## 7. 蚂蚁集团 ETCD NPKV 优化 (ZZ-261)

**来源:** [微信公众号](https://mp.weixin.qq.com/s/bPPSJHyOnnmUb0GEbX5VCQ)
**出品:** 蚂蚁集团

### 优化效果

通过关闭 Watch `PrevKV` 参数：

| 指标 | 优化前 | 优化后 | 改善 |
|------|--------|--------|------|
| Watch 延迟 P99 | 15ms | 1ms | **↓ 94%** |
| ETCD 吞吐 QPS | baseline | +29% | **↑ 29%** |

**技术细节：**
- `PrevKV=true` 会在每次 watch 事件中附带前一个版本的完整 KV 数据，产生大量额外序列化和传输开销
- 关闭后，通过 `reverseKeyFunc` 从 ETCD Key 提取 Name/Namespace 重建墓碑（tombstone）对象，无需 PrevKV 数据
- 相关 PR: [kubernetes/kubernetes#131862](https://github.com/kubernetes/kubernetes/pull/131862)

**适用场景：** 大规模 Kubernetes 集群中 ETCD 性能调优的实战经验，对自建 K8s 基础设施的团队有直接参考价值。

---

## 8. 数据存储原理 — 文件系统、存储引擎、磁盘 I/O (ZZ-377)

**来源:** [makingsoftware.com](https://www.makingsoftware.com/chapters/how-is-data-stored)
**系列:** Making Software

### 内容范围

深入讲解数据在计算机中如何被存储的底层机制：

- **文件系统** — inode、目录结构、块分配
- **数据库存储引擎** — heap files、B-Tree 索引、LSM-Tree
- **磁盘 I/O** — 顺序 vs 随机 I/O、page cache、fsync 语义

适合作为理解数据库内核（PG Buffer、ETCD 存储）的基础知识补充，与本组其他条目形成知识体系闭环。

---

## 主题关联图

```
存储底层原理 (ZZ-377)
    └── PostgreSQL Buffer (ZZ-476)
            ├── PG 配置调优 (ZZ-856)
            ├── PG 能力边界 (ZZ-855)
            │       └── ML 推理 in PG (ZZ-496)
            └── AI Agent 数据库工具 (ZZ-459)

数据库架构哲学 (ZZ-854: Stonebraker)
    └── 专用 vs 通用的权衡

分布式存储优化 (ZZ-261: ETCD)
```

---

*Reference 归档时间: 2026-03-08 | GTD 状态: Reference*
