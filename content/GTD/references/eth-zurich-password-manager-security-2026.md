---
title: "ETH Zurich 云密码管理器安全分析 (2026)"
date: 2026-02-17
tags: ["security", "password-manager", "cryptography"]
---

# Zero Knowledge (About) Encryption: 云密码管理器安全分析

## 来源
- 推文: https://x.com/intcyberdigest/status/2023537806325215302
- 论文: https://eprint.iacr.org/2026/058 (USENIX Security '26)
- 作者: Matteo Scarlata, Giovanni Torrisi, Matilda Backendal, Kenneth G. Paterson (ETH Zurich + USI)

## 核心内容
在"恶意服务器"威胁模型下，测试主流密码管理器的"零知识加密"是否名副其实。

## 发现
- **Bitwarden**: 12 个攻击向量（中低风险）
- **LastPass**: 7 个攻击向量
- **Dashlane**: 6 个攻击向量
- **1Password**: 完整版论文包含额外分析

攻击严重程度从单个用户 vault 完整性破坏到组织内所有 vault 完全泄露，大部分可恢复密码明文。

## 1Password 具体情况
- 端到端加密未被突破（需要账户密码 + Secret Key + 加密数据三要素）
- 问题在于公钥分发缺乏强验证机制：恶意服务器可在共享 vault / 添加成员时伪造公钥做中间人攻击
- 1Password 称此为已知架构限制（Security Design White Paper 附录 C 已披露）
- 不是被动泄露已有数据，而是在特定操作时可被 hijack

## 各家回应 (截至 2026-02)
- **Bitwarden**: 主动参与研究，所有问题已修复 ([博客](https://bitwarden.com/blog/security-through-transparency-eth-zurich-audits-bitwarden-cryptography/))
- **1Password**: 称无新攻击面 ([博客](https://1password.com/blog/eth-zurich-zero-knowledge-malicious-server-review))
- **LastPass**: [回应](https://blog.lastpass.com/posts/details-on-hardening-in-response-to-eth-zurich-reported-security-issues)
- **Dashlane**: [回应](https://www.dashlane.com/blog/zero-knowledge-malicious-server)

## 结论
前提是服务器被完全入侵（历史上未发生过），属于学术级极端威胁建模。日常使用仍安全，但论文揭示了 E2EE 系统设计中的常见反模式。
