---
id: research.Abdullajon1881-LiquidGlass
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Kotlin, AGSL]
  frameworks: [android-view, compose]
also_relevant: []
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: Abdullajon1881/LiquidGlass
  url: https://github.com/Abdullajon1881/LiquidGlass
  head: 72ad05c
  cloned_to: "%TEMP%/YoAgentResearch/Abdullajon1881--LiquidGlass"
studied_at: 2026-09-04
related:
  - research.apple-liquid-glass
  - research.android-liquid-glass-data-path
  - research.QWEA0-Liquid-Glass-Android
  - research.Kyant0-AndroidLiquidGlass
---

# Abdullajon1881/LiquidGlass

## 入选理由

明确把 Android 侧做成 iOS `GlassEffectContainer` 的对应物：多孩子登记形状、**smooth-min 并集**、共享 backdrop、`spacing` 控制融合距离。Yo 现在每块 `BackdropGlass` 独立采样，缺这一层。JVM 上有与 AGSL 同公式的 `GlassMath`，可单测光学。

## 项目是什么

「受 Apple 启发、独立重做」的液态玻璃，View + Compose + Expo 三套壳，核心在 `liquidglass-core`（shader 字符串、shape pack、tier）。定位声明与 Apple 无隶属关系。

## 架构

```text
BackdropRecorder / LiquidGlassProviderLayout
  dispatchDraw → RenderNode 录内容 → invalidate 所有 consumer

LiquidGlassContainer (Compose) / 多 LiquidGlassView (View)
  登记最多 8 个 packed RRect
  mergeSmoothing = spacing
  单 shader：scene SDF = smin(各形状)
  每像素：圆剖面透镜、touch bulge、色散、tint、角度高光

GlassMath          JVM 镜像 SDF / smin / 法线 / 透镜剖面
GlassRenderTier    SHADER API33 / BLUR API31 / SCRIM API21
```

`BackdropRecorder.drawRecorded`：硬件画布上 `beginRecording` 子树，再画到当前 canvas，然后刷新注册的玻璃。这就是「玻璃不能采样玻璃」的工程解：消费者不在被录的 display list 里。

`Modifier.liquidGlassContainer` 文档直接写：Android counterpart of `GlassEffectContainer`。`spacing` 是开始融化的距离。超出 8 个形状丢掉——作者引用 Apple「簇要小」。

`GlassMath.lensProfile`：边带内 0（板心不弯）→  Rim 处 1，斜率在 Rim 无穷，注释称为凸玻璃边的光学签名。与 Yo `LiquidGlassOptics`「fillet 上 inward travel、内部 identity」是同一类**边带透镜**，不是整板桶形。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 共享 BackdropRecorder + consumer 列表 | reuse-pattern | 升格现有 `BackdropHost.retain`：一 host 多 glass，禁止每块各录一棵树 |
| smin 并集 + spacing | adapt | L1 `GlassUnion`；上限跟 token，不要 8 写死在 shader 注释外 |
| JVM↔AGSL 同公式 | reuse-pattern | 已有 `LiquidGlassOptics`；merge 必须同样可测 |
| 档位 SHADER/BLUR/SCRIM | lesson-only | Yo 已有 HIGH/MID/LOW + MotionPolicy；不要第三套 tier 名 |
| Compose / RN / Expo 壳 | anti-pattern | 不进 youi_kit |
| 单 shader 塞 touch bulge | anti-pattern | Yo 触点光是 L6，折射不读指针（已冻结） |
| 最多 8 形状硬帽当 API | adapt | 策略上限可以有；超限应拒绝编译/登记，不要静默 drop |

## 架构设计经验

容器不是「多个 `YoBlur` 并排」。容器拥有 **一张** 场景 SDF 和 **一次** 捕获。子玻璃只贡献形状（中心、半宽、半径、id），不拥有采样权。统一适应 = 同一套 Regular/Clear 与同一 luminance 作用在并集上。

## 与当前工作

能用：`BackdropHost` 扩展为多 consumer；新 L2 `GlassCluster`。必须改写：touch 写进透镜。不要用：Expo module、Compose `GraphicsLayer` 作为 View 路径的实现。

## 数据链路（推录制）

H1 在 `LiquidGlassProviderLayout.dispatchDraw`：`BackdropRecorder` 把子树录进一份 `contentNode`，再 `postInvalidateOnAnimation` 所有 consumer。H2 是玻璃本地 node 对这份 list 做 `translate(inflate - relative)` 切片。View 的 `GlassViewRenderer` **强制 `MERGE_SMOOTHING=0`、只 pack 一块形状**；smin 容器只在 Compose `LiquidGlassContainerState`。逐跳对照见 [android--liquid-glass-data-path.md](./android--liquid-glass-data-path.md)。

## 阅读范围

`liquidglass-core/.../GlassMath.kt` / `LiquidGlassShaders.kt`；`liquidglass-view/.../LiquidGlassProviderLayout.kt`、`BackdropRecorder.kt`、`GlassViewRenderer.kt`（`drawEffectGlass` / `buildShaderEffect`）；`liquidglass-compose/.../LiquidGlassContainerState.kt`。未读 RN/Expo 包。未跑 sample。
