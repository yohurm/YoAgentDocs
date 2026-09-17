---
id: research.gstreamer-gst-plugins-base
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C]
  frameworks: [GStreamer]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: gstreamer/gst-plugins-base
  url: https://github.com/gstreamer/gst-plugins-base
  head: ce937bc
  cloned_to: "%TEMP%/YoAgentResearch/gstreamer--gst-plugins-base"
  note: sparse gst/videoscale + gst-libs/gst/video
studied_at: 2026-09-15
related:
  - research.FFmpeg-FFmpeg
  - research.obsproject-obs-studio
---

# gstreamer/gst-plugins-base（videoscale）

## 入选理由

管道里 **videoscale 与 videoconvert 是两个元件**。dest 宽高来自 caps；`method` 是元件属性。LGPL。Yohu 不引 GStreamer，要的是「管道元素 = 层」。

## 项目是什么

`GstVideoScale` 继承 `GstVideoFilter`（`gst/videoscale/gstvideoscale.h` 31–32）。结构体同时有 `method` 和 `GstVideoConverter *convert`（73、83）：尺寸变换走 method，像素格式仍可能经 converter——但对外仍是「缩放元件」，转色主路径是隔壁的 `videoconvert`。

## 架构

```text
… ! videoconvert ! videoscale method=lanczos ! …
              色/格式              caps 宽高 + method
```

`GstVideoScaleMethod`（50–62）：NEAREST / BILINEAR / 4TAP / LANCZOS / BILINEAR2 / SINC / HERMITE / SPLINE / CATROM / MITCHELL。属性文档把 NEAREST 写成 *fast and ugly*（37）。`gstvideoscale.c` 618+ 把 method 映射到 `GstVideoResampler` 的 taps/cubic 参数——核实现可换，caps 协商的 dest 不变。

`add_borders` 是 letterbox 旗标（74），与 method 并列：黑边是几何，不是核。`set_info` 里对内部 converter 设 `GST_VIDEO_MATRIX_MODE_NONE` + `GST_VIDEO_CHROMA_MODE_NONE`（约 698–702），**缩放元件禁止顺手转色**。源与 dest 同尺寸且无边时 `set_passthrough`（608–614）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| videoscale ≠ videoconvert | reuse-pattern | Convert / Scale 两模块 |
| dest 来自 caps，method 来自属性 | reuse-pattern | Fit 产出 Letterbox；核模块只读它 |
| add_borders 独立于 method | reuse-pattern | 占用黑边在舞台，不在核里 |
| 嵌 GStreamer 管道 | anti-pattern | 壳已是 Tauri + MF |

## 架构设计经验

1. **管道图让层可见。** 焊在一个 blit 里就无法单独测核。
2. **NEAREST 被文档标成丑。** 只该用于不缩小。

## 与当前工作

- 能直接用：两元件边界；borders 与 method 分字段。
- 不要用：GStreamer 运行时、把 letterbox 画进核。

## 阅读范围

`gst/videoscale/gstvideoscale.h`；`gstvideoscale.c` method 枚举、属性、618 附近 resampler 映射。未读完 gst-libs 全部 converter。
