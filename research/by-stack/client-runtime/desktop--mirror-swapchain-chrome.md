---
id: research.desktop-mirror-swapchain-chrome
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, C++, Rust]
  frameworks: [DirectComposition, Direct2D, SDL, libmpv]
also_relevant: [windows-desktop, ui-kit]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: other
  repo: desktop/mirror-swapchain-chrome
  url: https://learn.microsoft.com/en-us/windows/win32/directcomp/clipping
  cloned_to: "%TEMP%/YoAgentResearch/Genymobile--scrcpy"
studied_at: 2026-09-11
related:
  - research.synthesis.client-runtime
  - research.microsoft-Windows.UI.Composition-Win32-Samples
  - research.mpv-player-mpv-examples
  - research.Genymobile-scrcpy
---

# 投屏交换链铬：谁拥有回缓冲

## 入选理由

Yohu 浅色投屏卡在「开始 → 停止」后边框和空态一起退回白板。这不是 token 色值问题，是 **回缓冲所有权** 和 **DComp clip 动画** 抢同一条 Present 路径。本篇对照官方 clip 文档、已克隆的 Composition 样本、scrcpy `screen.c`、libmpv 嵌入/Render 分界。

## 项目是什么

主题深研，不是单一仓库。源码读自已有 Temp 克隆；规范读 Microsoft Learn *Clipping (DirectComposition)* 与 `IDCompositionVisual::SetBorderMode`。

## 架构

Microsoft Learn：`Clip` 裁的是 **visual 的位图内容**。圆角 clip 的边在合成器上切，画在位图最外一圈的 hairline 会被吃掉一半。clip 四边可以 `IDCompositionAnimation`，动画在合成器线程，不回头通知 CPU「播完了」。

Composition 样本（`CompositionHost.cpp`）：HWND 只当 target；动的是 Visual 的 Offset/Size，不改窗口客户区。Yohu 占用卡已经是这条：HWND/交换链铺满 avail，可见卡片是 `IDCompositionRectangleClip`。

scrcpy `app/src/screen.c` `sc_screen_render`：

```
SetRenderDrawColor(bg)
SDL_RenderClear
若有纹理 → 按 content rect 画
SDL_RenderPresent
```

没有纹理也 Clear+Present。窗口事件、尺寸、方向变化都再走完整一帧。**谁拥有窗口，谁每帧清+画+呈。** 没有「dirty 画一次就停」。

libmpv README：

| 路 | 能叠自己的 OSD/铬 | 窗口谁画 |
|----|-------------------|----------|
| `wid` 嵌入 | 否 | 播放器子窗铺满槽，letterbox 在原生侧 |
| Render API | 是 | 宿主 GL 上下文每帧 `mpv_render_context_render`，再画自己的层 |

Yohu 已选嵌入 HWND + 自己画铬，等价「嵌入槽 + 自己当 Render 宿主」：空态/加载/暂停时 **壳必须继续当回缓冲主人**。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 回缓冲主人每拍 Present | reuse-pattern | 空态不是 dirty overlay；`shows_chrome()` 为真就持续画 |
| clip 是合成器属性 | reuse-pattern | 禁止用 CPU 描边去追正在跑的 clip；动画期可跳过 hairline，结束后必须再画 |
| 用 dirty 一次画完空态 | anti-pattern | 停止后 contain→fill 动画 300ms 内 Present 一次、清 dirty，边框永远不再来 |
| 用改色值掩盖所有权 | anti-pattern | 色板对了，停完仍是白板 |

## 架构设计经验

交换链上同时存在「视频 VP」和「D2D 铬」时，**模式决定主人**：

```
Empty / Loading / Paused → chrome Present 每拍
Video                     → 帧 Present 每拍
```

clip 动画只改可见窗口，不改主人。`present_with_hairline` 在 `clip_animating` 时跳过描边是对的（避免和 DWM 抢边）；错的是把「这一拍没画上描边」当成「铬已经交付」。

## 与当前工作

- 能直接用：`Stage::chrome_draw` 在 `shows_chrome()` 期间一直给出规格，不要 `take_chrome_dirty`。
- 必须改写：Windows `present_chrome` 仍可在动画期跳过 hairline，但下一拍（动画结束后）必须还能走进同一条路径。
- 不要用：再叠一套 CSS 空态、再改一遍 ARGB 当修复。

## 阅读范围

- Learn：`directcomp/clipping`、`IDCompositionVisual::SetBorderMode`
- Temp：`microsoft--Windows.UI.Composition-Win32-Samples` 的 `CompositionHost.cpp`
- Temp：`Genymobile--scrcpy` 的 `app/src/screen.c` `sc_screen_render`
- Temp：`mpv-player--mpv-examples` 的 `libmpv/README.md`（嵌入 vs Render）
- 未整仓克隆 mpv / Win2D / QtAV
