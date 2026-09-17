---
id: research.android-classic-ble-connection-architecture
type: topic-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java, Kotlin]
  frameworks: [Android Bluetooth, GATT, A2DP, HFP]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://developer.android.com/develop/connectivity/bluetooth/ble/connect-gatt-server
  repos:
    - NordicSemiconductor/Android-BLE-Library
    - weliem/blessed-android
    - JuulLabs/kable
    - Freeyourgadget/Gadgetbridge
    - LineageOS/android_frameworks_base
  cloned_to:
    - "%TEMP%/YoAgentResearch/NordicSemiconductor--Android-BLE-Library"
    - "%TEMP%/YoAgentResearch/weliem--blessed-android"
    - "%TEMP%/YoAgentResearch/JuulLabs--kable"
    - "%TEMP%/YoAgentResearch/Freeyourgadget--Gadgetbridge"
    - "%TEMP%/YoAgentResearch/LineageOS--android_frameworks_base"
studied_at: 2026-09-17
related:
  - research.NordicSemiconductor-Android-BLE-Library
  - research.weliem-blessed-android
  - research.JuulLabs-kable
  - research.Freeyourgadget-Gadgetbridge
  - research.LineageOS-android_frameworks_base
  - research.LineageOS-android_packages_modules_Bluetooth
  - research.synthesis.client-runtime
---

# Android 经典 + BLE：连接对象与编排

