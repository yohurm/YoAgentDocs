---
id: research.harmony-corner-radius
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [ETS, C++, Java]
  frameworks: [harmonyos, arkui, android]
also_relevant: []
utilization: [reuse-pattern, adapt]
source:
  platform: other
  repos:
    - openharmony/graphic_graphic_2d
    - openharmony/arkui_ace_engine
  cloned_to:
    - "%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d"
    - "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.openharmony-graphic_graphic_2d-roundrect
---

# HarmonyOS 圆角：半径层级 + 圆弧 RRect + clip 所有权

主题笔记。实现 Agent 对齐规范与几何，不要把第三方源码拷进 YoAgentDocs / Android-YoUI。

本地规范：`E:\Dev\Doc\HarmonyOS-Developer-docs\设计\设计指南\通用设计基础\视觉风格\圆角参数.md`（2026-08-12）。官方定义圆角是「对矩形直角做圆弧化」，**不是** iOS 连续曲率 / squircle。

## 1. 设计：同层统一、层级正相关

| 常用值 | Token 量级 | 场景 |
|--------|------------|------|
| 4vp | level2 | 标签、角标 |
| 8vp | level4 | 图片、图标、列表内模块 |
| 16vp | level8 | 通知卡片、内容容器 |
| 20vp | level10 | 按钮、菜单、选项卡 |
| 32vp | level16 | 半模态、弹出框 |

半径随视觉层级增大：浮层 > 容器 > 小控件。同一类型控件必须同一半径。全量表 0–36，步长 2vp（缺 28、30、34）。

卡片 FAQ：内容圆角必须与外壳规格一致（1×2 / 2×2 宫格 18vp，2×4 / 4×4 宫格 22vp），否则抗锯齿露出白边。这是 **嵌套半径匹配**，不是「内层随便小一档」。

## 2. 几何：Skia / Rosen RoundRect（椭圆弧）

ArkGraphics `RoundRect(rect, xRadii, yRadii)`；`setCorner` 可四角独立、XY 独立。半径 ≤0 退化为直角。FAQ 示例用 `arcTo` / `arc` 四分之一圆，与 `addRoundRect` 同类。

Rosen `Drawing::RoundRect`（`round_rect.cpp`）：

- 四角各存 `(xRad, yRad)`。
- `ScaleRadii()`：若同一边上两角半径之和超过边长，**按比例缩小**，不是只夹 `min(r, half)`。
- 任一轴 ≤0 → 该角整角清零。

没有 superellipse、没有 `cornerCurve`。液态玻璃 SDF（`sdRoundBox`）也是这套圆弧 RRect。玻璃裁剪路径必须与 SDF 同几何，否则边缘光会离边。

## 3. 绘制所有权：borderRadius ≠ clip

官方 FAQ《圆角边框问题》：

- `borderRadius` 定义铬层圆角；子项默认仍按矩形画。
- 复杂渲染 / 自定义组件要 **`borderRadius` + `clip(true)`**，子项才裁进圆角。
- `clipShape` 会覆盖 `borderRadius`；只能有一条裁剪形状。
- 圆角处黑边：抗锯齿半透明像素透出下层。对策是背景色一致、上层半径略小、或下层略缩小，**不要加粗描边伪装**。

半径上限：组件宽或高的一半（胶囊 /  pill）。

## 4. 对 YoUI 的用法

| 点 | 方式 | 说明 |
|----|------|------|
| 半径 token 层级 | adapt | 已有 `Dimens`；不要在 View 里再写魔法 dp |
| RRect 比例缩放 | reuse-pattern | 邻角半径之和 > 边长时按比例缩，对齐 Rosen |
| 铬与内容区 clip | reuse-pattern | 与 `content-region.md` 一致：阴影在区外，内容/ripple 必须裁 |
| 玻璃 SDF | anti-pattern | Backdrop / immersive 不得改成连续圆角，否则 SDF 对不齐 |
| 黑边 | lesson-only | 叠层同半径 + 不同底色会出 AA 缝 |

## 阅读范围

本地《圆角参数》、FAQ《圆角边框问题》《如何绘制圆角矩形》《卡片圆角显示异常》；`graphic_graphic_2d` 的 `round_rect.h/.cpp`。未读闭源 HDS 控件实现。
