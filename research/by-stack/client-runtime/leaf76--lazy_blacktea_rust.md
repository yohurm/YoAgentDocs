---
id: research.leaf76-lazy_blacktea_rust
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [typescript, rust]
  frameworks: [tauri]
also_relevant: []
utilization: [lesson-only, anti-pattern]
source:
  platform: github
  repo: leaf76/lazy_blacktea_rust
  url: https://github.com/leaf76/lazy_blacktea_rust
  head: aecc469c4787057a667ca3a2d4393cb5b69c49fc
  cloned_to: "%TEMP%/YoAgentResearch/leaf76--lazy_blacktea_rust"
studied_at: 2026-09-11
related: []
---

# leaf76/lazy_blacktea_rust

## 入选理由

与 Yohu 同栈（Tauri 2 + Rust + WebView）的 ADB 工作台，且已有「命令库」结构。用来确认同代工具把自定义命令建成什么样——以及为什么单字符串模型撑不住命令块。

## 项目是什么

设备 / logcat / 文件 / APK / 无线配对 / scrcpy 拉起的桌面控制台。命令库是设置里的 pack：预置导入 + `custom_commands`。

## 架构

```
AdbCommandLibraryCommand
  id, title, category, command: String, description, tags[], risk
  → 一条 command 字符串，无 steps、无 gap
Settings.adb_command_library.custom_commands
  → 导入 pack / 收藏 id
```

前后端各有一份结构（`src/types.ts` 与 `src-tauri/src/app/config.rs`）。没有序列编排器；一次点击跑一条字符串。`interval_ms` 出现在性能监控和设备刷新，与命令库无关。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 同栈仍把命令建成单行 | lesson-only | 说明「命令库」默认不会长出块；Yohu 要显式改模型，不能只改编辑器 |
| tags / risk / description / category | anti-pattern | Yohu v1.85 已删这些；不要借命令块加回来 |
| 设置里堆命令库 | anti-pattern | Yohu 命令库在 `data/modules/adb-terminal/config/library.json`，不进 settings |

## 架构设计经验

Tauri ADB 壳可以只做「标签 + 一行」。一旦产品要「自动执行多条 + 选间隔」，单字段 `command: String` 会逼出换行 DSL 或双轨 `template`/`steps`。正确做法是一次换成 steps + gap，而不是在设置里加可选字段。

## 与当前工作

- 能用：对照「不要把命令库做成带风险标签的设置项」。
- 必须改：Yohu 已有组/命令/template，块是命令身体的升级，不是新设置页。
- 不要用：拉起外部 `scrcpy`、把间隔复用性能监控的 `interval_ms`、为将来预留 tags。

## 阅读范围

`src/types.ts`（`AdbCommandLibraryCommand`）、`src/adbCommandLibrary.ts`、`src-tauri/src/app/config.rs` 命令库段。未读 logcat/文件模块实现。
