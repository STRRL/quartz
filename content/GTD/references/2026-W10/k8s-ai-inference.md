---
title: "Kubernetes AI 推理三方势力：CNCF 趋势 + KServe/llm-d + Kthena"
date: 2026-03-06
tags: [kubernetes, ai, inference, llm, cloud-native]
---

# Kubernetes AI 推理基础设施

## 趋势：所有 AI 平台都在向 K8s 收敛（ZZ-780）

CNCF 博客，Amazon 的 Sabari Sawant 撰写。核心观点：不管你用 vLLM、TensorRT 还是自研引擎，调度层最终都是 K8s。

## 方案 A: KServe + llm-d（ZZ-771）

- KServe: K8s 原生 serving 框架（Red Hat + 社区）
- llm-d: **KV-cache aware 调度** — 把 KV-cache 状态暴露给调度器
- 核心创新：请求路由到已有缓存的 Pod，避免重复 prefill
- 适合：已有 K8s 集群、想要社区驱动方案

## 方案 B: Kthena / Volcano（ZZ-739）

- 华为 Volcano 团队（K8s batch/AI 调度器）的子项目
- 支持 vLLM/SGLang/Triton 多引擎
- 声明式模型生命周期管理 + 智能路由
- Prefill-decode disaggregation
- 适合：已用 Volcano 调度、华为云生态

## 对比

| | KServe + llm-d | Kthena |
|---|---|---|
| 生态 | Red Hat + CNCF 社区 | 华为 Volcano |
| 调度特色 | KV-cache aware routing | Prefill-decode 分离 |
| 多引擎 | 主要 vLLM | vLLM/SGLang/Triton |
| 成熟度 | 较高（KServe 已广泛使用） | 较新 |

## 推理引擎对比：vLLM vs SGLang vs Triton

| | vLLM | SGLang | Triton |
|---|---|---|---|
| 定位 | LLM 专用推理引擎 | LLM 推理 + 结构化生成 | 通用模型推理服务器 |
| 核心技术 | PagedAttention（KV-cache 分页管理） | RadixAttention（前缀树复用 KV-cache） | 多后端（TensorRT, ONNX, PyTorch 等） |
| 模型范围 | LLM 为主 | LLM 为主 | 任意模型（CV、NLP、语音等） |
| 结构化输出 | 支持（JSON mode 等） | 原生强项（constrained decoding） | 需自行实现 |
| 吞吐量 | 高（continuous batching） | 高（prefix caching 场景更优） | 取决于后端引擎 |
| 社区 | UC Berkeley, 社区最活跃 | UC Berkeley LMSYS | NVIDIA 官方 |
| 适合场景 | 通用 LLM serving | 多轮对话 / 共享前缀多的场景 | 混合模型部署、已有 TensorRT pipeline |

vLLM 和 SGLang 都出自 UC Berkeley，但不是同一批人：
- vLLM: Ion Stoica 实验室（Sky Computing Lab / RISELab），Zhuohan Li、Woosuk Kwon 等，后成立公司
- SGLang: Lianmin Zheng 主导（LMSYS 团队，也维护 Chatbot Arena）
- 两者技术路线不同，存在竞争关系

选型思路：
- 纯 LLM serving → vLLM（生态最大，KServe/llm-d 主推）
- 多轮对话或大量共享 system prompt → SGLang（RadixAttention 前缀复用优势明显）
- 需要同时部署非 LLM 模型或已有 NVIDIA 推理栈 → Triton
- Kthena 同时支持三者，统一调度层让用户按需选择

## TODO

- [ ] 评估哪个方案更适合 LayerZero 场景
- [ ] 关注 KV-cache aware scheduling 的实际效果数据

## 参考

- [ZZ-780] CNCF The Great Migration — https://www.cncf.io/blog/2026/03/05/the-great-migration-why-every-ai-platform-is-converging-on-kubernetes/
- [ZZ-771] KServe + llm-d — https://kserve.github.io/website/blog/cloud-native-ai-inference-kserve-llm-d
- [ZZ-739] Kthena — https://github.com/volcano-sh/kthena
