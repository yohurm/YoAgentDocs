---
id: research.LineageOS-android_packages_modules_adb
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [adbd, mDNS, TLS]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: LineageOS/android_packages_modules_adb
  url: https://github.com/LineageOS/android_packages_modules_adb
  head: e7ed41a
  cloned_to: "%TEMP%/YoAgentResearch/LineageOS--android_packages_modules_adb"
studied_at: 2026-09-10
related:
  - research.android-wireless-connect-paths
  - research.yume-chan-ya-webadb
  - research.synthesis.client-runtime
---

# LineageOS/android_packages_modules_adb（ADB Wifi）

## 入选理由

AOSP `packages/modules/adb` 的 Lineage 镜像。无线调试（配对、TLS 口、mDNS、遗产 `tcpip`）的协议源。Yohu 只能经 sidecar 调这些命令，不能自己讲 A_STLS。浅克隆 `lineage-23.2` HEAD `e7ed41a`（2026-08-19）。文档：[`docs/dev/adb_wifi.md`](https://github.com/LineageOS/android_packages_modules_adb/blob/lineage-23.2/docs/dev/adb_wifi.md)。

## 项目是什么

设备上的 `adbd` 与主机 `adb` 客户端同一棵树。本篇只读无线：遗产 TCP 与 Android 11+ TLS 两条听口，外加配对服务器。

## 架构

```text
用户拨「无线调试」
  → persist.adb.tls_server.enable = 1
  → TlsServer 听 0.0.0.0:0（随机端口）
  → register_adb_tls_service → mDNS _adb-tls-connect._tcp
  → Framework 读 service.adb.tls.port 做发现

用户点「配对码配对」
  → Pairing Server（短命）
  → mDNS _adb-tls-pairing._tcp
  → SPAKE2(共享密码) + 交换证书
  → 主机公钥进 /data/misc/adb/adb_keys

遗产 adb tcpip 5555
  → daemon 服务名 tcpip:<port> → restart_tcp_service
  → mDNS _adb._tcp
  → 明文，问候包是 A_AUTH，不是 A_STLS
```

要点：

- **两个听口不是一个开关。** 关「无线调试」只拆 TLSServer（`disable_wifi_debugging` → `kick_all_tcp_tls_transports`）。已经 `tcpip 5555` 的遗产口还在，直到重启或 `adb usb`。
- **配对口 ≠ 连接口。** 配对屏上的 `IP:port` 和无线调试主屏上的 `IP:port` 不同，都会变。
- **主机命令全是 adb server 的 host 服务。** `pair` 拼 `host:pair:CODE:IP:PORT`（`client/commandline.cpp`）。Yohu 调 sidecar 即可。
- **API 37+** 可用 adbdauth 生命周期替代傻等系统属性；端口仍随机。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 无线调试 = TLS 口 + 配对，不是 5555 | reuse-pattern | UI 文案不要写成「输入 5555」 |
| sidecar `adb pair` / `connect` / `mdns` | reuse-pattern | ADR-v6-008；ya-webadb 已示范 host 服务字符串 |
| 遗产 `tcpip:` 服务 | reuse-pattern | 不开无线调试时的无线路径 |
| 自己实现 SPAKE2 / A_STLS | anti-pattern | 密钥与握手在 adbd 里；客户端已有官方二进制 |
| 假设 mDNS 在 Windows 一定可用 | anti-pattern | 文档和现场都要求先 `mdns check` |

## 架构设计经验

- **「无线」在 ADB 里是运输，不是功能。** 配对成功后，shell / sync / logcat / app_process 与 USB 同一套服务。Yohu 不要为无线再做一套文件/日志模块。
- **开关是设备策略。** 主机不能远程打开「无线调试」（除非已有 ADB 或系统特权）。产品不能承诺「用户什么都不用开」。
- **明文 tcpip 与 TLS 口不要混连。** 对 TLS 口发遗产握手会失败；对 5555 走 `adb pair` 也失败。

## 与当前工作

- 能直接用：命令形状与三种 mDNS 类型；属性名（排障用）。
- 必须改写：发现失败时的手动 IP:端口；把「已配对」存工作台设置，不要抄 adbd 的 keystore 路径。
- 不要用：把本树编进 Yohu；在 Rust 里重写 pairing_connection。

## 阅读范围

`docs/dev/adb_wifi.md`、`daemon/adb_wifi.cpp`、`daemon/services.cpp`（`tcpip:` / `usb:`）、`client/commandline.cpp`（`pair` / `tcpip`）、`pairing_auth/include/adb/pairing/pairing_auth.h`。未读 mdns 后端实现与 Android 17 Rust mDNS 全量。
