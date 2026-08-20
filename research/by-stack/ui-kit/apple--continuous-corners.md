---
id: research.apple-continuous-corners
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [swift, objc, java]
  frameworks: [uikit, swiftui, android]
also_relevant: []
utilization: [adapt]
source:
  platform: other
  url: https://developer.apple.com/documentation/quartzcore/calayercornercurve
studied_at: 2026-08-20
related:
  - research.phamfoo-figma-squircle
  - research.racra-smooth-corner-rect-android-compose
---

# iOS 圆角：连续曲率 + 同心嵌套

主题笔记。Apple 不公开 `CALayer.cornerCurve.continuous` 的实现源码；几何以公开 API 与 Figma 复现文为准。

## 1. 两种曲线

| 曲线 | API | 观感 |
|------|-----|------|
| circular | `CALayerCornerCurve.circular` / `RoundedCornerStyle.circular` | 四分之一圆接到直线。接点曲率从 1/R 跳到 0，出现「鼓包」。 |
| continuous | `CALayerCornerCurve.continuous`（iOS 13）；SwiftUI 默认 `.continuous` | G2：直线→角曲率连续。角更「软」，贴边更长。`UIBezierPath(roundedRect:cornerRadius:)` 走这条。 |

系统控件、分组列表、卡片用 continuous。图标超椭圆更极端，不要拿图标 squircle 当按钮默认。

## 2. 几何（Figma / MartinRGB 复现）

公开复现（Figma《Desperately seeking squircles》，phamfoo/figma-squircle、racra SmoothCorner）每个角 = **两段三次贝塞尔 + 一段圆弧**：

- 平滑度 `s ∈ [0,1]`。`s=0` 退化为 90° 圆弧（鸿蒙/Skia）。`s=1` 圆弧消失，接近 iOS 7 图标。
- UIKit `cornerCurve.continuous` 对应 **s≈0.6–1.0**（Sketch/Figma 默认平滑 60%；racra 默认 60）。Yo 的 `CONTINUOUS` 取 **0.6**，不要用 PaintCode 图标系数 `1.52866483` 当控件默认。
- 过渡长度 `p = (1+s)·R`，且 `p` 不得超过该角在邻边上的预算（半边或与邻角分账）。

PaintCode iOS 7 路径是图标级固定三次曲线，不能四角独立半径，也不随尺寸做预算分配。只作对照，不进控件默认。

## 3. 同心（WWDC 2025 / Liquid Glass）

三种嵌套：

- **fixed**：半径常量。
- **capsule**：半径 = 短边一半。
- **concentric**：`inner = max(0, parentRadius − padding)`。

鸿蒙卡片 FAQ 的「内外半径一致」是防 AA 缝；iOS concentric 是光学嵌套。Yo 两者都要：外壳/内容同规格用 **匹配**；内边距里的子表面用 **concentric**。

## 4. Android 约束

- `Outline.setRoundRect` 只有 circular，硬件阴影走这条。
- continuous 必须 `Outline.setPath`（API 30+）或 `setConvexPath`（仍须凸）。阴影会略偏。
- 玻璃 SDF 是 circular RRect。连续路径与 SDF 混用会漏光。

## 5. 对 YoUI 的用法

| 点 | 方式 | 说明 |
|----|------|------|
| 双曲线 | adapt | `CIRCULAR` 默认（鸿蒙/SDF/Outline）；`CONTINUOUS` 给非玻璃表面 opt-in |
| 平滑 0.6 | adapt | 对齐 UIKit，不是图标 squircle |
| concentric | reuse-pattern | `inner = parent − inset`，下限 0 |
| 单 Path 多用 | reuse-pattern | fill / clip / outline / ripple 同一条几何 |
| 抄 PaintCode 常量进默认 | anti-pattern | 图标级，四角不能独立 |

## 阅读范围

Apple `CALayerCornerCurve` / `UICornerCurve` / `UnevenRoundedRectangle`；Figma 博文公式；phamfoo `draw.ts`；racra `SmoothCorner.kt`。未反编译 UIKit。
