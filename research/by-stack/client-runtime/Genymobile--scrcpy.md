---
id: research.Genymobile-scrcpy
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, Java]
  frameworks: [SDL, FFmpeg, MediaCodec, adb sidecar]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: Genymobile/scrcpy
  url: https://github.com/Genymobile/scrcpy
  cloned_to: "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
studied_at: 2026-08-25
related:
  - research.synthesis.client-runtime
  - research.yume-chan-ya-webadb
  - research.barry-ran-QtScrcpy
  - research.NetrisTV-ws-scrcpy
---

# Genymobile/scrcpy

## 入选理由

Android 投屏的协议源。官方 `scrcpy-server` 用 `app_process` 在设备上跑，不装 APK、不 root。桌面客户端是 C + SDL + FFmpeg（`app/src/server.c` 负责 push/隧道/启动）。Yohu 要 **按这份源码自己写工作台客户端**（画面进模块面板），不是把 `scrcpy.exe` 当黑盒拉起。协议无前后兼容：客户端版本字符串必须与 server 完全一致。先前一篇只读了拖入 push；本轮补视频/控制/隧道。浅克隆 HEAD `2926c06`（2026-07-12，v4.1）。

文档：[`doc/develop.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/develop.md)、[`doc/video.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/video.md)、[`doc/control.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/control.md)、[`doc/window.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/window.md)。

## 项目是什么

把 Android 屏幕（可选音频、摄像头）镜像到电脑并可用键鼠控制。不是文件管理器，也不是工作台模块。设备侧编码，电脑侧解码显示。

## 架构

```
adb push scrcpy-server → /data/local/tmp/scrcpy-server.jar
adb reverse localabstract:scrcpy_<SCID> tcp:27183   # 默认；失败再 forward
adb shell CLASSPATH=... app_process / com.genymobile.scrcpy.Server <version> key=value...

设备 MediaCodec(Surface) ──video socket──► 客户端 demuxer → decoder(FFmpeg) → SDL
设备 AudioRecord/MediaCodec ──audio socket─► demuxer → decoder → 播放器
客户端 SDL 事件 ──control socket──► Controller.injectInputEvent（双向：剪贴板回传）
```

要点：

- **三套独立 socket**，按 video → audio → control 顺序打开；任一可关，但不能全关。
- **默认 reverse**：电脑先听，设备来连，避免 forward 的竞态。forward 时第一路 socket 先收 1 字节 dummy，用来探测「隧道在、server 还没起来」。
- **SCID** 31-bit 随机，同一设备可多实例。
- **视频**：先 `u32` codec id（`h264`/`h265`/`av1`/`vp8`/`vp9`），旋转时发 12 字节 session packet（宽高），之后每包 12 字节帧头（config/key/PTS/size）+ MediaCodec payload。客户端**不感知旋转**，只跟帧尺寸。
- **编码在设备**：`ScreenCapture` + `SurfaceEncoder`；`KEY_REPEAT_PREVIOUS_FRAME_AFTER` 解决首帧空白和运动后糊尾。
- **控制**可关（`--no-control` / `-n`）：只看不摸。
- **质量**：默认 8 Mbps；`--max-size` 限制长边并保比例；`--max-fps`；编码失败自动降分辨率。
- **清理**：默认退出删掉设备上的 server；不留安装痕迹。

官方 win64 发行包还带 FFmpeg/SDL/adb。那是 **SDL 独立窗口客户端** 的体积，不是 Yohu 的否决条件。Yohu 不搬这条客户端：UI 已是 WebView2，像素应在面板里画，而不是再开 SDL 窗。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 只部署官方 `scrcpy-server`，用现有 adb 做 push/reverse/shell | reuse-pattern | 设备侧编码器不重写；ADB 仍走 sidecar（ADR-v6-008） |
| 版本锁死 + `key=value` 启动参数 | reuse-pattern | `audio=false control=false max_size=…` 即可做只读投屏 |
| reverse 优先、forward 回退、SCID 区分实例 | reuse-pattern | 与日志「每设备一路」同类：每 serial 至多一路投屏 |
| 会话槽位：push → 隧道 → app_process 长驻 → 按序 accept | reuse-pattern | 对齐 ADR-v6-016 的 Empty/Starting/Live/Stopping |
| 帧头 demux 后把 **encoded packet** 交给 UI，不在 core 解 H.264 | reuse-pattern | WebView2 用 WebCodecs 画进面板；core 解成像素再塞进 WebView 是多余一跳 |
| 整包 `scrcpy.exe` + SDL 窗口当模块画面 | anti-pattern | 黑盒进程，画面不在工作台；与「源码实现客户端」相反 |
| 视频走 log 风格逐行/逐帧 invoke | anti-pattern | 8 Mbps 量级；ADR-v6-007 是 logcat 行，不是视频 |
| 自己写 MediaCodec / 改 fork server | anti-pattern | 隐藏 API 反射按 Android 版本分叉；协议无兼容 |
| `.apk` 丢进画面即 install | anti-pattern | 文件/终端模块已有路径；投屏不要兼包管理 |

## 架构设计经验

- **投屏协议 ≠ ADB 协议。** ADR-v6-008 禁的是重写 ADB；scrcpy 帧头/控制消息可以（也必须）在 core 实现或从 MIT 实现改编，ADB 仍走 sidecar。
- **编码在设备、显示在壳。** SDL 客户端用 FFmpeg 是因为它自己画窗。Yohu 壳是 WebView2，解码应在页面 WebCodecs。不是因为安装包体积，是因为分层：core 出编码包，View 出画面。
- **只读是一等公民。** `--no-control` 关掉整条 control socket，不是 UI 里忽略点击。产测看屏应默认这条。
- **旋转是 server 的事。** UI 只按新宽高改 canvas，不要自己转矩阵。
- **长驻进程是第三种 ADB 能力。** 短命令收集退出码、流式泵 stdout、投屏则是「进程活着 + 另开 TCP」。现有 `run_streaming` 不够。

## 与当前工作

### 投屏显示（本轮）

- 能直接用：官方 server 二进制作 sidecar；`adb push` / `adb reverse|forward` / `adb shell app_process`；只读启动参数；每设备一路 + 掉线停槽。
- 必须改写：解码/渲染不在 SDL；视频通道不能走 log 批量事件；`selectionMode` 从 `none` 改为 `SingleRequired`；任务中心登记长驻会话；退出 3s 强杀进程树（已有约定）。
- 不要用：拉起 `scrcpy.exe` 当模块、改 server 源码、JPEG `screencap` 轮询、把投屏页做成第二套文件管理器。

### 文件拖入（2026-08-20，仍有效）

- 能直接用：`listen('tauri://drag-drop')` → 命中文件面板 → `files.push`。
- 不要用：无反馈的「丢进窗口就算成功」、APK 安装捷径、push 目标写死 Download。

## 阅读范围

本轮：`doc/develop.md`（连接/协议/帧头）、`doc/video.md`、`doc/control.md`、`doc/window.md`、`doc/shortcuts.md`、`README.md` 功能列表、`server/src/main/java/com/genymobile/scrcpy/video/SurfaceEncoder.java` 目录与 `ScreenCapture.java` 存在性。未读 OTG/HID/摄像头/V4L2 实现细节。

先前：`app/src/file_pusher.c`、`input_manager.c`、`adb/adb.c`、`doc/control.md` File drop、`cli.c` `--push-target`。
