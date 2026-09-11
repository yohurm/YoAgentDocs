---
id: research.tauri-apps-wry-child-visible
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Rust]
  frameworks: [WebView2, Win32]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: tauri-apps/wry
  url: https://github.com/tauri-apps/wry
  head: 792d035
  cloned_to: "%TEMP%/YoAgentResearch/tauri-apps--wry"
studied_at: 2026-09-11
related:
  - research.tauri-apps-wry
  - research.MicrosoftEdge-WebView2Samples
  - research.synthesis.client-runtime
---

# tauri-apps/wry（子 HWND 显隐）

## 入选理由

Yohu 窗口就是 wry。上一篇只读了拖入；本轮补 **子 WebView 的 ShowWindow + SetIsVisible 必须成对、Destroy 只在 Drop**。浅克隆 HEAD `792d035`（2026-09-08）。

## 项目是什么

跨平台 WebView 封装。Windows 上主 WebView 和 `new_as_child` 都是 HWND + `ICoreWebView2Controller`。

## 架构

`src/webview2/mod.rs`：

```
set_visible(visible)
  ShowWindow(self.hwnd, SW_SHOW | SW_HIDE)
  controller.SetIsVisible(visible)

Drop
  controller.Close()
  if is_child { DestroyWindow(self.hwnd) }
```

显隐是 **同一对象上的宿主 API**，不是页面路由。拆窗只在对象释放，不在 CSS 转场。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 藏窗同时改 HWND 与 controller | reuse-pattern | 只 `SW_HIDE` 或只拆解码都会漏一层 |
| 子表面寿命跟宿主对象，不跟页面节点 | reuse-pattern | PresentHost 跟「投屏模块是否为当前模块」 |
| 页面 onCleanup / HMR 里 DestroyWindow | anti-pattern | View 重挂会把还该在的表面拆掉，留下透明洞 |

## 架构设计经验

wry 自己都不敢让 HTML 决定子 HWND。Yohu 投屏 HWND 更不该由 `YoPresence` 推断。

## 与当前工作

- 能直接用：离开 `screen-mirror` 由壳 `setActive(false)` → `shutdown`；几何仍走 `mirror.layout`。
- 不要用：在 MirrorView 观察 `.yohu-presence[data-state=closed]`（`closest` 还能误中侧栏里别的 Presence）。

## 阅读范围

`src/webview2/mod.rs` 的 `Drop`、`set_visible`、`new_as_child`。未再读拖入篇已覆盖的 `drag_drop.rs`。
