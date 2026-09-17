---
id: research.NordicSemiconductor-Android-BLE-Library
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java, Kotlin]
  frameworks: [Android BLE, BluetoothGatt]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: NordicSemiconductor/Android-BLE-Library
  url: https://github.com/NordicSemiconductor/Android-BLE-Library
  head: 4a86e6c6b2631a29d7de6c5789b11326733f1503
  cloned_to: "%TEMP%/YoAgentResearch/NordicSemiconductor--Android-BLE-Library"
studied_at: 2026-09-17
related:
  - research.weliem-blessed-android
  - research.JuulLabs-kable
  - research.android-classic-ble-connection-architecture
  - research.synthesis.client-runtime
---

# NordicSemiconductor/Android-BLE-Library

## 入选理由

第三方 Android GATT 的行业标准实现。`USAGE.md` 把架构写死：一个 `BleManager` 实例对应一台外设；对应用暴露设备级 API，不暴露 `writeCharacteristic`。能直接回答 HS01 里「长寿 `GattConnectionManager` 持有一个 `BluetoothGatt`、拆链等回调、超时再强拆」是不是正路。仓仍在维护（本快照 2026-02），作者同时在写 Kotlin BLE Library 2.0，但现役产品仍是本库。

官方规范先读 [Connect to a GATT server](https://developer.android.com/develop/connectivity/bluetooth/ble/connect-gatt-server) 与 [`BluetoothGatt.close`](https://developer.android.com/reference/android/bluetooth/BluetoothGatt#close())。

## 项目是什么

Java 库。`BleManager` 是门面，`BleManagerHandler` 持有唯一 `BluetoothGatt`、请求队列、bonding 广播。连接、发现、初始化、读写都是 `Request`，串行入队。不负责扫描。

## 架构

```
App  ──高阶 API──►  子类 BleManager（一设备一实例）
                         │
                         ▼
                   BleManagerHandler
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     ConnectRequest  操作队列      Bond / BT 广播
          │
          ▼
   device.connectGatt(..., TRANSPORT_LE)
          │
          ▼
   断开 → close() 把 bluetoothGatt 置 null
```

### GATT 对象

`BleManagerHandler.internalConnect`（约 639–796 行）写明两条重连路：

1. 复用同一 `BluetoothGatt` 调 `connect()`：系统会把 `autoConnect` 强制成 `true`。
2. `close()` 后再 `connectGatt`：官方直连路径。

注释写死：`gatt.close()` 是异步的，立刻再 `connectGatt` 时，部分老机（Nexus 4 / 5.0.1）服务发现永不结束。他们把这个平台瑕疵收成 **一处 `Thread.sleep(200)`**，不是会话状态机。直连（`autoConnect=false`）走关旧开新；`autoConnect` 重连才复用对象。

API 23+ 一律 `TRANSPORT_LE`。双模耳机不指定 transport 时，栈可能走 BR/EDR，GATT 133 / 服务发现残缺。这是公开 API，不是猜测。

### 连接是请求，不是补丁循环

`connect(device)` 返回 `ConnectRequest`，可 `.retry(n, interval)`、`.timeout(ms)`、`.useAutoConnect()`。重试是请求属性。`disconnect()` 也是请求。`USAGE.md`：上一台断开后可以复用 manager，**推荐新实例**。

`close()`：`gatt.close()`、`bluetoothGatt = null`、清空队列、状态回到 `DISCONNECTED`。官方示例同样在用完后 `close()` 并把引用置空。

### 配对与连接分开

`ACTION_BOND_STATE_CHANGED` / `ACTION_PAIRING_REQUEST` 只服务 bonding 流程。连接态是 `BluetoothProfile.STATE_*`。配对弹窗不会把已连接写成断开。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 一设备一 manager，推荐新实例 | reuse-pattern | 替换设备 = 退役旧 manager，不是在同一对象上叠 pending |
| `connectGatt(TRANSPORT_LE)` | reuse-pattern | 双模耳机 GATT 必须锁 LE |
| 关旧 GATT 再开新；200ms 只隔离老机 | adapt | 可抄「关旧开新」；不要把 sleep 做成通用状态 |
| 重试/超时是 ConnectRequest 属性 | reuse-pattern | 不要在 Manager 里用 retryCount + 拆链超时互等 |
| 子类暴露「开灯」而不是写特征 | adapt | HS01 命令层已有；GATT 层不该再懂业务 |
| 复用同一 Gatt + `gatt.connect()` 当直连 | anti-pattern | Nordic 自己写了：这会变成 autoConnect |

## 架构设计经验

- GATT 对象死后不可复活。下一个会话必须是新的 `connectGatt` 返回值。
- 平台瑕疵（close 异步）可以有一处隔离，不能扩散成 Session / AutoConnect / Runtime 三套超时。
- 「已连接」对库来说是：链路起来 + 必要服务在 + `initialize()` 队列跑完。不是 `onConnectionStateChange(CONNECTED)` 立刻对外成功。

## 与当前工作

HS01 的 `GattConnectionManager` 是长寿单例，字段里同时有 `mBluetoothGatt`、`pendingReplaceDevice`、`retryCount`、5s 拆链超时。Nordic 的对应物是：**ConnectRequest 拥有一次尝试；对象替换靠 close + 新 connectGatt；推荐换实例。**

能直接用：`TRANSPORT_LE`、关旧开新、连接/断开都是请求、配对广播不改连接态。

必须改写：不要引入 Nordic 的 `BleManager` 子类体系（Kotlin/Java 库依赖）；不要 `Thread.sleep` 当默认路径；不要把命令初始化塞进 GATT manager。

明确不要用：把 `BluetoothGatt.connect()` 当「换设备直连」。

## 阅读范围

读过：`USAGE.md`、`ble/src/main/java/no/nordicsemi/android/ble/BleManager.java`（构造、connect/disconnect/close）、`BleManagerHandler.java` 的 `close` / `internalConnect` / 队列清空。未读 GATT Server、packet merger、测试 app。
