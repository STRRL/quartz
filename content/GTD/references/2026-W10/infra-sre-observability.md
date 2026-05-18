---
title: "基础设施 / SRE / K8s / 可观测性 参考合集"
date: 2026-03-08
week: 2026-W10
tags: [infrastructure, sre, kubernetes, observability, cloud, docker, ebpf, monitoring]
---

# 基础设施 / SRE / K8s / 可观测性 参考合集

> 2026-W10 整理。覆盖云 VM 选型、容器技术十年回顾、K8s 思维模型、eBPF 可观测性、Feature Flag、监控工具等 16 个 items。

---

## 云基础设施与 VM 选型

### ZZ-849 — Cloud VM Benchmarks 2026

**来源：** [devblog.ecuadors.net](https://devblog.ecuadors.net/cloud-vm-benchmarks-2026-performance-price-1i1m.html) | Dimitrios Kechagias | 2026-02-27

2026 年云 VM 大横评，覆盖 AWS/GCP/Azure/OCI/Akamai/DigitalOcean/Hetzner 共 7 家、44 种 VM 类型。

**核心结论：**
- **AMD EPYC Turin 统治级表现**：单线程性能比所有竞品高出一个档次，是历年横评中领先幅度最大的一次。AWS C8a（Turin，禁用 SMT）最亮眼。
- **Intel Granite Rapids** 比 Emerald Rapids 更稳定（后者在繁忙节点上 boost 行为导致性能波动大）。
- **ARM 阵营**：Google Axion 领跑（达到 AMD Genoa 单线程水平）；Azure Cobalt 100 介于 Graviton3 和 Graviton4 之间。
- **OCI 性价比碾压**：Oracle Cloud Turin VM 仅 $29/月（2vCPU），远低于 AWS/GCP/Azure 同配置的 $50-90/月。
- **Hetzner 最便宜**：CX23 共享核 $4.31/月，ARM CAX11 $5.46/月（仅限欧洲）。
- **DigitalOcean 落后**：仍在用 Broadwell/Cascade Lake/Rome 等老旧 CPU，性能垫底。

**实用建议：** 有云 VM 选型需求时（CI、服务部署），OCI + Turin 的性价比值得优先考虑。

---

## 容器技术

### ZZ-847 — CACM: A Decade of Docker Containers

**来源：** [Communications of the ACM](https://cacm.acm.org/research/a-decade-of-docker-containers/) | Docker 团队 | 2026

Docker 十年技术演进回顾，从 Linux namespace/cgroup 基础到 AI 时代 GPU 支持。

**关键技术演进：**

- **2013 核心创新**：不是发明 namespace（2001 年就有），而是找到 VM 级隔离和 OS 原语易用性之间的实用平衡点。
- **2015 架构拆分**：从单体 daemon 拆为 buildkit（构建镜像）+ containerd（运行容器）。
- **2016 Docker for Mac/Windows**：把 hypervisor 嵌入用户态应用（library VMM "HyperKit"），用 MirageOS unikernel 实现用户态 TCP/IP 栈（vpnkit，OCaml 编写），bug 报告减少 99%+。
- **LinuxKit**：所有组件在容器内运行的最小 Linux，WSL2 后来采用了类似方案。
- **多架构支持**：OCI multiarch manifest + binfmt_misc/QEMU，Apple Rosetta 集成。
- **AI 时代 GPU 挑战**：GPU 没有类似 Linux syscall ABI 的稳定接口，CDI（Container Device Interface）2023 年起支持动态绑定 GPU 设备。

**规模数据：** Docker Hub 1400 万+ 镜像，月拉取量 110 亿+，GitHub 340 万+ 公开 Dockerfile。

---

## Kubernetes

### ZZ-141 — How I Think About Kubernetes

**来源：** [garnaudov.com](https://garnaudov.com/writings/how-i-think-about-kubernetes)

将 Kubernetes 重新定性为"声明式基础设施的运行时 + 类型系统"，而非简单的容器编排工具。

**核心思维模型：**
- **K8s 是运行时**：声明期望状态，系统持续协调收敛（不是执行一次性命令）。
- **类型系统类比**：Pod/Deployment/Service 等 `kind` 类似编程语言类型，CRD = 自定义类型，Operator = 实现这些类型语义的控制器。
- **核心循环**：`declare → persist → reconcile → place → execute`，持续进行。
- **调试技巧**：`spec` = 你想要的，`status` = 运行时观察到的 → 调试时先看这两者的差异。
- **GitOps 完美契合**：Git 是 source of truth，cluster 是运行时，手动修改被 GitOps 管理的资源会被回滚，这是预期行为。

### ZZ-193 — Multi-Cluster Interconnect (Skupper)

**来源：** [skupper.io](https://skupper.io/)

Skupper 是一个多集群互联方案，通过应用层网络（无需 VPN 或特殊网络权限）连接不同 Kubernetes 集群中的服务。适合跨云、跨区域的微服务互联场景。

### ZZ-165 — CloudNativePG (CNPG)

**来源：** [github.com/cloudnative-pg](https://github.com/cloudnative-pg) | CNCF Sandbox

在 Kubernetes 上全生命周期管理 PostgreSQL 的开源平台。

**要点：**
- 管理 PostgreSQL 在 K8s 上的完整运维生命周期（备份、HA、扩缩容）。
- CNCF Sandbox 项目，Apache 2.0 授权，由 EDB (EnterpriseDB) 创建赞助。
- 健康分 81/100，Star/贡献者 YoY 增长 40-100%。
- 社区活跃：每月 Office Hours + 每四周开发者会议。

---

## 可观测性 (Observability)

### ZZ-244 — Inspektor Gadget 2025 年度回顾

**来源：** [inspektor-gadget.io](https://inspektor-gadget.io/blog/2025/12/2025-year-in-review/)

eBPF 可观测性工具 Inspektor Gadget 的 2025 年度总结。

**重大进展：**
- **Image-Based Gadgets**：从内置 gadgets 转向 OCI 镜像架构，支持自定义 gadgets、版本管理、策略控制。
- **OpenTelemetry 集成**：原生 OTel operators，gadgets 直接发 metrics/logs，连接 Prometheus/Grafana。
- **MCP Server**：自然语言描述问题 → LLM 自动调用正确的 gadget + 参数 + 过滤器，实现 AI-driven troubleshooting。
- **核心定位**：让内核级可观测性（eBPF）对所有人可用。

### ZZ-80 — System Observability: Metrics, Sampling, and Tracing

关于系统可观测性三大支柱的介绍：Metrics（指标）、Sampling（采样）、Tracing（链路追踪）。可观测性基础概念参考。

### ZZ-79 — AI + Observability

**来源：**
- [Greptime 微信文章](https://mp.weixin.qq.com/s/ywoFue7WgW6pQyzMQWKzbw)
- [O'Reilly: Distributed Systems Observability](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch01.html)
- [Pragmatic Engineer: Observability with Charity Majors](https://newsletter.pragmaticengineer.com/p/observability-the-present-and-future)
- [Galileo: 9 Key Challenges in Monitoring Multi-Agent Systems](https://galileo.ai/blog/challenges-monitoring-multi-agent-systems)

AI 与可观测性结合的前沿探索。包含 Greptime 的 AI+Observability 实践、Charity Majors 对可观测性现状与未来的观点、以及 Multi-Agent Systems 监控的 9 大挑战。

### ZZ-219 — Geth Metrics 监控配置

**来源：** [geth.ethereum.org/docs/monitoring/metrics](https://geth.ethereum.org/docs/monitoring/metrics)

Go-Ethereum (geth) 的 metrics 监控配置参考。

**配置要点：**
- 使用 `--metrics` 和 `--metrics.prometheus` 开启。
- 默认在 `:6060/debug/metrics/prometheus` 暴露 Prometheus 格式指标。
- 常配合 Grafana 监控 Peer 数、区块处理时间和磁盘 I/O。

### ZZ-398 — Beszel — 轻量级服务器监控平台

**来源：** [beszel.dev](https://beszel.dev/guide/what-is-beszel)

caicai 推荐的轻量级服务器监控平台。

**特点：**
- 基于 PocketBase 构建，Hub + Agent 架构。
- 监控项：CPU、内存、磁盘、网络、温度、GPU、Docker 容器。
- 支持告警，Docker 一键部署。
- 定位：个人/小团队的轻量替代方案（对比 Grafana Stack）。

### ZZ-29 — Vercel Log Drain + Vector.dev

Vercel 的 Log Drain 功能配合 vector.dev 做日志收集管道的方案。适合将 Vercel 部署的应用日志接入自建的日志基础设施。

---

## DevOps 工具链

### ZZ-462 — zizmor — GitHub Actions 安全静态分析

**来源：** [github.com/zizmorcore/zizmor](https://github.com/zizmorcore/zizmor) | 推荐人：@shcallaway

GitHub Actions workflow 文件的静态分析安全工具，发现 CI/CD 配置中的安全漏洞。

### ZZ-687 — Google Cloud Logging Query Language

**来源：** [Google Cloud 官方文档](https://docs.cloud.google.com/logging/docs/view/logging-query-language)

GCP Logging 查询语言官方参考文档。

**关键点：**
- 适用于 Logs Explorer、Logging API、gcloud CLI、创建 sink 和 log-based metrics。
- 布尔表达式语法，支持 AND/OR/NOT，基于 LogEntry indexed fields。
- 支持 `resource.type`、`severity`、时间范围、label、jsonPayload 字段、正则表达式过滤。
- 字符串比较不区分大小写。
- 基于 [google.aip.dev/160](https://google.aip.dev/160) 规范。

### ZZ-138 — OpenFeature — CNCF Feature Flag 标准

**来源：** [cncf.io/projects/openfeature](https://www.cncf.io/projects/openfeature/) | CNCF Incubating

统一 Feature Flag 标准 SDK，解决厂商锁定。

**要点：**
- 2022-06 进入 CNCF Sandbox，2023-11 晋升 Incubating。
- 统一 API，不绑定特定平台，支持 LaunchDarkly、Flagsmith、自建系统等 provider 接入。
- 健康分 81/100，Star/贡献者 YoY 增长 40-100%。
- 核心价值：Feature Flag 基础设施的"可移植性"。

---

## 网络与连接

### ZZ-179 — SDWAN 与中国运营商拥堵分析

**来源：** [blog.mfwt.top](https://blog.mfwt.top/index.php/archives/912/) | 墨枫梧桐

深度分析中国三大运营商跨省/跨网/跨境链路拥堵的成因与对策。

**核心分析：**
- **跨省拥堵根源**：省间结算政策导致运营商对家宽用户实施严格 QoS，流量越大越是"负价值用户"。
- **跨网拥堵**：三网互联链路容量不足 + 运营商升级意愿低，晚高峰严重。
- **跨境拥堵**：晚高峰家宽丢包率普遍 20-30%，CMI 等优质线路也受波及。
- **应对方案**：KCP 协议（抗丢包传输）、SD-WAN（智能选路）；专线/流量卡 QoS 等级更高但价格贵。
- **历史背景**：从 2019 年到 2025 年网络环境的变化轨迹。

---

## 内容创作参考

### ZZ-470 — Kubernetes in 60 Seconds 内容风格

**来源：** [@devops_nk on X](https://x.com/devops_nk/status/2024431910018245093)

@devops_nk 的短视频 + 配音技术知识分享风格，60 秒讲解 Kubernetes 概念。适合学习技术内容传播和个人品牌建设的创作方法。

---

*本文由 GTD Master 自动整理，2026-W10 Weekly Review。*
