---
id: research.openharmony-arkui_ace_engine-tabs-motion
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++]
  frameworks: [arkui]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: gitcode
  repo: openharmony/arkui_ace_engine
  url: https://gitcode.com/openharmony/arkui_ace_engine
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.harmony-hdstabs-motion
  - research.openharmony-arkui_ace_engine-immersive
---

# openharmony/arkui_ace_engine（Tabs 动效）

## 入选理由

HdsTabs 闭源。底部页签动效的可执行定义在 ArkUI `TabBarPattern`：切页默认时长、栏 translate 弹簧、选中 Symbol bounce 触发点、SVG mask 旁路。

## 项目是什么

OpenHarmony 方舟 UI。本篇只读 **Tabs / TabBar / TabContent / ControlInteractionBase** 动效，不重复沉浸光感那篇。

## 架构

```text
TabsController / 点击
  → TabBarPattern.HandleClick / ChangeIndex
       ├─ UpdateAnimationDuration → Swiper duration（底部样式 = 0）
       ├─ HandleBottomTabBarChange
       │    ├─ UpdateImageColor / UpdateSymbolStats（色瞬时）
       │    └─ UpdateSymbolEffect isActive（bounce）
       └─ StartHide/ShowTabBar（TRANSLATE_CURVE 228/30 + opacity）
ControlInteractionBase（材质 interactive，与 TabBar 平行）
```

关键常量（`tab_bar_pattern.cpp`）：

- `TRANSLATE_CURVE = InterpolatingSpring(0, 1, 228, 30)`
- `TabBarPhysicalCurve = InterpolatingSpring(-1, 1, 228, 30)`（指示器物理）
- `DurationCubicCurve = Cubic(0.2, 0, 0.1, 1)`（指示器时长曲线，非栏显隐）
- `MASK_ANIMATION_DURATION = 200`（仅 SVG）
- `TRANSLATE_THRESHOLD = 26vp`

`UpdateAnimationDuration`：styles 含 `BOTTOMTABBATSTYLE` 且 API≥11 → 0。

`UpdateSymbolStats`：新选中且 `preIndex != -1` 才 `UpdateSymbolEffect(..., true)`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 底部切页 duration=0 | reuse-pattern | YoTabs 不要滑页 |
| 栏位移 | reuse-pattern | `SetTabBarTranslate` 整节点，不是光感 |
| 栏透明度 | anti-pattern | `SetTabBarOpacity` 是独立 API；本轮不用淡出冒充藏栏 |
| 跟手 offset 再弹簧 | adapt | 像素累加，过 26vp 交 228/30 |
| Symbol bounce 触发条件 | reuse-pattern | 仅选中 id 变化 |
| SVG 200ms mask | anti-pattern | 位图页签不要抄 |
| 整项/整栏 scale | anti-pattern | 与 interactive 抢变换 |

## 架构设计经验

栏位移和内容切页不是同一条动画。底部导航把切页关掉，把反馈放到图标 bounce 和材质按压。显隐是物理弹簧，不是 `LOCAL` 贝塞尔。

## 与当前工作

**藏栏对象是 TabBar 宿主节点的 transform translate**（`SetTabBarTranslate` → `UpdateTransformTranslate`），并同步平移 divider。`SetTabBarOpacity` 是另一条公开 API。沉浸光感 / interactive / 点光源不在这条链上。

YoUI 必须平移 `barSlot`（胶囊铬层整棵子树），VEIL 当官方 divider 一样跟同一 `translationY`。禁止用 alpha / rim / liftSample 冒充藏栏。

能直接映射到 `ComponentStateMotion` 像素跟手 + `listItemSwipeSpring`。Bounce 振幅无开源数，Android 用 token 近似 DOWN 位移。

## 阅读范围

`frameworks/core/components_ng/pattern/tabs/tab_bar_pattern.{h,cpp}`（StartShow/Hide、UpdateTabBarHiddenOffset、HandleBottomTabBarChange、UpdateAnimationDuration、UpdateSymbolStats）；`tab_content_model_ng.cpp` UpdateSymbolEffect；`component_material_interaction.cpp` 按压弹簧。未读 Swiper 翻页内部插值（底部路径 duration=0 用不到）。
