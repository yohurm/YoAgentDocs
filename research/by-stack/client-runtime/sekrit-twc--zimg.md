---
id: research.sekrit-twc-zimg
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: []
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: sekrit-twc/zimg
  url: https://github.com/sekrit-twc/zimg
  head: 67e0603
  cloned_to: "%TEMP%/YoAgentResearch/sekrit-twc--zimg"
studied_at: 2026-09-15
related:
  - research.haasn-libplacebo
  - research.FFmpeg-FFmpeg
---

# sekrit-twc/zimg

## 入选理由

专业图像图：调用方只给 **源格式、目标格式、图参数**，库自己插入 colorspace / resize / dither 节点。VapourSynth、若干播放器当高质量 CPU 缩放。WTFPL。Yohu 不链 zimg（CPU SIMD、不是 D3D），要的是「色与尺寸是图上的不同节点」。

## 项目是什么

`zimg_filter_graph_build(src_format, dst_format, params)`（`src/zimg/api/zimg.h` 679）。`zimg_image_format` 带宽高、像素类型、色彩族、矩阵/传递/原色。`zimg_graph_builder_params` 另带 resample 与 dither。测试 `graphbuilder_test.cpp` 把期望节点列成 `"colorspace"`、`"resize"` 字符串——**只改色彩只出 colorspace 节点；只改尺寸才出 resize**。

## 架构

```text
zimg_image_format src  ──宽高 + 色
zimg_image_format dst  ──宽高 + 色
zimg_graph_builder_params
    resample_filter      默认 BICUBIC
    resample_filter_uv   默认 BILINEAR
    dither_type          默认 NONE
        ↓
zimg_filter_graph   运行时按需插入节点
```

核枚举与色枚举分开（`zimg.h` 375–381、365–368）：`ZIMG_RESIZE_POINT` 注释写 **never anti-aliased**。POINT 是显式核，不是「没选核」。`ZIMG_ERROR_NO_COLORSPACE_CONVERSION` 与 `ZIMG_ERROR_RESAMPLING_NOT_AVAILABLE` 是两类错误（118–120）。

`GraphBuilder` 按差分插节点，顺序跟放大/缩小走：`test_upscale_colorspace` 先把色度抬到 4:4:4，再 `colorspace`，再拉 luma（`graphbuilder_test.cpp` 407–423）。色转在 float 444 工作空间（`connect_color_channels`）。`FilterObserver` 把 colorspace / resize / depth 收成三个虚函数（`graphbuilder.h` 45–63）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 格式差 → 自动插节点 | reuse-pattern | Yohu：内容尺寸变了只动 Fit；YUV 变了只动 Convert |
| resize 滤镜独立于 matrix | reuse-pattern | VP 色空间 ≠ 面积核 |
| POINT = 永不抗混叠 | reuse-pattern | 只给 1:1 / 整数放大 |
| 链 zimg 进壳 | anti-pattern | 热路径是 D3D；图引擎是 CPU |
| 默认 bicubic 当缩小 | anti-pattern | 嵌入槽 >2:1 应用面积/盒，不是默认 bicubic |

## 架构设计经验

1. **源/目标是完整图像描述，参数是算法。** dest 宽高在 format 上，核在 params 上。
2. **图构建器按差分插节点。** 同尺寸转色不应跑 resize。
3. **错误按层分类。** 色转失败不是核失败。

## 与当前工作

- 能直接用：Convert 与 Scale 分错误、分模块；POINT 永不抗混叠。
- 必须改写：GPU 两 pass（VP + HLSL），不是 CPU 图。
- 不要用：VapourSynth 式用户可选 7 种 resize。

## 阅读范围

`src/zimg/api/zimg.h` 79–120、365–381、590–679；`test/graph/graphbuilder_test.cpp` `test_colorspace_only` / `test_downscale_colorspace`。未读完各 SIMD resize 实现。
