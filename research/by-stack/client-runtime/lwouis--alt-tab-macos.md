---
id: research.lwouis-alt-tab-macos
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Swift]
  frameworks: [appkit, core-animation]
also_relevant: [ui-kit]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: lwouis/alt-tab-macos
  url: https://github.com/lwouis/alt-tab-macos
  head: 2f3c677
  cloned_to: "%TEMP%/YoAgentResearch/lwouis--alt-tab-macos"
studied_at: 2026-09-08
related:
  - research.apple-motion-system
---

# lwouis/alt-tab-macos

## 入选理由

真实 macOS 窗口切换器：大量缩略图、AppKit 布局。用来确认桌面端如何**禁止**隐式 layer 动画，以及窗口 frame 不是动画通道。

## 项目是什么

开源 Alt-Tab。切窗面板、瓷砖缩略图、设置窗。活跃维护。

## 架构

`TileView.disableImplicitLayerAnimations`：`wantsLayer = true`，并把 layer `actions` 里 `position` / `bounds` / `frame` / `hidden` / `opacity` / `transform` 全部设为 `NSNull()`。注释写明：AppKit 布局若在 `CATransaction` 外改这些键，系统会插隐式动画；NSNull 是「此键无动画」的文档哨兵。

真正要动的装饰（Pro 按钮扫光）才显式 `CABasicAnimation` + `CATransaction.setCompletionBlock`。

设置窗用 `setFrameAutosaveName` 记几何，**不是** animate 到新 frame。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 关掉 layout 隐式动画 | reuse-pattern | Windows 上等价：不要靠 DWM 拉伸 HWND 冒充过渡 |
| 显式 CA 动画才动 transform/opacity | reuse-pattern | 与鸿蒙 / Composition 一致 |
| 缩略图 tile 的 frame 补间 | anti-pattern | 他们主动禁掉 |

## 架构设计经验

桌面壳里「窗口/单元格矩形」属于布局。合成器默认会对 layer 属性做隐式动画，必须白名单。Yohu 的 bug（展开时图标抖动）就是 USER32 拷贝旧客户区 + GDI 重绘两套尺寸对打，和这里要禁 `frame` 动画是同一类问题。

## 与当前工作

启动交接只允许白名单属性：冻结快照的 scale / offset / opacity。HWND 外框最多改 **一次**（放到最终矩形或创建 overlay）。

## 阅读范围

`src/switcher/main-window/TileView.swift`（`nullActions`）；`HelperExtensions.swift` 里 `CATransaction.setDisableActions`。未读整个切换器手势与快捷键。
