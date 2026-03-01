---
title: "为什么 etcd 在大规模 Kubernetes 集群中会成为瓶颈"
date: 2026-02-24
tags: ["kubernetes", "etcd", "scalability", "distributed-systems"]
---

# 为什么 etcd 在大规模 Kubernetes 集群中会成为瓶颈

> 参考来源：[Learnk8s - Why etcd breaks at scale](https://learnkube.com/etcd-breaks-at-scale)、[How etcd works in Kubernetes](https://dev.to/danielepolencic/how-etcd-works-in-kubernetes-373l) by @danielepolencic、AWS EKS 技术博客、Google GKE 博客、OpenAI 博客、阿里云容器平台博客等。

## 背景

在 Kubernetes 中，只有 API server 直接与 etcd 通信。scheduler、controller manager、kubelet、kubectl 和所有 operator 都通过 API server 间接访问数据。etcd 是 API server 的私有后端数据库。

对于单节点 API server，理论上你可以用 SQLite、PostgreSQL 甚至文件来存储集群状态。但生产集群需要高可用——多个 API server 共享一致的状态——这就是 etcd 存在的理由。

---

## 一、etcd 内部机制与瓶颈

### 1.1 Raft 共识：单 Leader 瓶颈

etcd 使用 [Raft 共识算法](https://raft.github.io/)，核心特性：

- **单 Leader 写入**：所有写请求无论发送到哪个节点，都会转发给 Leader
- **多数派确认**：Leader 将写入追加到日志，复制到 follower，等待多数派（quorum）确认后才提交
- **写延迟下界**：至少一次到 follower 的网络 RTT + 每个节点的 `fdatasync`

**关键限制：**

| 问题 | 说明 |
|------|------|
| **写吞吐受限于单节点** | 增加 etcd 节点不会提升写容量，反而更慢（更多 follower 需要复制） |
| **Leader 网络/CPU 饱和** | 高写入 + 大量 watch 事件分发时，Leader 成为集群瓶颈 |
| **Log replication 延迟** | follower 落后太多时需要发送全量 snapshot，期间 Leader 性能下降 |
| **选举中断** | Leader 崩溃后的选举期间，集群短暂不可用 |

生产建议：etcd 集群通常 3 或 5 节点。增加到 7+ 节点带来的可用性收益不值得写性能的下降。

### 1.2 BoltDB/bbolt：单文件存储的限制

etcd 使用 [bbolt](https://github.com/etcd-io/bbolt)（B+ tree key-value store），数据存储在**单个文件**中：

| 限制 | 数值 |
|------|------|
| 推荐最大数据库大小 | **8 GiB**（etcd 官方） |
| 默认 backend quota | **2 GiB** |
| 单个 value 上限 | **1.5 MiB/请求** |
| 单个 key-value 上限 | **1 MiB** |

**为什么这么严格？** 因为 Raft 复制机制：当 follower 落后太远或新节点加入时，Leader 需要发送整个数据库的 snapshot。8 GiB 的 snapshot 传输时间长，恢复慢，风险高。

**阿里巴巴的发现**：bbolt 的空闲页管理使用线性搜索算法，当数据量增大时，页分配性能急剧下降。阿里团队设计了基于 segregated hashmap 的空闲页管理算法，将 etcd 存储容量从推荐的 2 GB 提升到 100 GB，并已贡献到社区（etcd 3.4+）。

### 1.3 MVCC revision 堆积

etcd 使用**多版本并发控制（MVCC）**：

- 每次写入不覆盖旧值，而是创建整个数据集的**新 revision**
- Kubernetes 的 `resourceVersion` 就是 etcd 的 revision
- controller 使用 revision 来 watch 变更、恢复 watch、检测冲突

**问题**：旧 revision 不会自动消失。如果有 10,000 个 pod，每分钟更新一次，就是每分钟 10,000 个新 revision 堆积。

### 1.4 Compaction 与 Defragmentation

- **Compaction**：丢弃某个时间点之前的所有旧 revision。不做 compaction，数据库单调递增
- **Defragmentation**：compaction 后空间标记为可重用但**文件不会缩小**（bbolt 的 copy-on-write 页机制），需要 defrag 重建文件回收空间

**灾难场景**：如果 mutation rate 超过 compaction 速度 → 数据库超过 quota → etcd 进入 **alarm mode** → 只读 → **整个 Kubernetes 控制平面停止接受变更**，无法创建新 pod、无法 scale、无法 deploy。

---

## 二、Watch 机制的扩展性问题

### 2.1 Watch 基本原理

Kubernetes controller 不轮询 API server，而是建立长连接（watch），接收对象变更事件流。API server 维护到 etcd 的 watch 连接，当 key 变化时，etcd 将事件流式发送给所有订阅者。

**扩展性问题**：每次 pod status 更新，etcd 需要评估哪些 watcher 关心这个 key 并发送事件。对象数 × controller 数 × namespace 数 = Leader 每次写入的额外工作量。大规模下，**Leader 花在分发 watch 事件上的时间可能超过处理写入的时间**。

### 2.2 API Server Watch Cache

API server 在 etcd 之上维护了一层 **watch cache**：

- 对每种资源类型在内存中维护完整缓存 + 历史事件窗口
- 多个客户端 watch 同一资源时，API server 只需对 etcd 维护一个 watch，然后本地扇出
- 显著减少 etcd 的 watch 连接数

**问题**：watch cache 要求 API server 至少在内存中持有每个 collection 的所有对象，加上历史窗口，内存开销大。

### 2.3 Watch Bookmark

问题：当 watch 断开重连时，如果没有最新的 `resourceVersion`，客户端需要从很早的 revision 重新 list，导致大量数据传输。

**Watch Bookmark**（Kubernetes 1.15+ 引入）：API server 定期发送 "bookmark" 事件（只含 resourceVersion，不含对象数据），让客户端保持最新的 revision 位置，断线重连时可以从最近的位置恢复。

### 2.4 Consistent Read from Cache（Kubernetes 1.31+）

传统上，每个强一致性读都会打到 etcd。kubelet list 自己节点上的 pod 时，API server 要从 etcd 获取所有 pod 再过滤。

**新机制**：API server 追踪 watch cache 的新鲜度，当确认缓存是最新的时，直接从缓存提供强一致性读。效果：**CPU 使用降低 30%，list 请求速度提升 3 倍**（AWS EKS 的测试数据）。

### 2.5 Streaming List（Kubernetes 1.33+）

大 LIST 操作过去需要在内存中缓冲整个响应才能发送。50,000 个 pod 的集群中，这是巨大的内存分配。

**Streaming list response** 允许 API server 增量编码和传输 collection 中的对象，内存效率和 list 请求并发提升约 **8 倍**。

---

## 三、实际规模数据

### 3.1 各大厂的集群规模与 etcd 瓶颈

| 组织 | 集群规模 | etcd 相关问题/方案 |
|------|----------|-------------------|
| **阿里巴巴** | 10,000 节点 / 200,000 pods / 1,000,000 对象 | etcd 读写延迟导致 pod 调度延迟高达 10s；将不同类型对象分到不同 etcd 集群；优化 bbolt 页分配算法 |
| **OpenAI** | 7,500 节点 | 5 API server + 5 etcd 节点；单独的专用节点运行 etcd；关注 API server 429/5xx 率 |
| **Reddit 用户报告** | ~4,000 节点 / 60,000 pods | Events 导致 etcd churn 过高，需要将 events 分离到独立 etcd 集群 |
| **AWS EKS** | 100,000 节点 | 完全替换 etcd 后端（详见下文） |
| **Google GKE** | 65,000 → 130,000 节点 | 用 Spanner 替换 etcd（详见下文） |
| **Kubernetes 官方推荐上限** | 5,000 节点 | [官方文档](https://kubernetes.io/docs/setup/best-practices/cluster-large/)的建议上限 |

### 3.2 典型瓶颈出现的规模

- **~1,000 节点**：node heartbeat 导致 etcd 写入压力明显（每 10s 更新 ~15KB 的 node 对象 × 1000 = 大量 etcd 事务日志）
- **~5,000 节点**：开始触碰 etcd 数据库大小和 watch fan-out 限制
- **~10,000 节点**：原生 etcd 基本无法支撑，需要分片或替换
- **>10,000 节点**：需要深度架构改造

**阿里的具体数据**：10,000 节点集群中，每分钟 etcd 产生近 **1 GB 事务日志**（仅 node heartbeat），API server 处理心跳的 CPU 开销超过 **80%**。引入 Lease API 后，kubelet 每 10s 只更新极小的 Lease 对象，node 对象更新间隔拉长到 60s。

---

## 四、替代方案深度对比

### 4.1 K3s / Kine

[Kine](https://github.com/k3s-io/kine)（"Kine is not etcd"）是 K3s 项目开发的 etcd API shim：

- **原理**：实现 etcd gRPC API 的子集，将请求转换为关系数据库操作
- **支持后端**：SQLite、PostgreSQL、MySQL/MariaDB、NATS
- **关键洞察**：API server 不直接用 etcd 内部实现，只用 etcd gRPC API。实现该 API 就能透明替换

**限制**：
- 只实现 Kubernetes 实际使用的 etcd API 子集，不是通用替代
- watch 效率依赖后端的 polling 实现
- revision 语义是近似的，非原生
- 适合边缘部署和小集群，不适合超大规模

### 4.2 AWS EKS：Journal Service

AWS 在 2025 年宣布支持 100,000 节点集群，做了三个核心替换：

| 改造 | 说明 |
|------|------|
| **Consensus offload** | 将 Raft 共识替换为内部 **journal** 组件（AWS 内部开发 10+ 年的有序数据复制服务），消除单 Leader 写瓶颈，无需 quorum，消除 peer-to-peer 通信 |
| **In-memory database** | 将 bbolt 从 EBS 卷迁移到 **tmpfs**（内存存储），读写吞吐数量级提升，延迟可预测，最大数据库大小提升到 **20 GB** |
| **Partitioned keyspace** | 按资源类型分片到不同 etcd 集群，写吞吐提升 **5 倍** |

关键点：**保持了 etcd API**，因为改写 Kubernetes API server 的存储接口成本远高于重写 etcd。

### 4.3 Google GKE：Spanner 后端

Google 的方案：

- 2024 年底宣布 **65,000 节点** GKE 集群，用 **Spanner**（Google 的全球分布式数据库）实现 etcd API
- 2025 年底推到 **130,000 节点**
- Spanner 没有 bbolt 的单文件大小限制，也没有单 Leader 写瓶颈

**但即使有 Spanner，超大集群仍有限制**：
- 不支持 cluster autoscaler
- headless service 限制 100 个 pod
- 每个节点一个 pod

因为存储层只是瓶颈之一，API server 自身的序列化/反序列化、admission controller、watch 事件分发、scheduler 单线程处理、kubelet 状态上报、网络带宽都有各自的天花板。

### 4.4 对比总结

| 方案 | 规模目标 | 一致性 | Watch 支持 | 运维复杂度 | 可用性 |
|------|----------|--------|-----------|-----------|--------|
| **原生 etcd** | ~5,000 节点 | 原生强一致 | 原生 | 中等 | 开源标准 |
| **Kine + SQLite** | 单节点/边缘 | 近似 | 基于 polling | 低 | K3s 生态 |
| **Kine + PostgreSQL** | 中小集群 | 近似 | 基于 polling | 中 | 社区支持 |
| **AWS Journal** | 100,000 节点 | 强一致 | 原生兼容 | N/A（托管） | 仅 EKS |
| **GKE Spanner** | 130,000 节点 | 强一致 | 原生兼容 | N/A（托管） | 仅 GKE |

### 4.5 etcd 社区自身的改进

- **bbolt 优化**：阿里巴巴贡献的 segregated hashmap 页分配算法（etcd 3.4+）
- **Raft Learner**：只读副本，不参与投票，可安全添加节点，减少对集群稳定性的影响
- **分片讨论**：Kubernetes 原生支持按资源类型将不同资源存到不同 etcd 集群（`--etcd-servers-overrides`），但不是 etcd 自身的分片
- **etcd 3.5/3.6 改进**：并发读优化、watch 性能改进、compaction 效率提升
- ⚠️ etcd v3.5.0 曾有严重的**数据损坏 bug**，直到 v3.5.4 才完全修复

---

## 五、生产实践

### 5.1 关键调优参数

| 参数                            | 推荐值                               | 说明                            |
| ----------------------------- | --------------------------------- | ----------------------------- |
| `--quota-backend-bytes`       | `8589934592`（8GB）                 | 数据库大小配额，超过则进入 alarm mode      |
| `--auto-compaction-mode`      | `periodic` 或 `revision`           | 自动 compaction 模式              |
| `--auto-compaction-retention` | `5m`（periodic）或 `10000`（revision） | compaction 保留策略               |
| `--heartbeat-interval`        | `100ms`（默认）                       | 建议为节点间 RTT 的 0.5-1.5 倍        |
| `--election-timeout`          | `1000ms`（默认）                      | 至少为 heartbeat-interval 的 10 倍 |
| `--snapshot-count`            | `10000`（默认）                       | 触发 snapshot 的事务数              |
| `--max-request-bytes`         | `1572864`（1.5MB）                  | 单次请求最大大小                      |

**磁盘 I/O 是最关键的因素**：
- etcd 对 fdatasync 延迟极其敏感
- **必须使用 SSD**，推荐低延迟 NVMe SSD
- 避免与其他 I/O 密集型工作负载共享磁盘
- OpenShift 建议控制平面节点间 RTT < 33ms，最大不超过 66ms

### 5.2 Monitoring 要点

关键 Prometheus 指标：

| 指标                                          | 含义                   | 告警阈值              |
| ------------------------------------------- | -------------------- | ----------------- |
| `etcd_mvcc_db_total_size_in_bytes`          | 数据库物理文件大小            | > 6 GiB 警告        |
| `etcd_mvcc_db_total_size_in_use_in_bytes`   | 实际使用大小（compaction 后） | 与物理大小差距过大需 defrag |
| `etcd_disk_wal_fsync_duration_seconds`      | WAL fsync 延迟         | p99 > 10ms 警告     |
| `etcd_disk_backend_commit_duration_seconds` | 后端提交延迟               | p99 > 25ms 警告     |
| `etcd_server_slow_apply_total`              | 慢 apply 次数           | 持续增长需关注           |
| `etcd_server_slow_read_indexes_total`       | 慢读次数                 | 持续增长需关注           |
| `etcd_network_peer_round_trip_time_seconds` | peer 间 RTT           | > 50ms 警告         |
| `etcd_server_proposals_failed_total`        | 失败的 Raft 提案          | 任何增长需关注           |
| `etcd_server_leader_changes_seen_total`     | Leader 切换次数          | 频繁切换需排查           |
| `grpc_server_handled_total`（code != OK）     | gRPC 错误              | 错误率上升需关注          |

### 5.3 常见故障模式

#### 1. Quota Alarm（数据库满）
- **现象**：etcd 进入只读模式，Kubernetes 无法创建/更新任何资源
- **原因**：compaction 跟不上写入速度，或从未配置 auto-compaction
- **恢复步骤**：
  ```bash
  # 1. 执行 compaction
  etcdctl compact $(etcdctl endpoint status --write-out=json | jq '.[0].Status.header.revision')
  # 2. 执行 defragmentation
  etcdctl defrag --endpoints=...
  # 3. 清除 alarm
  etcdctl alarm disarm
  ```

#### 2. Leader 频繁切换
- **现象**：集群不稳定，延迟波动大
- **常见原因**：磁盘 I/O 慢（fdatasync 延迟高）、网络不稳定、CPU 竞争
- **排查**：检查 `etcd_disk_wal_fsync_duration_seconds` 和 `etcd_network_peer_round_trip_time_seconds`

#### 3. Snapshot 压力
- **现象**：数据库大且有 follower 落后时，Leader 发送多 GB 的 snapshot，期间正常操作性能下降
- **缓解**：控制数据库大小、确保网络带宽充足

#### 4. Watch 事件积压
- **现象**：controller 处理延迟，看到的状态落后于实际
- **原因**：watch 连接太多 + 变更频率高，Leader CPU/网络饱和

#### 5. 数据损坏（etcd 3.5.0 - 3.5.3）
- **现象**：数据不一致
- **根因**：etcd v3.5.0 的已知 bug
- **预防**：保持 etcd 版本至少在 v3.5.4+，定期备份

### 5.4 架构最佳实践

1. **etcd 运行在专用节点**：不与其他工作负载共享，OpenAI 在 7,500 节点集群中使用 5 个专用 etcd 节点
2. **Events 分离**：将 Kubernetes Events 存入独立 etcd 集群（`--etcd-servers-overrides`），Events 是最高频的写入源
3. **定期备份**：`etcdctl snapshot save` + 异地存储
4. **多 API server**：分散读负载（OpenAI 用 5 个）
5. **监控先行**：Prometheus + Grafana 仪表盘（推荐 [kube-prometheus](https://github.com/coreos/kube-prometheus) 提供的 etcd dashboard）
6. **Lease 心跳**：确保 kubelet 使用 Lease API（Kubernetes 1.14+ 默认），将 node 更新间隔从 10s 拉长到 60s

---

## 六、上游 Kubernetes 的演进方向

Kubernetes 正在逐步解耦与 etcd 的依赖，但这是个渐进过程：

- **1.14**：引入 Lease API，减少 node heartbeat 对 etcd 的写入
- **1.15**：Watch Bookmark，减少 watch 断线重连时的数据传输
- **1.31**：Consistent reads from cache（Beta），强一致读不再打到 etcd
- **1.33**：Streaming list response，减少大 LIST 的内存开销

但核心存储接口（`k8s.io/apiserver/pkg/storage/interfaces.go`）仍然是 **etcd-shaped** 的——每个操作都是 revision-aware 的，`CompactRevision()` 直接暴露 etcd 的 compaction 状态。

更重要的是，**整个生态都围绕 etcd**：kubeadm 引导 etcd、备份工具针对 etcd snapshot、监控 dashboard 追踪 etcd 指标、运维手册描述 etcd 恢复流程。改变存储后端不仅是代码变更，整个生态都要跟上。

云厂商能绕过这个问题，是因为他们控制全栈——编译自己的 API server、接入自己的存储实现、跑自己的一致性测试。对于运行上游 Kubernetes 的所有人来说，**etcd 仍然是唯一支持的后端**。

---

## 七、总结

| 集群规模 | 建议 |
|----------|------|
| < 500 节点 | 标准 3 节点 etcd，默认配置基本够用 |
| 500 - 2,000 节点 | 调优参数、SSD、Events 分离、监控告警 |
| 2,000 - 5,000 节点 | 5 节点 etcd、专用节点、Lease heartbeat、watch cache 优化 |
| 5,000 - 10,000 节点 | 按资源类型分片 etcd、深度调优 API server、考虑 bbolt 优化补丁 |
| > 10,000 节点 | 使用托管服务（EKS/GKE）或自行替换 etcd 后端 |

**核心结论**：etcd 的限制不是 bug，而是其设计权衡的必然结果。Raft 单 Leader 写入、bbolt 单文件存储、MVCC revision 堆积、watch 事件扇出——每个单独看都是合理的设计选择，但在大规模下会叠加。大多数集群永远不会触碰这些限制。但如果你在推动 10,000+ 节点的规模，选择要么是深度改造，要么是站在云厂商已经替你改造好的基础上。

> "改写 etcd 比改写 Kubernetes 便宜" —— 这就是 AWS 和 Google 选择的路径。

## Takeaway

- 关键参数优化和监控
	- https://monitoring.mixins.dev/etcd/
	- 
- 搞一个其他后端
	- 玩玩, 但是用不上
	- 500 节点下其实都不需要, 我啥时候能玩到这种机器啊
	- 想玩万卡集群