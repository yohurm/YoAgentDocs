---
id: research.desktop-mirror-aspect-scale
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, TypeScript, Rust]
  frameworks: [SDL, Media Foundation, D3D11, WebCodecs]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  repo: desktop/mirror-aspect-scale
  url: https://github.com/Genymobile/scrcpy/blob/master/doc/window.md
  cloned_to: "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
studied_at: 2026-09-15
related:
  - research.synthesis.client-runtime
  - research.Genymobile-scrcpy
  - research.yume-chan-ya-webadb
  - research.mpv-player-mpv-examples
  - research.desktop-mirror-swapchain-chrome
---

# 投屏占用比例与清晰度

## 入选理由

Yohu 真机 USB 直播编码 **1220×2712**，硬解纹理常变成 **1248×2720**（32 对齐 / MF `STREAM_CHANGE`）。占用描边若跟纹理走，卡片比手机胖一圈，右侧/底侧画出对齐填充，看起来像「边框对不上、框里有黑边」。缩小 1220→446 再叠一次错误比例，画面发糊。本篇把官方客户端、WebCodecs 画布、已有嵌入槽笔记收成一条尺寸契约。

规范：scrcpy [`doc/window.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/window.md)、[`doc/video.md`](https://github.com/Genymobile/scrcpy/blob/master/doc/video.md) Size；Microsoft Learn [`ID3D11VideoContext::VideoProcessorSetStreamSourceRect`](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videocontext-videoprocessorsetstreamsourcerect)。源码：已克隆 scrcpy / ya-webadb；mpv 嵌入见既有篇。

## 项目是什么

主题深研，不是新仓库。回答两句：

1. 可见「手机框」跟谁的宽高比？
2. 缩小到占用盒时，谁负责清晰度？

## 架构

```
协议 session 宽高（设备这一次编码的内容）
  → 占用卡片 / 窗口比例 / 触控坐标   ← 唯一内容尺寸
硬解输出纹理（可更大：alignment / coded）
  → 只存在解码器与 GPU
  → blit 前裁到内容矩形（SDL 建纹理用 content；D3D 用 SourceRect）
占用盒（avail 内 contain）
  → dest 铺满占用盒（比例已相同）
  → 1:1 YUV→RGB（内容分辨率）再 mip / 三线性缩到 dest
```

scrcpy 默认把 **整个窗口** 锁到 `content_size`。工作台做不到改 avail 的格子，所以 **clip + 描边** 必须扮演那扇窗：卡片跟 session，avail 里多出来的是舞台底，不是框内 letterbox。

ya-webadb 默认 canvas backing store = `codedWidth×codedHeight`，CSS 再 contain；显示盒必须先被调成正确比例。他们承认 `coded` 和显示可见区不是一回事。

mpv `wid`：子窗铺满槽，letterbox 在播放器里。Yohu 已选「槽=avail、可见卡=clip」，letterbox 不得再在 clip 内侧用第二套尺寸算一遍。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 内容尺寸单源：session / EncodedFrame | reuse-pattern | 占用、描边、dest、触控、截图裁切同一份 |
| 纹理大于内容时裁源 | adapt | `VideoProcessorSetStreamSourceRect`；(0,0)–(content_w,content_h) 夹在纹理内 |
| 缩小走 mip / 三线性，整数倍放大才 nearest | reuse-pattern | 对标 scrcpy `texture.c`；禁止 VP 一次从真机压到 clip |
| 设备端先 `-m` 再投 | anti-pattern | 糊在编码器里，桌面救不回来 |
| 占用用 A、Present 用解码器 B、再 `fit_letterbox` | anti-pattern | 框内黑边 + 二次缩放 |
| 把 avail 当手机框 | anti-pattern | 描边铺满 HWND，画面 contain，必出「宽框 + 黑边」 |

## 架构设计经验

三种像素不得混用：

| 名字 | 来源 | 谁用 |
|------|------|------|
| 内容 | scrcpy session 包 → `EncodedFrame.width/height` → Live 事件 | Stage 占用 / dest / 描边 / 触控 |
| 纹理 | MF / VT / D3D `GetDesc` | 只建纹理、建 VP view |
| 占用 | `contain(avail, 内容)` | DComp clip、hairline、dest 原点 |

`even_px` / `STREAM_CHANGE` 改的是纹理，不是内容。Live 事件已经带 session 宽高：Loading 就可以把 clip 收到手机比例，不必等首帧 Present。

清晰度：**嵌入槽短边 991，对 2712 高的内容，mip 与整数栅格都错。** contain dest 是 **446×991**（约 0.37×，两边 >2:1）。`GenerateMips` + 三线性 lod≈1.45，混 610 与 **305**，比 dest 还软。整数 1/3 dest **406×904** 最近邻少画显示器像素，锯齿当「清晰」。scrcpy 作者称 mip 是 quick-and-dirty；MiniEngine 写明 >2:1 禁止单点双线性。

正确契约：占用 = contain dest（尽量多像素）；RGB = 内容 1:1 无 mip；每个 dest 像素面积平均其源矩形。放大才 nearest。禁止 mip 当缩小器，禁止为清晰去砍 `max_size`。详见 [缩小核](desktop--mirror-downsample.md)。

## 与当前工作

- 能直接用：USB `max_size=0`；session 宽高已在 `yohu-mirror` 泵和 `mirror/state`；VP 已有 SourceRect API。
- 必须改写：PictureBank / Stage 不得再发布或采用解码器输出尺寸；GPU 不得用纹理 desc 覆盖内容；`dest()` 不得在占用盒内再 contain。
- 不要用：恢复 YUV shader dest-rect；View 报 `video_width`；拉 `scrcpy.exe`；编码前降长边换「清晰」。

## 阅读范围

scrcpy `doc/window.md`、`doc/video.md` Size、`screen.c` 比例锁与 `compute_content_rect`、`texture.c` mipmaps、`Size.java` / `SurfaceEncoder.java` alignment。ya-webadb `canvas.ts` `canvasSize`、`webgl.ts` texel。mpv-examples 既有篇（嵌入槽 letterbox）。Microsoft Learn VP SourceRect。moonlight-qt 已另篇：contain 单函数 + 裁填充；不是整数 dest 样本。
