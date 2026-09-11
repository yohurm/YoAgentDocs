---
id: research.android-wireless-connect-paths
type: topic-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++, C, TypeScript, Java]
  frameworks: [adb, scrcpy-server, MediaProjection, Miracast]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: other
  url: https://developer.android.com/tools/adb
  repos:
    - LineageOS/android_packages_modules_adb
    - Genymobile/scrcpy
    - yume-chan/ya-webadb
    - bk138/droidVNC-NG
  cloned_to:
    - "%TEMP%/YoAgentResearch/LineageOS--android_packages_modules_adb"
    - "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
    - "%TEMP%/YoAgentResearch/yume-chan--ya-webadb"
    - "%TEMP%/YoAgentResearch/bk138--droidVNC-NG"
studied_at: 2026-09-10
related:
  - research.LineageOS-android_packages_modules_adb
  - research.Genymobile-scrcpy
  - research.yume-chan-ya-webadb
  - research.bk138-droidVNC-NG
  - research.synthesis.client-runtime
---

# Android 无线连接：无线调试 / 经典 tcpip / 厂商投屏

主题笔记。回答两件事：Yohu 怎么做 **WiFi 无线调试**；以及「厂商电脑应用不开无线调试也能投屏」对第三方 ADB 工作台意味着什么。规范先读 [adb 官方文档](https://developer.android.com/tools/adb) 与 AOSP [`docs/dev/adb_wifi.md`](https://android.googlesource.com/platform/packages/modules/adb/+/HEAD/docs/dev/adb_wifi.md)；实现对照 scrcpy `--tcpip`、ya-webadb 的 `host:pair` 封装、droidVNC-NG 的 MediaProjection。厂商电脑管家无开源协议栈，只按官方能力与权限模型对照，不假装读过闭源二进制。

## 先把词拆开

用户口头里的「WiFi 无线调试」经常混了四条完全不同的路。Yohu 今天四条都还没做编排，但 **目录已经能吃 `tcp:`**：`DeviceInfo.connection` 来自 `adb devices -l`，`is_tcp_connection` 会把投屏切到 wifi 编码并默认 forward。

| 路 | 用户要开什么 | 要不要 USB | 运输 | 投屏后还有文件/日志/终端吗 |
|----|--------------|------------|------|----------------------------|
| A USB ADB | USB 调试 | 一直插着 | USB | 有。现状 |
| B 经典 `adb tcpip 5555` | USB 调试。**不要**「无线调试」开关 | 每次重启后插一次，开端口后可拔 | 明文 TCP，端口通常 5555 | 有。设备一旦进 `adb devices` 四模块都能用 |
| C Android 11+ 无线调试 | 开发者选项 + **无线调试** 开关 | 配对后不用 | TLS，端口随机；配对口与连接口是两个 | 有。同样是 ADB |
| D 厂商电脑管家 / 多屏协同 | 同品牌账号或 NFC/蓝牙确认。不要开发者选项 | 不要 | BLE 发现 + Wi-Fi Direct / 私有 UDP。不是 ADB | **没有** logcat / `adb shell` / SafetyRoot 文件。只有他们自己的画面、键鼠、互传 |
| E 系统「无线投屏」Miracast | 手机设置里的投屏。不要开发者选项 | 不要 | Wi-Fi Display | 只看画面（偶有音）。不能控、不能文件、不能日志 |
| F 自研/第三方 APK（droidVNC 一类） | 装 APK + 录屏授权 + 无障碍 | 不要 ADB | MediaProjection / VNC | 只有投屏和有限控制。没有 ADB 能力 |

厂商能「不开无线调试就投屏」，是因为他们走 **D 或 E**，从来不是 C。Yohu 是 ADB 工作台，抄不了 D，也不该把 E/F 塞进 `yohu-mirror`。

## 对本工作台的结论

1. **不开「无线调试」也能无线投屏 / 文件 / 日志 / 终端：能。** 走 B。USB 调试仍要开。重启后通常要再插一次 USB（未 root 不能把 `tcpip` 做成开机常驻）。这就是 scrcpy `--tcpip` 无参模式。
2. **完全不要开发者选项、像小米/华为电脑管家那样一碰即连：第三方做不到同等体验。** 那些是系统签名应用 + 特权输入 + 私有 P2P。`INJECT_EVENTS` 是 `signature\|privileged`；普通 APK 只能无障碍或 ADB shell。
3. **C 仍然值得做。** 它解决的是「不想插 USB」，不是「用户没开开发者选项」。Android 17 + platform-tools 37 的 ADB Wi-Fi 2.0 可把当前 Wi-Fi 标成受信网络，连上后自动开无线调试；市售机短期内仍是 Android 11–16 的随机端口 + 手动开开关。
4. **缺口在设备栏编排，不在投屏协议。** 现有 `yohu-mirror` / 文件 / 日志只要 `adb devices` 里有 Online 条目就能跑。源码里没有 `adb pair` / `tcpip` / `connect`。
5. **无线 ADB 上 `adb reverse` 常失败。** scrcpy `adb_tunnel.c` 写明 over `adb connect` 时 reverse 失败就 fallback forward。Yohu 已对 `tcp:` 默认 `start_force_forward`，这条要对。

## 五条路的源码级对照

### B 经典 tcpip（不开无线调试）

官方仍把这条写成 Android 10 及以下的主路径，并注明 11+ 也能用，只是要先 USB 一次。AOSP 自己说它 **明文、可被窃听**。

设备侧：`daemon/services.cpp` 把 `tcpip:<port>` 交给 `restart_tcp_service`，adbd 改听 TCP。主机侧：`adb tcpip 5555` → `adb connect IP:5555`。

scrcpy 把向导收成两条（`app/src/server.c`）：

```text
已 USB、不知道 IP
  sc_adb_get_device_ip  ← adb shell ip route，只收 wlan* 那行的 src
  已在听则复用端口，否则 sc_adb_tcpip(5555) 再轮询最多 40×250ms
  sc_adb_connect(ip:port)

已经在听
  scrcpy --tcpip=192.168.1.1[:5555]
  前缀 + 则先 disconnect 再连
```

ya-webadb `AdbTcpIpService` 补了一层属性优先级：`service.adb.listen_addrs` → `service.adb.tcp.port` → `persist.adb.tcp.port`。前两个或 `persist` 非空时，有的 ROM **出厂就开着 TCP 监听**；`setPort` 只改得了 `service.adb.tcp.port`。

重启后 `tcpip` 模式丢。未 root 不能靠普通应用把它写回。这是 Android 限制，不是 scrcpy 没做。

### C 无线调试（必须开那个开关）

AOSP 明确：这 **不是** 遗产 `tcpip` 套接字。遗产口用 `A_AUTH`；无线调试口用 `A_STLS`，全程 TLS。开关写 `persist.adb.tls_server.enable`；`daemon/adb_wifi.cpp` 的 `TlsServer` 绑 `port=0`（内核分配随机端口），再 `register_adb_tls_service` 发 `_adb-tls-connect._tcp`。关开关就 `kick_all_tcp_tls_transports`。

配对是另一条短命服务 `_adb-tls-pairing._tcp`。共享秘密是 6 位配对码或二维码里的密码。`pairing_auth` 用 SPAKE2 + AES-128-GCM。主机命令是 sidecar 已有的：

```text
adb pair IP:PAIR_PORT [CODE]     → host:pair:CODE:IP:PORT
adb connect IP:CONNECT_PORT      → host:connect:IP:PORT
adb mdns services                → 列出 _adb._tcp / _adb-tls-pairing / _adb-tls-connect
```

两个端口会变。配对信任能留，连接口重启或开关拨一下就换。ya-webadb `WirelessCommands` 只封装官方 adb server 的 host 服务，**浏览器里并不自己做 TLS 配对**（作者说明：无线调试未在 WebUSB 路径实现）。

Android 17（API 37）起：`always allow on this network` 把 SSID+BSSID 标成受信网；adbd 在受信网上自动拉起 TLSServer。主机要 platform-tools ≥ 37.0.0，`adb server-status` 里 `mdns_backend: LIBADBMDNS`。这改善的是「开过一次之后少动手」，**第一次仍然要开无线调试并配对**。

Windows 上 mDNS 经常哑。`adb mdns check` 报 `unknown host service` 时，旧栈要 `ADB_MDNS_OPENSCREEN=1`；2.0 文档反过来要 `ADB_MDNS=1` 且 `ADB_MDNS_OPENSCREEN=0`。Yohu 若做发现，先 `adb mdns check` / `server-status`，不要假设局域网一定能看见设备。

### D 厂商电脑管家

公开产品行为（华为电脑管家「多屏协同」、小米电脑管家 / HyperConnect、荣耀/OPPO/一加同类）一致：

- 发现：蓝牙 LE，有的加 NFC
- 传输：Wi-Fi Direct / 5 GHz P2P，不是连路由器再打 5555
- 画面：设备侧硬件编码，电脑侧硬解
- 控制：系统应用注入输入，不是 Accessibility
- 身份：同品牌账号或本机配对，不是 `adb_keys`

华为另有一条 **设置 → 手机投屏** 打到电脑「无线投屏」，那是 E（Miracast），和管家协同不是同一条。官方说明：管家连着手机时，系统投屏可能被占住。

第三方不能复刻 D：没有预装 `priv-app`、没有平台签名、没有厂商账号服务器。逆向闭源协议不进本库、也不该进 Yohu。

### E Miracast

手机设置里的「投屏 / 无线显示」是 Wi-Fi Display。Windows 可选功能「无线显示」+「投影到此电脑」就是系统 sink。Win10 1903+ 有 `Windows.Media.Miracast.MiracastReceiver`，应用能自建 sink，但要 Wi-Fi Direct，且通常 **只看不控**。这不是 `yohu-mirror` 的 scrcpy 管道，不要混进同一槽位。

### F 第三方 APK

droidVNC-NG：`MediaProjectionService` 建 `VirtualDisplay`；控制走 `InputService extends AccessibilityService`。Android 10+ 每次启动都要录屏授权；Android 14+ 必须 `foregroundServiceType=mediaProjection`，token 一次性。README 写明：想免弹窗得 `adb shell cmd appops set … PROJECT_MEDIA allow`——免弹窗本身又回到 ADB。

这条能「不用无线调试」投屏，但要用户装包、授无障碍、每次点允许。Yohu 的身份是 **不装 APK、官方 scrcpy-server + sidecar adb**。F 是另一个产品。

## Yohu 若做（调研导出，供架构设计引用）

P0 设备栏「无线」向导，走 B，**不要求无线调试开关**：

1. 已 USB 且 Online：读 `ip route` 的 `wlan*` src（对标 scrcpy 解析，不要 `awk '{print $9}'` 第一行）。
2. sidecar `adb -s SERIAL tcpip 5555`，轮询端口起来。
3. `adb connect IP:5555`。目录扫描会看到 `tcp:IP:5555`。
4. 提示：拔线后仍可用；重启要再插一次。空态写清「这不是开发者选项里的无线调试」。
5. 投屏继续用现有 `tcp:` → wifi 编码 + 默认 forward。

P1 无线调试（C），给不想插 USB 的 Android 11+：

6. 设备上用户自己打开「无线调试」。Yohu 提供配对码输入（先 CLI 形状，后可加二维码 `WIFI:T:ADB;S:studio-…;P:…;;`）。
7. sidecar `adb pair` / `adb connect` / `adb mdns services`。禁止重实现 pairing SPAKE2。
8. 持久化「已配对 GUID」，下次只 connect。连接口变了就重新发现，不要写死 5555。
9. 启动时 `adb mdns check`；失败则走手动 IP:端口，不要假装能自动发现。

P2 远期：

10. Android 17 受信网自动连：探测 `mdns_service_version: "2.0"`，再谈少一步开关。
11. 单独「投屏接收」入口（Miracast sink）——若产品真要零开发者选项的只看。**新模块，不进 `yohu-mirror`。**

明确不做：

- 逆向小米 / 华为 / OPPO 电脑管家协议。
- 为投屏发一只常驻 APK 当主路径。
- 把 Miracast 画面接到 scrcpy FramePipe。
- 重实现 ADB / TLS 配对（ADR-v6-008）。
- 宣传「重启后不用 USB 也不用开无线调试」——未 root 做不到。

## 阅读范围

本轮：AOSP/LineageOS `docs/dev/adb_wifi.md`、`daemon/adb_wifi.cpp`、`daemon/services.cpp` 的 `tcpip:`、`client/commandline.cpp` 的 `pair`/`tcpip`、`pairing_auth.h`；scrcpy `doc/connection.md`、`server.c` tcpip 向导、`adb_tunnel.c` reverse 回退；ya-webadb `wireless.ts`、`tcpip.ts`、`m-dns.ts`；droidVNC-NG README、`MediaProjectionService.java`、`InputService.java`；官方 adb 文档（含 Android 17 Wi-Fi 2.0）；华为「手机投屏至计算机」支持页；Windows `MiracastReceiver` API。未拆厂商管家 PE，未在真机跑 pair。
