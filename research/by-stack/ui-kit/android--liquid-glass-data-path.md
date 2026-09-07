---
id: research.android-liquid-glass-data-path
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Java, Kotlin, AGSL]
  frameworks: [android-view, compose]
also_relevant: []
utilization: [adapt, anti-pattern, reuse-pattern]
source:
  platform: github
  url: https://developer.android.com/develop/ui/views/graphics/render-effect
  repos:
    - QWEA0/Liquid-Glass-Android
    - Abdullajon1881/LiquidGlass
    - Kyant0/AndroidLiquidGlass
  cloned_to:
    - "%TEMP%/YoAgentResearch/QWEA0--Liquid-Glass-Android"
    - "%TEMP%/YoAgentResearch/Abdullajon1881--LiquidGlass"
    - "%TEMP%/YoAgentResearch/Kyant0--AndroidLiquidGlass"
studied_at: 2026-09-04
related:
  - research.apple-liquid-glass
  - research.QWEA0-Liquid-Glass-Android
  - research.Abdullajon1881-LiquidGlass
  - research.BarredEwe-LiquidGlass
---

# Android 液态玻璃：完整数据链路（四套实现对照）

主题笔记。对象是 **像素从场景走到玻璃板** 的每一跳，不是官网摘要。四套都读过源码：QWEA0 View 拉采样、Abdullajon Provider 推录制、Kyant Compose GraphicsLayer、YoUI `BackdropHost` 混合。Apple 的 `UIVisualEffectView` 片元仍闭源；Android 能复述的只有这条链路。

## 1. 共同拓扑（六跳）

所有能跑起来的实现都拆成同一条链。差在每一跳的所有者。

```text
H0  场景树（内容，不得含正在采样自己的玻璃）
H1  捕获：把 H0 变成可复用的 display list / bitmap
H2  切片：按玻璃在场景上的屏幕矩形裁一块（含模糊/折射外扩）
H3  霜：高斯 / RenderEffect.blur（可降采样）
H4  透镜：AGSL 在霜面上沿 SDF 法线位移采样
H5  铬：阴影 / tint / rim / overlay / 前景孩子
```

H1 与 H2 不能由同一 `RenderNode.beginRecording` 再 `draw` 到正在录制的祖先，否则 `IllegalStateException` 或 display list 成环。四套都用「玻璃不在被录的 list 里」解决，手段不同。

## 2. QWEA0：每块玻璃自己拉（pull）

入口：`LiquidGlassView.onDraw` → `drawGlassEffect`。

档位（`tryDrawLensGlass` / `tryDrawHardwareBlur` / `renderGlassEffectSync`）：

| 条件 | 路径 |
|------|------|
| REDUCE 透明 / FORCE_OPAQUE | H5 实色圆角，无 H1–H4 |
| API 33+ 硬件画布、`useShaderPipeline`、无自定义 Bitmap 捕获 | **透镜 2.0** |
| 否则 API 31+ 硬件、无色散 | **旧 GPU 模糊±色差** |
| 其余 | **CPU Bitmap 三级缓存** |

### 2.1 透镜 2.0 逐跳

`GlassLensRenderer.draw`：

1. **H1+H2 合一。** 新建/复用 `RenderNode("LiquidGlassLens")`，`setPosition(-margin,-margin,w+margin,h+margin)`。`margin = max(3σ_blur, 32)` 对齐 16px。注释写明：折射必须向内，`RuntimeShader` 子输入越界是透明黑；margin **只服务模糊**，不给外折射。
2. **坐标。** `getLocationOnScreen(glass) - getLocationOnScreen(backdropView)`。`backdropView` 默认 `parent`，可设成任意祖先/兄弟（RecyclerView）。非法：source 是玻璃自己或后代。
3. **排除自身。** `isCapturingBackdrop=true` **拦不住硬件 `dispatchDraw`**（走 `updateDisplayListIfDirty`，不进 `View.draw`）。真正排除在 `BackdropCapture`：沿 source→glass 的分支 `setTransitionVisibility(INVISIBLE)`，对该层调用公开 `View.draw`，再单独补画分支。跨层祖先时：中间容器的缩放/旋转丢失；玻璃之后的同层兄弟会被盖住。
4. **H3→H4 链。** `createChainEffect(lensRuntimeShader, blurEffect)`。内层先模糊。AGSL `content.eval` 读的是已模糊纹理。
5. **H4 片元。** `sdRoundedBox4`（逐角）∪ 可选 `sminPoly` 第二形状 → 外法线中心差分 → `t = clamp(-d/bevel)`、`slope = (1-t)²` → `offset = -n * slope * refractPx`（按压加大）→ RGB 分通道 → 饱和 → Regular 逐像素 adaptive / Clear `* (1-dim)` → 本体 `glassTint` 吸收+散射 → `dot(N,-L)` 高光 + 背光内阴影 → `alpha = cov`。
6. **H5。** 着色器输出覆盖率，**不再 clipPath**。边缘高光在 shader 里，Kotlin 不再描边。

