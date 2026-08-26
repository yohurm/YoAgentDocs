---
id: research.openharmony-graphic_graphic_2d-roundrect
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++]
  frameworks: [openharmony, rosen, skia]
also_relevant: []
utilization: [reuse-pattern, adapt]
source:
  platform: github
  repo: openharmony/graphic_graphic_2d
  url: https://gitcode.com/openharmony/graphic_graphic_2d
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d"
studied_at: 2026-08-20
related:
  - research.harmony-corner-radius
---

# openharmony/graphic_graphic_2d（RoundRect）

## 入选理由

鸿蒙圆角的运行时真源。ArkUI `borderRadius` 最终落到 Rosen `Drawing::RoundRect`，与 Android `Path.addRoundRect` / Skia `SkRRect` 同类。要抄「怎么夹半径」，不要抄「连续圆角」——这里没有。

## 项目是什么

方舟 2D 图形栈。本次只读 `rosen/modules/2d_graphics` 的 RoundRect，不读合成器全貌。

## 架构

- `RoundRect`：外接矩形 + 四角 `(xRad, yRad)`。
- `IsSimpleRoundRect`：四角相同可走快路径。
- `ScaleRadii()`：四条边分别看「两角半径之和 vs 边长」，取最小比例；再 `AdjustRadiiX/Y` 把浮点误差收到边长内。
- `ClampToZero`：任一轴 ≤0 则该角 XY 都清零（椭圆弧缺一轴就不是角）。
- NDK `OH_Drawing_RoundRectCreate(rect, xRad, yRad)` 与 ArkTS `new drawing.RoundRect(rect, xRadii, yRadii)` 是同一对象。

没有平滑度参数。`ClipRoundRect` 与 `DrawRoundRect` 共用该对象——**裁剪形状 = 填充形状**。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 比例缩放 | reuse-pattern | 不要只 `min(r, w/2)`；邻角抢边时长边也要一起缩 |
| XY 独立半径 | adapt | Yo 第一期四角各一个标量半径（正圆角）；椭圆角以后再说 |
| 简单矩形快路径 | reuse-pattern | 四角相同走 `setRoundRect` / `addRoundRect` |
| 在此仓找 squircle | anti-pattern | 没有 |

## 架构设计经验

几何对象与绘制指令分离：先有 RoundRect，再 Draw/Clip。Yo 应对齐成「一份 Radii → Path / Outline / Drawable」，不要 View 一份、Drawable 一份、clipPath 再一份。

## 与当前工作

能直接用：缩放规则、四角枚举、clip 与 fill 同几何。必须改写：Android Outline 没有 xRad≠yRad 的 roundRect。不要用：把 Rosen 头文件搬进 App。

## 阅读范围

`include/utils/round_rect.h`、`src/drawing/utils/round_rect.cpp`、NDK `drawing_round_rect.h`。未读完整 Skia 后端栅格化。
