---
id: research.mpv-player-mpv-examples
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++, Qt]
  frameworks: [libmpv, Win32 HWND embed]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: mpv-player/mpv-examples
  url: https://github.com/mpv-player/mpv-examples
  head: e0d1a84
  cloned_to: "%TEMP%/YoAgentResearch/mpv-player--mpv-examples"
studied_at: 2026-09-11
related:
  - research.synthesis.client-runtime
  - research.Genymobile-scrcpy
---

# mpv-player/mpv-examples

## 入选理由

桌面里「把原生视频窗嵌进宿主槽」最清楚的小样本。libmpv README 把 **窗口嵌入** 和 **Render/合成 API** 分成两条路，正好对照 Yohu「HWND 独占像素、不把帧送进 JS」。浅克隆 HEAD `e0d1a84`（2024-06-07）。

## 项目是什么

libmpv 客户端示例。本轮读 `libmpv/README.md` 与 `libmpv/qt/qtexample.cpp`。

## 架构

窗口嵌入：

```
宿主创建原生槽（Qt：WA_NativeWindow 的 QWidget）
  → winId() 作为 wid
  → libmpv 再建子视频窗，父=槽 HWND
  → 子窗铺满槽，比例不对就 letterbox
```

槽是 **工具窗口件**，不是 DOM。藏槽 / 拆主窗，子视频窗跟着走。页面 CSS 不参与。

Render API：在宿主 GL 上下文里画，才能在视频上叠自己的 OSD。官方写：嵌入简单但 OS 相关；要叠层用 Render。

Yohu 已选「壳内 HWND + DComp clip」，等价嵌入 + 自己画 chrome，不搬 libmpv。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 宿主拥有槽 HWND，解码器只往里画 | reuse-pattern | PresentHost 拥有舞台；`yohu-mirror` 只给 FramePipe |
| 嵌入窗铺满槽，letterbox 在原生侧 | reuse-pattern | 与 avail 铺满 + contain clip 同构 |
| 用网页 opacity 藏嵌入视频窗 | anti-pattern | 子 HWND 不吃 CSS |
| 把 FFmpeg/libmpv 拉进 core | anti-pattern | ADR-v6-024 禁 FFmpeg |

## 架构设计经验

嵌入表面的生命周期 = **槽还在不在**。槽的开关必须和「这个面板是不是当前页」同一拍，不能等淡出结束，更不能让淡出中的 View 继续报 `visible=true`。

## 与当前工作

- 能直接用：工作台在 `activeModuleId` 离开 `screen-mirror` 的同一拍关掉槽。
- 不要用：mpv、FFmpeg、独立播放器窗（QtScrcpy 已是否决）。

## 阅读范围

`libmpv/README.md`；`libmpv/qt/qtexample.cpp`。未读 cocoa / qml / sdl。
