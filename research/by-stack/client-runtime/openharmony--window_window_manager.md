---
id: research.openharmony-window_window_manager
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [openharmony, rosen, arkui]
also_relevant: [ui-kit]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: openharmony/window_window_manager
  url: https://github.com/openharmony/window_window_manager
  head: 9c8cabb
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--window_window_manager"
studied_at: 2026-09-08
related:
  - research.harmony-motion-system
  - research.microsoft-Windows.UI.Composition-Win32-Samples
---

# openharmony/window_window_manager

## 入选理由

唯一能在开源里看到「系统启动页 / 窗口进出场」怎么接到合成器上的仓。回答：启动动画动的是哪块表面、主窗布局何时到位、默认 Scale/Opacity 参数。

## 项目是什么

OpenHarmony 窗口管理：WMS + window_scene。应用窗口、starting window、键盘、动画配置。渲染走 Rosen（`RSTransitionEffect` / `RSNode::Animate`），WMS 不自己画帧。

## 架构

```
配置（AnimationConfig）
  → WindowController::UpdateWindowAnimation 给 surface 挂 RSTransitionEffect
  → StartingWindow 在 leash 上挂 startingWinSurfaceNode_
  → 应用 surface 首帧回调 → RSNode::Animate(alpha) → RemoveChild
```

窗口默认进出场（`animation_config.h`）：

- 时长协议默认 **200ms**，曲线 `EASE_OUT`
- `scale_ { 0.7, 0.7, 1 }`，`opacity_ = 0`
- `UpdateWindowAnimation`：`RSTransitionEffect::Create()->Scale()->Rotate()->Translate()->Opacity()` 设到 `leashWinSurfaceNode_` 与 `surfaceNode_`

启动页（`starting_window.cpp` `SetStartingWindowAnimation`）：

- `SetAlpha(opacityStart_)` 然后 `Animate` 到 `opacityEnd_`（配置默认 1→0）
- **窗口矩形不变**；finish 时从 leash 摘掉 starting 节点
- 可配置关掉（`transAnimateEnable_`），则首帧直接 RemoveChild

测试钉死 starting 配置默认：`enabled_ false` 时仍带 `duration_ 200`、`opacityStart_ 1`、`opacityEnd_ 0`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 启动面 ≠ 窗口布局 | reuse-pattern | 主窗一次放到最终尺寸；快照/启动面在合成器上淡出或 morph |
| Scale+Opacity 在 surface | reuse-pattern | Windows 对应 DComp Visual / 分层 HWND 的 transform，不是 `SetWindowPos` 尺寸 |
| 等首帧再卸启动面 | adapt | Yohu 等 workbench hydrate，不是等进程启动时刻 |
| 默认 0.7 scale 窗口出现 | lesson-only | 那是应用窗口从桌面起来；Yohu 同屏要用共享容器（小→大），不要 0.7 弹主窗 |
| 每帧改 WindowRect | anti-pattern | 本仓不这么做 |

## 架构设计经验

窗口管理器把 **layout** 和 **presence animation** 拆开。Layout 写最终几何；动画只改合成树属性。启动页是 leash 上的临时 child，不是「小 HWND 长大」。

## 与当前工作

Yohu `native_splash` 的 GDI 小窗可以留作 **starting surface 的绘制源**。交接必须改成：主窗最终矩形一次到位 → 快照 visual 做共享容器或淡出 → 卸启动面。禁止 `animate_hwnd` 插值 RECT。

## 阅读范围

`wmserver/include/animation_config.h`；`wmserver/src/window_controller.cpp` `UpdateWindowAnimation`；`wmserver/src/starting_window.cpp` `SetStartingWindowAnimation` / `HandleClientWindowCreate`；`wmserver/src/window_manager_service.cpp` `ConfigStartingWindowAnimation`；对应 unittest 断言。未读输入法动画全路径、多屏虚拟屏。