同帧 **N 块玻璃 = N 次** `BackdropCapture.draw` 整棵 source。没有共享 host。`setSecondaryShape` 只是同一 shader 里第二枚 RRect，不是第二块 View。

### 2.2 旧 GPU 模糊

`HardwareBackdropBlur.draw`：同一套 Capture + 屏幕坐标差，但 **无 margin**（`setPosition(0,0,w,h)`）。链：blur →（可选）位移贴图色差 RuntimeShader → 饱和 ColorFilter。绘制时 `clipPath`。色散或自定义 Bitmap 捕获会打回 CPU。

### 2.3 CPU 回退

`EnhancedBlurEffect.captureBackdrop`：软件 `Canvas(Bitmap)`，`parent.draw`；此时 `isCapturingBackdrop` **有效**（软件走 `View.draw`）。三级 Bitmap：L1 原图 / L2 模糊（C++ IIR NEON 或 Kotlin）/ L3 色差或色散。8×8 像素签名决定是否脏；`enableDynamicBackground` 跳过签名每帧当脏。模糊半径帽走 native，不是 `RenderEffect`。

**对 Yo：** pull 在 Dialog 未入树、跨 window 时有用。默认热路径不应每块玻璃重画整页。Capture 的 `View.draw` + `TRANSITION_VISIBILITY` 与 Yo `OverlayExclusion.hideInside` 同构。

## 3. Abdullajon：Provider 推一次（push）

树约定（`LiquidGlassProviderLayout` 注释）：

```text
Parent
  LiquidGlassProviderLayout    ← 内容，H1 在这里
  LiquidGlassView (sibling)    ← 玻璃，禁止放进 Provider
```

### 3.1 H1

`dispatchDraw` → `BackdropRecorder.drawRecorded`：

- API 29+ 硬件：`contentNode.beginRecording` → `super.dispatchDraw(recording)` → `canvas.drawRenderNode(contentNode)`。
- 然后对每个 registered consumer `postInvalidateOnAnimation()`。

玻璃 **不在** Provider 子树，所以录制天然不含玻璃。这是 iOS「玻璃不能采样玻璃」的最干净工程解。

### 3.2 H2–H5（View，单板）

`GlassViewController.draw`：

1. `getLocationInWindow(host) - getLocationInWindow(provider)` → `relativeX/Y`。
2. 档位 `GlassRenderTier.select(api, override)`：SHADER 33 / BLUR 31 / 否则有 recording 则 API29 裁切 backdrop+scrim / 再否则纯 scrim。
3. `GlassViewRenderer.drawEffectGlass`：本地 `RenderNode` 尺寸 = 板 + `inflate`（`2*blur + |refract| + 4`）。recording：`translate(inflate - relative)` + `drawRenderNode(providerNode)`。即 **H2 是对共享 H1 的平移切片**，不再 `View.draw`。
4. H3+H4：`createChainEffect(runtimeShader, blur+saturation)`。View 路径 `MERGE_SMOOTHING = 0`，只 pack **一块** `PackedGlassShape`。多板融化 **不在 View 引擎**，在 Compose `LiquidGlassContainerState`（最多 8 形状、progress 做 morph）。
5. SHADER 档 H5 在 shader（mask/tint/rim/grain/press bulge）。BLUR 档 clipPath + 画 tint + LinearGradient rim。

### 3.3 关键副作用

每帧 Provider 录完会 `postInvalidateOnAnimation` **所有** consumer。N 块玻璃 → N 次额外 invalidate。Yo 的 `OverlayExclusion.invalidateSamplers` 只 invalidate overlay **根**，并在 capturing 栈非空时直接 return，避免风暴。

