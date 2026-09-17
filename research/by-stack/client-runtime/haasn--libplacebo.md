---
id: research.haasn-libplacebo
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, GLSL]
  frameworks: [Vulkan, D3D11, OpenGL]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: haasn/libplacebo
  url: https://github.com/haasn/libplacebo
  head: 3330a51
  cloned_to: "%TEMP%/YoAgentResearch/haasn--libplacebo"
studied_at: 2026-09-15
related:
  - research.synthesis.client-runtime
  - research.mpv-player-mpv
  - research.desktop-mirror-downsample
---

# haasn/libplacebo

## 入选理由

桌面视频呈现里把 **几何 / 色彩 / 核 / 合成** 拆得最清楚的开源库。mpv `gpu-next`、VLC Vulkan 路径、FFmpeg 的 Vulkan 滤镜都吃它。Yohu 嵌入槽真机约 0.37× 缩小，需要的是「缩小核是独立参数」，不是整库。LGPL-2.1，体量大，**禁止 vendoring**。

## 项目是什么

分层 GPU 图像处理库。公开 API 按 **tier** 叠：数学核（`filters.h`）→ GPU 抽象 → shader 生成 → `pl_render_image`。调用方只填 `pl_render_params` 和源/目标 `pl_frame`。

## 架构

```text
Tier 0  filters / colorspace / dither     核与色，不碰 GPU
Tier 1  gpu / swapchain / d3d11           设备与呈现
Tier 2  shaders + custom hooks            生成 GLSL
Tier 3  renderer.pl_render_image          按钩子阶段走完整通路
```

`pl_render_params` 把放大核、缩小核、平面核、色彩、抖动拆开（`renderer.h` 130–152）。`downscaler == NULL` 同时意味着 `skip_anti_aliasing`：GPU 自带双线性/最近邻不能抗混叠（136–137）。默认推荐档 `pl_render_default_params` 的缩小核是 **hermite**（`renderer.c` 207–210）；高质量档仍用 hermite 缩小，只把放大换成 EWA Lanczos（217–220）。

钩子阶段把「转 RGB」和「主核」切开（`shaders/custom.h` 120–126）：

```text
INPUT → CHROMA_SCALED → NATIVE → RGB → LINEAR
  → PRE_KERNEL → POST_KERNEL → SCALED → OUTPUT
```

`PL_HOOK_RGB` 可改尺寸；主核前后是独立插入点。目标矩形是 `pl_render_image` 的 `target`，不是核配置的一部分。

`pl_render_image` 热路径按函数切开（`renderer.c` 3643–3650）：

```text
pass_init / fix_refs_and_rects     Fit：crop → dst_rect
pass_read_image                    Convert：平面对齐 + YUV→RGB
pass_scale_main                    Scale：PRE_KERNEL → 核 → POST_KERNEL
pass_convert_colors                色管 / 色调（Yohu 不搬）
pass_output_target                 Compose + Present
```

`downscaler == NULL` 时 `SAMPLER_DIRECT`，主核可整段跳过（`renderer.c` 2164–2169 *Skipping main scaler (free sampling)*）。大比例缩小若走 fast hermite，采样器自己打日志：*will most likely result in nasty aliasing*（`sampling.c` 373–375）。Polar EWA 半径过大直接 `SH_FAIL`（`sampling.c` 802–805）。`pl_render_image` 默认按 crop 拉伸；要保比例须另调 `pl_rect2df_aspect_copy`（`renderer.h` 601–603）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 放大核 / 缩小核 / 色 / 抖动分字段 | reuse-pattern | Yohu：`present_dest` 管 dest，核只吃 `(src, dest)` |
| `downscaler == NULL` ⇒ 无抗混叠 | reuse-pattern | 嵌入槽 2.73× 缩小禁止把核留空交给 VP/采样器 |
| RGB 之后才 PRE_KERNEL | reuse-pattern | Convert 1:1 完成后再 Scale |
| 整库 / EWA / 3DLUT / 色管 | anti-pattern | LGPL + 体积；嵌入投屏不需要播放器级色管 |
| 高质量档的 EWA 放大 | anti-pattern | Yohu 主路径是缩小；EWA 大比例还会顶半径容量 |

## 架构设计经验

1. **dest 像素数是 renderer 目标，核是 params。** 改清晰度只换 `downscaler`，不改 contain。
2. **NULL 核不是「用硬件随便缩」。** 作者写明那就是跳过抗混叠。
3. **平面核（chroma）与图像核分开。** Yohu 的 NV12→RGB 已由 Video Processor 做完，Scale 只看 RGB crop。
4. **钩子顺序是契约。** 锐化若以后要做，只能 POST_KERNEL，不能写进 Fit。

## 与当前工作

- 能直接用：阶段名 Fit（target）/ Convert（到 RGB）/ Scale（kernel）/ Compose（OUTPUT 前混合）/ Present（swapchain）。
- 必须改写：Yohu 不链 libplacebo；Windows 核是自己的 HLSL，macOS 另写。
- 不要用：把 `gpu.rs` 当 `pl_render_image` 神文件；为「清晰」引入 EWA 或 mip。

## 阅读范围

`README.md` API tiers；`renderer.h` params / `skip_anti_aliasing` / crop 语义；`shaders/custom.h` 钩子；`renderer.c` 默认档、`pl_render_image` 3643–3650、`pass_scale_main` 跳过核；`shaders/sampling.c` 半径加宽与 Polar 容量。未读完每一支色管。
