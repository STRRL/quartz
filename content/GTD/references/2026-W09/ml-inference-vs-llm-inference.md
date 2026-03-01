---
title: "传统 ML 推理 vs LLM 推理：系统性深入参考"
date: 2026-02-24
tags: ["ml", "llm", "inference", "systems"]
---

# 传统 ML 推理 vs LLM 推理

> 基于 [Daily Dose of DS 的博客](https://blog.dailydoseofds.com/p/regular-ml-inference-vs-llm-inference) 展开的深入工程参考。

## 核心差异总览

| 维度 | 传统 ML（CNN/XGBoost 等） | LLM |
|------|--------------------------|-----|
| 输入/输出 | 固定大小 | 变长输入 + 变长自回归输出 |
| 计算模式 | 单阶段前向传播 | 两阶段：prefill（compute-bound）+ decode（memory-bound） |
| Batching | 静态 batch，所有样本同时完成 | 各请求完成时间不同，需要 continuous batching |
| 缓存 | 无状态，请求独立 | KV cache 跨 token 复用，请求间可共享前缀 |
| 扩展 | 简单复制 + round-robin | 需要 prefix-aware routing、KV cache 迁移 |
| 并行策略 | DP / pipeline | TP + PP + EP（MoE），更复杂 |

---

## 1. Continuous Batching（Orca 论文）

**论文**: Yu et al., "Orca: A Distributed Serving System for Transformer-Based Generative Models", OSDI 2022.

### 问题：Static Batching 的 GPU 浪费

传统 batch inference 把一组请求放进同一个 batch，等 **所有请求** 生成完 EOS 后才能调度下一批。由于各请求输出长度不同，短请求完成后 GPU slot 空转（padding waste）。

### Orca 的两个核心技术

#### 1.1 Iteration-Level Scheduling

关键思想：调度粒度从 **request-level** 降到 **iteration-level**（即每一步 decode iteration）。

- 每个 decode step 结束后，scheduler 检查哪些序列已输出 EOS
- 已完成的序列立即从 batch 中移除，slot 让给等待队列中的新请求
- 新请求先执行 prefill（处理完整 prompt），然后在下一个 iteration 加入 decode batch

**工程细节**：
- Scheduler 维护一个 **running pool**（当前 batch 中的活跃序列）和 **waiting queue**（排队请求）
- 每个 iteration：`running_pool = [seq for seq in running_pool if not seq.is_finished()] + new_seqs_from_queue`
- 最大 batch size 受 GPU 显存（主要是 KV cache 容量）限制

#### 1.2 Selective Batching

不是所有 Transformer 操作都适合 batching 不同长度的序列：

- **Attention**：每个序列有不同的 sequence length，不能简单 batch → Orca 对 attention 不做 batch，逐序列计算
- **FFN / LayerNorm 等**：只依赖当前 token 的 hidden state，维度固定 → 可以安全 batch

这样做的好处是避免了 padding，同时在可以 batch 的地方充分利用 GPU 并行。

> **后续影响**：vLLM、SGLang 等所有现代引擎都实现了 continuous batching，这已经是 LLM serving 的标配。

---

## 2. PagedAttention（vLLM 的核心创新）

**论文**: Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention", SOSP 2023.

### 问题：KV Cache 的内存碎片

每个请求的 KV cache 随生成长度线性增长。传统做法预分配连续内存块（按 max_seq_len），导致：

1. **内部碎片（Internal fragmentation）**：实际生成长度 << max_seq_len，预分配的尾部空间浪费
2. **外部碎片（External fragmentation）**：请求完成后释放的内存块大小不一，无法被新请求利用
3. **预留浪费（Reservation waste）**：必须为每个请求预留最大可能长度

实测中，传统方案的 KV cache 有效利用率仅 **20-40%**。

### PagedAttention 的解法：借鉴 OS 虚拟内存

核心类比：

| OS 虚拟内存 | PagedAttention |
|------------|----------------|
| 虚拟页（virtual page） | 逻辑 KV block |
| 物理页帧（physical frame） | GPU 上的物理 KV block |
| 页表（page table） | Block table |
| 按需分配（demand paging） | 按需分配新 block |

#### 具体机制

1. **Block 划分**：KV cache 被切分为固定大小的 block，每个 block 存放固定数量 token 的 K 和 V 向量（典型 block_size = 16 tokens）
2. **Block Table**：每个序列维护一个 block table，记录逻辑 block → 物理 block 的映射
3. **按需分配**：生成新 token 时，只在当前 block 填满后才分配新的物理 block
4. **非连续存储**：同一序列的 block 可以散布在 GPU 显存的任意位置

#### Attention 计算的适配

CUDA kernel 需要改写：不再假设 KV cache 连续存放，而是通过 block table 查找每个 block 的物理地址：

```
// 伪代码：PagedAttention kernel
for each query token q:
    for each logical_block_idx in seq.block_table:
        physical_block = block_table[logical_block_idx]
        k_block = k_cache[physical_block]  // 从非连续位置读取
        v_block = v_cache[physical_block]
        score += dot(q, k_block)
    output = softmax(score) @ v_blocks
```

vLLM 的 kernel 签名：
```cpp
paged_attention_kernel<scalar_t, HEAD_SIZE, BLOCK_SIZE, NUM_THREADS, PARTITION_SIZE>
    (out, q, k_cache, v_cache, ...)
// k_cache shape: [num_blocks, num_kv_heads, head_size/x, block_size, x]
// v_cache shape: [num_blocks, num_kv_heads, head_size, block_size]
```

#### 高级特性

- **Copy-on-Write (CoW)**：beam search 或 parallel sampling 时，多个序列共享相同前缀的 block，只在分歧时拷贝
- **Prefix sharing**：相同 system prompt 的请求共享 KV block，引用计数管理
- **Swap / offload**：显存不够时可以把 block swap 到 CPU 内存，之后再 swap 回来（类似 OS 的 page swap）

#### 效果

- KV cache 利用率从 ~30% 提升到 **>96%**
- 相同显存下可服务的并发请求数提升 **2-4x**
- 吞吐量提升 **2-4x**（通过支持更大 batch size）

---

## 3. 推理引擎对比

### 3.1 vLLM

- **核心创新**：PagedAttention
- **定位**：通用 LLM serving，社区最大，生态最广
- **特点**：
  - Python-first，易于 hack 和扩展
  - 支持 TP/PP、LoRA serving、guided decoding（结构化输出）
  - OpenAI-compatible API
  - 支持 prefix caching、chunked prefill
- **适合场景**：通用部署、快速上手、需要灵活定制

### 3.2 SGLang

- **出自**：LMSYS（Chatbot Arena 团队）
- **核心创新**：RadixAttention（基于 radix tree 的 KV cache 管理）+ 高效 structured generation
- **特点**：
  - 用 radix tree 管理所有请求的 KV cache 前缀，实现自动、细粒度的前缀复用
  - 对 structured output（JSON schema、正则约束）特别优化
  - 在 multi-turn / 共享前缀场景下性能显著优于 vLLM
  - 从 2024 年中开始在 benchmark 中频繁击败 vLLM 和 TensorRT-LLM
- **Benchmark 表现**：Llama3 系列模型上，SGLang 在 online serving 场景下 throughput 和 latency 均优于或持平 TensorRT-LLM（LMSYS 2024-07 benchmark）
- **适合场景**：高吞吐在线服务、structured output、multi-turn 对话

### 3.3 TensorRT-LLM

- **出自**：NVIDIA
- **核心优势**：极致硬件优化
- **特点**：
  - 底层 C++，深度使用 NVIDIA GPU 特性（FP8、custom CUDA kernel、Tensor Core 利用率最高）
  - 支持 in-flight batching（NVIDIA 版本的 continuous batching）
  - 需要预编译模型为 TensorRT engine（build 过程复杂）
  - 灵活性最低，新模型支持慢
  - 单请求延迟通常最低（kernel 级优化）
- **Benchmark 表现**：单请求吞吐最好，但高并发扩展性不如 SGLang（Clarifai GPT-OSS-120B benchmark 2025）
- **适合场景**：NVIDIA 专有环境、对延迟极端敏感、批量离线推理

### 3.4 LMCache

- **定位**：不是推理引擎，而是 **KV cache 管理层**，配合 vLLM / SGLang 使用
- **核心能力**：
  - 从 vLLM/SGLang 引擎中提取 KV cache，存到 GPU → CPU DRAM → 本地磁盘 → 远程存储的多级缓存
  - 跨引擎实例共享 KV cache（同一 prefix 在 Replica A 计算后，Replica B 可以直接加载）
  - 支持非前缀的 KV 复用（不仅是 prefix，任意共享片段都可以缓存）
  - CacheBlend 技术：部分 KV 复用 + 选择性重计算，保证精度
- **适合场景**：多副本部署、shared system prompt 重、enterprise-scale serving

### 对比总结

| 维度                | vLLM              | SGLang          | TensorRT-LLM | LMCache |
| ----------------- | ----------------- | --------------- | ------------ | ------- |
| 语言                | Python + CUDA     | Python + CUDA   | C++ + CUDA   | Python  |
| 上手难度              | 低                 | 低               | 高（需编译）       | 低（插件式）  |
| 社区                | 最大                | 快速增长            | NVIDIA 官方    | 学术团队    |
| 前缀复用              | 手动 prefix caching | Radix tree 自动管理 | 有限支持         | 多级、跨实例  |
| Structured output | 支持                | **最优**          | 有限           | N/A     |
| 单请求延迟             | 好                 | 好               | **最低**       | N/A     |
| 高并发吞吐             | 好                 | **最优**          | 一般           | 增强原引擎   |
| 硬件锁定              | 无                 | 无               | NVIDIA only  | 无       |
|                   |                   |                 |              |         |

---

## 4. Speculative Decoding

### 核心思想

LLM decode 阶段是 **memory-bandwidth bound**：每生成一个 token 都要加载整个模型权重，但 GPU 计算单元大量空闲。Speculative decoding 利用这个空闲：

1. **Draft 阶段**：用一个小模型（draft model）快速自回归生成 γ 个候选 token
2. **Verify 阶段**：用大模型（target model）**一次前向传播** 并行验证这 γ 个 token
3. **Accept/Reject**：基于 rejection sampling 决定接受多少 token

### 数学保证：Lossless

关键是 rejection sampling 的设计保证了输出分布与直接用大模型自回归生成 **完全一致**：

对于 draft model 分布 $q(x)$ 和 target model 分布 $p(x)$：
- 如果 $q(x_i) \le p(x_i)$：以概率 1 接受
- 如果 $q(x_i) > p(x_i)$：以概率 $p(x_i)/q(x_i)$ 接受
- 被拒绝时，从修正分布 $\text{norm}(\max(0, p(x) - q(x)))$ 重采样

这保证了最终分布恰好是 $p(x)$——**无损加速**。

### 加速比分析

- 如果 draft model 的 acceptance rate 为 α，每次投机 γ 个 token
- 期望每次验证接受的 token 数：$(1 - \alpha^{\gamma+1}) / (1 - \alpha)$
- Draft model 越接近 target model → α 越高 → 加速越明显
- 实践中典型加速 **1.5x - 3x**

### 变体

| 方法 | 特点 |
|------|------|
| **Draft model** | 独立小模型（如 Llama-68M draft Llama-70B） |
| **Self-speculative** | 用同一模型的跳层（skip layers）做 draft，无需额外模型 |
| **Medusa** | 在 target model 上加多个 prediction head，并行预测多个位置 |
| **EAGLE** | 用 target model 的 hidden states 训练 draft head |
| **Lookahead** | 使用 n-gram 和 Jacobi iteration，无需 draft model |
| **HiSpec** | 层级 speculation：多级 draft + 分级验证 |

### 工程考量

- Draft model 选型很关键：太大则 draft 本身慢，太小则 acceptance rate 低
- 需要 draft model 和 target model 共享 vocabulary
- vLLM、SGLang、TensorRT-LLM 均已原生支持 speculative decoding

---

## 5. Prefill-Decode 分离（Disaggregated Serving）

### 问题：Prefill 和 Decode 的资源冲突

| 阶段 | 计算特性 | 关键指标 |
|------|---------|---------|
| Prefill | Compute-bound（大量矩阵乘） | TTFT（Time-to-First-Token） |
| Decode | Memory-bandwidth-bound（每步只处理 1 token） | TPOT（Time-per-Output-Token） / ITL |

混合部署的问题：
- 一个长 prompt 的 prefill 会阻塞同 GPU 上所有 decode 请求（**prefill stall**）
- Decode 请求需要低延迟但计算量小，和 compute-heavy 的 prefill 争抢资源
- 难以独立扩缩 prefill 和 decode 的资源

### DistServe（OSDI 2024）

**核心思想**：将 prefill 和 decode 分配到不同的 GPU pool，中间通过 KV cache 传输连接。

#### 架构

```
Request → [Prefill GPU Pool] → KV cache transfer → [Decode GPU Pool] → Response
              ↑                                          ↑
         compute-optimized                        memory-BW-optimized
         (高 FLOPS 利用率)                         (高并发、低延迟)
```

#### 关键设计

1. **独立资源配比**：prefill 和 decode GPU 数量可以根据 workload 特征独立调整
2. **Goodput 优化**：不是最大化 throughput，而是最大化满足 SLO 的请求比例（goodput）
3. **KV cache 传输**：prefill 完成后通过 NVLink / PCIe / RDMA 将 KV cache 发送到 decode GPU
4. **调度算法**：基于 SLO-aware 的放置策略，预测每个请求的 prefill 和 decode 时间

#### 效果
- 相比 colocated serving（vLLM 默认），在相同 SLO 下 goodput 提升 **2x - 4.48x**
- 或在相同 goodput 下 SLO 收紧 **10.2x**

### Splitwise（Microsoft, 2024）

与 DistServe 同期独立提出，核心思路一致，但侧重不同：
- 更关注**异构硬件**：prefill 用高算力 GPU（如 H100），decode 用大显存/高带宽 GPU
- 强调 **cluster-level** 调度和混合工作负载管理
- 在 Azure 生产环境验证

### 后续发展（截至 2025）

DistServe 团队 18 个月后的回顾（2025-11）：

1. **Mooncake（Kimi/月之暗面）**：生产落地的 disaggregated serving，prefill-decode 分离 + KV cache 存储层
2. **vLLM V1** 已原生支持 disaggregated prefill
3. **Prefill-Decode Multiplexing**：新方向——不完全分离，而是在同一 GPU 上智能交错 prefill 和 decode micro-batch，减少 KV 传输开销
4. **Context caching as a service**：disaggregation 使得 KV cache 可以独立存储和共享（与 LMCache 的理念融合）

### 工程挑战

- **KV cache 传输延迟**：对于短请求，传输 overhead 可能抵消分离收益
- **网络带宽要求高**：KV cache 体积大（70B 模型，4K context ≈ 几 GB），需要高速互连
- **复杂度增加**：需要跨 GPU 协调调度，容错更复杂
- **适用条件**：请求量大、prompt 较长时收益最明显

---

## 6. 其他重要优化方向（简要提及）

- **Quantization**：W4A16、W8A8、FP8 等，减少模型大小和显存占用，提升吞吐
- **FlashAttention**：I/O-aware 的 attention 实现，减少 HBM 访问（已成为标配）
- **Chunked Prefill**：将长 prompt 的 prefill 分成多个 chunk，减少对 decode 的干扰
- **Prefix-Aware Routing**：多副本部署时，根据 KV cache 命中情况路由请求
- **Expert Parallelism**：MoE 模型特有的并行策略，expert 分散在不同 GPU

---

## 参考文献

1. Yu et al., "Orca: A Distributed Serving System for Transformer-Based Generative Models", OSDI 2022
2. Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention", SOSP 2023
3. Zhong et al., "DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving", OSDI 2024
4. Patel et al., "Splitwise: Efficient Generative LLM Inference Using Phase Splitting", ISCA 2024
5. Leviathan et al., "Fast Inference from Transformers via Speculative Decoding", ICML 2023
6. Chen et al., "Accelerating Large Language Model Decoding with Speculative Sampling", 2023
7. Zheng et al., "SGLang: Efficient Execution of Structured Language Model Programs", 2024
8. LMCache Tech Report, lmcache.ai, 2025
9. LMSYS Blog, "Achieving Faster Open-Source Llama3 Serving with SGLang Runtime", 2024-07
10. Hao AI Lab, "Disaggregated Inference: 18 Months Later", 2025-11

## Takeaway

- 看上去很有趣, 很直接很工程的提升, 加了个 memtable 就行了...
- 什么时候我也有机会做这种工作呢
- ML/LLM inference 得学起来