**对 Yo：** 默认 Dialog/Bar 应走 push host（已有 `BackdropHost.dispatchDraw`）。Abdullajon 的 inflate 切片公式可对照 `BackdropSampleSpace.pad`。View 单板 merge=0 证明：**容器必须在 L1 把多 plate 打进同一 shader**，不能靠多个 `LiquidGlassView` 并排。

## 4. Kyant：Compose GraphicsLayer 导出

`Modifier.drawBackdrop(backdrop, shape, effects)`：

1. **H1。** `rememberLayerBackdrop()` 持有一个 `GraphicsLayer`。内容侧 `layerBackdrop` modifier 把内容 `record` 进该 layer（`exportedBackdrop` 路径还会把玻璃自己的 behind+backdrop+surface 再录一份给下游）。
2. **H2。** 玻璃节点 `recordLayer`：按 `padding` 外扩尺寸，`onDrawBackdrop` 里 `LayerBackdrop.drawBackdrop`：`localPositionOf(glass, layer)`（失败则 window 差）`translate(-offset)` + `drawLayer(contentLayer)`。可选 `inverseTransform(layerBlock)` 抵消玻璃自己的 graphicsLayer（按压缩放时采样仍对世界）。
3. **H3–H4。** `BackdropEffectScope` 累加 `RenderEffect`：`blur()`、`lens(refractionHeight, refractionAmount, chromaticAberration)`。lens 只支持圆角 SDF 形状，其它 Shape 直接 throw。`refractionAmount` 传入 shader 为 **负数**（`-refractionAmount`）。
4. **绘制顺序（H5 拆开）。** `onDrawBehind` → backdrop layer → `onDrawSurface` → **`drawContent()` 前景** → `onDrawFront`。阴影/高光是 **独立 Modifier**（`ShadowElement` / `HighlightElement` / `InnerShadowElement`），不是透镜 pass 的 uniform。
5. 测量：`placeWithLayer` + `clip=true` + Offscreen compositing。

**对 Yo：** 铬与透镜分 Modifier = 鸿蒙/Yo 的 L4 chrome 不进 AGSL。Compose 坐标用 `localPositionOf`，View 只能 screen/window 差。Kyant **没有** smin 容器；每块 `drawBackdrop` 各自切一份同一 `LayerBackdrop`（共享 H1，重复 H2）。

## 5. YoUI 现状链路（证据）

```text
prepareBackdrop(scene)
  BackdropHost.installIfSibling(scene, glass)
    若 glass 在 scene 内 → null（禁环）
    否则把 scene 包进 BackdropHost

BackdropHost.dispatchDraw
  子脏或无 display list → sourceNode.beginRecording
  OverlayExclusion.beginCapture + hideInside(TRANSITION_VISIBILITY)
  super.dispatchDraw(recording)     // H1 push
  showInside / endCapture
  invalidateSamplers(this)          // 只打 overlay 根
  canvas.drawRenderNode(sourceNode)

BackdropGlass.onDraw
  OverlayExclusion.skips(this) → 空画（正在被祖先录）
  BackdropPolicy.resolve(live, MotionPolicy)
    SOLID / REDUCE 透明 / 高对比 / MINIMAL → 实色 H5
  HardwareGlassPipeline.draw        // API 31+ 硬件
    resolveSampledNode: host.sourceNode 优先，否则 GlassCaptureStage
    屏幕坐标差 origin + sampleOffsetY（FLUSH/VEIL 上抬）
    BackdropSampleSpace: 降采样 + pad = max(2·blur, sampleInset)
    sampleNode: 填 page 不透明底 + scale + drawRenderNode(sampled)   H2
    blurNode + RenderEffect.blur                                     H3
    bodyNode: 上采样霜 + tint；无透镜时 DST_IN clip / SDF silhouette
    lensNode: draw body，再 GlassLensProgram.install RuntimeShader
              禁止 createChainEffect（Motorola 忽略嵌套链）         H4
    compositeCached: shadow → plate → fade → stroke → rim → L6/L7   H5

无 host 时 PreDraw → GlassCaptureStage.recordSource
  OverlayExclusion.hideInside; source.draw(captureNode)             H1 pull
软件：SoftwareGlassPipeline bitmap + StackBlur，无 HIGH 透镜
```

