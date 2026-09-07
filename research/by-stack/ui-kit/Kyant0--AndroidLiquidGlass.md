---
id: research.Kyant0-AndroidLiquidGlass
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Kotlin, AGSL]
  frameworks: [compose]
also_relevant: []
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: Kyant0/AndroidLiquidGlass
  url: https://github.com/Kyant0/AndroidLiquidGlass
  head: 65ab177
  cloned_to: "%TEMP%/YoAgentResearch/Kyant0--AndroidLiquidGlass"
studied_at: 2026-09-04
related:
  - research.android-liquid-glass-data-path
  - research.apple-liquid-glass
---

# Kyant0/AndroidLiquidGlass

## 入选理由

Compose 侧把「内容层 GraphicsLayer」和「玻璃层 RenderEffect」拆开最清楚。Yo 是 View，不依赖它，但 H1 共享 / H5 铬分修饰符 / lens 只支持圆角 这三项要写进数据链路。QWEA0 README 也指定 Compose 绿场用本仓。

## 项目是什么

KMP `backdrop` 库：`Modifier.drawBackdrop(backdrop, shape) { blur(); lens(...) }`。Android 用 AGSL RuntimeShader；Skiko 另有实现。Catalog 里有按钮、底栏、对话框、自适应亮度等 demo，那是应用层，不是材料内核。

## 架构

```text
rememberLayerBackdrop()          一份 GraphicsLayer = H1
内容节点 record → 该 layer

Modifier.drawBackdrop
  Shadow / InnerShadow / Highlight   独立 element，先于玻璃
  DrawBackdropNode
    record 外扩 layer：translate + drawLayer(H1)     H2
    graphicsLayer.renderEffect = blur then lens      H3+H4
    draw: behind → plate → surface → content → front H5
```

`LayerBackdrop.drawBackdrop` 用 `localPositionOf` 对齐玻璃与内容层；失败回退 window 坐标差。玻璃自己的 `graphicsLayer` 变换用 `inverseTransform` 抵消，避免按压缩放把采样一起扭。

`lens()`：仅 `RoundedRectangularShape` / `CornerBasedShape`；`refractionAmount` 写入 shader 为负数。其它 Shape `throw`.

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 一份内容 layer，N 块玻璃切片 | reuse-pattern | 同 Abdullajon Provider；对应 BackdropHost |
| 阴影/高光不进透镜 pass | reuse-pattern | 对齐 Yo BackdropChrome |
| 按压逆变换采样 | lesson-only | View 没有 Compose layerBlock；Yo 透镜不读指针 |
| 非圆角 Shape 直接 throw | adapt | 玻璃锁 CIRCULAR，不要静默退化 |
| Compose Modifier 当 youi_kit 实现 | anti-pattern | 栈不对 |

## 与当前工作

能对照坐标切片与 effect 累加顺序。不要把 catalog 控件抄进 Yo。完整跳表见 [android--liquid-glass-data-path.md](./android--liquid-glass-data-path.md)。

## 阅读范围

`DrawBackdropModifier.kt`（draw/record/effectScope）；`LayerBackdrop.kt`；`effects/Lens.kt`；`internal/LayerRecorder.kt`。未读全部 Skiko expect；未跑 catalog。
