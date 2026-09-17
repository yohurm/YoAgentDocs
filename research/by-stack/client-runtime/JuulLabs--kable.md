---
id: research.JuulLabs-kable
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Kotlin]
  frameworks: [Kotlin Coroutines, Android BLE]
also_relevant: []
utilization: [reuse-pattern, adapt, lesson-only]
source:
  platform: github
  repo: JuulLabs/kable
  url: https://github.com/JuulLabs/kable
  head: 503e5d20c8423ebfce95a752f2f0931993f07490
  cloned_to: "%TEMP%/YoAgentResearch/JuulLabs--kable"
studied_at: 2026-09-17
related:
  - research.NordicSemiconductor-Android-BLE-Library
  - research.weliem-blessed-android
  - research.android-classic-ble-connection-architecture
  - research.synthesis.client-runtime
---

# JuulLabs/kable

## 入选理由

把「一次连接」做成可取消的资源，而不是 Manager 上的可变字段。`Peripheral.connect()` 挂起直到 adapter 连上、服务发现完、observation 接好；`disconnect()` 取消这次 action 并等到 settled。状态是 `StateFlow<State>`，阶段是 `Connecting.Bluetooth / Services / Observes / Connected`。本快照 2026-09-14，仍活跃。与 Nordic/Blessed 同主题但切法不同（连接是协程资源），故入选。

HS01 是 Java，不能直接搬 Kable。要的是状态模型和「Connection 对象有寿命」。

## 项目是什么

Kotlin Multiplatform BLE。Android 实现：`BluetoothDeviceAndroidPeripheral` + `Connection`（包住这一次的 `BluetoothGatt`）。扫描与连接分开。

## 架构

```
Peripheral
    state: StateFlow<State>
    connection: MutableStateFlow<Connection?>
    connectAction = sharedRepeatableAction(::establishConnection)

connect()
    → establishConnection
         → bluetoothDevice.connectGatt(..., TRANSPORT_LE) 得到新 Connection
         → 等到 State.Connecting.Services
         → discoverServices
         → 接到 State.Connected(scope)

disconnect()
    → cancelAndJoin(connectAction)
         → Connection.disconnect()：gatt.disconnect()，可配置超时等 DISCONNECTED
         → 无论是否等到，close()：gatt.close()，Connection 作废
```

`State.kt` 把 GATT 回调的四态拆成应用阶段：`CONNECTED` 回调对应 `Connecting.Services`，不是对外 Connected。I/O 在 `Connecting.Bluetooth` 抛 `NotConnectedException`，在 `Connecting.Services` 抛 `IllegalStateException`。

`Connection.close()` 无条件 `gatt.close()`。等断开回调有 `disconnectTimeout`；超时只打日志，仍然 close。**超时不是再试 connect 的理由。**

`connectGatt` 走 `TRANSPORT_LE`（`BluetoothDevice.kt`）。老 API 的 autoConnect 竞态用反射绕过（注释链到 issuetracker 36995652）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 一次连接 = 一个 Connection 对象 | reuse-pattern | 替换 = 取消旧 action，新 Connection；不要 pending 字段 |
| 对外 Connected = 发现+订阅完成 | reuse-pattern | 与 Blessed 一致 |
| 断开超时只用于 close，不用于重连 | reuse-pattern | 和 HS01「拆链 5s 再强拆再连」相反 |
| State 分 Bluetooth / Services / Observes | adapt | Java 可用枚举，不必上 Flow |
| 整库协程 API | lesson-only | HS01 不引入 Kable 依赖 |

## 架构设计经验

- 连接失败的原因写在 `Disconnected(status)` 上（对端拆、本机拆、建链失败、L2CAP、超时…），不要用一个 `retryCount` 吞掉。
- `Peripheral` 可长寿（代表那台设备）；`Connection` 必须短寿（代表这一次无线电会话）。
- 重复 `connect()` 是同一个 repeatable action，不是第二套 pending。

## 与当前工作

HS01 的 `BleGattState` 已有 IDLE/CONNECTING/CONNECTED/DISCOVERING/SERVICES_READY。缺的是：**Gatt 对象跟状态同生共死**。Kable 的 `connection: Connection?` 就是这件事。`pendingReplaceDevice` 是在长寿对象上模拟短寿 Connection。

能直接用：阶段划分、Connection 可空、断开超时只收尾。

必须改写：用 Java 会话对象，不用协程库。

明确不要用：把 Kable 的 `autoConnectPredicate` / 多平台 Peripheral 搬进 yobtservice。

## 阅读范围

读过：`State.kt`；`BluetoothDeviceAndroidPeripheral.kt` 的 establish/connect/disconnect；`Connection.kt` 的 disconnect/close；`BluetoothDevice.kt` 的 connectGatt/TRANSPORT_LE。未读 iOS/JS 后端、扫描过滤器、samples/sensortag。
