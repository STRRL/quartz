---
title: "ERC-4337: Account Abstraction"
date: 2026-02-24
tags: ["ethereum", "account-abstraction", "erc-4337", "smart-wallet"]
---

# ERC-4337: Account Abstraction（账户抽象）

## 1. ERC-4337 到底做了什么？核心机制是什么？

ERC-4337 实现了**不修改以太坊共识层协议**的账户抽象方案。

核心思路：在应用层引入一套平行于传统交易的新流程：

1. 用户构造 **UserOperation**（伪交易对象），发送到专用的 **alt mempool**（替代内存池）
2. **Bundler**（打包者）从 mempool 收集多个 UserOperation，打包成一笔普通交易
3. 这笔交易调用链上单例合约 **EntryPoint** 的 `handleOps()` 方法
4. EntryPoint 依次验证和执行每个 UserOperation

**核心价值**：让智能合约账户成为一等公民，用户不再需要持有 EOA 和 ETH 就能发起交易。

## 2. 和传统 EOA 的区别

| 特性 | EOA（外部账户） | ERC-4337 智能账户 |
|------|----------------|-------------------|
| 验证方式 | 固定 secp256k1 签名 | **任意验证逻辑**（passkey、多签、社交恢复等） |
| Gas 支付 | 必须自己用 ETH 支付 | 可由 **Paymaster 代付**，支持 ERC-20 代币支付 |
| 交易能力 | 单笔操作 | **批量操作**（一次调用多个合约） |
| 账户恢复 | 私钥丢失 = 资产丢失 | 可编程恢复机制（社交恢复、时间锁等） |
| 创建要求 | 需要私钥生成 | 可通过 Factory 合约创建，**无需预先持有 ETH** |
| 升级性 | 不可升级 | 可通过代理模式升级逻辑 |
| 权限控制 | 全有或全无 | 可设置 **session key**、权限分级、支出限额 |

## 3. 智能合约钱包原理：核心组件

### UserOperation（用户操作）

类似交易的数据结构，但由智能合约账户发起。关键字段：

- `sender` — 智能合约账户地址
- `nonce` — 防重放（支持 192-bit key + 64-bit sequence 的半抽象 nonce）
- `callData` — 要执行的操作数据
- `signature` — 签名（格式由账户合约自定义，不限于 ECDSA）
- `factory` / `factoryData` — 首次创建账户时的工厂合约信息
- `paymaster` / `paymasterData` — 代付 gas 的信息
- Gas 相关字段：`callGasLimit`, `verificationGasLimit`, `preVerificationGas`, `maxFeePerGas`, `maxPriorityFeePerGas`

### Bundler（打包者）

- 运行专用节点，监听 UserOperation mempool
- 验证 UserOperation 的有效性（模拟执行）
- 将多个 UserOperation 打包成一笔交易提交上链
- 承担 gas 成本风险，由用户账户或 Paymaster 补偿
- 无需信任假设，任何人都可以运行 Bundler

### EntryPoint（入口点合约）

- **全局单例合约**，是整个系统的信任核心
- 核心方法：`handleOps(PackedUserOperation[] calldata ops, address payable beneficiary)`
- 执行流程：
  1. **验证阶段**：调用每个账户的 `validateUserOp()` 验证签名和支付
  2. **执行阶段**：调用账户合约执行 `callData`
- 已经过审计和形式化验证
- EntryPoint v0.7 是当前最新版本

### Paymaster（付费者）

- 代替用户支付 gas 费用的合约
- 使用场景：
  - **Gas 赞助**：dApp 为用户代付 gas（免费体验）
  - **ERC-20 支付**：用户用 USDC 等代币支付 gas
  - **跨链 gas**：在 A 链支付 B 链的 gas
- 需要在 EntryPoint 存入 ETH 作为保证金
- 实现 `validatePaymasterUserOp()` 和可选的 `postOp()`

### Factory（工厂合约）

- 负责在用户首次操作时部署智能合约账户
- 使用 `CREATE2` 确保地址可预测（可在部署前就接收资产）
- 通过 UserOperation 的 `factory` 和 `factoryData` 字段触发

### 完整流程

```
用户 → 构造 UserOperation → alt mempool → Bundler 收集并验证
  → Bundler 打包成交易 → 调用 EntryPoint.handleOps()
    → EntryPoint 验证阶段：
      1. 如有 factory，先部署账户
      2. 调用账户的 validateUserOp()
      3. 如有 paymaster，调用 validatePaymasterUserOp()
    → EntryPoint 执行阶段：
      4. 调用账户执行 callData
      5. 如有 paymaster，调用 postOp()
    → Gas 费用结算给 beneficiary
```

## 4. 实际应用场景

### 通用场景

- **无 gas 入门**：新用户无需购买 ETH，Paymaster 代付 gas
- **社交登录**：用 passkey、Google 账号等创建钱包（自定义签名验证）
- **批量交易**：approve + swap 合并为一笔交易
- **自动化操作**：定时转账、自动止损（通过 session key）
- **多签钱包**：团队/DAO 资金管理
- **账户恢复**：社交恢复、guardian 机制
- **支出限额**：每日/每笔限额，类似银行卡

### AI Agent 场景 🤖

