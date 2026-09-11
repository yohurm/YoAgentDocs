---
id: research.apple-motion-system
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Swift]
  frameworks: [uikit, appkit, swiftui, core-animation]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://developer.apple.com/design/human-interface-guidelines/motion
  repos:
    - nathangitter/fluid-interfaces
    - lwouis/alt-tab-macos
  cloned_to:
    - "%TEMP%/YoAgentResearch/nathangitter--fluid-interfaces"
    - "%TEMP%/YoAgentResearch/lwouis--alt-tab-macos"
studied_at: 2026-09-08
related:
  - research.harmony-motion-system
  - research.nathangitter-fluid-interfaces
  - research.lwouis-alt-tab-macos
---

# Apple / iOS / macOS 动效体系（主题笔记）

## 背景

对照鸿蒙读 Apple：**HIG Motion**、WWDC 2018 Designing Fluid Interfaces、WWDC 2023 Animate with springs、WWDC 2024 Enhance your UI animations、AppKit `NSWindow.setFrame(_:display:animate:)`、iOS 18 zoom navigation。HIG 页面需 JS，条文来自 Apple 文档摘要与无障碍章节；实现读 Temp 里的开源仓。没有 UIKit/AppKit 源码可克隆。

Apple 的分层与鸿蒙同构，名字不同：

```
HIG（有目的、可关、可取消）
  → 弹簧为默认（感知 duration + bounce，不是随便填 mass）
    → Core Animation 渲染进程跑 transform/opacity
      → 转场配方（matchedGeometry / zoom / 交叉淡入）
        → 控件只改模型，layer 提交给合成器
```

## 关键结论

### HIG Motion

- 动效要**支撑任务**，不要为动而动；过频的自定义运动让人等。
- **Make motion optional**：重要信息不只靠运动传递；补触觉 / 声音。
- **Let people cancel motion**：能打断，不要卡到播完。
- Reduce Motion：去掉缩放、旋转、景深、周边运动；层级转场改 **cross-fade / 色变**，不要删掉「这里是子页」的信息。弹簧可拧紧以减小 bounce。

### 引擎：合成器，不是布局循环

- Core Animation 在 **render server**（独立进程）上跑。`CALayer` 的 `transform` / `opacity` / `position` 是动画属性。
- `UIView` 动画底层是 `CAAnimation`；SwiftUI 动画**没有** backing `CAAnimation`，iOS 18 起可用同一套 `SwiftUI.Animation` 驱动 UIKit/AppKit，避免三套时钟。
- **隐式 layer 动画是陷阱。** `alt-tab-macos` 把 `frame` / `bounds` / `position` / `opacity` / `transform` 的 `actions` 设成 `NSNull()`，避免 AppKit 布局路过时自动补间。要动就显式 `CATransaction` / `CABasicAnimation`。
- macOS `NSWindow.setFrame(..., animate: true)` 走 **AppKit 窗口服务器**的平滑 resize（时长 `animationResizeTime`），不是应用自己 `for` 循环改 frame。跨屏启动交接不要假装自己是窗口服务器。

### 弹簧（WWDC 2023）

- 系统默认弹簧；参数用 **perceptual duration + bounce**，不要从 mass/stiffness 拍脑袋。
- 可打断：新手势 / 新目标继承速度（`interactiveSpring` 连续 retarget）。
- 完成回调用感知时长，不要等物理 settling duration。
- `interpolatingSpring`：0→1 曲线再映射到属性区间（与鸿蒙 `curves.interpolatingSpring` 同族）。

### 转场配方

| 场景 | API | 几何关系 |
|------|-----|----------|
| 同层级共享几何 | `matchedGeometryEffect`（同一树） | 位置+尺寸跟过去 |
| 导航 push 英雄 | iOS 18 `matchedTransitionSource` + `navigationTransition(.zoom)` | 源格 morph 成目标页，可中途抓住 |
| 电脑层级 | 交叉淡入；不要手机式左右滑 | 与鸿蒙 PC 淡入淡出一致 |
| 启动 | 主窗已在最终 frame；启动面淡出（类似鸿蒙 starting surface） | 不要把主窗从 splash 尺寸拉大 |

Zoom 转场的前提：**源和目标在同一视觉连续性里**（同一屏、能对上 id）。没有「跨显示器共享元素」的系统配方。

### Fluid Interfaces（WWDC 2018 实现对照）

`nathangitter/fluid-interfaces` 把设计参数落到 `UIViewPropertyAnimator`：

- 弹簧：`response`（速度）+ `damping`（bounciness）→ `stiffness = (2π/response)²`，`damp = 4π·damping/response`。
- PiP / 抽屉：**跟手改 `center`/`transform`，松手弹簧落到离散锚点**；奖励动量（速度快 bounce 更大）。
- 动画对象是 **view transform**，不是反复改 Auto Layout 常量。

## 与当前工作的关系

| 用 | 改写 | 不要 |
|----|------|------|
| 同屏启动 = zoom / 共享容器（快照 visual morph 到主窗） | 异屏 = 小窗出场 + 主窗淡入；主窗 **一次** `SetWindowPos` 到最终矩形 | 应用层每帧改主窗宽高（不是 `NSWindow` 的 animate:true） |
| Reduce Motion / `SPI_GETCLIENTAREAANIMATION` → 瞬时 | 隐式动画关掉：只动我们声明的 scale/opacity | 布局属性（HWND 四边）当动画通道 |
| 弹簧参数用 response/damping 或鸿蒙 128/12，两边可换算 | 完成点按感知 300/350ms，不要等 spring settle | 跨屏 matchedGeometry |
| 可取消：hydrate 前可跳过 | 与鸿蒙共享「主窗先在、启动面后卸」 | 第二套 CSS 启动动画 |

## 来源与阅读范围

- https://developer.apple.com/design/human-interface-guidelines/motion
- https://developer.apple.com/design/human-interface-guidelines/accessibility（Reduce Motion）
- WWDC 2018/803、2023/10158、2024/10145（字幕/笔记，未下视频）
- `NSWindow.setFrame(_:display:animate:)` 文档摘要
- `%TEMP%/YoAgentResearch/nathangitter--fluid-interfaces`：`Spring.swift`、`Pip.swift`、`README.md`
- `%TEMP%/YoAgentResearch/lwouis--alt-tab-macos`：`TileView.disableImplicitLayerAnimations`（`actions = NSNull()`）
- 未读：SwiftUI 私有实现、AppKit 窗口服务器内部。
