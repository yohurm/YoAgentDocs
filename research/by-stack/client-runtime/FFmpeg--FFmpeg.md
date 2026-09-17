---
id: research.FFmpeg-FFmpeg
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C]
  frameworks: [libswscale]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: FFmpeg/FFmpeg
  url: https://github.com/FFmpeg/FFmpeg
  head: b04bbd8
  cloned_to: "%TEMP%/YoAgentResearch/FFmpeg--FFmpeg"
  note: sparse checkout libswscale only
studied_at: 2026-09-15
related:
  - research.haasn-libplacebo
  - research.sekrit-twc-zimg
---

# FFmpeg/FFmpeg（仅 libswscale）

## 入选理由

经典「缩放是独立库」。`sws_getContext(srcW, srcH, srcFmt, dstW, dstH, dstFmt, flags, …)` 把 **几何、像素格式、算法旗标** 分成三组参数（`libswscale/swscale.h` 551–554）。`SWS_AREA` 是面积平均（202）。Yohu **禁止** 链 FFmpeg（ADR-v6-024/028）。本篇只读 API 分层，sparse 未拉解码器。

## 项目是什么

FFmpeg 的软件缩放/转格式库。新 API 把 `SwsContext.scaler` 与 `src_w/dst_w`、`intent`（色意图）拆开（227–286）。旧旗标里算法互斥：POINT / BILINEAR / BICUBIC / AREA / LANCZOS…（197–207）。

## 架构

```text
src 尺寸 + 格式 ──► SwsContext ──► dst 尺寸 + 格式
                      flags / scaler   算法
                      dither           量化
                      intent           色意图
                      gamma_flag       是否线性光
```

`sws_scale` 只跑已配置的上下文（583–585）。另有 `sws_scale_frame`。注释写明某些路径「Like sws_scale_frame, but without actually scaling」（395 附近）——转格式可以不缩放。

`utils.c` 里 `SWS_AREA`：**缩小是盒积分，放大退化成双线性**（注释 *downscale only, for upscale it is bilinear*）。未指定算法时默认 bicubic，且算法 bit **只能有一个**。`SwsContext.scaler` 正在从旧 flags 里拆出来，色意图 `SwsIntent` 仍是第三轴。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| dest 是 getContext 参数，不是旗标 | reuse-pattern | Fit 产出 dest；Scale 消费 dest |
| `SWS_AREA` 具名 | reuse-pattern | 缩小档叫面积，不是 mip |
| 整库 / swscale 进直播 | anti-pattern | ADR-v6-024；LGPL |
| 用 swscale 代替 VP | anti-pattern | 丢掉硬解零拷贝 |

## 架构设计经验

1. **三十年的 API 仍坚持「尺寸、格式、核」三分。** 焊成一次 blit 是捷径，不是架构。
2. **AREA 是缩小算法，POINT 是另一算法。** 不能用 POINT 冒充缩小清晰度。

## 与当前工作

- 能直接用：三分参数；面积当缩小档。
- 不要用：任何 FFmpeg 链接、exe、LoadLibrary。

## 阅读范围

`libswscale/swscale.h` 96–108、170–315、395、439、551–585；`utils.c` 176–370、1921–1939。未读汇编后端。