`GlassLensProgram`：边带 `lip=(1-t)²`，`sampleAt = p - n * travel`（向内），内部 identity；可选 chroma；外 scatter 独立。**不读指针。**

`BackdropPolicy`：REDUCE 透明/高对比/HIDDEN/MINIMAL → SOLID。`reduceMotion` 或 REDUCED budget → CACHED（仍可走 GPU 霜，但不强制每帧 live）。

## 6. 四套对照（实现时必须锁的决策）

| 跳 | QWEA0 | Abdullajon View | Kyant | Yo 现状 | Yo 目标 |
|----|-------|-----------------|-------|---------|---------|
| H1 所有者 | 每块玻璃 pull | Provider 推一次 | LayerBackdrop 一份 | Host 推；无 Host 则 PreDraw pull | **默认 Host 推**；未入树 Dialog 才 pull |
| 排除玻璃 | Capture VISIBLE 跳分支 | 树结构（sibling） | 内容层不含玻璃 modifier | OverlayExclusion 注册表 + hideInside | 保持注册表；簇内板不得进 host |
| N 块采样次数 | N 次整树 | 1 次 + N 次切片 | 1 次 layer + N 次切片 | 1 次 host + N 次 pipeline | **1 次 host + 1 次 union shader** |
| H3/H4 接线 | `createChainEffect` | 同左 | Compose RenderEffect 累加 | **分 node**，透镜挂在 body 上 | 保持分 node（Motorola） |
| 折射方向 | `-n` 向内（注释：外向=黑边） | `-n * amount * lens` | shader 里 amount 为负 | `-n * travel` 向内 | 锁定向内；外沿只 scatter |
| 多板融合 | 同 View 第二 shape | Compose 最多 8；View=0 | 无 | 无 | GlassUnion 进 **同一** H4 |
| 触点 | 写进透镜 bulge | 写进透镜 + View scale | 独立 highlight modifier | L6 overlay，透镜不读指针 | **保持分通道** |
| 物化 | 无（alpha/scale） | Compose progress 缩 SDF | 无 | Dialog opacity/L7 | `materialProgress` 只缩放 fillet |
| a11y | 实色板 | 降 tier | 调用方 | MotionPolicy → SOLID | 同现状，Clear 仍要 dim token |

## 7. 源码锚点（本轮逐文件）

### QWEA0（clone `QWEA0--Liquid-Glass-Android`）

| 跳 | 文件 | 证据 |
|----|------|------|
| 档位 | `LiquidGlassView.drawGlassEffect` | opaque → `tryDrawLensGlass` → `tryDrawHardwareBlur` → CPU `renderGlassEffectSync` |
| H1+H2 | `GlassLensRenderer.draw` | `getLocationOnScreen` 差；`setPosition(-margin,…)`; `BackdropCapture.draw` |
| 排除 | `BackdropCapture.drawContent` | `setTransitionVisibility(INVISIBLE)` + 公开 `View.draw`；注释写明硬件 `dispatchDraw` 不进 `View.draw` |
| H3→H4 | `GlassLensRenderer.buildEffect` | `createChainEffect(lens, blur)`，内层先模糊 |
| 指针 | 同文件 uniforms | `press` / `touchPos` / `touchAmp` 写进透镜 |
| CPU | `EnhancedBlurEffect.captureBackdrop` | 软件 Canvas 上 `parent.draw`；此时 `isCapturingBackdrop` 才有效 |

### Abdullajon View（clone `Abdullajon1881--LiquidGlass`）

| 跳 | 文件 | 证据 |
|----|------|------|
| 树 | `LiquidGlassProviderLayout` 头注释 | 玻璃必须是 sibling，禁止放进 Provider |
| H1 | `dispatchDraw` → `BackdropRecorder.drawRecorded` | `contentNode.beginRecording` + `super.dispatchDraw`；然后对每个 consumer `postInvalidateOnAnimation` |
| H2 | `GlassViewRenderer.drawEffectGlass` | `inflate = 2*blur + |refract| + 4`；`translate(inflate - relative)` + `drawRenderNode(provider)` |
| H3+H4 | `buildShaderEffect` | `createChainEffect(shader, prepBlur+sat)`；`MERGE_SMOOTHING = 0f`；只 pack 一块 `PackedGlassShape` |
| 多板 | `LiquidGlassContainerState`（Compose） | 最多 8 形、progress morph；**不在 View 引擎** |

