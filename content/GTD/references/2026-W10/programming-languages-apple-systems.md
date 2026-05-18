---
title: "编程语言、Apple 生态与系统架构"
date: 2026-03-08
tags: [programming, apple, macos, shortcuts, cpp, systems]
gtd_week: 2026-W10
gtd_items: [ZZ-100, ZZ-113, ZZ-58]
---

# 编程语言、Apple 生态与系统架构

本文归档了三条关于编程语言设计、Apple Shortcuts 扩展以及 macOS 应用结构的 Reference 内容。

---

## ZZ-100 — 有史以来最烂的编程语言 C++

**来源：** [YouTube — Lazo Velko](https://www.youtube.com/watch?v=7fGB-hjc2Gc)（也可在 Bilibili 搜索 Lazo Velko）

C++ 以"向后兼容 C"为设计原则，在几十年的迭代中积累了大量历史包袱：指针算术、未定义行为（UB）、复杂的模板系统、`#include` 机制等。这个视频/系列从批判角度梳理了 C++ 的设计失误。

**核心槽点：**
- 未定义行为（Undefined Behavior）渗透整个语言规范，导致编译器可以"合法地"生成反直觉代码
- 头文件与 `#include` 的 copy-paste 模型是远古产物，C++20 Modules 才开始解决
- 运算符重载滥用、多重继承、虚函数 + 菱形继承问题
- 模板错误信息堪称天书

**延伸思考：** 对比 Rust（所有权系统消灭 UB）、Go（刻意简化）的设计哲学，C++ 的复杂性很大程度上是历史债务，而非必然选择。

---

## ZZ-113 — Apple Shortcuts 的"编程语言"：Jelly

**来源：**
- Telegram 频道讨论：[t.me/dinahzhang/290](https://t.me/dinahzhang/290)
- Jellycuts 文档：[docs.jellycuts.com](https://docs.jellycuts.com/)

Apple Shortcuts（捷径）是一个图形化自动化工具，但对于复杂逻辑来说，拖拽方块很快变得难以维护。**Jelly** 是一门专为编写 Siri Shortcuts 设计的文本编程语言，可通过 **Jellycuts** IDE 编写后编译为 Shortcuts。

**Jelly 语言特点：**
- 以函数调用为核心结构（对应 Shortcuts 的每个 Action）
- 标准库直接映射 Shortcuts 内置动作（Shortcuts Standard Library）
- 支持变量、条件、循环等基础控制流
- 可从文本代码生成可执行的 `.shortcut` 文件

**使用场景：** 需要版本控制 Shortcuts、在 Shortcuts 中实现复杂逻辑、或者对图形化界面过敏的开发者。

---

## ZZ-58 — Anatomy of a macOS App

**来源：** [eclecticlight.co — The Anatomy of a macOS App](https://eclecticlight.co/2025/12/04/the-anatomy-of-a-macos-app/)

这篇文章从历史演进角度讲解 macOS 应用程序的内部结构。

**历史沿革：**
- **Classic Mac OS**：使用 Resource Fork 存储 UI 资源、可执行 CODE 资源，工具如 ResEdit 可直接编辑
- **Mac OS X（NeXTSTEP 继承）**：引入 Bundle 结构，`.app` 本质是一个目录

**macOS App Bundle 标准结构：**
```
MyApp.app/
└── Contents/
    ├── MacOS/          # 主可执行文件 + 命令行工具
    ├── Resources/      # 图标、本地化文件、GUI 资源
    ├── Frameworks/     # 私有 Framework
    ├── PlugIns/        # 扩展
    └── Info.plist      # 元数据（Bundle ID、版本、权限声明等）
```

**关键点：**
- `Info.plist` 是 App 的"身份证"，声明 Bundle ID、最低系统版本、所需权限（entitlements）
- Finder 通过 `.app` 扩展名将目录伪装成单一文件，用户无感知
- 代码签名（Code Signing）覆盖整个 Bundle，任何文件改动都会导致签名失效
- Universal Binary（`lipo` 合并 arm64 + x86_64）存放在 `MacOS/` 目录中

**延伸：** 理解 Bundle 结构对调试代码签名问题、分析 App 内部资源、开发 App Extension 都有直接帮助。

---

## 关联主题

- C++ → Rust 迁移的理由
- Apple Shortcuts 自动化开发工具链
- macOS 应用打包、签名、公证（Notarization）流程
