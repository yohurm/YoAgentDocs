---
id: research.LineageOS-android_frameworks_base
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java]
  frameworks: [AOSP SettingsLib, Bluetooth profiles]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: LineageOS/android_frameworks_base
  url: https://github.com/LineageOS/android_frameworks_base
  head: 781c37c3f3c8566177b85ff80637e7affc834858
  cloned_to: "%TEMP%/YoAgentResearch/LineageOS--android_frameworks_base"
studied_at: 2026-09-17
related:
  - research.LineageOS-android_packages_modules_Bluetooth
  - research.Freeyourgadget-Gadgetbridge
  - research.android-classic-ble-connection-architecture
  - research.synthesis.client-runtime
---

# LineageOS/android_frameworks_base（SettingsLib 蓝牙）

## 入选理由

系统设置如何看待「一台经典耳机」。这是 AOSP 对 BR/EDR + profile 的标准模型，不是第三方 GATT 库。稀疏克隆只取 `packages/SettingsLib/src/com/android/settingslib/bluetooth`。本快照 2026-09-15（lineage 默认枝）。

官方 profile 步骤见 [Bluetooth profiles](https://developer.android.com/develop/connectivity/bluetooth/profiles)：`getProfileProxy` → 用 proxy 观察/操作。设置里的「已连接」是 **任一音频 profile 已连**，不是 ACL，也不是 GATT。

## 项目是什么

`LocalBluetoothManager` 组装三件套：

- `CachedBluetoothDeviceManager`：每台远程设备一个 `CachedBluetoothDevice`
- `BluetoothEventManager`：广播 → 主线程 Handler
- `LocalBluetoothProfileManager`：A2DP / Headset / HearingAid / LeAudio… 各一个 `LocalBluetoothProfile`

## 架构

```
BluetoothEventManager
   ACL_CONNECTED / DISCONNECTED  →  CachedDevice.onAclStateChanged(state, transport)
   BOND_STATE_CHANGED            →  onBondingStateChanged
   ACTION_UUID                   →  补 profile 再 connect
   A2DP/HFP CONNECTION_STATE     →  onProfileStateChanged

CachedBluetoothDevice
   connect()
      ensurePaired()          // BOND_NONE → createBond()，本次返回
      connectDevice()         // mDevice.connect()   ← 特权
   isConnected()              // 任一 profile == STATE_CONNECTED
   isBusy()                   // profile 正在连/断，或 BOND_BONDING
   mIsAclConnectedLe / BrEdr  // ACL 只记账，不定义 isConnected
```

### 发起

当前树的 `connectDevice()` 调 `BluetoothDevice.connect()`（即 `connectAllEnabledProfiles`）。Settings 是特权应用，过得了 `BLUETOOTH_PRIVILEGED`。**第三方抄这条会在摩托罗拉 Android 15 等机型上 `SecurityException`。** 这与 HS01 已测事实一致。

单 profile 路径 `connectInt` → `profile.setEnabled(device, true)`。`A2dpProfile.setEnabled` 走 `BluetoothA2dp.setConnectionPolicy(ALLOWED|FORBIDDEN)`，不是隐藏的 `BluetoothA2dp.connect()`。`setConnectionPolicy` 同样不是给普通 App 当主路径的。

未配对：`ensurePaired()` 只 `createBond()`，等 `ACTION_UUID` / 后续 `connect()`。配对和连 profile 是两步。`isBusy()` 把 `BOND_BONDING` 算忙，不算断。

### 观察

`AclStateChangedHandler` 读 `EXTRA_TRANSPORT`，默认当 BR/EDR。`onAclStateChanged` 分别记 LE / BR-EDR ACL，并只在「两条 ACL 都还没起来」时刷新 `mConnectAttempted`。**`isConnected()` 仍然只扫 profile。** 系统设置关心「能不能出声」，不关心「ACL 在不在」。

`BOND_NONE`：清 `mProfiles`，不把 ACL 字段当成用户已连。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| EventManager：ACL / bond / UUID / profile 分 Handler | reuse-pattern | 对标 ClassicBtConnectionMonitor |
| 配对与连 profile 分步 | reuse-pattern | `createBond` 后才发起 |
| ACL 分 LE / BR-EDR | reuse-pattern | API 33+ `EXTRA_TRANSPORT` |
| `isConnected` = 任一 A2DP/HFP | anti-pattern | HS01 已定：profile 断 ≠ 经典断 |
| `mDevice.connect()` | anti-pattern | 特权；第三方不能当发起 |
| `setConnectionPolicy` | anti-pattern | 同样不是 3P 主路径 |

## 架构设计经验

- 「设备」聚合多个 profile，每个 profile 自己的连接态。不要用一个布尔代替。
- 谁定义「已连接」取决于产品：Settings = 音频 profile；陪伴 App = ACL（加耳机）+ GATT（命令）。
- 系统发起用特权 API；第三方只能：`createBond` + 隐藏 `BluetoothA2dp.connect` / `BluetoothHeadset.connect`（只查 CONNECT/ADMIN）。Binder 事实见既有 AOSP Bluetooth 模块深研，不要以 API 37 javadoc 的 CDM 说法为准。

## 与当前工作

`ClassicAclStateStore`（`linkUp && (BONDING\|BONDED)`）比 Settings 的 `isConnected()` 更适合 HS01。Settings 证明：**ACL 事件要收，但不能用 A2DP/HFP 定义用户可见已连。**

能直接用：广播分发结构、配对/连 profile 分步、ACL 分运输。

必须改写：发起改走 profile `connect` 隐藏 API（HS01 已有 `ClassicBtProfileConnect`）；不要 `BluetoothDevice.connect()`。

明确不要用：把 Settings 的 `isConnected()` 搬过来当 Store。

## 阅读范围

读过：`CachedBluetoothDevice` 的 connect / ensurePaired / isConnected / isBusy / onBondingStateChanged / onAclStateChanged；`BluetoothEventManager` 注册与 Bond/Acl Handler；`A2dpProfile` 的 proxy 与 `setEnabled`；`LocalBluetoothProfileManager` 如何挂 A2DP/HFP。未读 HearingAid / LeAudio / CSIP 细节，未读整棵 frameworks_base。
