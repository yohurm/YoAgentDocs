---
id: rules.type.ui-kit.content-region
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 列表项、卡片、按钮、揭示区等有圆角与按压反馈的控件
related: [rules.type.ui-kit.layering]
---

# 内容区

内容区是 **L4 的布局契约**，不是调 ripple 参数的副作用。

## 必须

- 每个带圆角的可交互表面，先定义内容区矩形（含内边距后、圆角内的孩子可绘制范围）。
- **区内超出一律裁剪**：文字、图标、操作钮、揭示块可见部分、ripple。裁的是绘制，不是把孩子往里挪躲开圆角。
- 划内容区时**不要改 ripple 的几何算法**。Ripple 仍按 L1 原规格画，再被内容区 clip。圆角 clip / fill 走 `YoCorner`，与 ripple 共用同一条 Path。
- 右侧操作、圆形按钮等必须落在内容区内，不能压在圆角外侧。
- 横滑揭示：停住时露在内容区外的部分被裁掉；不要为了「不被裁」把揭示块整体左移。

## 应当

- 内容区与铬层（阴影、外描边、光感边）分开：阴影可以在区外，内容与 ripple 不能。
- 手势冲突（横滑 vs 竖滑）放 L3 策略，不在 item view 里用临时阈值补丁。

## 反例

- 把 ripple inset 加大来假装裁进圆角。
- 揭示区画在内容区外却不 clip，或为避 clip 改布局结构。
