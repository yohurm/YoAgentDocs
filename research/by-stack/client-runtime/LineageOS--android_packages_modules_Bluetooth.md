---
id: research.LineageOS-android_packages_modules_Bluetooth
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [AOSP Gabeldorsche, Bluetooth HCI]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: LineageOS/android_packages_modules_Bluetooth
  url: https://github.com/LineageOS/android_packages_modules_Bluetooth
  head: 778718dad41d6192ef33ea010fb76654b21b33dc
  cloned_to: "%TEMP%/YoAgentResearch/LineageOS--android_packages_modules_Bluetooth"
studied_at: 2026-09-10
related: [research.wireshark-wireshark, research.mauricelam-btsnoop-extcap, research.thelok1s-btsnoop-adb, research.synthesis.client-runtime]
---

# LineageOS/android_packages_modules_Bluetooth（设备侧 HCI snoop）

## 入选理由

Yohu 问的是「实时 HCI 日志」能不能做。设备上的唯一生产者是 AOSP 蓝牙栈的 `SnoopLogger`，不是 logcat。本仓是 AOSP `packages/modules/Bluetooth` 的可克隆镜像（`lineage-23.2`，2026-08-19）。只读 HAL 落盘 / 套接字 / 属性，不读协议栈其余部分。

官方规范：[Verify and debug](https://source.android.com/docs/core/connect/bluetooth/verifying_debugging) 写明 BTSnoop 像 RFC 1761，路径在 `data/misc/bluetooth/logs`，未开「Enable Bluetooth HCI snoop log」时内存环只记非个人字段。

## 项目是什么

Android 主线蓝牙模块（Gabeldorsche）。HCI 抓包在 `system/gd/hal/`：`SnoopLogger` 编排，`SnoopLoggerFile` 写文件，`SnoopLoggerSocket` 在 `127.0.0.1:8872` 推流。

## 架构

```
Controller ↔ Host HCI
        │
        ▼
SnoopLogger::Capture(packet, direction, type)
        │
        ├─ mode == disabled
        │     内存 btsnooz 环（ACL 截到约 14B）
        │     DumpSnoozLogToFile() → btsnooz_hci.log（bugreport 用）
        │
        ├─ mode == kernel
        │     用户态不写
        │
        └─ mode == full | filtered
              ├─ SnoopLoggerFile
              │     /data/misc/bluetooth/logs/btsnoop_hci.log
              │     超限轮转到 *.last
              └─ 仅 is_debug_build()（ro.build.type != user）
                    SnoopLoggerSocketThread
                    bind 127.0.0.1:8872
                    新客户端先发 16 字节文件头，再跟记录
```

### 开关

`GetBtSnoopMode()`（`snoop_logger.cc`）：

1. 默认 `disabled`。
2. `ro.build.type != user` 时，可读 `persist.bluetooth.btsnoopdefaultmode`，缺省 `filtered`。
3. 再覆盖 `persist.bluetooth.btsnooplogmode`（开发者选项「Enable Bluetooth HCI snoop log」写的就是它：`disabled` / `filtered` / `full` / `kernel`）。
4. 改完必须重启蓝牙栈（`svc bluetooth disable && enable`），构造函数只在启动时读属性。

本快照里 **8872 套接字另有一道门**：`is_debug_build()`。注释引用 b/375056207：user 构建要过安全评审才能开套接字。因此市售 `user` 机即使打开 HCI snoop，也只写文件，不听 8872。Insinuator（2026-07）写 Android 17 多了一个「Enable Bluetooth HCI snoop log socket」；**本 lineage-23.2 树里没有这个开关**，不能把 Android 17 当已落地契约。

### 文件路径

`system/gd/os/android/parameter_provider.cc`：

| 产物 | 默认路径 |
|------|----------|
| 全量 / 过滤 snoop | `/data/misc/bluetooth/logs/btsnoop_hci.log`（filtered 加 `.filtered`） |
| 内存环导出 | `/data/misc/bluetooth/logs/btsnooz_hci.log` |

目录属 `bluetooth`，不在 `/sdcard`。无 root 的 `adb pull` 会被 SELinux 拒。官方退路是 `adb bugreport`，zip 里常见 `FS/data/misc/bluetooth/logs/btsnoop_hci.log`（三星等 OEM 偶发 `FS/data/log/bt/`）。

### 记录格式（`Capture` 写出）

文件头（`SnoopLoggerCommon::kBtSnoopFileHeader`，16 字节）：

- magic `btsnoop\0`
- version 1（按端序打包）
- datalink **1002**（HCI UART H4）

每条记录：24 字节大端头 + 载荷。头字段：`original_length` / `included_length` / `flags` / `dropped` / `timestamp`。`flags` bit0 = 方向（0 主机→控制器，1 反方向），bit1 = 命令/事件 vs ACL/SCO/ISO。时间戳 = Unix 微秒 + `0x00dcddb30f2f8000`（年 0 纪元）。H4 载荷第一字节是 HCI 类型（CMD=1 ACL=2 SCO=3 EVT=4 ISO=5）。

套接字与文件写同一套头：新 TCP 客户端先收到 16 字节文件头，之后每包 `Write(header)` + `Write(payload)`。`Send(..., MSG_DONTWAIT)`，拥塞直接丢包（`EAGAIN`），不断栈。

`disabled` 时的 btsnooz：CMD/EVT 全长，ACL 默认只留 HCI+L2CAP 头，上限约 14 字节，SCO/ISO 不记。这就是「没开 snoop 的 bugreport 只有残包」的源码原因。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| HCI ≠ logcat | reuse-pattern | 二进制记录，不能塞进 `yohu-logsrv` 的 `LogLine` |
| 属性 + 重启蓝牙才生效 | reuse-pattern | 探测用 `getprop persist.bluetooth.btsnooplogmode` |
| user 机默认无 8872 | reuse-pattern | 实时套接字必须先探测，不能当默认 |
| `/data/misc/bluetooth/logs` | anti-pattern | 不在 SafetyRoot；禁止为 HCI 放开文件模块安全根 |
| 把 Wireshark 当产品 UI | anti-pattern | 栈只负责产出 btsnoop，展示是另一层 |
| filtered / 隐私截断 | lesson-only | v1 只要 `full`；btsnooz 不能冒充全量 HCI |

## 架构设计经验

设备侧有三条互斥出口，不是一个「开了就能拉」的流：

1. **文件**：`full`/`filtered` 才写；路径受保护。
2. **套接字**：文件开了还不够，还要 debug 构建（本树）或未来的 socket 开发者选项。
3. **btsnooz**：关 snoop 时的隐私环，只适合 bugreport，不适合实时。

主机必须按能力探测，不能假设 8872 在。

## 与当前工作

- 能直接用：属性名、默认路径、btsnoop 头/记录布局、8872 只绑 localhost、改模式后重启蓝牙。
- 必须改写：Yohu 走 sidecar `adb`，不重实现 ADB 线协议；探测用 `adb shell getprop` / `adb forward` 试连，不要抄 androiddump 的 `host:transport` 帧。
- 不要用：把 HCI 当 logcat 过滤；`adb pull /data/misc/...` 当无 root 主路径；为 HCI 扩大 SafetyRoot。

## 阅读范围

`system/gd/hal/snoop_logger.{h,cc}`、`snoop_logger_file.{h,cc}`、`snoop_logger_socket.{h,cc}`、`snoop_logger_socket_thread.{h,cc}`、`snoop_logger_common.h`、`system/gd/os/android/parameter_provider.cc`。未读 dumpstate 如何打进 bugreport zip、Settings 开发者选项 Java、Android 17 socket 开关（本树无）。
