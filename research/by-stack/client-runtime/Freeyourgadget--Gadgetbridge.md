---
id: research.Freeyourgadget-Gadgetbridge
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java]
  frameworks: [Android BLE, Classic RFCOMM]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: Freeyourgadget/Gadgetbridge
  url: https://github.com/Freeyourgadget/Gadgetbridge
  head: a0948ee1cbc2a870f91d313f8e37df5f524465f7
  cloned_to: "%TEMP%/YoAgentResearch/Freeyourgadget--Gadgetbridge"
studied_at: 2026-09-17
related:
  - research.NordicSemiconductor-Android-BLE-Library
  - research.LineageOS-android_frameworks_base
  - research.android-classic-ble-connection-architecture
  - research.synthesis.client-runtime
---

# Freeyourgadget/Gadgetbridge

## 入选理由

开源里最完整的「多设备、双运输、产品编排」。UI 不碰 GATT。`DeviceCommunicationService` 按设备建 `DeviceSupport`，连之前先 `dispose` 旧的。GATT 在 `BtLEQueue`：换连先 `disconnect()+close()`，再新 `connectGatt(TRANSPORT_LE)`。耳机走另一条：`AbstractHeadphoneDeviceSupport` 只开 RFCOMM，**不发起 A2DP**。这正好对照 HS01「经典 ACL + BLE 同时在、SPP 不是加耳机成功」。

GitHub 是镜像，本浅克隆 HEAD 停在 2024-12-22。规范与分层仍与现网 [gadgetbridge.org 开发文档](https://gadgetbridge.org/internals/development/project-overview/) / [New gadget tutorial](https://gadgetbridge.org/internals/development/new-gadget/) 一致。要追最新设备实现应改从 Codeberg 拉。

## 项目是什么

无云厂商替代 App。Pebble / 米环 / 索尼耳机 / Soundcore 等。运输可以是 BLE GATT、Classic RFCOMM、偶发 Wi-Fi。每种设备一个 `DeviceCoordinator` + `DeviceSupport`。

## 架构

```
Activity / DeviceService（客户端 API）
        │  Intent ACTION_CONNECT / DISCONNECT
        ▼
DeviceCommunicationService
        │  每台 GBDevice 一份 DeviceStruct
        │  再 connect：removeDeviceSupport() → dispose 旧的
        │  createDeviceSupport() → support.connect()
        ▼
DeviceSupport          （只此 Service 能碰实现）
   ├─ AbstractBTLE*     → BtLEQueue（GATT）
   ├─ AbstractSerial / AbstractHeadphone → IoThread（RFCOMM）
   └─ XiaomiSupport     → 二选一：XiaomiBleSupport 或 XiaomiSppSupport
        ▼
GBDevice.State
  NOT_CONNECTED → CONNECTING → CONNECTED → INITIALIZING → INITIALIZED
```

`GBDevice.State` 把「无线电已连」和「协议初始化完」分开。对外可用是 `INITIALIZED`，不是 `CONNECTED`。`AUTHENTICATION_*` 是设备协议配对，不是 Android `BOND_*`。

### BtLEQueue：GATT 生命周期

`connect()`（约 252–297 行）：

- 已连则忽略。
- `mBluetoothGatt != null`：注释写 *Tribal knowledge says you're better off not reusing existing BluetoothGatt connections*，先 `disconnect()`。
- `disconnect()`：引用置空，`gatt.disconnect(); gatt.close();`，状态立刻 `NOT_CONNECTED`。**不等 `onConnectionStateChange`。**
- 再 `connectGatt(context, false, cb, TRANSPORT_LE)`。注释：`autoConnect=true` 太容易失败。

`handleDisconnected`：若开了 autoReconnect 才 `mBluetoothGatt.connect()`；否则再 `disconnect()` 保证下次从干净对象开始。重连是策略，不是换设备补丁。

### 编排：换设备 = 换 Support

`connectToDevice`：该设备已在连就 `continue`。否则 `removeDeviceSupport`（`dispose`）再 `createDeviceSupport`。Support 与这一次会话同生共死，和 Nordic「推荐新实例」、Kable `Connection` 短寿是同一件事。

### 双运输怎么切

`XiaomiSupport.createConnectionSpecificSupport`：`ConnectionType.BOTH` 仍会落到 **一个** `XiaomiBleSupport` 或 `XiaomiSppSupport`。是运行时选路，不是 ACL 与 GATT 同时维持。

索尼耳机：`SonyHeadphonesIoThread` 继承 `BtClassicIoThread`，连厂商 UUID 的 RFCOMM。系统 Settings 已经把 A2DP/HFP 连上之后，Gadgetbridge 只做命令通道。`AbstractHeadphoneDeviceSupport.connect()` 只 `getDeviceIOThread().start()`。**加耳机成功不是这条 RFCOMM。**

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| UI → Service → Support，GATT 不下沉到 Activity | reuse-pattern | 对标 YoSPPApi / YoBLEApi 门面 |
| 再 connect 先 dispose 旧 Support | reuse-pattern | 不要 pendingReplace |
| `disconnect+close` 后立刻新 connectGatt | reuse-pattern | 这是换会话的正路 |
| `INITIALIZED` ≠ `CONNECTED` | reuse-pattern | 命令通道就绪再对 App 发布 |
| 耳机 RFCOMM 当「已戴上」 | anti-pattern | HS01 已禁止；索尼是厂商命令通道 |
| Xiaomi BOTH 只选一条运输 | lesson-only | 不能当 HS01 双模同时在线的样板 |
| GitHub 镜像停更 | lesson-only | 深研以本快照 + 官网文档为准 |

## 架构设计经验

- 产品编排层只调度 Support，不写 Gatt 超时。
- 运输层（BLE queue / RFCOMM thread）可替换；设备状态机共用。
- 系统音频 profile 与 App 命令通道是两条线。Gadgetbridge 耳机不发起 A2DP。

## 与当前工作

HS01 要的是 Gadgetbridge **没有整段做完**的组合：经典 ACL（用户可见已连）**和** GATT（命令）同时在。能搬的是编排与 GATT 对象寿命，不是索尼 RFCOMM，也不是小米二选一。

能直接用：Service 编排、dispose 再 create、BtLEQueue 的 close 后新连、State 分 CONNECTED/INITIALIZED。

必须改写：经典侧用 ACL Store + profile 发起，不要用 RFCOMM 当加耳机。

明确不要用：把 SPP/RFCOMM 成功回调当成 `onConnected`；用 autoConnect=true 当主路径。

## 阅读范围

读过：`DeviceCommunicationService` 的 connectToDevice / setDeviceSupport / removeDeviceSupport；`GBDevice.State`；`BtLEQueue` connect/disconnect/handleDisconnected；`AbstractHeadphoneDeviceSupport`；`SonyHeadphonesIoThread` 开头；`XiaomiSupport` 运输选择。未读各厂商协议、固件升级、扫描实现。
