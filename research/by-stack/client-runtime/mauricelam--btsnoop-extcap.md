---
id: research.mauricelam-btsnoop-extcap
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Rust]
  frameworks: [tokio, extcap, adb CLI]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: mauricelam/btsnoop-extcap
  url: https://github.com/mauricelam/btsnoop-extcap
  head: 76df8240b27a7dbd1064c7c4d853bcd7cf1a03e2
  cloned_to: "%TEMP%/YoAgentResearch/mauricelam--btsnoop-extcap"
studied_at: 2026-09-10
related: [research.thelok1s-btsnoop-adb, research.LineageOS-android_packages_modules_Bluetooth, research.synthesis.client-runtime]
---

# mauricelam/btsnoop-extcap（root 下文件尾随）

## 入选理由

8872 在 user 机构造里经常不存在。这条仓给出 **另一条实时链**：root 后 `tail -F` 受保护文件。Rust + spawn `adb`，比 androiddump 更接近 Yohu。解析依赖同作者 [btsnoop-rs](https://github.com/mauricelam/btsnoop-rs)（已克隆，格式库，不单独成篇）。

## 项目是什么

Wireshark extcap：列出 `adb devices -l`，接口 `btsnoop-<serial>`。采集时 `adb root`，再 `adb shell tail -F -c +0 /data/misc/bluetooth/logs/btsnoop_hci.log`，把 stdout 当成无限增长的 btsnoop 文件。

## 架构

```
adb devices -l
  → 每台设备一个 extcap 接口
  → Capture:
        adb -s SERIAL root
        id -u 必须是 0，否则 RootDeclined
        getprop persist.bluetooth.btsnooplogmode
        若不是 full：工具栏按钮
              setprop persist.bluetooth.btsnooplogmode full
              svc bluetooth disable
              sleep 2s
              svc bluetooth enable
        adb shell "tail -F -c +0 <path>"
        读 16 字节 FileHeader
        循环 24 字节 PacketHeader + payload
        默认丢掉第一秒（display_delay），避免灌历史
        按 HCI 类型字节判方向，写成 PCAP BLUETOOTH_HCI_H4_WITH_PHDR
```

`adb.rs` 把开发者选项映射成属性：`disabled` / `filtered` / `full`。空属性当 disabled。

默认路径与 AOSP `ParameterProvider` 一致。`local:<hostpath>` 只给测试。

README 写明：androiddump 依赖 2015 年就默认关掉的 8872；本工具改读文件，所以 **必须 root / userdebug `adb root`**。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| spawn adb CLI，不讲 5037 协议 | reuse-pattern | 对齐 ADR-v6-008 |
| `tail -F -c +0` 不是 `cat` | reuse-pattern | `cat` 到 EOF 就退，没有实时 |
| 属性 + 重启蓝牙 | adapt | 可做「打开 HCI snoop」动作；要提示会断蓝牙 |
| 前 N 秒丢历史 | adapt | 文件含开机以来的包；实时 UI 应跳过或单独「载入历史」 |
| `adb root` 当唯一路径 | anti-pattern | 市售 user 机（如项目真机 moto）会失败；只能当能力档 |
| 方向靠 payload[0] 猜 | lesson-only | 设备侧记录已有 flags；优先信 flags |

## 架构设计经验

文件尾随 = **特权 + 文本 shell + 二进制 stdout**。Yohu 已有 `spawn_long_lived`，缺的是二进制泵（`stream_lines` 按行切会拆包）。

`tail -F` 在文件轮转到 `btsnoop_hci.log.last` 时可能跟丢。AOSP `SnoopLoggerFile` 超包会 rename。长采要准备重新打开或侦测 magic。

## 与当前工作

- 能直接用：属性读写、`tail -F -c +0`、先 `id -u` 再承诺 root 档。
- 必须改写：extcap FIFO → 自己的环；Wireshark 工具栏 → Yohu 页眉开关。
- 不要用：root 失败还空转；HCI 走 `yohu-files` / SafetyRoot。

## 阅读范围

`src/main.rs`、`src/adb.rs`、`src/btsnoop_ext.rs`、`README.md`。btsnoop-rs 读了 `src/lib.rs` 的头/包/方向标志。未读 `install.rs` 的 Wireshark 安装路径细节。
