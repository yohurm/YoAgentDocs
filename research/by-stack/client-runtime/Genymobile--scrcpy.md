---
id: research.Genymobile-scrcpy
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C]
  frameworks: [SDL, adb sidecar]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: Genymobile/scrcpy
  url: https://github.com/Genymobile/scrcpy
  cloned_to: "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# Genymobile/scrcpy

## 入选理由

桌面 ADB 工具里最干净的 **「拖入窗口 = adb push」** 实现：SDL 给出本机路径，后台队列串行 `adb push`，目标目录可配，成功后通知设备媒体扫描。维护活跃（浅克隆 HEAD 2026-07）。明确 **不做** 设备→电脑（issue #5585：「Out of scope」）。用来对照「拖入可以很薄」和「拖出完全是另一件事」。

文档：[doc/control.md — File drop](https://github.com/Genymobile/scrcpy/blob/master/doc/control.md)。

## 项目是什么

投屏/控制 Android 的原生窗口。文件能力只是控制通道上的附属：把电脑文件丢进画面。不是文件管理器，没有目录浏览、没有进度条、没有拖出。

## 架构

```
SDL_EVENT_DROP_FILE  (event->data = 本机路径)
        │
        ▼
input_manager_process_file
        │  .apk → INSTALL_APK
        │  其它 → PUSH_FILE
        ▼
file_pusher 队列（mutex + cond，工作线程懒启动）
        ▼
sc_adb_push(serial, local, push_target)   // 默认 /sdcard/Download/
        ▼
成功 → SCAN_FILE 控制消息（媒体库刷新）
```

`sc_adb_push` 就是 `adb -s <serial> push <local> <remote>`，目录也能进，因为 adb 本身递归 push。scrcpy 不做 SafetyRoot、不做进度解析、窗口内零 UI。`--push-target` 原样传给 adb。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 拖入 = 路径列表 + 后台队列 + 已有传输引擎 | reuse-pattern | Yohu 已有 `TransferRunner`；缺的是把 wry 的 paths 接到 `files.push` |
| 工作线程懒启动、退出丢未处理项 | reuse-pattern | 与任务中心登记一致：drop 入队，不堵 UI |
| 固定丢到 `/sdcard/Download`、无进度、无命中测试 | anti-pattern | Yohu 文件页有当前目录；drop 必须落到当前文件夹，且走 `transfer.progress` |
| `.apk` 自动 install | anti-pattern | 文件模块不是包管理；安装是终端/另入口 |
| 不做拖出 | lesson-only | 作者把拖出划出范围，说明双向不是「再加一个事件」 |

## 架构设计经验

- **拖入的最小闭环不经过 HTML。** 原生窗口拿到的是文件系统路径。WebView 上等价物是 Tauri `tauri://drag-drop` 的 `paths`，不是 `DataTransfer.files`（那个没有完整路径）。
- **目标目录是产品决策，不是协议。** scrcpy 用启动参数；Yohu 用当前浏览路径 + `SafetyRoot`。
- 中文路径乱码（#5585）出在 Windows 控制台代码页，不在队列模型。Yohu 走 IPC 传 UTF-8 路径，不要再拼 cmdline 给用户看。

## 与当前工作

- 能直接用：`listen('tauri://drag-drop')` → 命中文件面板 → `fileStore.push(local, basename)`；多文件入现有传输队列。
- 必须改写：目标 = 当前 `session.path`（且 `check_descendant`）；目录要让 `TransferRunner` 接受 `adb push` 目录，而不是 `is_file()` 直接拒绝；命中测试用事件里的 `position`（wry 给的是相对 WebView 坐标）。
- 不要用：无反馈的「丢进窗口就算成功」、APK 安装捷径、把 push 目标写死成 Download。

## 阅读范围

`app/src/file_pusher.c` / `file_pusher.h`、`app/src/input_manager.c`（`sc_input_manager_process_file`）、`app/src/adb/adb.c`（`sc_adb_push`）、`doc/control.md`、`app/src/cli.c`（`--push-target`）。未读投屏/控制协议。
