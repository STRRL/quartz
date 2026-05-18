---
title: "基础设施 / SRE / 性能优化"
date: 2026-03-07
tags: [sre, performance, postgres, rust, systems-thinking, gtd-reference]
week: 2026-W10
---

# 基础设施 / SRE / 性能优化

> 2026-W10 Reference 归档 · 8 篇精选

---

## 🧮 ZZ-638 · SREcon Napkin Math — Simon Eskildsen

**第一性原理估算系统成本和性能**

### 核心基准数字（背下来）

| 资源 | 成本 |
|------|------|
| 1 CPU core | ~$10/月 |
| 1 GB SSD | ~$1/GB/月 |
| 1 GB 内存 | ~$10/GB/月 |

### 关键延迟数字

- **Redis GET**：~25μs（其中 TCP 开销 ~15μs，纯计算只需 ~10μs）
- **TCP 慢启动初始窗口**：15KB（超过 15KB 就触发额外 RTT）
- **随机 SSD vs 随机内存**：差 **4000x**（Big O 分析完全忽略这个差异！）

### Napkin Math 实战

```
10K RPS × 100ms latency = 1000 并发请求
= 1000 CPU cores
= $10,000/月
```

### 核心洞察

Big O 只描述算法形状，**不反映硬件特性**。O(1) 的随机内存访问比 O(1) 的随机 SSD 快 4000 倍——这不是常数，这是系统设计的生死线。

---

## 📊 ZZ-702 · PostgreSQL random_page_cost 实测 — Tomas Vondra

**Postgres 默认配置对 SSD 的严重低估**

### 问题

PostgreSQL 默认 `random_page_cost = 4.0`，这个值是为旋转磁盘设计的。在 SSD 上严重偏低，导致 Query Planner 错误地偏好 Index Scan（认为随机读很贵），实际上 SSD 的随机读远比预设便宜。

### 实测数据

| 操作 | 实测时间 |
|------|---------|
| `seq_page_time` | 1.559 μs |
| `random_page_time` | 39.298 μs |
| 比值 | **~25x** |

→ 因此 `random_page_cost` 应设为 **25–35**（而非默认 4.0）

### 验证方法

设为 `30.0` 后，Query Planner 的 cost 估算与实际执行时间**完美对齐**。

```sql
ALTER SYSTEM SET random_page_cost = 30.0;
SELECT pg_reload_conf();
```

### 影响

这一个参数的错误，会让所有涉及 Index vs Seq Scan 选择的查询走错执行计划。

---

## 🚀 ZZ-786 · PostgreSQL 全面调优

**从默认配置到 +343% TPS 的完整路径**

### 调优阶段对比

| 阶段 | TPS | 提升 |
|------|-----|------|
| 默认配置 | 6,400 | baseline |
| 15个关键参数 | 8,300 | **+30%** |
| 全参数优化 | 22,000 | **+343%** |

### 关键参数（必改）

```ini
shared_buffers = 25% 系统内存   # 最重要的单一参数
random_page_cost = 30.0          # SSD 必改（见 ZZ-702）
work_mem = 根据并发调整
max_connections = 合理上限
checkpoint_completion_target = 0.9
wal_buffers = 64MB
```

### 华为云贝叶斯调参

使用贝叶斯优化自动搜索参数空间，比手工调参更系统。关键发现：**大多数性能提升来自少数几个参数**，长尾参数调优边际收益递减。

### 实操建议

1. 先改 `shared_buffers`（25% RAM）
2. 再改 `random_page_cost`（SSD: 30）
3. 用 `pgbench` 基准测试验证每次变更
4. 监控 `pg_stat_bgwriter` 检查 checkpoint 压力

---

## ⚡ ZZ-691 · P99CONF — 尾延迟优化

**减少线程数、消除锁、理解硬件**

### 核心论点

**线程不是越多越好**——在高并发场景，过多线程带来的上下文切换和锁竞争会**增加** P99 延迟，而非减少。

### 三个关键技术

#### 1. 减少线程数降尾延迟
- 线程池大小应匹配 CPU 物理核数，而非请求并发数
- 排队发生在应用层，而非 OS 调度器层

#### 2. Rust Borrow Checker 的性能优势
- 编译期消除数据竞争，运行时零开销
- 无 GC pause = 更低的 P99/P999

#### 3. ArcSwap 消除锁
- `ArcSwap` 允许无锁地原子替换共享数据（如配置、路由表）
- 读路径完全无锁，适合读多写少场景

### Mechanical Sympathy

> 了解你运行的硬件，才能写出真正快的软件。

结合 ZZ-638 的 Napkin Math：知道 L1 cache hit ~1ns vs RAM ~100ns，才能做出正确的数据结构选择。

---

## 🦀 ZZ-807 · Apache DataFusion

**用 Rust 造数据库的框架**

### 定位

