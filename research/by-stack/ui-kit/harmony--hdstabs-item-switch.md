---
id: research.harmony-hdstabs-item-switch
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++, ArkTS]
  frameworks: [harmonyos, arkui, hds]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  repo: HarmonyOS HdsTabs item switch
  url: https://developer.huawei.com/consumer/cn/doc/design-guides/bottomtab-0000001956787789
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-24
related:
  - research.harmony-hdstabs-motion
  - research.openharmony-arkui_ace_engine-tabs-motion
---

# HarmonyOS HdsTabs 导航项切换（主题笔记）

本篇只回答「点页签时项上发生什么」，不重复栏显隐 / 点光源。HDS 无公开仓；可执行定义在 ArkUI `TabBarPattern::HandleBottomTabBarChange` 与 `SymbolGlyph` `BounceSymbolEffect`。

## 入选理由

YoTabs 要对齐底部页签的选中告知。社区自定义 Tab 下划线、`animationDuration` 滑页、整项 scale 都不是这条通道。

## 官方通道（项内）

| 通道 | 所有者 | 触发 | 属性 |
|------|--------|------|------|
| 选中 bounce | `BounceSymbolEffect(DOWN)`，默认 `scope=LAYER` | 选中 id **真正变化** | 图标 translationY 下弹再回基线 |
| 色 / 资源 | `UpdateSymbolStats` / `UpdateImageColor` | 同上 | **瞬时**换色、换图 |
| 切页 | `animationDuration` | 点项 / `changeIndex` | BottomTabBarStyle **默认 0** |

`preIndex == index` 早退；`preIndex == -1`（首次 bind）不激活 effect。再点当前项不弹，走回顶。

闭源 Symbol 渲染器没有公开振幅。Android 用 token 近似 DOWN 位移，禁止 `bounceScale`（与栏 L5 press warp 抢 scale）。

运营位出血 4vp 是基线位移；bounce 必须叠在出血上，不能弹回 0。

切页与项 bounce 不是一条动画。底部导航关掉滑页，把反馈放到图标。

## 栏显隐（相邻通道，须同一配方）

`StartHide/ShowTabBar`：`InterpolatingSpring(0, 1, 228, 30)` + `opacity = 1 - |ty| / barHeight`。对象是 TabBar 节点 translate，VEIL/divider 跟同一 `translationY`。禁止只用淡出冒充藏栏。跟手是 P1；程序化显隐必须可打断。

`DurationCubicCurve(0.2, 0, 0.1, 1)` 是指示器 / 按压 mask，不是栏显隐。

## 与 YoTabs

**直接用：** `YoBounce.bounceDown`、切页 GONE/VISIBLE、`MotionTokens.listItemSwipeSpring()` ≡ 228/30。

**要对齐：** bind 换项时先 `YoMotion.cancel` 再写基线，否则上一帧 bounce 会盖住未选项；栏显隐从 `YoTranslate` + LOCAL 贝塞尔改成弹簧 + 耦合 alpha。

**不做：** MiniBar、Symbol LAYER 分层、切页 300ms、子页签 indicator。
