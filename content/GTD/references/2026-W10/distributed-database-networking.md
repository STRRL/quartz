---
title: "分布式 / 数据库 / 网络"
date: 2026-03-07
tags: [distributed-systems, database, networking, crdt, duckdb, postgres, p2p, rosetta2]
---

# 分布式 / 数据库 / 网络

2026-W10 阅读集，8 篇。三条主线：**分布式一致性理论与实践**、**数据库技术四连**、**底层网络与运行时黑科技**。

---

## 分布式一致性

### ZZ-800 · Peter Bourgon: Consistency Without Consensus

**链接:** https://www.youtube.com/watch?v=em9zLzM8O7c  
**演讲者:** Peter Bourgon（SoundCloud 基础设施工程师，Go 社区知名人物）  
**时长:** 41 分钟

**核心命题：** 不要用 Paxos/Raft 硬解所有分布式问题。CRDT（Conflict-free Replicated Data Types）可以在**完全无共识**的情况下实现可证明的最终一致性。

#### 理论基础

- **CALM 原则**（Consistency as Logical Monotonicity）：系统只朝一个方向增长，天然避免冲突
- **ACID 2.0**（不是传统 ACID）：Associative + Commutative + Idempotent — 满足这三条性质即为 CRDT，可证明最终一致，无需协调

#### SoundCloud Roshi 实战

**播放计数器问题：** 整数加法不幂等 → 重建模为 user ID 集合，Union 做 merge，幂等性自然满足

**Activity Feed (Roshi) 架构 — 4 层：**
1. **Redis 分片** — 底层存储，sorted set + Lua 原子操作
2. **Pool 映射** — 管理分片路由
3. **Cluster** — 暴露 CRDT 语义（修改版 LWW-Element-Set）
4. **Farm** — 多副本 + 写 quorum，读时 fan-in

**读取优化策略：**
- 90% 直接返回第一个 cluster 响应（低延迟）
- 后台 targeted read-repair
- 低频 key 每 8 小时全扫