DataFusion **不是数据库**，是**造数据库的工具箱**。它提供：
- SQL 解析 & 执行引擎
- 向量化查询处理
- 可插拔存储层

### 谁在用

| 项目 | 用途 |
|------|------|
| **InfluxDB IOx** | 时序数据库查询引擎 |
| **delta-rs** | Delta Lake 的 Rust 实现 |
| **Ballista** | 分布式查询执行 |

### 扩展方式

实现 `TableProvider` trait，即可让 DataFusion 用 SQL 查询**任意数据源**：

```rust
#[async_trait]
impl TableProvider for MyDataSource {
    async fn scan(&self, ...) -> Result<Arc<dyn ExecutionPlan>> {
        // 返回你的数据
    }
}
```

### 为什么值得关注

如果你在构建分析型工具、数据管道、或需要 SQL 接口的系统，DataFusion 是比从零实现查询引擎**省 90% 工作量**的选择。

---

## 📬 ZZ-727 · Platformatic Job Queue — Matteo Collina

**生产级 Node.js 任务队列的正确实现**

### 核心功能

| 特性 | 说明 |
|------|------|
| **去重（Deduplication）** | 同一 job 不会重复入队 |
| **enqueueAndWait** | 入队并等待结果，像 RPC 一样使用队列 |
| **Stalled Job Reaper** | 自动清理卡死的 job |
| **Graceful Shutdown** | 正在执行的 job 不会被强杀 |
| **TypeScript 原生** | 完整类型支持 |

### 设计哲学（Matteo Collina 风格）

- 队列是基础设施，**可靠性优先**
- 去重和幂等性应该内置，而不是让业务代码处理
- `enqueueAndWait` 填补了"我需要异步处理但又想知道结果"的场景空白

### 适用场景

发邮件、图片处理、第三方 API 调用等需要异步化但又需要结果反馈的操作。

---

## 🧘 ZZ-814 · 避免紧急文化

**好经理的四个特质**

### 为什么紧急文化有害

持续的紧急状态 = 永远在救火 = 没时间建防火墙。团队陷入响应模式，战略工作无法推进。

### 好经理四特质

1. **知道多难**：对工程复杂度有真实感知，不会凭空压 deadline
2. **知道什么重要**：能区分真正的优先级，而非所有事都是"紧急"
3. **有心智模型**：理解系统如何运作，预见问题而非被问题追着跑
4. **在乎团队**：关心团队的可持续性，而非短期产出最大化

### 与系统思维的关系

紧急文化本身是一个**增强回路**（见 ZZ-833）：越救火 → 越没时间预防 → 越多火灾 → 越救火。打破它需要**平衡回路**介入：主动预防、技术债偿还、容量规划。

---

## 🗺️ ZZ-833 · Systems Thinking

**复杂系统的思维工具**

### Rumsfeld Matrix（知识四象限）

|  | 知道自己知道 | 不知道自己知道 |
|--|------------|--------------|
| **知道自己不知道** | Known Knowns | Unknown Knowns |
| **不知道自己不知道** | Known Unknowns | **Unknown Unknowns** |

Unknown Unknowns 是系统性风险的来源——你甚至不知道要去问这个问题。

### Causal Loop Diagrams（因果回路图）

**增强回路（Reinforcing Loop）**：
```
压力增加 → 错误增加 → 更多救火 → 更少预防 → 压力增加 ↺
```

**平衡回路（Balancing Loop）**：
```
技术债增加 → 投入偿还 → 技术债减少 → 投入减少 ↺
```

### 核心原则

> **地图不是领土**（The map is not the territory）

你的架构图、监控 dashboard、指标——都是现实的**模型**，不是现实本身。当模型和现实冲突，相信现实。

---

## 🔗 Cross-Insights

### Postgres 性能三连

```
ZZ-638 Napkin Math
    ↓ 发现 random_page_cost 成本估算应该更高
ZZ-702 random_page_cost 实测（默认 4.0 → 应为 30）
    ↓ 修正后叠加全面调优
ZZ-786 全参数优化（+343% TPS）
```

一个错误的默认值，可能让你的数据库长期跑在 30% 性能水位。

### Mechanical Sympathy 闭环

**ZZ-638**（硬件成本/延迟基准）+ **ZZ-691**（尾延迟 & 减少线程）= 硬件理解是所有性能优化的**地基**。不知道 SSD vs RAM 差 4000x，就不知道为什么要把热数据塞进内存。

### 管理 × 架构

**ZZ-814**（避免紧急文化）需要 **ZZ-833**（系统思维）提供方法论支撑：
- 用因果回路图识别"救火增强回路"
- 用 Rumsfeld Matrix 找 Unknown Unknowns（潜在风险）
- 打破紧急文化，需要管理者有意识地引入**平衡回路**

---

*归档于 2026-W10 · GTD Reference*
