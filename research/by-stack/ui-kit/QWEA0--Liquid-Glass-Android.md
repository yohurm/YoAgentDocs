---
id: research.QWEA0-Liquid-Glass-Android
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Kotlin, AGSL, C++]
  frameworks: [android-view]
also_relevant: []
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: QWEA0/Liquid-Glass-Android
  url: https://github.com/QWEA0/Liquid-Glass-Android
  head: 82a9640
  cloned_to: "%TEMP%/YoAgentResearch/QWEA0--Liquid-Glass-Android"
studied_at: 2026-09-04
related:
  - research.apple-liquid-glass
  - research.android-liquid-glass-data-path
  - research.Abdullajon1881-LiquidGlass
  - research.Kyant0-AndroidLiquidGlass
---

# QWEA0/Liquid-Glass-Android

## 入选理由

YoUI 是 Android **View** 体系。该库把 iOS 26 Regular/Clear、SDF 透镜、液态 merge、Scroll Edge、无障碍降级接到 `FrameLayout`，API 33+ 单 pass AGSL，与 `youi_kit` 的 `BackdropGlass` 同栈。不选 Compose 专用库。

## 项目是什么

独立复现「液态玻璃观感」的 View 组件，**不是** Apple SDK。作者写明与官方实现无关。`LiquidGlassView` 把前景孩子放进玻璃板；背景由 `BackdropCapture` 从指定 source 录制。

## 架构

```text
LiquidGlassView (FrameLayout)
  -> BackdropCapture     跳过自身/祖先路径，避免 RenderNode 重入与环
  -> RenderNode 模糊
  -> GlassLensRenderer   单 AGSL：SDF / smin / 斜面 / 折射 / 色散 / 高光 / 内阴影 / tint
  -> GlassMaterial       REGULAR | CLEAR
  -> LightSourceController 重力 → lightDir（可关）
  -> BackdropLuminanceMeter 自适应 tint + 通知前景 light/dark
  -> ScrollEdgeBlurView  与玻璃分离的渐进模糊带
  -> GlassAccessibility  高对比 / 去动画 / 省电
```

`GlassLensRenderer` 注释中的光学顺序：圆角 SDF（主形状可逐角半径）→ 可选第二形状 `sminPoly` → 斜面厚度 → 数值梯度法线 → 沿法线折射 → 分通道色散 → `dot(N,L)` 高光 → 内阴影 → Regular 自适应染色或 Clear 的 `dimAmount`。

`BackdropCapture` 的关键约束与 Yo 的 `BackdropHost` 相同：不能 `source.draw` 走到正在 recording 的祖先，也不能让玻璃进被采样的 display list。跨层祖先时改走公开 `View.draw` + `setTransitionVisibility(INVISIBLE)` 跳过玻璃分支。

`GlassMaterial`：

| | Regular | Clear |
|--|---------|-------|
| blurScale | 1.0 | 0.35 |
| dimAmount | 0 | 0.16 |
| adaptiveTint | true | false |

`ScrollEdgeBlurView`：API 31+ 弱模糊铺满渐变带 + 强模糊贴边，`DST_IN` 渐变遮罩；更低 API 用 scrim。这是独立 View，不是透镜 shader 的一个 uniform。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Regular/Clear 参数分叉 | adapt | 变体编译进 recipe，不要运行时 if 调半径 |
| 捕获排除自身 / 禁环 | reuse-pattern | 已有 `OverlayExclusion` + `BackdropHost`；对照补跨层 `View.draw` 路径 |
| SDF + smin 双形状 | adapt | 容器 merge 的最小实现（最多两块）。多块见 Abdullajon |
| Scroll edge 独立节点 | reuse-pattern | 不要把渐变模糊写进透镜 pass |
| 无障碍 → 不透明/去弹性 | adapt | 接到已有 `MotionPolicy`，不要另开 FORCE_FULL 开关给业务 |
| 重力传感器默认开 | anti-pattern | 官方高光是「部分场景」；Yo L6 已是触点光。默认跟 recipe，不跟 TYPE_GRAVITY |
| RGB 色散默认开 | anti-pattern | 官方讲 lensing，不规定光谱边纹；HIGH 可选、默认 0 |
| 预置 TabBar/Dialog/FAB 控件树 | anti-pattern | 宿主只走 Yo 组件；禁止再做一套 Material 玻璃控件 |
| 包名 `com.example.liquidglass` | lesson-only | 示范仓，不依赖 |

## 架构设计经验

- **捕获/模糊/透镜分节点；光学公式可对照。** QWEA0 用 `createChainEffect(lens, blur)`；Yo 因 Motorola 必须 blurNode → bodyNode → lensNode 分挂。不要把链搬过来，也不要再叠位移贴图。
- **变体是模型，不是布尔 `moreTransparent`。** Clear 关掉 adaptive 并打开 dimming，三件事一起编译。
- **merge 是 SDF 并集，不是两块独立采样再 blend。** 两块玻璃必须共享同一张 backdrop。

## 与当前工作

能直接对照：`GlassLensRenderer` 的 SDF / 覆盖率 AA / 斜面带宽 与 `LiquidGlassOptics` + `GlassLensProgram`。必须改写：色散、重力、预置控件。明确不要：把 `LiquidGlassView` 当依赖；不要 `setSecondaryShape` 暴露给应用。

## 数据链路（拉采样）

热路径是 **每块玻璃自己 pull**：`onDraw` → `GlassLensRenderer.draw` 建外扩 RenderNode → `BackdropCapture` 沿祖先路径 `View.draw` + `TRANSITION_VISIBILITY` 跳玻璃分支 → `createChainEffect(lens, blur)`。N 板 = N 次整树。`isCapturingBackdrop` 只对 CPU 软件 `parent.draw` 有效。逐跳对照见 [android--liquid-glass-data-path.md](./android--liquid-glass-data-path.md)。

## 阅读范围

`README.md`；`LiquidGlassView.drawGlassEffect` / `tryDrawLensGlass`；`GlassLensRenderer.kt` 全文（AGSL + `buildEffect`）；`BackdropCapture.kt` 全文；`HardwareBackdropBlur.kt`；`EnhancedBlurEffect.captureBackdrop`；`GlassMaterial.kt`；`GlassAccessibility.kt`；`ScrollEdgeBlurView.kt`。未跑 demo APK。未读 C++ NEON 模糊核细节。
