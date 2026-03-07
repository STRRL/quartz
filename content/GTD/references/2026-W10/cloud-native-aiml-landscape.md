---
title: "Cloud Native AI/ML Landscape 整理"
date: 2026-03-07
tags: [kubernetes, ai, ml, cloud-native, landscape]
---

# Cloud Native AI/ML Landscape

## 1. 模型推理引擎（Inference Engines）

| 项目 | 维护方 | 定位 |
|---|---|---|
| vLLM | UC Berkeley / 社区 | LLM 专用，PagedAttention |
| SGLang | UC Berkeley LMSYS | LLM + 结构化生成，RadixAttention |
| Triton Inference Server | NVIDIA | 通用多后端推理服务器 |
| TensorRT-LLM | NVIDIA | LLM 推理优化（编译优化） |
| llama.cpp / ollama | 社区 | 轻量级本地推理，CPU/消费级 GPU |
| TGI (Text Generation Inference) | Hugging Face | LLM serving，HF 生态集成 |

## 2. 模型 Serving / 编排（Model Serving & Orchestration）

| 项目 | 维护方 | 定位 |
|---|---|---|
| KServe | Red Hat + CNCF 社区 | K8s 原生模型 serving 框架，标准化推理协议 |
| Seldon Core | Seldon | 模型部署 + A/B 测试 + 可解释性 |
| BentoML | BentoML | 模型打包 + serving，开发者友好 |
| Ray Serve | Anyscale | 基于 Ray 的模型 serving，支持复杂 pipeline |
| llm-d | Red Hat | KServe 的 LLM 专用数据面，KV-cache aware 调度 |

## 3. 调度与资源管理（Scheduling & Resource Management）

| 项目 | 维护方 | 定位 |
|---|---|---|
| Volcano | 华为 / CNCF | K8s batch/AI 调度器，gang scheduling |
| Kthena | 华为 Volcano 团队 | Volcano 子项目，LLM 推理编排 |
| Kueue | Google / K8s SIG | K8s 原生作业队列，quota 管理 |
| YuniKorn | Apache | 大数据/ML 统一调度器 |
| DRA (Dynamic Resource Allocation) | K8s SIG | K8s 1.30+ GPU/加速器动态分配 |

## 4. GPU 虚拟化与共享（GPU Virtualization & Sharing）

| 项目 | 维护方 | 定位 |
|---|---|---|
| NVIDIA GPU Operator | NVIDIA | K8s GPU 全栈管理（驱动、插件、监控） |
| NVIDIA MPS | NVIDIA | 多进程共享 GPU |
| NVIDIA MIG | NVIDIA | A100/H100 硬件级 GPU 分区 |
| HAMi (k8s-vGPU-scheduler) | 社区 | K8s GPU 虚拟化 + 显存/算力隔离 |
| Fluid | 阿里 / CNCF Sandbox | 数据集编排和加速（缓存亲和调度） |

## 5. 训练平台（Training Platforms）

| 项目 | 维护方 | 定位 |
|---|---|---|
| Kubeflow | Google / CNCF | 端到端 ML 平台（训练 + pipeline + notebook） |
| Kubeflow Training Operator | CNCF 社区 | 分布式训练（PyTorch/TensorFlow/MPI） |
| DeepSpeed | Microsoft | 大模型分布式训练优化 |
| Megatron-LM | NVIDIA | 大规模 LLM 训练框架 |
| Ray Train | Anyscale | 基于 Ray 的分布式训练 |

## 6. ML Pipeline / Workflow

| 项目 | 维护方 | 定位 |
|---|---|---|
| Kubeflow Pipelines | Google / CNCF | ML pipeline 编排 |
| Argo Workflows | Akuity / CNCF | 通用 K8s workflow 引擎 |
| MLflow | Databricks / Linux Foundation | 实验跟踪 + 模型注册 + 部署 |
| Flyte | Union.ai / Linux Foundation | 类型安全的 ML workflow |
| Metaflow | Netflix / Outerbounds | 数据科学 workflow |

## 7. 模型注册与格式（Model Registry & Formats）

| 项目 | 维护方 | 定位 |
|---|---|---|
| Hugging Face Hub | Hugging Face | 模型/数据集托管平台 |
| OCI Artifacts / ORAS | OCI / CNCF | 用容器 registry 存储模型 |
| ModelCar (KServe) | KServe 社区 | OCI image 打包模型，init container 加载 |
| ONNX | Linux Foundation | 跨框架模型交换格式 |
| Safetensors | Hugging Face | 安全高效的模型权重格式 |

## 8. 可观测性与 AI Gateway

| 项目 | 维护方 | 定位 |
|---|---|---|
| OpenTelemetry + GenAI SIG | CNCF | LLM 调用的 trace/metrics 标准化 |
| Envoy AI Gateway | Envoy / CNCF | LLM 流量管理（路由、限流、可观测） |
| LiteLLM | 社区 | 多 LLM provider 统一代理 |
| Portkey | Portkey | AI Gateway（缓存、fallback、审计） |

## 9. Vector Database / RAG 基础设施

| 项目 | 维护方 | 定位 |
|---|---|---|
| Milvus | Zilliz / Linux Foundation | 云原生向量数据库 |
| Weaviate | Weaviate | 向量数据库 + 混合搜索 |
| Qdrant | Qdrant | Rust 实现的向量数据库 |
| Chroma | Chroma | 轻量嵌入式向量数据库 |
| pgvector | 社区 | PostgreSQL 向量搜索扩展 |

## 全景总结

```
用户请求
  ↓
[AI Gateway / 路由] ← Envoy AI Gateway, LiteLLM
  ↓
[Model Serving 编排] ← KServe, Seldon, BentoML
  ↓
[推理引擎] ← vLLM, SGLang, Triton, TGI
  ↓
[GPU 调度与共享] ← Volcano/Kthena, Kueue, DRA, HAMi
  ↓
[GPU 硬件] ← GPU Operator, MIG, MPS

[训练侧]  Kubeflow, DeepSpeed, Ray Train
[Pipeline] Argo, Kubeflow Pipelines, MLflow
[数据]     Fluid, Vector DB (Milvus, pgvector)
[可观测]   OpenTelemetry GenAI SIG
```
