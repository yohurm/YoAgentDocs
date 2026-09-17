---
id: research.weliem-blessed-android
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java]
  frameworks: [Android BLE, BluetoothGatt]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: weliem/blessed-android
  url: https://github.com/weliem/blessed-android
  head: 44bba83cc4cb5f19a1da28cd4548c4a5b8a1c816
  cloned_to: "%TEMP%/YoAgentResearch/weliem--blessed-android"
studied_at: 2026-09-17
related:
  - research.NordicSemiconductor-Android-BLE-Library
  - research.JuulLabs-kable
  - research.android-classic-ble-connection-architecture
  - research.synthesis.client-runtime
---

# weliem/blessed-android

## 入选理由

和 Nordic 同是 GATT 库，但对象切法不同：`BluetoothCentralManager` 拥有连接表，`BluetoothPeripheral` 包装一台外设的 GATT。Java，和 HS01 同语言。能对照「谁拥有 connect / 谁拥有 BluetoothGatt」。本快照 2026-05。另有 `blessed-android-coroutines`，结构同构，不重复入选。

## 项目是什么

Android 8+ BLE 中心/外设库。中心侧：扫描、直连、autoConnect、bonding。外设侧：本机 GATT Server。本文只读中心侧连接生命周期。

## 架构

```
BluetoothCentralManager
    connectedPeripherals : Map<addr, Peripheral>
    unconnectedPeripherals : Map<addr, Peripheral>
           │
           │ connectPeripheral()  // 同地址已连/在连则直接 return
           ▼
BluetoothPeripheral
    state = DISCONNECTED | CONNECTING | CONNECTED | DISCONNECTING
    bluetoothGatt
           │
           ├─ 仅当 state==DISCONNECTED 才 connect()
           ├─ connectGatt(..., transport=LE)
           └─ 任何断开路径 completeDisconnect() → gatt.close(); gatt=null
```

`ConnectionState` 与 `BondState` 是两个枚举，分文件。配对丢失走 `bondLost` + 1s 延迟再 `completeDisconnect`，不把 `BONDING` 写成 GATT 断开。

### 关键契约（源码，不是 README）

`BluetoothPeripheral.connect()`：`state != DISCONNECTED` 就拒绝，「not yet disconnected, will not connect」。没有 pendingReplace、没有等 5 秒。

`completeDisconnect()`：立刻 `bluetoothGatt.close()` 并置 null，清命令队列。这是所有成功/失败断开的汇合点。

`cancelConnection()`：已连走 `gatt.disconnect()`，等系统回调；若还在 `CONNECTING`，他们知道系统常常不给 `DISCONNECTED`，于是 50ms 后自己补一次 `onConnectionStateChange(DISCONNECTED)`，再进 `completeDisconnect`。这是对「连接中取消」的平台瑕疵隔离，不是换设备状态机。

`BluetoothCentralManager.connectPeripheral`：同地址已在 `connected` / `unconnected` 表里则忽略。换设备是另一台 Peripheral，不是在旧对象上改 pending。

Javadoc 写「同时只能有一个 outstanding connect」，实现只防同一地址重复，不串行化不同地址。以代码为准。

### 何时对外「已连接」

`onConnectionStateChange(CONNECTED)` 只做内部 `successfullyConnected()`（去发现服务）。`listener.connected()` 在 `onServicesDiscovered` 成功之后才发。链路起来 ≠ 对外成功。

直连 `autoConnect=false`，超时约 30s（三星约 5s）。`autoConnect()` 不超时。失败重试（最多 1 次，且不是 `CONNECTION_FAILED_ESTABLISHMENT`）在 Central，不在 Peripheral。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 未 DISCONNECTED 禁止 connect | reuse-pattern | 替换 = 先 close 到 DISCONNECTED，再新 Peripheral/新 connectGatt |
| 断开汇合点一律 close+null | reuse-pattern | 不要留死对象等回调 |
| Bond 与 Connection 分枚举 | reuse-pattern | 配对弹窗不是 GATT 态 |
| 对外成功 = 服务发现完成 | reuse-pattern | 与「命令通道就绪再发布」可对齐，但那是更上层 |
| CONNECTING 取消时自补 DISCONNECTED | adapt | 只用于取消中的 connect，不要做成换设备 5s 循环 |
| 同地址重复 connect 直接忽略 | adapt | HS01 同 BLE MAC 应复用进行中的会话，不是再挂 pending |

## 架构设计经验

- Central 管「有哪些外设、谁在连」；Peripheral 管「这一台的 Gatt 对象」。两层不要合成一个 Manager。
- 状态机前置条件比超时更硬：不满足 DISCONNECTED 就不发起。
- 双模设备默认 `Transport.LE`。`autoConnect` 遇到 `PeripheralType.CLASSIC` 直接拒绝。

## 与当前工作

HS01 把「当前 Gatt」「要换成谁」「重试次数」「拆链超时」放在同一个 `GattConnectionManager`。Blessed 的切法是：Central 的 map + Peripheral 的单一 Gatt；替换不是 pending 字段。

能直接用：DISCONNECTED 才连、断开必 close、TRANSPORT_LE、bond/connection 分开。

必须改写：不要引入 Blessed 的命令队列 API；HS01 已有 `GattOperationQueue`。

明确不要用：在 CONNECTING 上再发一次 connect，指望超时救场。

## 阅读范围

读过：`BluetoothCentralManager` 的 connect/autoConnect/cancel/caches；`BluetoothPeripheral` 的 GattCallback、connect/autoConnect/cancelConnection/completeDisconnect、bonding；`ConnectionState` / `BondState` / `Transport`。未读 GATT Server（`BluetoothPeripheralManager`）和 example app 协议解析。
