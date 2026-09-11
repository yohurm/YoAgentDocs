---
id: research.microsoft-Windows.UI.Composition-Win32-Samples
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [win32, windows-ui-composition, directcomposition]
also_relevant: [ui-kit]
utilization: [reuse-pattern, adapt]
source:
  platform: github
  repo: microsoft/Windows.UI.Composition-Win32-Samples
  url: https://github.com/microsoft/Windows.UI.Composition-Win32-Samples
  head: ee50e2e
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--Windows.UI.Composition-Win32-Samples"
studied_at: 2026-09-08
related:
  - research.openharmony-window_window_manager
  - research.harmony-motion-system
---

# microsoft/Windows.UI.Composition-Win32-Samples

## 入选理由

Win32 上官方的「合成器 visual + 独立线程动画」样本。Yohu 投屏已用 DirectComposition；启动交接应落到同一层，而不是 USER32 布局动画。

## 项目是什么

演示在 C++ Win32 / WPF / WinForms 里托管 Windows.UI.Composition（Visual Layer）。README 写明：这是 Windows 10+ 上 **DirectComposition 的推荐上层**；动画在合成器线程 60fps，不绑 UI 线程。

## 架构

`cpp/HelloComposition`：

1. `DispatcherQueueController`（当前线程 ASTA）
2. `Compositor` + `ICompositorDesktopInterop::CreateDesktopWindowTarget(hwnd)`
3. `ContainerVisual` 作 root（`RelativeSizeAdjustment = 1`）
4. `SpriteVisual`：`Offset` / `Size` / `Brush`
5. `CreateVector3KeyFrameAnimation` → `StartAnimation(L"Offset", animation)`

动的是 **Visual.Offset**，HWND 尺寸不变。另有 VirtualSurfaces、ScreenCaptureforHWND（把 HWND 内容送进 composition）。

Yohu 投屏路径已是更底层的 `DCompositionCreateDevice` + `IDCompositionAnimation`（clip 四边），语义相同：合成器属性，不是 `SetWindowPos`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| HWND 当 target，内容是 Visual 树 | reuse-pattern | 交接 overlay 固定为目标矩形；只动画 snapshot visual 的 scale/offset/opacity |
| KeyFrame / cubic 在 compositor | reuse-pattern | 与 `mirror_present` `IDCompositionAnimation::AddCubic` 同族 |
| Visual Layer 替代每帧 GDI | adapt | 启动快照可 UpdateLayeredWindow 或 DComp 位图 brush；不要 StretchBlt 活客户区 |
| UWP DispatcherQueue | lesson-only | 壳已是 DirectComposition，不必为 splash 再引 WinRT Compositor |

## 架构设计经验

Win32 桌面动画的正确寄宿：一个 HWND + compositor visual。改 `Size` 的 HWND 会强制 DWM 拉伸旧帧（Yohu 投屏 ADR 已禁 `SetWindowPos` 冒充占用过渡）。启动展开触发同样的拷贝位图问题。

## 与当前工作

同屏 morph：overlay HWND = 主窗最终外框（一次创建），snapshot visual 从 splash 相对矩形缩放到铺满。异屏：splash overlay 只做 scale+opacity；主窗 HWND 一次到位。复用 `IDCompositionAnimation` 或分层窗口逐帧提交冻结位图（源尺寸不变）。

## 阅读范围

`cpp/HelloComposition/README.md`、`CompositionHost.cpp`（Initialize / AddElement 动画）。未深读 VirtualSurfaces 贴图管理、WPF HwndHost 包装。
