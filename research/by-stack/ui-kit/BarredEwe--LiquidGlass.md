---
id: research.BarredEwe-LiquidGlass
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Swift, Metal]
  frameworks: [uikit, swiftui]
also_relevant: []
utilization: [anti-pattern, lesson-only]
source:
  platform: github
  repo: BarredEwe/LiquidGlass
  url: https://github.com/BarredEwe/LiquidGlass
  head: 81d9218
  cloned_to: "%TEMP%/YoAgentResearch/BarredEwe--LiquidGlass"
studied_at: 2026-09-04
related:
  - research.apple-liquid-glass
---

# BarredEwe/LiquidGlass

## 入选理由

iOS 上**不依赖 iOS 26 私有材料**的 Metal 复现。用来对照「错误的 L0」：截图层 vs display list。README 自己写明与 Apple Liquid Glass 不同、官方用私有 API。

## 项目是什么

Swift 包：把祖先层级画成 `CGImage` / `MTLTexture`，再跑自定义 fragment（SDF 盒、噪声 jitter 模糊、SDF 折射、描边高光）。SwiftUI + UIKit 共用 Metal 后端。更新模式 `.continuous` / `.once` / `.manual`。

## 架构

```text
HierarchySnapshotCapturer
  找到根视图 → convert glass.bounds
  glass.layer.opacity = 0
  targetView.layer.render(in: cgContext)    // CPU 截图层
  恢复 opacity
BackgroundTextureProvider  → MTLTexture + GPU blur
MetalShaderView / LiquidGlassShader.metal
```

`boxSDF` 与 Inigo 圆角盒相同。折射：`dfdx/dfdy(sdf)` 当梯度，`pow(abs(sdf), 10) * -0.16` 沿梯度偏移。模糊是 9 次 jitter LOD 采样 + 饱和，不是系统高斯。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `layer.render` 整树截图 | anti-pattern | 贵、非 live、易把玻璃/叠加层画进去；Yo 必须继续 `RenderNode` 录兄弟场景 |
| 用 opacity=0 排除自身 | lesson-only | 意图对，手段错；Yo 用 `OverlayExclusion` + 不把 glass 放进 host |
| 连续/一次/手动刷新 | reuse-pattern | 对应已有 LIVE / CACHED / 显式 `refresh()` |
| 噪声假模糊 | anti-pattern | 官方 frost 是真散射；Yo 走 `RenderEffect` |
| 把 time uniform 当材料 | anti-pattern | 液态是形状/触控/环境，不是 shader 里自己涨的 time |

## 架构设计经验

没有系统 `UIGlassEffect` 时，社区会退回「截背景 + 后处理」。那是 **UIBlur 时代的玻璃拟态**，不是 iOS 26 的共享采样材料。Yo 已经越过这一层；本仓只作反例，避免有人再加 `PixelCopy` 热路径。

## 与当前工作

不要用。捕获对比足以证明 `BackdropHost` 方向正确。

## 阅读范围

`HierarchySnapshotCapturer.swift`；`LiquidGlassShader.metal` 前 80 行；`README.md`。未读 UIKit 包装器全文。
