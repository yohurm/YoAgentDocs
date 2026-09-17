---
id: research.desktop-mirror-downsample
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, HLSL, Rust]
  frameworks: [OpenGL, D3D11, libplacebo]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  repo: desktop/mirror-downsample
  url: https://github.com/Genymobile/scrcpy/issues/1394
  cloned_to: "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
studied_at: 2026-09-15
related:
  - research.synthesis.client-runtime
  - research.desktop-mirror-aspect-scale
  - research.Genymobile-scrcpy
  - research.haasn-libplacebo
  - research.mpv-player-mpv
  - research.moonlight-stream-moonlight-qt
  - research.obsproject-obs-studio
  - research.Blinue-Magpie
  - research.sekrit-twc-zimg
  - research.lemenkov-libyuv
  - research.FFmpeg-FFmpeg
  - research.gstreamer-gst-plugins-base
  - research.microsoft-DirectX-Graphics-Samples
---

# 投屏缩小：面积核，不是栅格或 mip

## 入选理由

Yohu 嵌入槽真机约 **1008×991**，USB 内容 **1220×2712**。contain  dest ≈ **446×991**（约 0.37×，两边都 >2:1）。上一轮把 dest 收成整数 1/3（406×904 + 最近邻）是为了躲开 `GenerateMips` 的 lod 1.45；实机仍糊/糙，因为那是换了一种错误栅格，不是换了一条缩小契约。

规范与源码：scrcpy `texture.c` + [issue 1394](https://github.com/Genymobile/scrcpy/issues/1394)；MiniEngine 已克隆 [`GenerateMipsCS.hlsli`](microsoft--DirectX-Graphics-Samples.md)；缩小核分层见 libplacebo / mpv / OBS / Magpie / zimg / libyuv / swscale / GStreamer / moonlight（均已克隆，见各篇）。

## 项目是什么

主题深研。只回答：短嵌入槽里，从内容分辨率画到 dest 时，**dest 取多少像素、每个 dest 像素怎么读源**。

## 架构

```
内容（session）──contain(avail)──► dest（占用 = dest，尽量多像素）
硬解纹理 ──SourceRect 裁内容──► RGB = 内容（1:1，无 mip 链）
每个 dest 像素 ──面积平均其源矩形──► 回缓冲 viewport
```

对照三条被否决的通路：

| 通路 | dest | 取样 | 真机 1220×2712 @ 1008×991 |
|------|------|------|---------------------------|
| mip + 三线性 | contain 446×991 | lod≈1.45，混 610 与 **305** | 比 dest 还软 |
| 整数栅格 + 最近邻 | 406×904（更少像素） | 每 3×3 取 1 | 锯齿 + 信息更少 |
| **面积核** | contain 446×991 | 3×3 盖住该像素的源矩形 | 用满格子，不降到 1/4 mip |

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| dest = contain，占用跟 dest | reuse-pattern | scrcpy `compute_content_rect`；边框已按此收口 |
| 作者原话：mip 不是最好的缩小 | reuse-pattern | scrcpy#1394；Windows SDL Direct3D 路径甚至不建 mip |
| >2:1 禁止单点双线性 | reuse-pattern | MiniEngine：「One bilinear sample is insufficient when scaling down by more than 2x」 |
| 核半径跟缩小比走 | adapt | mpv `correct-downscaling`；Yohu 用源矩形上的 3×3 盒，不搬 libplacebo |
| 整数缩小 dest | anti-pattern | 嵌入槽远小于手机；少画 40px 宽换「整数」是丢清晰度 |
| GenerateMips 当缩小器 | anti-pattern | 2×2 递归 + 三线性 lod，不是对准 dest 的核 |
| 为清晰砍 `max_size` | anti-pattern | 糊进编码器 |

## 架构设计经验

1. **清晰度先是 dest 像素数，再是核。** 整数 1/3 比 contain 少约 10% 宽、9% 高，再怎么 nearest 也补不回这些显示器像素。
2. **mip 是给「随便缩一下」的预计算，不是对准某一 dest 的滤波器。** scrcpy 用负 LOD bias 补锐，仍承认不如手写 bicubic。MiniEngine 原文：缩超过 2× 一次双线性不够。
3. **>2:1 必须覆盖源足迹。** 单次 `Sample` 只看邻域 2×2；2.73× 会漏源 texel。4 点双线性（MiniEngine `NON_POWER_OF_TWO==3`）或 3×3 盒都可以；禁止再 `GenerateMips`。
4. **放大才 nearest。** 1:1 点采。缩小走面积核 + 线性 sampler（无 mip 层）。
5. **Fit 与 Scale 必须分模块。** 十份源码共同层：几何 dest ≠ 转色 ≠ 核 ≠ 合成。OBS 在 `dest<src/2` 静默丢掉 Area、moonlight 只有 Fit 没有核层，都是反例。详见横向总结「呈现五层」。

## 与当前工作

- 能直接用：内容/纹理/占用三分；VP 1:1 转 RGB；USB `max_size=0`。
- 必须改写：删 `present_raster` 作占用契约；Stage dest 回到 contain；GPU 一趟面积缩小。
- 不要用：mip bias 微调、整数栅格与 contain 双轨、VP 一次从垫过的纹理双线性压到 clip。

## 阅读范围

scrcpy `app/src/texture.c`、`screen.c` `compute_content_rect`、issue 1394。MiniEngine 克隆后的 `GenerateMipsCS.hlsli` 77–113。mpv / libplacebo / OBS / Magpie / zimg / libyuv / swscale / gst videoscale / moonlight 见各自深研篇。不再「只读手册不克隆」。
