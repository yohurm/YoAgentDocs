---
id: research.NetrisTV-ws-scrcpy
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [Node.js, WebSocket, WebCodecs, MSE, WASM]
also_relevant: [windows-desktop, frontend]
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: NetrisTV/ws-scrcpy
  url: https://github.com/NetrisTV/ws-scrcpy
  cloned_to: "%TEMP%/YoAgentResearch/NetrisTV--ws-scrcpy"
studied_at: 2026-08-25
related:
  - research.synthesis.client-runtime
  - research.Genymobile-scrcpy
  - research.yume-chan-ya-webadb
---

# NetrisTV/ws-scrcpy

## 入选理由

最早把 scrcpy **嵌进浏览器**的完整产品（~2500 star，HEAD 2026-08-24）。Node 进程 push server、用 WebSocket 把视频/控制送到页面，页面里换解码器（MSE / Broadway WASM / TinyH264 / WebCodecs）。Yohu 同样是「本地后端 + WebView」。它证明：**视频不能走普通 HTTP/JSON，要有一条二进制通道**。同时它 **fork 了 scrcpy-server**（版本串 `1.19-ws8`），这是明确不要学的。

## 项目是什么

自托管网页：设备列表、投屏、触控、剪贴板、浏览器里 `adb shell`、文件列表、iOS 实验通路。依赖 PATH 上的 `adb`。README 写明用的是 **修改过的** scrcpy。

## 架构

```
HttpServer + WebSocketServer
    WebsocketMultiplexer（一条 WS 上开 channel）
        ScrcpyServer.run(device)
            push vendor/.../scrcpy-server.jar
            app_process 参数：版本=1.19-ws8, type=web, port=8886
            等 PID 文件 /data/local/tmp/ws_scrcpy.pid
        浏览器 StreamClientScrcpy
            StreamReceiverScrcpy
            可注册多个 Player（MSE / Broadway / TinyH264 / WebCodecs）
            GoogToolBox + FeaturedInteractionHandler（触控、多指模拟）
```

`Constants.ts`：`SERVER_VERSION = '1.19-ws8'`。这不是官方 4.x 协议。server 在设备上听 WebSocket 端口，而不是 scrcpy 的三路 TCP。PID 文件用于「进程在、WS 还没起来」的对账。

解码器做成插件：`registerPlayer`，按浏览器能力挑。WebCodecs 注释写明仅 Chromium 系——WebView2 正好落在这。

产品还塞了 xterm shell、文件 listing、APK 拖入、DevTools。和投屏共享同一个 WebSocket 多路复用器。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 本地后端 ↔ 页面 用二进制通道传编码帧 | adapt | Yohu 不要 Node；可用 localhost TCP、Tauri 自定义协议或专用事件流，但必须独立于 log 批量 IPC |
| 解码器按能力注册，WebCodecs 优先 | adapt | 与 ya-webadb 一致；Yohu 只做 WebCodecs，不带 WASM |
| 启停用 PID/世代对账，失败可杀旧版本 server | reuse-pattern | 对齐日志 capture generation |
| 修改 scrcpy-server 让设备直接听 WS | anti-pattern | 锁死 1.19；跟不上官方 4.x；多一份 Java 维护 |
| MSE 把 NAL 封 MP4 再喂 `<video>` | anti-pattern | 延迟和封装复杂度都高于 WebCodecs |
| 投屏模块顺手做 shell/文件/DevTools | anti-pattern | Yohu 已有终端/文件；模块禁止互 import |
| 依赖 PATH 里的 adb | anti-pattern | sidecar 解析顺序已有约定 |

## 架构设计经验

- **Fork server 是技术债的根。** 官方协议声明无兼容；fork 等于永远停在旧版本。Yohu 必须 pin **未修改** 的 GitHub Release `scrcpy-server-vX.Y`。
- **WebView 投屏的本质是「把已编码字节送到 VideoDecoder」。** 多路复用器、MSE、WASM 都是历史补丁。2026 年 WebView2 只保留 WebCodecs 一条。
- **一条 WS 承载所有模块会耦死。** 他们把文件/shell/投屏都挂 multiplexer。Yohu 模块 IPC 已经按域拆开；投屏通道只服务投屏。
- **fitToScreen 是显示策略，不是编码策略。** HEAD 刚把 `fitToScreen=true` 写进 stream 链接。面板适应窗口 ≠ 改 `max_size`。

## 与当前工作

- 能直接用：二进制传输与 log IPC 分离；WebCodecs 作为唯一播放器；启停对账。
- 必须改写：server 用官方 4.x；隧道在主机用 adb reverse/forward，不要设备听 8886；无 Node、无 webpack 播放器插件框架。
- 不要用：`1.19-ws8` jar、Broadway/TinyH264、把终端/文件塞进投屏页、iOS qvh。

## 阅读范围

`README.md` 功能与解码器列表、`src/common/Constants.ts`、`src/server/goog-device/ScrcpyServer.ts`、`src/server/mw/WebsocketMultiplexer.ts`、`src/app/googDevice/client/StreamClientScrcpy.ts`。未读 iOS 通路与文件 listing 细节。
