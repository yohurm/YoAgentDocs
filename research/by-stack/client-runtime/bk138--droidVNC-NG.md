---
id: research.bk138-droidVNC-NG
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Java]
  frameworks: [MediaProjection, AccessibilityService, VNC]
also_relevant: []
utilization: [lesson-only, anti-pattern]
source:
  platform: github
  repo: bk138/droidVNC-NG
  url: https://github.com/bk138/droidVNC-NG
  head: 2b5c428
  cloned_to: "%TEMP%/YoAgentResearch/bk138--droidVNC-NG"
studied_at: 2026-09-10
related:
  - research.android-wireless-connect-paths
  - research.Genymobile-scrcpy
  - research.synthesis.client-runtime
---

# bk138/droidVNC-NG

## 入选理由

活跃的「无 ADB、无 root」Android 投屏开源对照。证明：第三方要在不开无线调试（甚至不开 USB 调试）时出画面，就必须 **装自己的 APK**，用 MediaProjection + 无障碍，而不是 scrcpy-server。用来划清 Yohu 不该走的产品边界。浅克隆 HEAD `2b5c428`（2026-09-07）。

## 项目是什么

Android 7+ VNC 服务器。F-Droid / Play 分发。电脑用任意 VNC 客户端或内置 noVNC。不经过 adbd。

## 架构

```text
用户点开始
  → MediaProjectionRequestActivity
        createScreenCaptureIntent()     # Android 10+ 每次都要同意
  → MediaProjectionService (FGS mediaProjection)
        getMediaProjection → VirtualDisplay
  → 编码/拷帧进 VNC framebuffer
  → 客户端连 :5900

控制（可选）
  → InputService extends AccessibilityService
        dispatchGesture / 全局动作（Home/Back/Recents）
```

README 写的「免再次授权」捷径是 `adb shell cmd appops set … PROJECT_MEDIA allow`。没有 ADB 就没有这条捷径。Android 11+ 开机自启还要求无障碍已经连上。

备用截屏（`EXTRA_FALLBACK_SCREEN_CAPTURE`）可少一次弹窗，但慢，且要无障碍。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 无 ADB 投屏 = 装包 + 录屏授权 +（控制则）无障碍 | lesson-only | 解释厂商之外第三方的真实代价 |
| 每次会话都要 MediaProjection 同意（官方，Android 14 更严） | lesson-only | 做不到管家那种无感 |
| 把 VNC/APK 当 Yohu 投屏主路径 | anti-pattern | 与官方 scrcpy-server、不装包、ADR-v6-015 相反 |
| 用无障碍冒充 scrcpy 注入 | anti-pattern | 能力弱、审核风险、和 shell 注入不是一条路 |

## 架构设计经验

- **权限模型决定产品形态。** scrcpy 用 `app_process` + shell 的 `INJECT_EVENTS`；普通 APK 没有。所以无 ADB 方案看起来「更亲民」，其实把复杂度从「开 USB 调试」换成「装包 + 无障碍 + 每次录屏」。
- **免弹窗仍然依赖 ADB。** 不能既宣传零调试选项，又用 `appops` 预授权。
- **VNC 是另一条视频管道。** 帧格式、控制、文件传输都不是 scrcpy。不能接到 `FramePipe`。

## 与当前工作

- 能直接用：向用户解释「为什么电脑管家能、Yohu 不能无感」时的对照。
- 不要用：在 Yohu 里带一只 VNC/录屏 APK；把无障碍当默认控制。

## 阅读范围

README、`MediaProjectionService.java`、`MediaProjectionRequestActivity.java`、`InputService.java` 头部与职责、`MainService.java` 启动步骤、`doc/Intent-Interface.md` 摘要。未读 native VNC 编码与 noVNC 前端。
