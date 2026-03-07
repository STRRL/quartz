---
title: "The Slow Death of the Power User"
source: "https://fireborn.mataroa.blog/blog/the-slow-death-of-the-power-user/"
author: "fireborn"
date: 2026-01-01
captured: 2026-03-02
tags: [technology, computing, power-user, platform, philosophy]
---

# The Slow Death of the Power User

## 核心论点

科技公司用二十年时间把用户从「理解工具的人」变成「消费者」，把计算机变成家电，把技术素养变成小众怪癖。Power user 这个物种正在灭绝，而行业在庆祝这场葬礼，还美其名曰"进步"。

## 关键论据

### 1. 一整代人的计算机心智模型是破碎的
不是说不会用——他们用得很熟练。破碎在于理解在玻璃屏幕前就停止了。文件系统、DNS、SSH、本地 IP vs 公网 IP——这些二十年前第一周就要学的东西，现在连很多在职开发者都不知道。

根本原因：框架把这一切抽象掉了，而框架「够用」。Optional until it isn't——直到某天框架救不了你，你连问题出在哪个层都不知道。

### 2. 移动平台是最大推手，且是蓄意为之
Apple 2007 年把一台真正的 BSD Unix 电脑包装成家电：
- 无 user-accessible filesystem（超过十年）
- 无 inter-app 通信（Apple 不开放的部分）
- 无真正的文件所有权（iCloud「优化」本地存储，你的文件在哪里你不知道）
- 无 sideload，App Store 垄断分发且抽 30%

这不是技术局限，是商业决策。Apple 用「用户体验」包装了「控制权转移」。

### 3. 开发者也未能幸免
能一路做到生产的开发者：
- 从未开过 Wireshark（免费存在几十年）
- 从未看完整 stack trace 的每一帧
- 不知道自己的 App 为什么发了 20 个 network request 而不是 3 个
- 不知道 Wireshark 这样的工具存在

因为框架处理了网络层，框架「够好」，理解底层是可选的。直到不再可选。

### 4. 两个核心转变
- 用户 → 消费者
- 工具 → 家电（appliance）

结果：一代人无法解压 zip 文件而不借助专门 App，行业还把这叫创新。技术素养从必备技能变成了小众怪癖，懂的人开始说「抱歉，这只是我个人爱好」。


## 延伸思考

这篇文章和 [Houseplant Programming](./houseplant-programming.md) 构成有趣的对话：houseplant programming 是对这种趋势的一种个人抵抗——用「只为自己写软件」来保持对工具的理解和掌控。

也与 [The Case for Betting on Youth](./betting-on-youth.md) 中那些创始人的早期经历形成对比——他们大多是在「工具还没有被简化成家电」的年代长大的，那种深入理解工具的能力塑造了他们后来的判断力。

## Takeaway

For developers who want to build products and make money:
- Ride the wave: users pay for "not needing to understand", the appliance-ification IS the product opportunity
- Go niche: power user market is small but high willingness-to-pay and loyalty (Raycast, Obsidian, Linear), big companies don't bother, less competition
- Either way, stay a power user yourself: deep understanding of the stack is your edge against big companies with more headcount
- "Optional until it isn't" hits product builders earlier and harder: if you don't understand what's under the hood, your product quality has a ceiling
