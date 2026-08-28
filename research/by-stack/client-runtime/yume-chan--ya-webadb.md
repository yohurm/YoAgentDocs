---
id: research.yume-chan-ya-webadb
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [WebCodecs, WebUSB, Streams]
also_relevant: [windows-desktop, frontend]
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: yume-chan/ya-webadb
  url: https://github.com/yume-chan/ya-webadb
  cloned_to: "%TEMP%/YoAgentResearch/yume-chan--ya-webadb"
studied_at: 2026-08-25
related:
  - research.synthesis.client-runtime
  - research.Genymobile-scrcpy
  - research.NetrisTV-ws-scrcpy
---

# yume-chan/ya-webadb

## 入选理由

唯一把 **官方 scrcpy-server 协议**做成可组合 TypeScript 包、并且用 **WebCodecs** 在 Chromium 里解码的活跃实现（~3200 star，HEAD 2026-08-20）。Yohu 的 UI 就是 WebView2，这条解码路径零额外原生库。包边界清楚：`@yume-chan/scrcpy` 只谈协议，`@yume-chan/adb-scrcpy` 用他们自己的 ADB 客户端做 push/隧道，`@yume-chan/scrcpy-decoder-webcodecs` 只解码渲染。文档：[tangoadb.dev/scrcpy](https://tangoadb.dev/scrcpy/)。

## 项目是什么

Tango：浏览器里的 ADB + scrcpy。产品是 tangoapp.dev；库是一套 MIT 包。**不是**桌面工作台。ADB 走 WebUSB 或 Node TCP，等于重实现了 ADB 客户端。

## 架构

```
@yume-chan/adb          ← 重实现 ADB（Yohu 禁止抄）
        │
@yume-chan/adb-scrcpy   ← pushServer + reverse/forward + 按序打开 3 socket
        │
@yume-chan/scrcpy       ← 按 server 大版本分目录（1_15 … 4_1）
        │                 帧头解析 / 控制消息序列化 / Options key=value
        ▼
@yume-chan/scrcpy-decoder-webcodecs
        VideoDecoder + 渲染器优先级：
        InsertableStream → <video>
        WebGL canvas
        Bitmap canvas
        另有 snapshot：canvas → Blob（截图）
```

`AdbScrcpyClient.start`：push jar → 建 `AdbScrcpyConnection`（Forward 带 dummy byte + 最多 100 次重试）→ `app_process` → 得到 `videoStream` / `audioStream` / `controlStream`。`AdbScrcpyExitedError` 带 server stdout，启动失败可对账。

`@yume-chan/scrcpy` **不依赖 Web/Node API**，也不依赖他们的 ADB 包。协议按版本文件夹演进，`latest.ts` 指向当前。控制消息拆成 `inject-key-code` / `inject-text` / `uhid` / `start-app` / `resize-display` 等小模块。

解码器 `WebCodecsVideoDecoder.isSupported = typeof VideoDecoder !== "undefined"`；capabilities 声明 h264/h265/av1。暂停是解码器级 `PauseController`，不是停设备编码。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 协议与 ADB 分家：scrcpy 包零 ADB 依赖 | reuse-pattern | Yohu 应对齐：`yohu-adb` 只做隧道/进程，`yohu-mirror` 做帧头与会话 |
| 大版本目录（`4_1/`）锁协议 | reuse-pattern | sidecar server 版本与解析器版本同一常量 |
| WebCodecs + canvas/video 渲染优先级 | adapt | WebView2 有 VideoDecoder；UI 模块画布用 `@yohu/ui` 壳包一层，不要引入他们的组件 |
| canvas → Blob 截图 | adapt | 不经过 core 再走一遍视频；保存对话框已有 |
| Forward 隧道 dummy byte + 重试 | reuse-pattern | 与 scrcpy develop.md 一致 |
| 把 `@yume-chan/adb` 接到 WebView | anti-pattern | ADR-v6-008；Yohu 已有官方 adb sidecar |
| UI 依赖 `@yume-chan/adb-scrcpy` 直接 start | anti-pattern | 会把 ADB 会话拖进前端；编排必须在 core |
| tinyh264 / WASM 软解当主路径 | anti-pattern | WebView2 有硬件 WebCodecs；WASM 是旧浏览器补丁，不是本栈主路径 |

## 架构设计经验

- **解码是 UI 能力，部署是 core 能力。** 他们把两者分成三个包；Yohu 对应 UI 消费端 vs `yohu-mirror` 会话。不要在 Solid 视图里 `adb push`。
- **协议要版本化，不要「跟最新 scrcpy 源码」。** 每个 server 发行版一个解析实现。升级 server = 升级解析器 + 换 sidecar 文件，一次提交。
- **暂停画面 ≠ 停采集。** 与日志 Space 冻可见区同构：解码器可暂停，设备编码可继续（或按产品选择停 socket）。
- **MIT 协议代码可改编进 Rust。** 若不想前端 npm 依赖 `@yume-chan/scrcpy`，就在 `yohu-protocol` 写帧头；不要复制他们的 ADB 栈。

## 与当前工作

- 能直接用：WebCodecs 是否可用的探测；渲染器降级顺序；截图走 canvas；启动失败收集 server 输出。
- 必须改写：push/隧道/shell 全部经现有 `yohu-adb` sidecar；视频字节如何进 WebView（localhost 流 / 专用 IPC，不是他们的 Adb socket）。
- 不要用：`@yume-chan/adb`、WebUSB、在模块里 npm 引入整棵 tango 应用壳。

## 阅读范围

`libraries/adb-scrcpy/src/client.ts`、`connection.ts`、`libraries/scrcpy/src/index.ts`（版本表到 4_1）、`libraries/scrcpy-decoder-webcodecs/src/video/decoder.ts`、`render/` 目录、`utils/snapshot.ts`。未读音频播放器与 HID 全量。