主题笔记。回答：陪伴耳机 App 在 Android 10–17 上，**经典 BR/EDR 与 GATT 应如何分层**，以及「换设备 / 清配对再连」该不该在旧 `BluetoothGatt` 上等断开回调。规范先读 [Connect to a GATT server](https://developer.android.com/develop/connectivity/bluetooth/ble/connect-gatt-server)、[`BluetoothGatt.close`](https://developer.android.com/reference/android/bluetooth/BluetoothGatt#close())、[Bluetooth profiles](https://developer.android.com/develop/connectivity/bluetooth/profiles)。实现对照五份源码，不靠 README 下结论。

本篇停在深研。不改 HS01 业务代码。

## 背景

HS01 是双模耳机：经典 MAC（例如 `E8:…`）与 BLE MAC（例如 `E9:…`）同时存在。产品约束已定：

- `YoSPPApi` 签名与超时接线不动
- 加耳机成功 ≠ SPP RFCOMM
- A2DP/HFP 断开 ≠ 经典断开
- 第三方不能用 `BluetoothDevice.connect()`（Binder 要 `BLUETOOTH_PRIVILEGED`）

旧 GATT 路径把「当前对象 / 要换成谁 / 重试次数 / 5s 拆链超时」叠在一个长寿 `GattConnectionManager` 上。这是在旧对象模型上打补丁。开源里没有「再等一个断开回调」的正路。

五份实现结构互不重复：Nordic = 一设备一 manager + 请求队列；Blessed = Central 表 + Peripheral；Kable = 连接是可取消资源；Gadgetbridge = Service 编排 + 运输可替换；SettingsLib = 经典设备聚合 profile。

## 关键结论

### 1. 三条线，三种「已连接」

| 线 | 事实源 | 谁当「已连接」 | 谁不当 |
|----|--------|----------------|--------|
| 经典 ACL + bond | ACL 广播 + `BOND_*` | HS01：`linkUp && (BONDING\|\|BONDED)` | Settings：`isConnected` 是 profile |
| 音频 profile | A2DP/HFP proxy | Settings 设置页 | HS01 加耳机；profile 断保持经典 |
| GATT | `BluetoothGatt` 回调 | 库：发现服务（+ 初始化）后 | `STATE_CONNECTED` 立刻对外 |

没有一份开源把这三条合成一个布尔。Settings 收 ACL（还分 LE / BR-EDR）但 UI 用 profile。Gadgetbridge 耳机不发起 A2DP，RFCOMM 只当厂商命令。Nordic/Blessed/Kable 不管经典。

### 2. GATT 对象是一次会话，不是单例字段

官方：`connectGatt` 得到实例；用完 `close()`；关掉的对象不要再 `connect`。

源码汇合：

| 库 | 换会话怎么做 |
|----|----------------|
| Nordic | 直连：`close()` 后新 `connectGatt`。200ms sleep 只隔离老机 close 异步 |
| Blessed | `state!=DISCONNECTED` 拒绝 connect。任何断开 `completeDisconnect` → `close()+null` |
| Kable | `Connection?` 短寿。`disconnect` 可超时等回调，**超时只 close，不重连** |
| Gadgetbridge `BtLEQueue` | 注释禁止复用 Gatt。`disconnect(); close();` **不等回调**，立刻新 `connectGatt(TRANSPORT_LE)` |

共同决策：**替换 = 结束旧对象 + 新 `connectGatt`。** 不是 `pendingReplaceDevice` + 等 `DISCONNECTED` + 5s 强拆 + `retryCount`。

Blessed 在 **CONNECTING 取消** 时 50ms 自补 DISCONNECTED，因为系统常不回调。这是取消路径的瑕疵隔离，不是换设备状态机。Nordic 的 200ms 同理。

### 3. 双模 GATT 必须 `TRANSPORT_LE`

Nordic、Blessed、Gadgetbridge、Kable 在 API 23+ 都锁 `TRANSPORT_LE`。官方从 API 23 起公开该参数；API 37 起用 `BluetoothGattConnectionSettings.setTransport(TRANSPORT_LE)`。双模耳机默认 `TRANSPORT_AUTO` 可能走 BR/EDR，表现为 133 或服务不全。这是文档与源码同时成立的，不是 OEM 猜测。

`autoConnect=true`：Gadgetbridge 写 *doesn't really work*；Blessed 把它留给「已缓存地址的后台重连」，且从不超时。主路径一律直连 `false`。

### 4. 编排：UI 不拥有运输，再连先退役

Gadgetbridge：`DeviceCommunicationService` 是唯一碰 `DeviceSupport` 的人。再 connect 先 `dispose` 旧 Support。`GBDevice.State` 区分 `CONNECTED` 与 `INITIALIZED`。

Nordic `USAGE.md`：一实例一外设，断开后推荐新实例。Kable：`Peripheral` 可长寿，`Connection` 必须短寿。Blessed：Central 的 map 按地址持有 Peripheral。

HS01 对应：

- `YoSPPApi` / `YoBLEApi` = 门面（可保持签名）
- 编排 = 「这副耳机」的会话：经典运输 + GATT 运输
- GATT 运输 = 短寿对象，关旧开新
- 命令就绪再对 App 发布（已有 Runtime 口径，对应 `INITIALIZED`）

### 5. 经典发起：Settings 的路第三方走不通

Settings `CachedBluetoothDevice.connectDevice()` 调 `mDevice.connect()`。AOSP Binder 要特权。HS01 摩托罗拉 15 已炸。

第三方能做的，仍是：

- 未配对：`createBond()`（公开）
- 已配对：隐藏 `BluetoothA2dp.connect` / 尝试 `BluetoothHeadset.connect`（CONNECT/ADMIN；HFP 在新树可能还要 `MODIFY_PHONE_STATE`）
- 成功只看 ACL Store，不看 profile 回调，不看 RFCOMM

Settings 的观察结构（EventManager 分 ACL / bond / UUID / profile）可以搬；它的 `isConnected()` 和发起 API 不能搬。

### 6. 配对不是断开

Blessed：`BondState` ≠ `ConnectionState`。Nordic：bonding 广播不改 `connectionState`。Settings：`BOND_BONDING` 只让 `isBusy()`，`BOND_NONE` 才清 profile。耳机侧清配对再扫：ACL 仍在且进入 `BONDING` 时，不应拆 GATT。拆 GATT 的条件是经典 ACL 掉或变成 `BOND_NONE`，或用户主动拆。

### 7. 开源里没有「ACL + GATT 同时维持」的整库

Xiaomi `BOTH` 仍是运行时选 BLE **或** Classic SPP。索尼耳机假定系统已经连上 A2DP，App 只开 RFCOMM。因此 HS01 不能「抄一个库」；要 **组合**：SettingsLib 的观察切法 + 第三方 profile 发起 + Nordic/Blessed/GB 的 GATT 对象寿命 + GB 的 Service 编排。

## 目标分层（从源码拼出来的，不是在旧 Manager 上改字段）

```
HeadsetSession          一副耳机（经典 MAC + BLE MAC）
  ├─ ClassicTransport   观察 ACL/bond；发起 createBond 或 profile connect
  ├─ GattTransport      短寿：connectGatt(TRANSPORT_LE) → 发现 → 队列
  └─ Orchestrator       经典已连才 start GattTransport；
                        经典 ACL 掉或 BOND_NONE 才 close GATT；
                        换 BLE 地址 = close 旧 GattTransport + new
```

Orchestrator 不持有 `BluetoothGatt`，不写拆链重试。GattTransport 不读配对弹窗，不读 A2DP 状态。ClassicTransport 不读 GATT。

## 与当前工作的关系

**能直接用**

- GATT：关旧 `close()` + 新 `connectGatt(TRANSPORT_LE)`（GB / Nordic / Blessed）
- 未 DISCONNECTED 不发起下一次 connect（Blessed）
- 断开超时只收尾，不驱动重连（Kable）
- 经典：EventManager 式分发；ACL 分运输；配对与连 profile 分步（SettingsLib）
- 对外：无线电 CONNECTED ≠ 对 App 发布（GB `INITIALIZED`、Kable `Connected`、Blessed 发现后）

**必须改写**

- 拆掉长寿 Manager 上的 `pendingReplace` / `retryCount` / 拆链等待。换成短寿 GattTransport。
- 编排从 Runtime + AutoConnect + Manager 互等，收成 Orchestrator 对两个运输的订阅。
- Java 自写，不引入 Nordic/Kable/Blessed/GB 依赖。

**明确不要用**

- `BluetoothDevice.connect()` / Settings `setConnectionPolicy` 当 3P 发起
- Settings `isConnected()`（profile）当加耳机
- 索尼 RFCOMM / 本仓 SPP 当加耳机成功
- Xiaomi「BOTH 选一条」当双模同时在线
- `gatt.connect()` 当换设备直连
- FastBle 式单例再包一层超时
- 把 Nordic 200ms / Blessed 50ms 扩散成通用状态机

## 来源与阅读范围

官方：Connect GATT、`BluetoothGatt.close`、Profiles。AOSP Binder / `BLUETOOTH_PRIVILEGED` 见已有 `LineageOS--android_packages_modules_Bluetooth` 与先前 Android 10–17 笔记。

克隆与单仓阅读范围见五篇 project-study。未读 androidx.bluetooth 单体仓、RxAndroidBle 源码（只作落选对照）、Nordic Kotlin BLE Library 2.0（未完成）、各 OEM 闭源耳机 App。
