---
title: "Lifo — 浏览器中的 Linux API 映射层分析"
date: 2026-02-24
tags: ["sandbox", "browser", "linux", "ai-agent"]
---

## 概述

[Lifo](https://lifo.sh/)（[GitHub](https://github.com/lifo-sh/lifo)）是一个将 Linux/Unix 风格 API 映射到浏览器 Web API 的 TypeScript 库，目标场景是 AI agent 代码的浏览器端沙箱执行。MIT 开源。

**核心定位：** 不是 VM、不是模拟器、不是容器——是一个**纯 JS 库**，在浏览器 tab 内提供类 Linux 的 API 表面（API surface）。

## 在哪一层做的 Mapping？

**答案：Node.js 标准库级别 + 命令行工具级别，不是 syscall 级别，也不是 POSIX C API 级别。**

具体来说，Lifo 有两层 mapping：

### 第 1 层：Node.js Compatibility Layer（核心）

在 `packages/core/src/node-compat/` 下，Lifo 对 19 个 Node.js 标准库模块提供了 shim：

| 模块 | 底层 Web API | 完整度 |
|------|-------------|--------|
| `fs` / `fs/promises` | 内存 INode 树 + IndexedDB 持久化 | ✅ 较完整：readFile/writeFile/stat/mkdir/readdir/unlink/rename/symlink/chmod(no-op)/fd 操作等 |
| `path` | 纯 JS 实现 | ✅ 完整 |
| `process` | performance API, 硬编码值 | ⚠️ 部分：argv/env/cwd/stdout/stderr/exit/nextTick/hrtime/memoryUsage 有；chdir 抛异常；platform 返回 `'lifo'` |
| `http` / `https` | Fetch API + 虚拟端口注册表 | ⚠️ 部分：request() 用 fetch 实现；createServer() 是虚拟的（进程内端口映射），不是真正的 HTTP 服务器 |
| `child_process` | Shell 解释器回调 | ❌ 极简：仅 exec() 通过内置 shell 执行；execSync/spawn/fork 全部抛异常 |
| `crypto` | Web Crypto API | ⚠️ 部分：randomBytes/randomUUID/createHash(SHA 系列)/randomInt；无 cipher/sign/verify |
| `buffer` | Uint8Array 包装 | ⚠️ 部分实现 |
| `events` | 纯 JS EventEmitter | ✅ 完整 |
| `stream` | 纯 JS Readable/Writable | ⚠️ 基础实现 |
| `os` | 硬编码值 | ⚠️ stub 为主 |
| `url` | 浏览器 URL API | ✅ 完整 |
| `util` | 纯 JS | ⚠️ 部分 |
| `timers` | setTimeout/setInterval | ✅ 完整 |
| `zlib` | CompressionStream API | ⚠️ 部分 |
| `string_decoder` | TextDecoder | ⚠️ 部分 |
| `querystring` | URLSearchParams | ✅ 完整 |
| `assert` | 纯 JS | ⚠️ 基础实现 |

### 第 2 层：60+ Unix 命令的 JS 重新实现

在 `packages/core/src/commands/` 下，每个命令是一个 async TypeScript 函数，调用第 1 层的 VFS API。不是编译的原生二进制，是纯 JS 实现。

| 类别 | 命令 | 说明 |
|------|------|------|
| 文件系统 | ls, cat, cp, mv, rm, mkdir, rmdir, touch, ln, chmod, chown, stat, file, find, tree, du, df, realpath, dirname, basename, mktemp | 操作 VFS 内存文件系统 |
| 文本处理 | grep, sed, awk, sort, uniq, wc, head, tail, cut, tr, rev, nl, diff | 纯 JS 实现，基本功能可用 |
| I/O | printf, tee, xargs, yes | 基础实现 |
| 网络 | curl, wget, ping, dig | curl/wget 用 Fetch API；ping/dig 是模拟的 |
| 系统信息 | ps, top, kill, env, uname, whoami, hostname, uptime, free, date, cal, bc, sleep, watch, which, man, help | 返回虚拟信息 |
| 压缩 | tar, gzip, gunzip, zip, unzip | 用 CompressionStream / pako |
| 运行时 | node（运行 JS + node-compat）, pkg（包管理） | node 命令接入第 1 层兼容层 |

## 架构层次

```
┌─────────────────────────────────────┐
│  60+ Unix 命令 (纯 JS async 函数)    │  ← 用户交互层
├─────────────────────────────────────┤
│  Node.js Compat Layer (19 个模块)    │  ← API 兼容层
├─────────────────────────────────────┤
│  Bash-like Shell (lexer/parser/AST) │  ← 管道/重定向/变量展开
├─────────────────────────────────────┤
│  VFS (内存 INode 树)                 │  ← "内核"
│  + /proc, /dev 虚拟提供器            │
│  + IndexedDB 持久化                  │
├─────────────────────────────────────┤
│  浏览器 Web API                      │  ← 真正的运行时
│  (IndexedDB, Fetch, Crypto,         │
│   CompressionStream, performance)   │
└─────────────────────────────────────┘
```

## 关键发现

### ✅ 做得不错的

- **VFS 实现较完整**：INode 树支持文件/目录/符号链接/硬链接、权限位（模拟的）、stat 元数据、fd 表、read/write/seek
- **Shell 功能丰富**：管道、重定向、逻辑运算符、变量展开、命令替换、glob、brace expansion、tab 补全、job control
- **fs shim 可用度高**：同步和异步 API 都有，包括 fd 级操作（open/read/write/close/seek）

### ⚠️ 本质限制

1. **不是 syscall 级别的映射**——没有拦截任何 syscall，不能运行编译好的 ELF 二进制
2. **只能运行 JavaScript/TypeScript**——没有原生代码执行能力（WASM 支持在 roadmap）
3. **文件系统是内存中的**——IndexedDB 持久化，但有浏览器配额限制
4. **没有真正的进程隔离**——所有"进程"共享主 JS 线程（Web Workers 可选）
5. **没有原始 socket**——网络只能走 Fetch API，没有 TCP/UDP
6. **child_process 几乎是 stub**——spawn/fork 直接抛异常，exec 只能调内置 shell
7. **http.createServer() 是虚拟的**——在进程内维护端口映射，不开真正的端口
8. **chmod/chown 是 no-op**——权限模型是模拟的

### 🔮 Roadmap 中但尚未实现的

- Git 集成（isomorphic-git）
- 端口隧道（tunnel local ports to real domains）
- 包管理器安装 WASM 运行时（ffmpeg, ImageMagick, SQLite）
- 快照/休眠/恢复
- Node.js / serverless 运行时支持

## 与同类对比

| | Lifo | WebContainers (StackBlitz) | v86/JSLinux |
|---|---|---|---|
| **映射层级** | Node.js API shim | 近完整 Node.js | 真正的 x86 模拟 + Linux 内核 |
| **能运行原生二进制？** | ❌ 只有 JS | ❌ 只有 JS/WASM | ✅ |
| **运行在** | 浏览器 tab | 浏览器 (WASM) | 浏览器 (WASM) |
| **启动时间** | ~0ms | 2-5s | 10-30s |
| **许可证** | MIT | 商业 | GPL |
| **成熟度** | 早期 (2025 weekend project) | 生产可用 | 研究/演示 |

## 总结

Lifo 本质上是一个 **Node.js API polyfill + Unix userland 命令的纯 JS 重写**，运行在浏览器中。它不做 syscall 拦截，不做二进制翻译，mapping 发生在 **JavaScript API 层面**。

对于 AI agent 场景：如果 agent 生成的是 **Node.js 风格的 JS 代码**，并且只需要基础的 fs/path/http 操作，Lifo 可以提供一个零成本、零延迟的浏览器内沙箱。但如果需要运行 Python、系统命令、或任何原生二进制，Lifo 无法胜任。

**一句话：这是一个挺不错的 "Node.js-in-the-browser" shim 库，有 60+ Unix 命令的 bonus，但 "Linux API mapping" 的说法有些过度营销——它映射的是 Node.js API，不是 Linux API。**

## Takeaway

- 垃圾