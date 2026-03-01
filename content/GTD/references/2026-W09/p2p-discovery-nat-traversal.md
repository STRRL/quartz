---
title: "P2P 节点发现与 NAT 穿透机制"
date: 2026-02-24
tags: ["p2p", "networking", "nat-traversal", "libp2p", "ethereum", "tailscale"]
---

# P2P 节点发现与 NAT 穿透机制

## 1. libp2p 的连接建立

### Multiaddr 格式

libp2p 使用 **multiaddr**（多地址）格式来描述网络地址，它是自描述、可组合的：

```
/ip4/198.51.100.1/tcp/4001/p2p/QmPeerID
/ip6/::1/udp/9090/quic-v1/p2p/12D3KooW...
/dns4/example.com/tcp/443/wss/p2p/12D3KooW...
```

特点：
- **协议栈从左到右叠加**：网络层 → 传输层 → 应用层 → peer 身份
- `/p2p/<peer-id>` 最后一层标识节点的密码学身份（通常是公钥的 multihash）
- 支持 TCP、UDP/QUIC、WebSocket、WebTransport 等多种传输
- 与以太坊的 enode URL 不同，multiaddr 天然支持多协议栈（一个节点可以广播多个 multiaddr）

### Aqua 的节点发现与连接

[Aqua](https://github.com/quailyquaily/aqua) 是基于 libp2p 的 AI agent 消息工具，其网络层工作方式：

1. **身份生成**：`aqua id <nickname>` 生成 secp256k1/Ed25519 密钥对，派生出 PeerID（如 `12D3KooW...`）
2. **直连模式**：节点启动后监听本地端口，广播 multiaddr（如 `/ip4/x.x.x.x/tcp/6372/p2p/<peer-id>`），对端直接 dial 该地址
3. **Relay 模式**：`--relay-mode auto` 先尝试直连，失败则通过 Circuit Relay v2 中继
4. **联系人管理**：手动 `aqua contacts add "<multiaddr>"` 交换地址（没有自动 DHT 发现）
5. **E2EE**：基于 libp2p Noise 协议建立加密通道

Relay 地址格式示例：
```
/dns4/aqua-relay.mistermorph.com/tcp/6372/p2p/<relay-peer-id>/p2p-circuit/p2p/<your-peer-id>
```

### Circuit Relay v2（NAT 穿透）

libp2p 的 NAT 穿透方案，核心思路是通过一个公网可达的中继节点转发流量：

```
A (behind NAT) ←→ Relay (public) ←→ B (behind NAT)
```

**工作流程：**
1. 节点 A 连接到公网 Relay 节点，请求 **reservation**（预留中继槽位）
2. Relay 同意后，A 获得一个 relay circuit 地址：`/p2p/<relay>/p2p-circuit/p2p/<A>`
3. 节点 B 想连接 A 时，先连接 Relay，通过 relay circuit 地址建立中继连接
4. （可选）通过 relay 连接交换直连信息，尝试 **DCUtR（Direct Connection Upgrade through Relay）** 进行 hole punching

**v2 vs v1 的改进：**
- v1 是无限制中继，容易被滥用；v2 有时间限制（默认 2 分钟）和流量限制（默认 128KB）
- v2 设计为"临时中继"，目的是让双方快速交换信息后尝试直连
- v2 中继节点资源消耗可控

**局限性：**
- 如果双方都在严格 NAT 后面且 hole punching 失败，只能靠 relay 转发，受限于 relay 的带宽和时间限制
- 需要已知的 relay 节点地址（类似 Tailscale 需要 DERP 服务器）

## 2. 以太坊 Discovery 协议

### discv4

以太坊最初的节点发现协议，基于 **Kademlia DHT**：

**节点地址格式（enode URL）：**
```
enode://<node-id-hex>@<ip>:<tcp-port>?discport=<udp-port>
```
- `node-id` 是 secp256k1 公钥的 hex 编码（64 bytes）
- 用 `keccak256(pubkey)` 的 XOR 距离做 Kademlia 路由

**协议消息（UDP）：**
- **Ping/Pong**：存活检测 + endpoint proof（防放大攻击）
- **FindNode/Neighbors**：Kademlia 递归查找，找到目标附近最近的 16 个节点
- **ENRRequest/ENRResponse**（EIP-868 新增）：获取节点的 ENR 记录

**Kademlia 路由表：**
- 256 个 k-bucket，每个 bucket 最多 16 个节点
- 按 XOR 距离分桶：`2^i ≤ distance < 2^(i+1)`
- 最近通信的节点排在 bucket 尾部（LRU 策略）

**已知问题：**
- 依赖系统时钟（`expiration` 字段用绝对时间戳），时钟不准会导致连接问题
- endpoint proof 不精确，实现之间有差异
- 无加密，容易被被动监听

### discv5

discv4 的演进版本（v5.1），主要改进：

| 特性 | discv4 | discv5 |
|------|--------|--------|
| 通信加密 | ❌ 明文 | ✅ 加密 |
| 时钟依赖 | ✅ 依赖系统时钟 | ❌ 不依赖 |
| 身份加密 | 仅 secp256k1 | 可扩展 |
| Topic 广告 | ❌ | ✅ 可按 topic 搜索节点 |
| 任意元数据 | ❌ | ✅ ENR 可存任意 KV |

**Topic Advertisement** 是 discv5 的核心新功能：
- 节点可以注册"我提供 X 服务"的广告
- 其他节点可以搜索"谁提供 X 服务"
- 这让不同子协议的节点能在同一个 discovery 网络中互相找到

**生产环境表现：**
- discv4 自 2016 年以太坊主网上线以来一直在用，**基本稳定但有坑**（时钟问题、DHT eclipse 攻击）
- discv5 在以太坊 2.0（Beacon Chain）中使用，逐步成熟
- Geth、Lighthouse、Prysm 等主流客户端都已支持 discv5
- 大规模网络（数千节点）下 Kademlia 路由收敛通常在几分钟内完成

### enode vs multiaddr

| | enode URL | multiaddr |
|---|-----------|-----------|
| 格式 | `enode://pubkey@ip:port` | `/ip4/x/tcp/y/p2p/hash` |
| 协议栈 | 固定（TCP+UDP） | 可组合任意协议栈 |
| 身份 | secp256k1 公钥原文 | PeerID（公钥的 multihash） |
| 多地址 | 一个节点一个 enode | 一个节点可多个 multiaddr |
| 传输 | TCP only（数据）、UDP only（发现） | TCP/QUIC/WS/WebTransport... |

两者**没有直接关系**，是不同生态的设计。部分项目（如某些 Ethereum 2.0 客户端）内部用 libp2p，但 discovery 层仍用 discv5。

### Kademlia DHT 的角色

Kademlia 在这些协议中的核心作用是**节点路由**：
- 提供 O(log n) 的节点查找效率
- 每个节点只需维护少量路由信息（~256 × 16 个条目）
- 天然抗节点流失（churn），路由表自动更新
- discv4/v5 用 Kademlia 做**节点发现**（不存任意数据，只存节点记录）
- libp2p 的 Kademlia DHT 还可以存任意 key-value（如 IPFS 的 content routing）

## 3. Tailscale 的打洞方式

### NAT 穿透技术栈

Tailscale 使用**自研方案**，但核心技术来自标准协议：

1. **STUN**：用于发现自己的公网 IP:port（NAT 映射）
2. **协调服务器**（login.tailscale.com）：交换节点的公钥和网络信息（类似 ICE 的 signaling）
3. **同时发包（Simultaneous Transmission）**：双方同时向对方的公网 IP:port 发 UDP 包，打开各自防火墙的状态表
4. **DERP 中继**：打洞失败时的 fallback

**不用标准 ICE/TURN**，而是自研了一套更轻量的方案。

### DERP（Designated Encrypted Relay for Packets）

Tailscale 的中继服务器协议：

- **本质**：加密的 TCP/HTTPS relay，当 UDP 直连失败时转发 WireGuard 包
- **全球部署**：Tailscale 在全球多个区域部署 DERP 服务器
- **加密**：DERP 只转发加密后的 WireGuard 包，relay 服务器看不到明文
- **协调功能**：同时充当 signaling channel，帮助节点交换打洞信息
- **自动选择**：客户端根据延迟自动选择最近的 DERP 服务器
- **可自建**：Tailscale 开源了 DERP 服务器（`cmd/derper`）

### 打洞流程

```
1. 两个节点都连接到 coordination server，交换 WireGuard 公钥
2. 节点通过 STUN 探测自己的公网地址
3. 通过 DERP 交换各自的候选地址（直连地址 + DERP relay 地址）
4. 双方同时向对方发 UDP 探测包（hole punching）
5. 如果直连成功 → 使用直连（最优路径）
6. 如果打洞失败 → 通过 DERP 中继（始终可用的 fallback）
7. 后台持续尝试直连升级
```

### 成功率

Tailscale 官方数据：**~92% 的连接最终实现直连**（不通过 DERP）。剩余 8% 通过 DERP 中继，主要是严格对称 NAT 或企业防火墙场景。

## 4. 横向对比

### NAT 穿透方案对比

| | libp2p Circuit Relay v2 | Ethereum discv4/v5 | Tailscale |
|---|---|---|---|
| **NAT 穿透** | Relay + DCUtR hole punch | 无（仅节点发现） | STUN + hole punch + DERP relay |
| **中继协议** | libp2p relay (TCP/QUIC) | N/A | DERP (TCP/HTTPS) |
| **节点发现** | mDNS / DHT / Bootstrap | Kademlia DHT (UDP) | 中心化协调服务器 |
| **去中心化程度** | 高（任何节点可做 relay） | 高（纯 P2P DHT） | 低（依赖 Tailscale 基础设施） |
| **打洞成功率** | 中等 | N/A（不做打洞） | 高（~92% 直连） |
| **延迟** | relay 时较高 | N/A | 直连时极低（WireGuard） |
| **稳定性** | 一般（relay 有时间/流量限制） | 生产验证（以太坊主网） | 很高（商业级 SLA） |
| **加密** | Noise 协议 | discv5 有 / discv4 无 | WireGuard |

### Tradeoff 三角

```
        去中心化
        /      \
       /        \
    discv5    libp2p
      |          |
      |          |
    稳定性 ———— 延迟
              Tailscale
```

- **Tailscale**：最中心化但最稳定、最低延迟。适合企业/个人 VPN。DERP 是中心化 fallback 但直连路径是纯 P2P。
- **libp2p**：中间路线。去中心化程度高，但 relay 性能受限。适合需要去中心化的应用（IPFS、Aqua）。
- **discv4/v5**：纯去中心化节点发现，不提供 NAT 穿透。假设节点有公网 IP 或自行解决 NAT 问题。适合区块链网络。

### 实际选型建议

- **运维区块链节点**：节点通常有公网 IP，discv4/v5 足够。NAT 穿透不是核心需求。
- **Agent 间通信（如 Aqua）**：需要 NAT 穿透，libp2p relay 是合理选择。
- **设备互联/远程访问**：Tailscale 是最实用的方案，打洞成功率高，体验好。
- **混合方案**：Ethereum 2.0 客户端就是 discv5（发现）+ libp2p（传输）的组合。

## 参考链接

- [libp2p 文档](https://docs.libp2p.io/)
- [Aqua - CLI message tool for AI agents](https://github.com/quailyquaily/aqua)
- [Ethereum devp2p - discv4](https://github.com/ethereum/devp2p/blob/master/discv4.md)
- [Ethereum devp2p - discv5](https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md)
- [How NAT traversal works - Tailscale](https://tailscale.com/blog/how-nat-traversal-works)
- [How Tailscale works](https://tailscale.com/blog/how-tailscale-works)
- [libp2p Circuit Relay](https://github.com/libp2p/specs/tree/master/relay)
- [DCUtR spec](https://github.com/libp2p/specs/blob/master/relay/DCUtR.md)
- [Tailscale DERP source](https://github.com/tailscale/tailscale/tree/main/cmd/derper)

## Takeaway

- libp2p vs webrtc