### Kyant Compose（clone `Kyant0--AndroidLiquidGlass`）

| 跳 | 文件 | 证据 |
|----|------|------|
| H1 | `rememberLayerBackdrop` / `LayerBackdrop` | 一份 `GraphicsLayer` |
| H2 | `LayerBackdrop.drawBackdrop` | `localPositionOf` 失败则 window 差；`translate(-offset)` + `drawLayer` |
| H3+H4 | `DrawBackdropModifier.ContentDrawScope.draw` + `effects/Lens.kt` | behind → plate → surface → `drawContent` → front；`refractionAmount` 写入负数 |
| 形状 | `Lens.kt` | 非圆角 Shape `throw` |

### YoUI（`youi_kit/.../backdrop/`）

| 跳 | 文件 | 证据 |
|----|------|------|
| Host 准入 | `BackdropHost.installIfSibling` | `android.R.id.content` / 未入树 / glass 在 scene 内 → `null` |
| H1 push | `BackdropHost.dispatchDraw` | 脏则 `sourceNode.beginRecording` + `hideInside` + `super.dispatchDraw`；录完 `invalidateSamplers` |
| 排除 | `OverlayExclusion` | `skips` 仅 contained；`invalidateSamplers` 在 capturing 栈非空时 return；只 invalidate overlay 根 |
| 接线 | `BackdropGlass.bindSource` / `drawGlass` / PreDraw | 有 host 走 GPU；无 host PreDraw 才 `hardware.recordSource` 或 `software.capture` |
| H1 pull | `GlassCaptureStage.recordSource` | `source.draw(recording)` + hideInside；capturing 中直接 return |
| H2–H5 | `HardwareGlassPipeline.draw` | sample（填 page 底）→ blurNode → bodyNode → `installRestOptics` 把 RuntimeShader **挂在 lensNode** → `compositeCached` |
| Motorola | 同文件类注释 + `GlassLensProgram.install` | 禁止 nested `createChainEffect`；`lensNode.setRenderEffect(createRuntimeShaderEffect)` |
| 政策 | `BackdropPolicy.resolve` | REDUCE 透明/高对比/MINIMAL → SOLID；reduceMotion → CACHED |

## 8. 明确不要从 Android 开源搬的跳

- QWEA0 CPU `parent.draw` 当默认热路径。
- QWEA0 每玻璃一次整树 Capture。
- Abdullajon 每 consumer `postInvalidateOnAnimation` 风暴。
- Kyant/Abdullajon `createChainEffect(shader, blur)` 当 HIGH（Motorola 已否）。
- 向外采样「放大一圈真实背景」（子输入越界黑边；QWEA0 已踩过）。
- 位移贴图色差链（旧 GPU）当透镜 2.0 的替代。
- 自定义 `setCustomBackdropCapture: Bitmap` 打掉 GPU。

## 阅读范围

QWEA0：`LiquidGlassView.kt`（onDraw 档位、tryDrawLensGlass、CPU sync、Capture 标志）；`GlassLensRenderer.kt` 全文（AGSL + draw + chain）；`HardwareBackdropBlur.kt` draw；`BackdropCapture.kt`；`EnhancedBlurEffect.captureBackdrop`；`GlassMaterial.kt`；`BackdropLuminanceMeter` 未逐行，只见回调接线。

Abdullajon：`LiquidGlassProviderLayout`；`BackdropRecorder`；`GlassViewController.draw`；`GlassViewRenderer`；`LiquidGlassShaders.kt`；`GlassRenderTier`；`LiquidGlassContainerState`（Compose morph）。View 路径确认 merge=0。

Kyant：`DrawBackdropModifier` 的 `ContentDrawScope.draw` / `recordBackdropBlock`；`LayerBackdrop.drawBackdrop`；`effects/Lens.kt`；`LayerRecorder.recordLayer`。未读全部 catalog demo。

YoUI：`BackdropHost.dispatchDraw`；`OverlayExclusion`；`BackdropGlass.drawGlass` / PreDraw；`HardwareGlassPipeline` sample→blur→body→lens→composite；`GlassCaptureStage`；`GlassLensProgram` AGSL；`BackdropPolicy`；`SoftwareGlassPipeline` 头。

未跑四套 demo；未在 Motorola 上复现 chainEffect 忽略（以现有注释与真机史为据）。