**开源：** ~2000 行 Go，[github.com/soundcloud/roshi](https://github.com/soundcloud/roshi)

> **金句：** "Sometimes that requires bending the problem rather than trying to hammer it into a square-shaped hole."

**亮点：** 这个演讲的精髓是"重新建模问题"——播放计数不是一个整数，而是一个集合。这种思维方式转换是 CRDT 实用化的关键。

---

### ZZ-564 · Disaggregated OLTP Systems — 存算分离数据库全景

**链接:** https://transactional.blog/notes-on/disaggregated-oltp

云原生时代 OLTP 数据库架构的系统性梳理，覆盖学术界和工业界主要论文。

#### 核心思想：Log is the Database

Aurora 首创的设计哲学：**存储层只需要理解 redo log**，计算层不再负责落盘。这一理念催生了整个存算分离数据库生态。

#### 主要系统对比

| 系统 | 厂商 | 关键指标 |
|------|------|---------|
| **Aurora** | Amazon | 35x TPS 提升，87% IO 减少 |
| **Socrates** | Microsoft (SQL Server) | 四层解耦架构 |
| **PolarDB** | 阿里云 | 共享分布式存储 |
| **TaurusDB / GaussDB** | 华为 | 国内自研存算分离 |

**亮点：** 这篇 Notes 带★标注推荐论文，是进入存算分离 OLTP 领域的最佳导读。

---

#### 跨文连线：ZZ-800 vs ZZ-564

- ZZ-800（CRDT）选择**放弃强一致性**，用数学性质保证收敛 → 适合写多读多、可接受最终一致的场景（feed、计数）
- ZZ-564（存算分离 OLTP）通过**共享日志**维持强一致，用网络换 IO → 适合传统 OLTP 工作负载
- 两者代表分布式系统设计的两条路：relaxed consistency vs shared truth

---

## 数据库四连

### ZZ-569 · Postgres JSONB Columns and TOAST Performance Guide

**链接:** https://www.snowflake.com/en/engineering-blog/postgres-jsonb-columns-and-toast/

深入解析 PostgreSQL JSONB 遇到 TOAST 机制时的性能陷阱。

#### TOAST 机制回顾

PostgreSQL 对超过 ~2KB 的字段自动启用 TOAST（The Oversized-Attribute Storage Technique）：压缩或切片存入侧表（`pg_toast_*`）。每次访问含 TOAST 字段的行都需要额外 IO。

#### "TOAST 税"实测

| 场景 | 查询延迟 |
|------|---------|
| 10K 行，40KB JSON，直接查 | **500ms**（TOAST 税） |
| Generated Column（提取常用字段） | **5ms** ✅ |
| Expression Index | **80ms** |
| GIN Index（精确值匹配） | **23ms** |

#### 三种逃生方案

1. **Generated Columns（最优）** — `ALTER TABLE ... ADD COLUMN field TEXT GENERATED ALWAYS AS (data->>'field') STORED`，物理存储为真实列，查询最快
2. **Expression Index** — 不存数据，但加速 WHERE 过滤
3. **GIN Index** — `jsonb_path_ops`，适合精确包含查询

**核心教训：** 把可预测、高频访问的 JSONB 字段提取为普通列；JSONB 只存真正灵活/稀疏的部分。

---

### ZZ-599 · LLMs Are Good at SQL — Mendral 用 AI 查 TB 级 CI 日志

**链接:** https://www.mendral.com/blog/llms-are-good-at-sql

Mendral 公司让 AI agent 通过自由编写 SQL 查询 ClickHouse 中的 TB 级 CI 日志，实现自动化 flaky test 根因分析。

#### 惊人规模

- **每周 15 亿日志行、70 万 jobs** → 全部进 ClickHouse
- 压缩比 35:1；5.31 TiB 未压缩 → 154 GiB 磁盘（每行含全部元数据仅 **21 字节**！）
- commit_message 字段单独压缩比 **301:1**

#### Agent 工作模式

**8534 次调查，52312 条查询**（平均每次 4.4 条 SQL）：

```
先广后深策略：
job 元数据（便宜，索引好）→ 原始日志行（贵，全文搜索）
```

典型扫描：33.5 万行；P95：9.4 亿行

#### 关键设计洞察

> **SQL > 固定 API** — 给 agent 一个 SQL 接口，而不是 `get_failure_rate()` 这类预设函数。LLM 能问出设计者没预想到的问题。

**ClickHouse 优化技巧：**
- 主键排序（降低扫描范围）
- Bloom filter skip index
- ngram 全文搜索
- 物化视图预聚合常用统计

**反直觉设计：** 每行日志带 **48 列元数据**（完全反规范化），但列式存储 + 压缩让这几乎免费。

**亮点案例：** agent 追踪一个 flaky test 到 3 周前的依赖升级，数百次查询数亿行日志，几秒完成。这就是"给 LLM 结构化数据访问"的威力。

---

### ZZ-623 · SQLRooms — DuckDB-WASM 浏览器端数据分析

基于 DuckDB-WASM 的 React 组件库，让浏览器成为完整的数据分析环境。

**亮点：** 零后端、零服务器，直接在浏览器内运行 DuckDB，查询本地文件（CSV/Parquet）或远程数据源。构建块设计，可嵌入任意 React 应用。

---

### ZZ-617 · DuckDB in Action — Manning 新书

Manning 出版的 DuckDB 实操指南，韩文版已出版。

**亮点：** DuckDB 生态成熟度的标志——有 Manning 级别的书，说明这不再是小众工具。

---

#### 跨文连线：数据库四连

```
ZZ-569（Postgres JSONB）→ 传统 RDBMS 的性能精调
ZZ-599（LLM + ClickHouse）→ 列式 OLAP 的 AI agent 接入
ZZ-623（DuckDB-WASM）→ 浏览器端嵌入式 OLAP
ZZ-617（DuckDB书）→ 嵌入式 OLAP 生态成熟
```

四个维度：调优、AI 接入、端侧部署、学习资源。DuckDB 占了后三席，俨然成为新一代数据分析基础设施。

---

## 底层网络与运行时

### ZZ-637 · libp2p vs WebRTC — P2P 技术选型

深入对比两种 P2P 网络技术。

#### 定位差异

| | WebRTC | libp2p |
|--|--------|--------|
| **层次** | 低级 P2P API（浏览器原生） | 高层模块化 P2P 网络栈 |
| **开发者** | W3C/IETF 标准 | Protocol Labs |
| **传输** | 单一（DTLS/SRTP/SCTP） | 多种（TCP/QUIC/WebRTC/WebTransport） |
| **内置能力** | 音视频 + 数据通道 | Peer Discovery、DHT、PubSub、Relay |
| **适用场景** | 浏览器音视频/数据实时通信 | 构建完整 P2P 网络（IPFS、区块链节点） |

#### 关键关系：不是竞争，是包含

**libp2p 将 WebRTC 作为传输层之一使用**，专门用于：
- 浏览器 ↔ 服务器连接
- 浏览器 ↔ 浏览器连接（无需 CA 证书即可加密，优于 WebSocket）

#### NAT 穿透

- WebRTC：公网 NAT 穿透成功率约 **80%**（STUN/TURN）
- libp2p：有 **relay 机制**作为 fallback，成功率更高

**亮点：** 很多人把两者对立，实际上选择 libp2p 并不排斥 WebRTC——libp2p 在需要时会用 WebRTC 做传输。真正的选择是：直接用 WebRTC API，还是要 libp2p 提供的上层抽象（路由、发现、多协议）。

---

### ZZ-633 · Apple Rosetta 2 Linux VM 揭秘

**链接:** https://blog.inoki.cc/2026/02/28/Apple-Rosetta-Linux-VM-Secret/

ARM64 内核上如何伪装成 x86-64 运行环境——Rosetta 2 在 Linux VM 中的实现黑科技。

#### 架构欺骗三层

1. **系统调用拦截** — `uname()` 返回 `x86_64`，让程序以为自己在 Intel 机器上
2. **硬编码 CPU 信息** — `/proc/cpuinfo` 的 model name、MHz、cache 全是假数据
3. **三层翻译** — 静态翻译（AOT） + 动态翻译（JIT） + 系统调用映射

但有一个漏洞：`/proc/device-tree` 会暴露真实的 `arm-v8` 硬件信息。

#### 实际意义

**OrbStack 和 Docker Desktop 的 `linux/amd64` 支持就是用这套技术实现的。** 你在 M1 Mac 上跑 x86 Docker 镜像时，底层经历了：

```
x86-64 二进制
  → Rosetta 2 翻译（指令集）
  → ARM64 Linux 内核（系统调用被映射）
  → Apple Hypervisor（虚拟化）
  → M 系列芯片（实际执行）
```

**彩蛋：** 这篇文章的作者正在为 libvirt 开发 Virtualization.framework 后端，他的工作会让 Linux 上的 macOS 虚拟化栈更完整。

**亮点：** Rosetta 2 最常被描述为"无缝透明"——这篇文章把黑盒打开了，透明背后是精心设计的系统级欺骗。

---

#### 跨文连线：ZZ-637 + ZZ-633

两篇都在研究"抽象层下面发生了什么"：
- libp2p 把 WebRTC 包装成传输层 → 用户看到的是统一 P2P 接口
- Rosetta 2 把 ARM 伪装成 x86 → 用户看到的是无缝兼容

底层黑科技让上层开发者无感知，这是系统软件最优雅的地方。

---

## 本周总结

| 主题 | 代表条目 | 核心洞察 |
|------|---------|---------|
| CRDT 无共识一致性 | ZZ-800 | 重建模问题，而非强套共识协议 |
| 存算分离 OLTP | ZZ-564 | Log is the database |
| Postgres 性能调优 | ZZ-569 | JSONB + TOAST = 性能陷阱，Generated Column 是出路 |
| LLM + OLAP | ZZ-599 | 给 AI 一个 SQL 接口比固定 API 强 10 倍 |
| DuckDB 生态 | ZZ-623 + ZZ-617 | 嵌入式 OLAP 正在成为标准基础设施 |
| P2P 选型 | ZZ-637 | libp2p ⊃ WebRTC，不是竞争关系 |
| 运行时黑科技 | ZZ-633 | Rosetta 2 靠系统级欺骗实现透明兼容 |


## Takeaway

- Consistency Without Consensus
  - https://github.com/soundcloud/roshi
  - https://www.youtube.com/watch?v=em9zLzM8O7c
- libp2p / webrtc