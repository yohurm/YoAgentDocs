---
id: research.MicrosoftEdge-WebView2Samples
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [WebView2, Win32, DirectComposition]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: MicrosoftEdge/WebView2Samples
  url: https://github.com/MicrosoftEdge/WebView2Samples
  head: 22f2fd1
  cloned_to: "%TEMP%/YoAgentResearch/MicrosoftEdge--WebView2Samples"
studied_at: 2026-09-11
related:
  - research.synthesis.client-runtime
  - research.tauri-apps-wry
---

# MicrosoftEdge/WebView2Samples

## 入选理由

Yohu 主窗是 **窗口化 WebView2**（wry 默认），投屏是另一块 `WS_CHILD` HWND。官方样本是「宿主控制器拥有可见性 / 合成树里才能叠层」的第一手契约，不是博客转述。对照文档 [Windowed vs. Visual hosting](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/windowed-vs-visual-hosting)。浅克隆 HEAD `22f2fd1`（2026-08-04）。

## 项目是什么

Microsoft Edge 的 WebView2 官方示例集。本轮只读：`WebView2APISample/ViewComponent.cpp`（`put_IsVisible`）、`WebView2SampleWinComp/CompositionHost.cpp`（Visual 宿主与兄弟 visual）。

## 架构

窗口化路径：

```
宿主 HWND
  └─ ICoreWebView2Controller
       put_IsVisible / put_Bounds / NotifyParentWindowPositionChanged
```

`ViewComponent::ToggleVisibility` 只改 **controller**，不改页面 CSS。最小化时 `put_IsVisible(FALSE)`，还原再按宿主自己的 `m_isVisible` 打开。文档写明：`IsVisible=FALSE` 让 WebView 不绘制，**不等于**藏掉传入 `CreateCoreWebView2Controller` 的那个容器 HWND。

合成路径（`WebView2SampleWinComp`）：

```
DesktopWindowTarget(主窗)
  └─ root Visual
       ├─ 应用自己的 Sprite/Container
       └─ WebView RootVisualTarget
```

析构时 `put_RootVisualTarget(nullptr)` 再拆 visual 树。只有这条路能让原生 visual **和** WebView 同树叠层；窗口化 WebView2 的 HWND 按 USER32 z-order **遮挡**底下内容。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 可见性是宿主 controller 的属性 | reuse-pattern | 切模块由壳改舞台开关，不由页面 `opacity` / Presence |
| 最小化藏内容、还原再按宿主旗标打开 | reuse-pattern | 与「离开投屏页拆表面、回来再按 avail 建」同构 |
| 指望窗口化 WebView 挖洞露出底下 HWND | anti-pattern | ADR-v6-026 已写：NOREDIRECTIONBITMAP 子窗不进 WebView backing，拆窗变白板/黑洞 |
| 把 WebView 改成 Visual 宿主只为叠一层投屏 | lesson-only | 要重做 wry 输入转发；Yohu 已否决为 chrome 再开 WebView，也不为叠层改整窗宿主模式 |

## 架构设计经验

窗口化 WebView2 与 `WS_CHILD` 投屏窗是 **两个 HWND**。页面淡出改变不了子窗。正确开关在 **PresentHost / controller**，调用方是工作台壳，不是模块 View 里的 MutationObserver。

## 与当前工作

- 能直接用：`mirror.present.setActive` 由 `@yohu/workbench` 在模块身份变化时调用；`IsVisible` 语义对齐 `PresentHost` 的 active。
- 必须改写：Yohu 拆的是投屏 HWND，不是 WebView controller。
- 不要用：把 Presence `data-state` 当成原生表面生命周期。

## 阅读范围

`SampleApps/WebView2APISample/ViewComponent.cpp`（ToggleIsVisible / 最小化）；`SampleApps/WebView2SampleWinComp/CompositionHost.cpp`；Learn「Windowed vs. Visual hosting」。未读全部 Scenario。
