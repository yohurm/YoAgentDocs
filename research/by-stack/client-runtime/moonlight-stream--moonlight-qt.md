---
id: research.moonlight-stream-moonlight-qt
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [Qt, SDL, D3D11VA, FFmpeg]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: moonlight-stream/moonlight-qt
  url: https://github.com/moonlight-stream/moonlight-qt
  head: e3fd29e
  cloned_to: "%TEMP%/YoAgentResearch/moonlight-stream--moonlight-qt"
studied_at: 2026-09-15
related:
  - research.desktop-mirror-aspect-scale
  - research.desktop-mirror-downsample
---

# moonlight-stream/moonlight-qt

## 入选理由

远程串流客户端。**一份 contain 函数**同时喂给所有渲染后端和鼠标/触控。D3D11VA 把 `frame->width` 与硬解纹理 `hw_frames_ctx->width` 拆开，UV 裁掉 alignment 填充。Yohu 已有 session 内容 vs 纹理；本篇给跨后端证据。

## 项目是什么

GameStream / Sunshine 桌面客户端。解码后按窗口画，键鼠映射回桌面坐标。不是工作台嵌入模块。

## 架构

```text
StreamUtils::scaleSourceToDestinationSurface(src, dst)
        │
        ├─ d3d11va / dxva2 / sdlvid / egl / vaapi / vdpau / drm / plvk / mmal
        ├─ session.cpp（叠加层几何）
        ├─ input/mouse.cpp
        └─ input/abstouch.cpp
```

实现就是一次 contain（`streamutils.cpp` 128–141）：按源宽高比改 `dst`，多出来的边居中。调用方先把 `dst` 设成窗口，函数再收成贴合盒。

D3D11VA（`d3d11va.cpp` 888–908）：dest 用 `frame->width/height` 跑同一 helper；UV `uMax = frame->width / hw_frames_ctx->width`，注释写明 **Don't sample from the alignment padding area**。这就是 Yohu 的 `SetStreamSourceRect` / session vs 1248×2720 纹理。

缺口：Fit 统一了，**Convert 与 Scale 仍多半是一次 blit**（D3D11VA：YUV→RGB + 线性采样 + NDC 四边形一次 `DrawIndexed`）。视频路径永远 `MIN_MAG_MIP_LINEAR`；`SDL_ScaleModeNearest` / `GL_NEAREST` 只给叠加层。README 里的 `v7.349.0` 是 **libplacebo 版本号**，不是整数缩放 issue。仓内没有整数 dest 模式。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 一份 contain，渲染 + 输入共用 | reuse-pattern | Yohu：`present_dest` + `map_client_to_video` 同一 `Letterbox` |
| coded / display 拆开，裁填充 | reuse-pattern | 内容尺寸锁占用；纹理只建资源 |
| 各后端自己再 contain 一次 | anti-pattern | moonlight 已经避免了；Yohu 也曾在 occupancy 后再 `fit_letterbox` |
| 每事件重算 contain、不缓存 Letterbox | anti-pattern | 13 个文件 15 处调用；Yohu 按 avail 变一次算完 |
| 输入用 stream 宽高、渲染用 frame 宽高 | anti-pattern | 填充裁切失败时触控与画面会分叉；Yohu 只认 session 内容 |
| 把 README `v7.349.0` 当整数缩放 issue | anti-pattern | 那是 libplacebo semver |
| FFmpeg 解码栈 | anti-pattern | ADR-v6-024 |

## 架构设计经验

1. **Fit 是跨后端契约，不是 D3D 细节。** 九个 renderer 都调用同一函数。
2. **输入必须吃同一 dest。** 否则触控点落在黑边或错格。
3. **裁填充属于 Convert 前的源矩形，不属于核。** UV 裁完再采样。
4. **只有 Fit、没有 Scale 层，清晰度只能碰采样器。** Yohu 不能停在这一层。
5. **色度 UV 也要裁。** `d3d11va.cpp` 1000–1004 在有填充时另算 `chromaUVMax`；420 平面不能只裁 luma。

## 与当前工作

- 能直接用：contain 单函数；触控跟 dest；裁 alignment。
- 必须改写：Yohu 还要独立 Scale 模块（moonlight 没拆）。
- 不要用：一次 blit 当完整呈现架构；再引入 FFmpeg；把 README 版本号当 issue。

## 阅读范围

`app/streaming/streamutils.cpp` 128–157；`streamutils.h` 15；`ffmpeg-renderers/d3d11va.cpp` 888–916、1000–1004、1060–1073、1615；`input/mouse.cpp` 100–140；`abstouch.cpp` 62–80；`README.md` libplacebo `v7.349.0`。未跟完每个 renderer 的 blit 核。
