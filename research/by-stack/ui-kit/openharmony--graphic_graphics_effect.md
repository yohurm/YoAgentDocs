---
id: research.openharmony-graphic_graphics_effect
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [cpp, sksl]
  frameworks: [openharmony, rosen, runtime-shader]
also_relevant: [client-runtime]
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: openharmony/graphic_graphics_effect
  url: https://github.com/openharmony/graphic_graphics_effect
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--graphic_graphics_effect"
  head: 18d3ed1
studied_at: 2026-08-20
related:
  - research.harmony-sdf-edge-light
  - playbook.online-deep-research
---

# openharmony/graphic_graphics_effect

## 入选理由

这是 HarmonyOS 沉浸光感边缘高光的**算法仓**。ArkUI 只接线参数，Rosen 只做 NG Effect 容器；真正的 SDF 距离衰减、bloom 内外宽度、`lightMask` 调制都在本仓 RuntimeShader 里。要回答「环境光沿边缘柔和流转」是不是 canvas stroke，必须读这里，不能停在官方设计文案。

同主题对照仓（本篇不复制全文）：

- `openharmony/graphic_graphic_2d`：`RSNGSDFEdgeLightEffect` / `RSNGFrameGradientMask` 属性袋，浅克隆 `%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d`（HEAD `38f018d`）。
- `openharmony/arkui_ace_engine`：ArkUI 默认数值与「HIGH 清边框」，浅克隆 `%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine`（HEAD `8626690d`，origin 为 gitcode）。

## 项目是什么

OpenHarmony 图形子系统的视效算法库：模糊、扭曲、光照、SDF 形状、遮罩。对外经 ArkUI / UIEffect / EffectKit。效果分成四类：`GEShaderFilter`（图像滤镜）、`GEShader`（直接画着色器）、`GEShaderMask`、`GEShaderShape`。

边缘光有两条实现，**都不是 `Canvas.drawPath` stroke**：

| 类 | 类型 | 用途 |
|----|------|------|
| `GESDFEdgeLight` | Filter | 对已有图像做边缘光，再 **RGB 相加** 合成 |
| `GESDFEdgeLightShader` | Shader | overlay 直接画；只填轮廓环带（内接矩形挖空） |

当前树里路径已从早期 `include/sdf/` 迁到 `include/effect/filter/` 与 `include/effect/shader/`。SkSL 字符串在 `.cpp` 内，不在独立 `.sksl` 文件。

## 架构

```text
ArkUI (默认参数 + FrameGradientMask)
    → RSNGSDFEdgeLightEffect / RSNGSDFEdgeLightFilter
        → GESDFEdgeLightFilterParams / GESDFEdgeLightEffectParams
            → RuntimeShader
                inputs: sdf (image 或 shape), lightMask, uniforms
                out: lightColor * intensity(mask) * (thinBorder(d) + bloom(d))
```

静止态「沿边流转」其实有两层，本仓都提供算法：

1. **材质滤镜内的对角高光**（`GEFrostedGlassEffect`）：SDF 带宽 × 扇形 mask。这是均衡档文案「环境光沿边缘柔和流转」的落点。
2. **SDF 边缘光 overlay**（本篇重点）：SDF 场光照 × `lightMask`。给 `edgeLight` API、Sheet 出现光、交互 `lightEffect` 用。

二者共同点：能量沿 **SDF 法线（到轮廓的有符号距离）** 衰减，再被切向/轴向 mask 压成「顶部亮、底沿暗」，不是 360° 发丝线圈。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| SDF 法线衰减公式 | adapt | `thinBorder` + `bloomMultiplierFromDist`；不要抄 SkSL 原文进业务仓 |
| `lightMask` 乘法 | adapt | mask.r=0 则整像素熄灭；流转来自 mask，不是加粗 stroke |
| Inner 0.8 / Outer 0.2 | adapt | bloom 宽是 thickness 的比例，不是 GE 默认 30/30 |
| Additive 合成 | lesson-only | `dst.rgb += light.rgb`；白底上加白几乎看不见 |
| canvas 闭合描边 | anti-pattern | HIGH/MID 官方清掉 `borderWidth`，高光必须来自滤镜 |
| 把 GE 默认 64/30/30 当 ArkUI 默认 | anti-pattern | ArkUI overlay 另有 47.8 / 8.7 / 10.3 套 |

## 架构设计经验

- **距离场是几何，mask 是光照。** 把「边在哪」和「哪段被照亮」拆开，才能做顶部锥而不画闭合线圈。
- **Bloom 内外不对称。** 内宽远大于外宽：光坐在玻璃里，不是元件外一圈雾。
- **高 `bloomFalloffPow`（ArkUI 8.7）** 让晕很贴边；GE 测试默认 2.0 会糊成光雾，不要当产品默认。
- **Filter 路径会把 SDF 图再模糊一遍（半径 3）** 专供 bloom 的 `smoothD`；核心细边用未模糊 SDF。软晕和锐边不是同一张距离图。
- **白底可见性来自接触变暗 / Fresnel，不是更白的描边。** 合成是加色；白加白对比度为 0。

## 与当前工作

能直接用的：参数语义、衰减公式、mask × SDF 的拆分、HIGH 无边框。

必须改写：Android 没有 Rosen RuntimeShader 容器。YoUI 应对齐公式与能量分配，用 AGSL / 现有 `GlassLensProgram` 重写，不要移植 C++ 类。

明确不要用：把 `BackdropChrome.drawRim` 的 SweepGradient stroke 加粗当「更像官方」；不要把第三方 SkSL 拷进 YoAgentDocs 或 Android-YoUI。

## 阅读范围

实际读过：

- `include/effect/filter/ge_sdf_edge_light.h`
- `src/effect/filter/ge_sdf_edge_light.cpp`（含 Filter SkSL、SDF decode、additive merge）
- `include/effect/shader/ge_sdf_edge_light_shader.h`
- `src/effect/shader/ge_sdf_edge_light_shader.cpp`（含 Shader SkSL、环带裁剪）
- `include/effect/filter/ge_sdf_edge_light_filter.params.in`
- `include/effect/shader/ge_sdf_edge_light_effect.params.in`
- `include/effect/mask/ge_frame_gradient_shader_mask.h`
- `src/effect/mask/ge_frame_gradient_shader_mask.cpp`（轴向 envelope）
- `src/effect/shader/ge_frosted_glass_effect.cpp`（对角扇形高光，静止材质滤镜）
- `src/core/ge_visual_effect_impl.cpp` 中 `SetSDFEdgeLightParams`
- `README.md`

未读：HPS 加速路径、全部模糊/扭曲滤镜、fuzz 测试。算法要点见主题笔记 [harmony--sdf-edge-light.md](harmony--sdf-edge-light.md)。
