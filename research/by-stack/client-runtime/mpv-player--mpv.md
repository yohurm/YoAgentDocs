---
id: research.mpv-player-mpv
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C]
  frameworks: [libplacebo, OpenGL, D3D11]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: mpv-player/mpv
  url: https://github.com/mpv-player/mpv
  head: 0b7ed67
  cloned_to: "%TEMP%/YoAgentResearch/mpv-player--mpv"
studied_at: 2026-09-15
related:
  - research.haasn-libplacebo
  - research.mpv-player-mpv-examples
  - research.desktop-mirror-downsample
---

# mpv-player/mpv

## 入选理由

把 **窗口几何** 和 **四套核旋钮** 拆开的播放器。嵌入槽研究已在 [mpv-examples](mpv-player--mpv-examples.md)；本篇只读 scaler。Yohu 禁止搬 libplacebo，要的是选项分层。

## 项目是什么

媒体播放器。视频输出（VO）算 dest 矩形；`--scale` / `--dscale` / `--cscale` / `--tscale` 选核。`gpu-next` 把选项映射到 `pl_render_params`。

## 架构

```text
VO dest（窗口 / wid 槽）     ← 几何，与核无关
  scale   放大
  dscale  缩小
  cscale  色度平面
  tscale  时间轴
  correct-downscaling  缩小时拉长卷积半径
  linear-downscaling   线性光里缩（要 ≥16bit FBO）
```

手册：`--dscale`「和 `--scale` 一样，但只用于缩小」（`DOCS/man/options.rst` 6012–6013）。默认缩小核 **hermite**（5980–5984）。`--correct-downscaling` 默认开：卷积核在缩小时加大支撑，质量换性能（6132–6135）。`gpu-next` 一行对齐 libplacebo：`skip_anti_aliasing = !correct_downscaling`（`vo_gpu_next.c` 2843）。旧 `vo=gpu` 在 `video.c` 2692–2693：缩小比 `f<1` 时用 `1/f` 当 scale_factor 拉半径。真机槽 446×991 ← 1220×2712 约 **0.365× → inv_scale ≈ 2.74**；hermite 半径 1 会被拉到约 2.74（`filter_kernels.c` `src_radius = radius * filter_scale`）。几何在 `aspect.c` `mp_get_src_dst_rects`，与核文件无关。

`--scaler-resizes-only` 默认开：1:1 时不用花核，双线性即可完美复原（6126–6130）。这是「核跟着是否在缩放走」，不是「核改 dest」。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 放大 / 缩小两套核 | reuse-pattern | Yohu：1:1 与整数放大 nearest；缩小面积核。Fit 不选核 |
| `correct-downscaling` = 半径跟缩小比 | adapt | 面积核的 3×3 是「盖住 dest 像素的源足迹」，同一意图 |
| `skip_anti_aliasing = !correct_downscaling` | reuse-pattern | 关掉正确缩小 = 允许锯齿，不是「更清晰」 |
| 整份 libplacebo / EWA / 线性光 16bit FBO | anti-pattern | ADR-v6-024 禁 FFmpeg；也不引 libplacebo。8bit RGBA 上做 linear-downscale 不值 |
| `--dscale-antiring` 在正交缩小时会加锯齿 | anti-pattern | 手册 6101–6104；Yohu 不要叠锐化当缩小器 |

## 架构设计经验

1. **VO 算盒子，scaler 算取样。** 用户改窗口大小只改 dest；改 `--dscale` 只改核。
2. **缩小默认 hermite，不是 nearest，也不是 mip。**
3. **1:1 走便宜路径。** 与 Yohu `nearest` 旗标同构。
4. **嵌入槽研究另文。** wid / CSS 管不着表面，见 mpv-examples，不要和核混写。

## 与当前工作

- 能直接用：dest 与核分文件；缩小必须有抗混叠核。
- 必须改写：半径用源矩形面积平均，不搬卷积 LUT。
- 不要用：把 mpv 当嵌入播放器；为「更清晰」关 `correct-downscaling`。

## 阅读范围

`DOCS/man/options.rst` 5974–6153；`video/out/aspect.c` 128–221；`vo_gpu_next.c` 1643–1744、2834–2870；`video/out/gpu/video.c` 371–391、2626–2748；`filter_kernels.c` 55–94。未读完 vo_gpu 全部 pass。
