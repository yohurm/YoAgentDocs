---
id: research.HarmonyOS_Samples-Spatialization
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [ArkTS]
  frameworks: [harmonyos, hds]
also_relevant: []
utilization: [reuse-pattern, lesson-only]
source:
  platform: gitcode
  repo: HarmonyOS_Samples/Spatialization
  url: https://gitcode.com/HarmonyOS_Samples/Spatialization
  cloned_to: "%TEMP%/YoAgentResearch/HarmonyOS_Samples--Spatialization"
studied_at: 2026-08-20
related:
  - research.harmony-hdstabs-motion
---

# HarmonyOS_Samples/Spatialization（HdsTabs 滚动隐藏接线）

## 入选理由

官方空间化案例是公开代码里**唯一直接调用** `HdsTabsController.applyHideAnimation` / `applyShowAnimation` 的完整页。用来确认 SCROLL 模式的触发语义，不是曲线。

## 项目是什么

HarmonyOS 沉浸光感 / 空间化最佳实践。`ImmersiveLightView` 用 `HdsNavDestination` 包 `HdsTabs`，滚动回调决定底栏显隐。

## 架构

```text
WaterFlow onDidScroll yOffset
  → handleTabBarAnimation
       yOffset>0 且当前未向上 → applyHideAnimation(SCROLL_ANIMATION)
       yOffset<0 且当前向上   → applyShowAnimation(SCROLL_ANIMATION)
  大屏 needDynamicHideBar=false 则整段跳过
```

应用不写 translate/opacity/spring。动效在闭源 HDS 控制器。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 方向边沿触发，不是每帧跟手 | reuse-pattern | 与 YoTabs `TabsScrollBinder` dy 阈值布尔意图同类 |
| 隐藏能力可关 | reuse-pattern | `HidePolicy.NEVER` / 非 FLOATING 忽略 |
| 在应用里 animateTo 藏栏 | anti-pattern | 必须走组件控制器 |

## 架构设计经验

SCROLL 模式对外是「滚动意图」，对内才是弹簧。样本没有 CLICK_ANIMATION。不要从 Demo 反推 200ms EaseInOut。

## 与当前工作

YoTabs 保持 `HidePolicy.FOLLOW_SCROLL` → `setCollapsed`；把 **settle 曲线**改成 ArkUI 的 228/30。真 1:1 跟手位移留 P1。

## 阅读范围

`products/entry/src/main/ets/view/ImmersiveLightView.ets`。未反编译 HDS 控制器。
