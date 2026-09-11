---
id: research.wireshark-wireshark
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C]
  frameworks: [extcap, ADB protocol, pcap]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: wireshark/wireshark
  url: https://github.com/wireshark/wireshark
  head: 4e3f39cfaf203c400267cb1c08633c7927e0c986
  cloned_to: "%TEMP%/YoAgentResearch/wireshark--wireshark"
studied_at: 2026-09-10
related: [research.LineageOS-android_packages_modules_Bluetooth, research.synthesis.client-runtime]
---

# wireshark/wireshark（androiddump 实时 HCI）

## 入选理由

主机侧「无 root 实时 HCI」的参考实现就是 Wireshark 自带的 extcap `androiddump`。只稀疏读了 `extcap/`，不读解剖器。手册：[androiddump(1)](https://www.wireshark.org/docs/man-pages/androiddump.html)。

## 项目是什么

`extcap/androiddump.c`：连本机 adb-server（默认 `127.0.0.1:5037`），列出设备上的 logcat / 蓝牙 / tcpdump 接口，选中后把包写入 Wireshark FIFO。HCI 接口名 `android-bluetooth-btsnoop-net`。

## 架构

### 列出接口（Btsnoop Net 是否出现）

API ≥ 21 才考虑。步骤：

1. `host:transport:<serial>` 接到设备。
2. `shell:ps` 找蓝牙进程 PID（API 分级用不同 ps）。
3. 读该 PID 的 `/proc/net/tcp`，找本地端口十六进制 **`22A8`（8872）**。
4. 找不到就把接口标 disable——Wireshark 里根本没有「Android Bluetooth Btsnoop Net」。

这与设备侧 `is_debug_build()` 门闩一致：user 机构造函数不 `bind 8872`，androiddump 直接不画接口。

### 采集（`capture_android_bluetooth_btsnoop_net`）

**默认不是 `adb forward`。** 它自己讲 ADB 文本协议：

```
TCP 127.0.0.1:5037
  → host:transport:<serial>
  → tcp:8872          # ADB 服务名，接到设备 localhost:8872
  → recv 16 字节 btsnoop 文件头
  → 循环 recv 24 字节记录头 + included_length 载荷
  → 减 0x00dcddb30f2f8000 得到 Unix µs
  → flags bit0 → H4 伪头方向
  → extcap_dumper 写成 BLUETOOTH_H4_WITH_PHDR
```

`adb_send` 先发 4 位十六进制长度，再发服务字符串，等 `OKAY`。这是 **重实现 ADB 线协议**，不是 spawn `adb.exe`。

旧接口 `android-bluetooth-external-parser` 才有 `--bt-forward-socket`，会发 `host-serial:<sn>:forward:tcp:LOCAL;tcp:REMOTE`。Btsnoop Net 这条主路径不用它。

### 探测失败时的用户所见

IssueTracker 183305452：Android 10+ 生产机只剩 logcat 接口，没有 Btsnoop Net。androiddump 启动时打 `Btsnoop Net Port for <serial> is unknown`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 先探 8872 再承诺实时 | reuse-pattern | `/proc/net/tcp` 的 `22A8` 或试连 `tcp:8872` |
| 流是 btsnoop 记录不是文本行 | reuse-pattern | 不能走 `AdbClient::stream_lines` |
| 自讲 ADB 协议连 5037 | anti-pattern | ADR-v6-008 禁止；改 sidecar `adb forward` + 本机 TCP |
| 绑定 Wireshark FIFO / pcapng | anti-pattern | Yohu 要自己的包环 + UI，不要拉起 Wireshark |
| H4 伪头方向 | adapt | 列表 UI 需要 TX/RX；解析 flags bit0 即可 |

## 架构设计经验

实时套接字链是 **ADB 传输 + 设备 localhost TCP**，不是 `adb shell`。Yohu 合法等价物：

```
sidecar adb -s SERIAL forward tcp:<local> tcp:8872
  → std::net / tokio TcpStream 连 127.0.0.1:<local>
  → 读 16 字节头 + 循环 24+N
```

不要把 androiddump 链进安装包。探测失败要明白说「这台设备没有实时套接字」，不要空转。

## 与当前工作

- 能直接用：8872、文件头 16 / 记录头 24、时间戳偏移、用端口是否在听判断能力。
- 必须改写：5037 帧 → `ProcessRunner` + `adb forward`；FIFO → `yohu-hci` 环 + 批量事件。
- 不要用：重实现 ADB；把 logcat 接口当 HCI；依赖用户已装 Wireshark。

## 阅读范围

`extcap/androiddump.c`：`adb_connect` / `adb_connect_transport` / `adb_send` / 接口枚举（约 1228–1304 行）/ `capture_android_bluetooth_btsnoop_net`（1909–2028 行）。未读 logcat 文本采集、tcpdump、解剖器。
