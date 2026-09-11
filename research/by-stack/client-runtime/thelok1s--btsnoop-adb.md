---
id: research.thelok1s-btsnoop-adb
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Rust]
  frameworks: [adb CLI, libpcap]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: thelok1s/btsnoop-adb
  url: https://github.com/thelok1s/btsnoop-adb
  head: 2e8994f95e2107122661c3e65569c9478b2d0864
  cloned_to: "%TEMP%/YoAgentResearch/thelok1s--btsnoop-adb"
studied_at: 2026-09-10
related: [research.mauricelam-btsnoop-extcap, research.synthesis.client-runtime]
---

# thelok1s/btsnoop-adb（Rust 解析 + pcap）

## 入选理由

最小的「ADB 二进制流 → 解析记录 → 主机格式」闭环，和 Yohu 同语言。2026-07 把 `cat` 改成 `tail -c +1 -f`，正好踩中实时语义。Star 为 0，当对照实现，不当产品对标。

## 项目是什么

CLI：探路径 → `adb shell [su -c] tail -c +1 -f <path>` → `BtsnoopReader` → libpcap（LINKTYPE 187）→ Wireshark stdin 或文件。

## 架构

```
probe: adb shell su -c 'dd if=<path> bs=8 count=1'
         stdout 是否以 btsnoop\0 开头
  候选：
    /data/misc/bluetooth/logs/btsnoop_hci.log
    /data/misc/bluetooth/btsnoop_hci.log
  → AdbReader（Child stdout）
  → BtsnoopReader::new 校验 16 字节头
  → 循环 next_record()（24 字节大端头 + 载荷）
  → timestamp = raw_ts - 0x00dcddb30f2f8000
  → pcap 全局头 + 每包 16 字节头
```

`btsnoop.rs` 把格式写成可测契约：magic、version、DLT 1001/1002、flags bit0 方向。单测用合成字节流，不依赖真机。

`main.rs` 注释仍写 `cat`，实现已是 `tail`。以源码为准。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 先读 8 字节 magic 再开流 | reuse-pattern | 套接字档 / 文件档共用 |
| 流式记录解析 + 年 0 纪元 | reuse-pattern | 可原样搬进 `yohu-hci`，或依赖 `btsnoop` crate |
| `su -c` 与 `adb root` 分开关 | adapt | 探测两种提权 |
| 默认交给 Wireshark | anti-pattern | Yohu 是工作台模块，不是抓包前端 |
| 只做文件档 | lesson-only | 不覆盖 8872；Yohu 要两档并列 |

## 架构设计经验

主机解析层应与运输层切开：`BtsnoopReader<R: Read>` 不关心 R 是 TCP 还是 `adb shell` stdout。Yohu 应对 `TcpStream`（forward 8872）和 `ChildHandle` stdout（tail）复用同一解析器。

`dd` 探路径比盲 `tail` 便宜。magic 不对就换路径或降级到 bugreport。

## 与当前工作

- 能直接用：路径表、magic 探测、记录布局、`tail -c +1 -f`。
- 必须改写：输出到协议事件，不是 pcap stdin；解析器放 core，不放 CLI。
- 不要用：安装 Wireshark；把 pcap 当 IPC。

## 阅读范围

`src/{main,adb,btsnoop,pcap}.rs`、`README.md`。未读 `wireshark.rs` 的 Windows 路径探测细节。