ERC-4337 对 AI agent 特别有价值：

1. **Session Keys（会话密钥）**：
   - 给 AI agent 颁发受限的临时密钥
   - 限制：可调用的合约、最大金额、有效期
   - Agent 可自主操作，但在安全边界内

2. **可编程权限**：
   - 不同 AI agent 可有不同权限级别
   - 例：交易 agent 只能操作 DEX，不能转账到外部地址

3. **自动化执行**：
   - Agent 可批量执行策略（DeFi 套利、portfolio rebalancing）
   - 通过 Paymaster 无需 agent 持有 ETH

4. **多 Agent 协作**：
   - 多签机制 → 多个 agent 需要共同确认高风险操作
   - 一个 agent 提议，其他 agent 或人类批准

5. **审计和可追溯**：
   - 所有操作通过 UserOperation 上链，完全可审计
   - 可设置人类 override/暂停机制

## 5. 当前生态现状

### 基础设施

| 项目 | 角色 | 说明 |
|------|------|------|
| **eth-infinitism** | EntryPoint 官方实现 | Vitalik 团队，核心合约 |
| **Alchemy** | Bundler + SDK | Rundler（Rust bundler）、Account Kit |
| **Pimlico** | Bundler + Paymaster | Alto bundler、permissionless.js |
| **Biconomy** | 智能账户 + SDK | Nexus 智能账户、Paymaster 服务 |
| **ZeroDev** | Kernel 智能账户 | 模块化账户、session key 支持好 |
| **thirdweb** | SDK + 账户抽象 | 开发者工具链，易用 |
| **Safe** | 智能合约钱包 | 老牌多签钱包，已支持 4337 |
| **Stackup** | Bundler | Go 实现的 bundler |

### Adoption 情况

- **L2 主导**：大部分 ERC-4337 活动在 L2（Base、Polygon、Arbitrum、Optimism）
- Ethereum 主网由于 gas 成本高，智能账户操作开销较大
- 2023-2025 年 UserOperation 数量持续增长，尤其在 Base 和 Polygon 链上
- 主要用户场景：游戏、社交应用、DeFi 入门

### SDK/开发工具

- **viem** + **permissionless.js**（Pimlico）— 类型安全的 AA 库
- **Alchemy Account Kit** — 全套 AA 工具
- **ZeroDev SDK** — Kernel 账户 + session key
- **Biconomy SDK** — 智能账户 + Paymaster
- **userop.js** — 轻量级 UserOperation 构建库

## 6. 和 ERC-7702 等新提案的关系

### EIP-7702（Pectra 升级，2025 年上线）

EIP-7702 是**协议层**的变更，允许 EOA 在单笔交易中临时委托给智能合约代码执行：

| | ERC-4337 | EIP-7702 |
|---|---------|---------|
| 层级 | 应用层（无需协议变更） | 协议层（共识变更） |
| 账户类型 | 纯智能合约账户 | EOA 临时获得智能合约能力 |
| 用户迁移 | 需要新地址 | **现有 EOA 地址不变** |
| 功能 | 完整 AA（自定义验证、Paymaster 等） | 批量交易、gas 赞助、有限的自定义逻辑 |
| 持久性 | 账户逻辑永久存在 | 每笔交易临时委托（交易结束后 EOA 恢复） |
| 独立性 | 用户完全不需要 EOA | 仍然**需要 EOA 签名**来授权委托 |

### 互补关系

ERC-4337 和 EIP-7702 **不是竞争关系，而是互补**：

1. **ERC-4337 + EIP-7702**：ERC-4337 v0.8 已原生支持 EIP-7702，可以让现有 EOA 通过 7702 委托来使用 4337 的基础设施（Bundler、Paymaster 等）
2. **迁移路径**：用户可以先用 7702 升级现有 EOA，未来再迁移到纯 4337 智能账户
3. **长期愿景**：Vitalik 的路线图是最终实现 Native Account Abstraction（RIP-7560），将 4337 的机制内置到协议层

### 其他相关提案

- **ERC-6900**：模块化智能账户标准（插件架构）
- **ERC-7579**：模块化智能账户的最小接口标准
- **ERC-7562**：UserOperation mempool 的验证规则
- **RIP-7560**：Native Account Abstraction（协议层原生 AA，长期目标）
- **ERC-7677**：Paymaster Web Service 标准接口

## 参考资料

- [EIP-4337 原文](https://eips.ethereum.org/EIPS/eip-4337) — 作者：Vitalik Buterin 等
- [ERC-4337 官方文档](https://docs.erc4337.io/)
- [Alchemy: What is ERC-4337?](https://www.alchemy.com/overviews/what-is-account-abstraction)
- [Alchemy: EIP-3074 vs EIP-7702 vs ERC-4337](https://www.alchemy.com/overviews/eip-3074-vs-eip-7702-vs-erc-4337)
- [thirdweb: Account Abstraction ERC-4337](https://blog.thirdweb.com/account-abstraction-erc4337/)
- [QuickNode: Account Abstraction and ERC-4337 Guide](https://www.quicknode.com/guides/ethereum-development/wallets/account-abstraction-and-erc-4337)

## Takeaway

- Metamask + Thirdweb paymaster
- need dApp's support
	- for now, uniswap does not support it