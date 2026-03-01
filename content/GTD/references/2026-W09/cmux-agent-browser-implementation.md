---
title: "cmux agent-browser 实现分析"
date: 2026-02-24
tags: ["swift", "macos", "browser-automation", "agent-browser"]
---

## 概述

[cmux](https://github.com/manaflow-ai/cmux) 是一个纯 Swift + AppKit 的 macOS 原生终端应用，内嵌了浏览器自动化 API，其接口设计参考了 Vercel 的 [agent-browser](https://github.com/vercel-labs/agent-browser)。但实现方式完全不同——**没有嵌入 Node.js/Playwright，而是用 WKWebView + JavaScript 注入全部重写**。

## 核心架构

```
┌─────────────────────────────────────────────────┐
│  CLI (cmux browser snapshot / click / fill ...)  │
│  (独立二进制，Swift ArgumentParser)               │
└──────────────────┬──────────────────────────────┘
                   │ Unix Domain Socket (/tmp/cmux.sock)
                   │ JSON-RPC v2 协议
                   ▼
┌─────────────────────────────────────────────────┐
│  TerminalController (Sources/TerminalController.swift) │
│  - 监听 socket，路由 "browser.*" 方法              │
│  - 管理 element ref 映射表                        │
│  - 构造 JS 脚本并注入 WKWebView                   │
└──────────────────┬──────────────────────────────┘
                   │ WKWebView.evaluateJavaScript()
                   ▼
┌─────────────────────────────────────────────────┐
│  CmuxWebView (WKWebView 子类)                    │
│  - 嵌入在 BrowserPanel 中                        │
│  - 实际的网页渲染和 JS 执行                       │
└─────────────────────────────────────────────────┘
```

### 1. IPC 层：Unix Domain Socket + JSON-RPC

- `TerminalController` 在 `/tmp/cmux.sock` 上监听
- CLI 工具 (`CLI/cmux.swift`) 通过 `sendV2(method:params:)` 发送 JSON-RPC 请求
- 所有浏览器操作都是 `browser.*` 命名空间的方法，如 `browser.snapshot`、`browser.click`、`browser.fill` 等
- 每个请求带 `surface_id` 来定位具体的浏览器面板

### 2. 浏览器渲染：WKWebView

- **不是** Chromium/Playwright，而是 macOS 原生的 **WKWebView**（WebKit 引擎）
- `CmuxWebView` 是 `WKWebView` 的子类，处理键盘快捷键路由、右键菜单、焦点管理等
- `BrowserPanel` 管理 WKWebView 的生命周期、导航、历史记录、favicon 等
- 共享 `WKProcessPool` 实现跨面板 cookie 共享
- UA 伪装成 Safari：`Mozilla/5.0 ... Version/26.2 Safari/605.1.15`

### 3. Accessibility Tree Snapshot（核心创新）

**不依赖任何原生 accessibility API**，而是通过 **JavaScript 注入** 来构建"伪 accessibility tree"。

`v2BrowserSnapshot()` 的实现：
1. 构造一个大段 JS 脚本（约 150 行）注入页面
2. JS 遍历 DOM 树（`document.body` 开始递归 `element.children`）
3. 对每个元素：
   - 检查可见性（`getComputedStyle`、`getBoundingClientRect`）
   - 确定 ARIA role（先看 `role` 属性，再根据 tag 推断隐式 role）
   - 提取 accessible name（`aria-label` → `aria-labelledby` → `placeholder` → `title` → `innerText`）
   - 生成 CSS selector 路径（`#id` 或 `tag:nth-of-type(n) > ...`）
4. 返回结构化数据，Swift 端格式化为文本树：
   ```
   - document "Page Title"
     - heading "Welcome" [ref=e0]
     - textbox "Search" [ref=e1]
     - button "Submit" [ref=e2]
   ```

**关键参数：**
- `--interactive` / `-i`：只返回可交互元素（button, link, textbox, checkbox 等）
- `--compact`：过滤掉没有 name 的结构性 role
- `--cursor`：额外包含有 `cursor: pointer` / `onclick` / `tabindex` 的元素
- `--max-depth`：限制遍历深度（默认 12）
- `--selector`：限制 scope 到某个 CSS 选择器

### 4. Element Ref 系统

- 每次 snapshot 为每个元素分配递增编号 `@e0`, `@e1`, `@e2` ...
- 内部维护 `v2BrowserElementRefs: [String: V2BrowserElementRefEntry]` 映射表
- 每个 ref 记录 `surfaceId` + `CSS selector`
- 后续操作（click、fill 等）可以用 ref 或直接用 CSS selector

### 5. 交互操作实现

所有交互操作都是 **构造 JS → evaluateJavaScript → 同步等待结果**：

| 操作 | JS 实现方式 |
|------|------------|
| `browser.click` | `el.click()` 或 `dispatchEvent(new MouseEvent('click'))` |
| `browser.fill` | 先 focus，清空 value，设新值，触发 input/change 事件 |
| `browser.type` | 逐字符 `dispatchEvent(new KeyboardEvent(...))` |
| `browser.press` | `dispatchEvent(new KeyboardEvent('keydown/keyup'))` |
| `browser.hover` | `dispatchEvent(new MouseEvent('mouseover/mouseenter'))` |
| `browser.check/uncheck` | 设置 `checked` 属性 + 触发 change 事件 |
| `browser.select` | 设置 `<select>.value` + 触发 change 事件 |
| `browser.scroll` | `window.scrollBy()` 或 `element.scrollBy()` |
| `browser.wait` | 轮询 JS 条件表达式，支持 selector/text/url/自定义函数 |
| `browser.eval` | 直接执行任意 JS |
| `browser.screenshot` | WKWebView 截图 API |

**JS 执行是同步阻塞的**：用 RunLoop polling 等待 `evaluateJavaScript` 回调完成（最多 10s 超时）。

### 6. iframe 支持

- `browser.frame.select` 设置当前 frame selector
- 后续 JS 执行会先 `document.querySelector(frameSelector).contentDocument`，在 iframe 内执行
- `browser.frame.main` 恢复到主文档

### 7. 完整的 API 列表

导航：`open`, `navigate`, `back`, `forward`, `reload`, `url`, `wait`
快照：`snapshot`, `screenshot`
交互：`click`, `dblclick`, `hover`, `focus`, `type`, `fill`, `press`, `keydown`, `keyup`, `check`, `uncheck`, `select`, `scroll`, `scroll_into_view`
查询：`get text/html/value/attr/title/count/box/styles`, `is visible/enabled/checked`, `find role/text/label/placeholder/alt/title/testid/first/last/nth`
Frame：`frame.select`, `frame.main`
Dialog：`dialog.accept`, `dialog.dismiss`
Cookie/Storage：`cookies.get/set/clear`, `storage.get/set/clear`
Tab 管理：`tab.new/list/switch/close`
其他：`eval`, `highlight`, `console.list/clear`, `errors.list`, `addinitscript`, `addscript`, `addstyle`, `viewport.set`, `download.wait`, `state.save/load`

## 与原版 agent-browser (Node.js/Playwright) 的对比

| 维度 | cmux | agent-browser (Vercel) |
|------|------|----------------------|
| 运行时 | Swift + WKWebView (WebKit) | Node.js + Playwright (Chromium) |
| 浏览器引擎 | macOS 系统 WebKit | Chromium (可选 Firefox/WebKit) |
| Accessibility Tree | JS 注入遍历 DOM，模拟 ARIA tree | Playwright 原生 `page.accessibility.snapshot()` (CDP) |
| 元素操作 | JS `evaluateJavaScript` 注入 | Playwright 原生 API（用 CDP 协议） |
| 元素定位 | CSS Selector + 自增 ref 编号 | Playwright Locator（支持 role/text/testid 等高级选择器） |
| 事件模拟 | JS `dispatchEvent`（应用层） | CDP 输入事件（OS 层，更真实） |
| 无头模式 | 不支持（必须有 GUI 窗口） | 支持 |
| IPC | Unix Domain Socket + JSON-RPC | 进程内 API 调用 |
| 独立性 | 嵌入在终端 app 内 | 独立 Node.js 包 |
| 跨平台 | macOS only | 跨平台 |

## 关键差异和限制

1. **事件模拟层级不同**：cmux 的 click/type 是 JS `dispatchEvent`，无法触发浏览器原生行为（如 `<a>` 的导航、文件上传对话框等）。Playwright 通过 CDP 在更底层模拟输入。

2. **Accessibility tree 是"模拟的"**：不是真正的 accessibility tree（没有调用 macOS Accessibility API 或 WebKit 内部的 AX tree），而是 JS 遍历 DOM + 推断 ARIA role。对于复杂的 ARIA widget（如 `aria-owns`、shadow DOM）可能不完整。

3. **同步执行模型**：所有 JS 执行用 RunLoop polling 等待，会阻塞主线程。对于长时间运行的 JS 或大页面 snapshot 可能有性能问题。

4. **WebKit vs Chromium**：某些网站的兼容性可能不同（WebKit 是 Safari 内核）。

## 总结

cmux 的 agent-browser 是一个"全 JS 注入"的实现方案：
- **没有嵌入任何 JavaScript 运行时**（不是 JavaScriptCore standalone，而是利用 WKWebView 自带的 JS 引擎）
- **没有用 CDP/DevTools Protocol**
- **所有操作都是构造 JS 字符串 → `WKWebView.evaluateJavaScript()` → 解析结果**
- API 设计高度参考 agent-browser/Playwright，但底层实现完全不同
- 架构上通过 Unix Socket IPC 让 CLI 工具和终端内 agent 能远程操控嵌入的浏览器

这是一个工程上很聪明的方案：利用 macOS 自带的 WebKit 引擎，避免了打包 Chromium 的巨大体积，同时通过 JS 注入实现了 Playwright 大部分功能的等价物。代价是事件模拟的真实度较低，以及受限于 WKWebView 的 API 边界。

## Takeaway

- Ghostty + Webview
- Good Reference