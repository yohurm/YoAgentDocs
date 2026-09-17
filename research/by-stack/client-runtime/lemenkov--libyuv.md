---
id: research.lemenkov-libyuv
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, C++]
  frameworks: []
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: lemenkov/libyuv
  url: https://github.com/lemenkov/libyuv
  head: 2dd4257
  cloned_to: "%TEMP%/YoAgentResearch/lemenkov--libyuv"
  note: unofficial GitHub mirror of https://chromium.googlesource.com/libyuv/libyuv ；github.com/google/libyuv 不存在
studied_at: 2026-09-15
related:
  - research.sekrit-twc-zimg
  - research.FFmpeg-FFmpeg
---

# lemenkov/libyuv

## 入选理由

Chromium / WebRTC 的 YUV 库。公开头文件把 **Convert** 和 **Scale** 收成两套 API：`convert.h` / `convert_argb.h` 对 `I420ToARGB`；`scale.h` 对 `I420Scale` + `FilterMode`。官方不在 `github.com/google/libyuv`。BSD。Yohu 不链 libyuv（热路径是 MF VP），要的是「转色函数不带 dest 宽高，缩放函数不带矩阵」。

## 项目是什么

平面转换、旋转、缩放。入口 `include/libyuv.h` 分别 `#include` convert 与 scale，不合成一个 `Blit()`。

## 架构

`FilterMode`（`scale.h` 24–29）：

| 枚举 | 含义 |
|------|------|
| `kFilterNone` | 点采，最快 |
| `kFilterLinear` | 只水平滤 |
| `kFilterBilinear` | 比 box 快，缩小质量较差 |
| `kFilterBox` | 平均，缩小最高质量 |

`I420Scale` 注释（67–74）：None=最近邻；Bilinear=插值；**Box=平均，缩小更好**。dest 宽高是函数参数，与过滤枚举并列。`I420ToARGB`（`convert_argb.h` 102）只有源/目标平面和 **同一** width/height——转色不缩放。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| convert 与 scale 分头文件 | reuse-pattern | Yohu：VP Convert ≠ HLSL Scale |
| Box = 缩小质量档 | reuse-pattern | 面积核是 GPU 上的 box |
| Bilinear 注释写明缩小更差 | reuse-pattern | 不要用 VP 双线性一次压到 dest |
| 链 libyuv 做呈现 | anti-pattern | CPU；ADR-v6-024 要 GPU 硬解 |

## 架构设计经验

1. **同尺寸转色是一种函数，改尺寸是另一种。** 合成 `ToARGBAndScale` 会把两层焊死。
2. **缩小质量档叫 box，不叫 mip。**
3. **点采是枚举值，不是缺省「硬件」。**

## 与当前工作

- 能直接用：API 边界；box 当缩小。
- 必须改写：实现是 VP + HLSL，不是 C 平面。
- 不要用：WebRTC 的整包缩放进壳。

## 阅读范围

`include/libyuv.h`；`include/libyuv/scale.h` 23–80；`include/libyuv/convert.h` 开篇；`include/libyuv/convert_argb.h` `I420ToARGB`。未读 SIMD 行实现。
