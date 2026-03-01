---
title: "FineTune - macOS Per-App Volume Mixer"
date: 2026-02-24
source: "https://github.com/ronitsingh10/FineTune"
tags: [macos, audio, swift, open-source]
---

# FineTune - macOS Per-App Volume Mixer

macOS 开源菜单栏应用，提供 per-app 音量控制，免费替代 SoundSource。

## Key Takeaways

### 核心 API
- **`CATapDescription` + `AudioHardwareCreateProcessTap()`** — macOS 14+ Process Tap API，按 PID 拦截指定进程的音频流
- **`AudioHardwareCreateAggregateDevice()`** — 创建 Aggregate Device，将 Process Tap 和输出设备组合
- **`AudioDeviceCreateIOProcIDWithBlock()`** — 实时音频 I/O 回调，做 per-sample 音量缩放、EQ、软限幅
- **DDC/CI (I2C over IOKit)** — 控制外接显示器硬件音量

### 架构
```
AudioEngine (中央协调器)
├── AudioProcessMonitor — 监听系统音频进程
├── AudioDeviceMonitor — 监听设备热插拔
├── ProcessTapController (每个 app 一个)
│   ├── CATapDescription → 拦截音频
│   ├── Aggregate Device → 路由到输出
│   ├── I/O Callback → 音量/EQ/限幅
│   └── CrossfadeState → 设备切换无缝过渡
└── DDCController — 外接显示器音量
```

### 亮点
- 实时回调严格 RT 安全（无锁、无分配、无 ObjC）
- ARM64 对齐原子读写代替锁做跨线程通信
- Per-app 独立设备路由 + 多设备同步输出
- 设备断连自动降级，重连自动恢复
- brew install --cask finetune

## Links
- [GitHub](https://github.com/ronitsingh10/FineTune)
- [推文](https://x.com/tom_doerr/status/2024836850293072233)
