---
title: "ForkSpacer - Kubernetes Environment Forking Tool"
date: 2026-02-24
source: https://github.com/forkspacer/forkspacer
tweet: https://x.com/learnk8s/status/2025623134716477884
tags: [kubernetes, devtools, environments, operator]
---

## 概述

ForkSpacer 是一个开源的 Kubernetes Operator，核心功能是**声明式地创建、fork（分叉）和休眠整个 Kubernetes 环境**。开发者可以从一个 baseline workspace 克隆出完整的隔离环境用于测试，空闲时自动休眠以节省资源。

⚠️ **注意**：LearnK8s 推文描述较为宽泛（提到 VM-based environments），但从源码看，当前实现聚焦于 Kubernetes 环境，通过 Helm chart 管理应用。

## 核心概念

### 两个 CRD

1. **Workspace**（`batch.forkspacer.com/v1`）
   - 代表一个环境（如 dev、staging、prod）
   - 类型：`kubernetes`（连接现有集群）或 `managed`（自动创建 vcluster/k3d/kind）
   - 支持 `from` 字段实现 fork：从另一个 workspace 克隆所有 modules
   - 支持 auto-hibernation（cron 定时休眠/唤醒）
   - 连接方式：in-cluster / kubeconfig secret

2. **Module**（`batch.forkspacer.com/v1`）
   - 代表 workspace 里的一个应用/服务
   - 两种类型：
     - **Helm module**：管理 Helm chart（支持 repo、git、configmap 来源）
     - **Custom module**：自定义容器镜像
   - 支持 adopt 已有的 Helm release（`existingRelease`）
   - 支持 hibernation（sleep/resume）
   - 支持 configurable parameters（Config 系统，类型安全的 int/bool/string/option）

### Fork 工作流程

`workspace_fork.go` 揭示了 fork 的实现：
1. 获取源 workspace 的所有 modules
2. 确认所有源 modules 都是 Ready/Sleeped 状态
3. 为每个 module 创建副本（去掉 existingRelease，因为是新安装）
4. 等待新 module 就绪
5. 如果 `migrateData: true`，执行数据迁移（PVC、Secret、ConfigMap）

### 数据迁移

Fork 时可选迁移：
- **PVC**：通过 `PVCMigrationService` 跨 namespace 复制持久化数据
- **Secret**：复制密钥
- **ConfigMap**：复制配置

迁移过程会先休眠源和目标 module（停止工作负载），完成后恢复。

## 架构

```
┌─────────────────────────────────┐
│  ForkSpacer Operator            │
│  ├── WorkspaceReconciler        │
│  │   ├── fork workspace         │
│  │   ├── managed cluster create │
│  │   └── auto-hibernation       │
│  ├── ModuleReconciler           │
│  │   ├── helm install/upgrade   │
│  │   ├── hibernate/resume       │
│  │   └── custom module mgmt    │
│  └── Webhooks (validation)      │
├─────────────────────────────────┤
│  CRDs: Workspace, Module        │
├─────────────────────────────────┤
│  Services                       │
│  ├── HelmService                │
│  ├── PVCMigrationService        │
│  ├── SecretMigrationService     │
│  └── ConfigMapMigrationService  │
└─────────────────────────────────┘
```

- 标准 kubebuilder 项目结构
- 通过 Helm chart 部署（含 operator-ui 和 api-server 可选组件）
- 需要 cert-manager 作为前置依赖

## Managed Cluster 支持

Workspace 类型为 `managed` 时，可自动创建虚拟集群：
- 后端选择：**vcluster**（默认）、k3d、kind
- vcluster 发行版：k3s（默认）、k0s、k8s、eks

## 适用场景

- 每个开发者需要从共享 baseline fork 出个人环境
- Dev / Test / Pre-prod / Prod 环境管理
- 空闲环境自动休眠节省云资源
- GitOps 风格的环境声明式管理

## 技术栈

- Go + kubebuilder（Operator SDK）
- Helm 作为应用部署引擎
- CRD: `batch.forkspacer.com/v1`
- Apache 2.0 许可证

## Takeaway

- fancy project
- I do not get the point, if we can not handle the "state", like persisted state PV, or in-memory state, even config, it will not work.