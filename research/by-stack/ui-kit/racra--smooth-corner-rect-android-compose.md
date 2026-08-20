---
id: research.racra-smooth-corner-rect-android-compose
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [kotlin]
  frameworks: [jetpack-compose]
also_relevant: []
utilization: [adapt, lesson-only]
source:
  platform: github
  repo: racra/smooth-corner-rect-android-compose
  url: https://github.com/racra/smooth-corner-rect-android-compose
  cloned_to: "%TEMP%/YoAgentResearch/racra--smooth-corner-rect-android-compose"
studied_at: 2026-08-20
related:
  - research.phamfoo-figma-squircle
  - research.apple-continuous-corners
---

# racra/smooth-corner-rect-android-compose

## 入选理由

同一套 Figma 公式的 Android 路径写法：相对顶点的控制点 + `arcToRad`。证明 Compose `Outline.Rounded`（circular）与 `Outline.Generic`（continuous）必须分支——Yo 的 `Outline.setRoundRect` vs `setPath` 同理。

## 项目是什么

Compose `CornerBasedShape`。`SmoothCorner` 算点；`AbsoluteSmoothCornerShape` 拼 Path。

## 架构

- `smoothnessAsPercent == 0` → `Outline.Rounded`（系统 RRect）。
- 否则 `Outline.Generic`。
- 半径大于半短边时在 circular 与 smooth 之间插值，避免胶囊被超椭圆化。
- 默认平滑 60%。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 0 平滑走 RRect | reuse-pattern | Yo `CIRCULAR` 不要走 cubic 近似圆 |
| 胶囊回退 circular | reuse-pattern | `radius ≥ min(w,h)/2` 时 continuous 改 circular |
| 依赖 Compose Shape | anti-pattern | YoUI 是 View 体系 |

## 架构设计经验

曲线类型是绘制策略，不是又一个 radius token。token 仍是 dp；curve 是 path 怎么连。

## 与当前工作

能直接用：分支与胶囊回退。必须改写：View `Path` / `ViewOutlineProvider`。不要用：Compose 主题 `Shape` 注入。

## 阅读范围

`SmoothCorner.kt`、`AbsoluteSmoothCornerShape.kt`。未把 demo app 当规范。

## 落选对照

`stoyan-vuchev/squircle-shape`：每角一条 cubic，平滑只是控制点系数。无 G2、无圆弧段，对不齐 UIKit。已克隆作反例，不单独成篇。
