---
title: "Tailscale Peer Relays is GA"
date: 2026-02-20
source: "https://tailscale.com/blog/peer-relays-ga"
tags: [tailscale, networking, wireguard, relay]
---

# Tailscale Peer Relays is GA

Tailscale Peer Relay 正式 GA。自部署的高吞吐量 UDP 中继节点，当 P2P 直连因 NAT/防火墙失败时提供接近直连的性能。

## 与 DERP 的区别

- **DERP**: Tailscale 运营的共享中继，TCP over HTTPS，免费兜底
- **Peer Relay**: 自部署，UDP，带宽由你的硬件决定，流量不经过 Tailscale

## 关键特性

- 锁竞争优化、多 UDP socket 分流，吞吐量大幅提升
- Static Endpoints: 支持固定 IP:port，可放 AWS NLB 后面
- 可替代 subnet router，支持 full-mesh、Tailscale SSH、MagicDNS
- 可部署多个 relay 节点，Tailscale 自动选延迟最低的

## 限制

- Relay 之间不支持级联（无 relay-to-relay 链路）
- 跨国加速场景需要额外的网络架构设计（exit node + 专线）

## Links

- [官方博客](https://tailscale.com/blog/peer-relays-ga)
- [HN 讨论](https://news.ycombinator.com/item?id=47063005)
