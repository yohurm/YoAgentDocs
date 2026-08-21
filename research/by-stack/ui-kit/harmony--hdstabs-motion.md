---
id: research.harmony-hdstabs-motion
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [cpp, arkts]
  frameworks: [harmonyos, arkui, hds]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: other
  repo: HarmonyOS HdsTabs motion
  url: https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ui-design-hdstabs
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.openharmony-arkui_ace_engine-tabs-motion
  - research.harmonyos-samples-spatialization
  - research.harmony-hdstabs-point-light
---

# HarmonyOS HdsTabs 动效（主题笔记）

HDS / UIDesignKit **无公开实现仓**。动效算法不在 Demo 里：样本只调 `HdsTabsController.applyHideAnimation` / `applyShowAnimation`。开源等价物是 ArkUI `TabBarPattern` + `ControlInteractionBase` + Symbol `BounceSymbolEffect`。不要假装读过 HDS 私有仓。

对照：

| 路径 | 角色 |
|------|------|
| `%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine` | 切页时长、栏显隐弹簧、选中 bounce 触发、按压形变 |
| `%TEMP%/YoAgentResearch/HarmonyOS_Samples--Spatialization` | 滚动方向 → `SCROLL_ANIMATION` 接线，无曲线 |
| `%TEMP%/YoAgentResearch/HarmonyOS_Samples--HarmonyOSComponentUXExamples` | BottomTabBarStyle 样本；切页时长 0/300 |
| `E:\Dev\Doc\HarmonyOS-Developer-docs` | 《底部页签》、HdsTabs API、动效属性、SymbolGlyph |

## 入选理由

回答 YoTabs 动效六问：切页要不要滑、图标怎么弹、栏怎么藏、跟手还是点击、按压形变与光是否抢变换、缺哪条引擎能力。

## 1. 自底而上：五条互不抢权的通道

```text
L0  曲线 / 时长 token
L1  共享配方（spring 228/30、BounceSymbolEffect、Immersive interactive）
      ↓
YoTabs 只接线，不自写第二套插值
```

| 通道 | 官方所有者 | 触发 | 属性 |
|------|------------|------|------|
| 切页 | `animationDuration` → Swiper | 点 Tab / `changeIndex` | 内容位移。BottomTabBarStyle **默认 0** |
| 选中告知 | `BounceSymbolEffect` DOWN | 选中 id 变化 | 图标弹跳（Symbol 渲染器） |
| 栏显隐 | `applyHide/ShowAnimation` | 滚动 / 点击 | translateY + opacity。曲线 228/30 |
| 光感形变 | `interactive` + `lightEffect` | 按住栏 | 胶囊 mesh/scale/offset + 点光源。弹簧 228/16 |
| MiniBar 折叠 | `applyMiniBarStyle` | 迷你栏 | 宽度/圆形互折。YoTabs **不做** |

设计《底部页签》：选中要「细微点击弹跳」；悬浮禁止横滑切页；跟手隐藏只属于悬浮栏。

## 2. 切页时长：底部页签默认关

HdsTabs `animationDuration`：存在 BottomTabBarStyle 时默认 **0ms**；非底部样式默认 300ms。ArkUI `TabBarPattern::UpdateAnimationDuration` 同源：API 11+ 且 styles 含 `BOTTOMTABBATSTYLE` → duration = 0。

性能文档：未设 BottomTabBarStyle 时默认 300ms，拉长会拖完成时延。

**YoTabs 是应用级底部页签 → 切页保持 GONE/VISIBLE，不要接 ContentTransition / Pager 滑页。** FLOATING 官方本来就不让横滑切页。

子页签下划线（`SubTabBarStyle` + indicator）是另一组件，不要塞进 YoTabs。

## 3. 选中弹跳：DOWN 位移，不是整项 scale

设计推荐 `BounceSymbolEffect(EffectDirection.DOWN)`。API 默认 `direction=DOWN`、`scope=LAYER`。ArkUI 在 `HandleBottomTabBarChange` → `UpdateSymbolStats` 里对**新选中** Symbol `SetIsTxtActive(true)`。首次 bind / 同 id remap 不弹（`preIndex == index` 早退；`preIndex == -1` 不激活 effect）。

真正的弹跳关键帧在闭源 HarmonyOS Symbol 渲染器，ace_engine 只接线。Android 无 Symbol：用 **translationY 向下弹再回基线** 近似，不要 `YoBounce.bounceScale`（那是强调缩放，且会与栏 L5 press warp 抢 scale）。

颜色 / 图标资源：ArkUI `UpdateSymbolStats` **瞬时**换色。200ms mask 只给 SVG `CheckSvg` 路径，YoTabs 位图不要抄。

运营位出血 4vp 是基线位移；bounce 必须叠在出血上，不能把 translationY 弹回 0。

## 4. 栏显隐：弹簧 228/30 + 透明度耦合

开源 ArkUI（HDS 闭源的最近等价）：

| 常量 | 值 |
|------|----|
| `TRANSLATE_CURVE` | `InterpolatingSpring(0, 1, 228, 30)` |
| `TRANSLATE_THRESHOLD` | 26vp（跟手位移跨过才 commit 弹簧） |
| opacity | `1 - \|ty\| / barHeight` |
| 跟手 | `UpdateTabBarHiddenOffset` 1:1 改 ty，不跑动画 |
| 离手/程序化 | `StartHideTabBar` / `StartShowTabBarImmediately` 同一条弹簧，可打断 |

Hds `HdsAnimationMode`：`SCROLL_ANIMATION=0`、`CLICK_ANIMATION=1`。样本 Spatialization 只在 `yOffset` 变号时调 SCROLL。CLICK 应对齐「无跟手、直接弹簧」。HDS 闭源，两种 mode 的时长差未见公开。

`DurationCubicCurve(0.2, 0, 0.1, 1)` 是 **指示器 / 按压 mask**，不是栏显隐。

## 5. 按压形变与光：已有 L5/L6，禁止再给项加 scale

`ControlInteractionBase`：按下 spring 228/16、时长 1000ms；移动直接写 scale/offset；松手同一弹簧回 1/0。灯 z=80vp、intensity=3。见 [harmony--hdstabs-point-light.md](./harmony--hdstabs-point-light.md)。

YoTabs：FLOATING 胶囊走 `YoImmersiveLight` L5/L6；项只转发 pointer。选中弹跳必须只打图标 translationY。

## 6. 对 YoTabs / 动画系统

**已有、能直接用：**

- `MotionTokens.listItemSwipeSpring()` ≡ 228/30（`SpringToken.LIST_SWIPE`）
- `ComponentStateMotion`：跟手 `follow` + 离手 `settle`，可打断
- `YoImmersiveLight` L5/L6（按压 + 触点光）
- 切页瞬时（GONE/VISIBLE）——与官方默认 0ms 对齐
- `YoMotion` 关键帧配方（给 bounce DOWN 近似）

**要改写：**

- 栏显隐：`YoTranslate` + `DurationToken.LOCAL` 贝塞尔 → `listItemSwipeSpring` + opacity
- 图标：`YoBounce.bounceScale` → `bounceDown`（translationY，基线含出血）

**本轮不做 / 引擎缺口：**

- MiniBar 折叠、握持左右搬家、侧栏 ≥840、Symbol 分层 LAYER bounce（闭源）
- 真跟手 `UpdateTabBarHiddenOffset`（ScrollBinder 仍是 dy 阈值布尔意图；跟手是 P1）
- 对外暴露 `applyHideAnimation(mode)`（现有 `HidePolicy.FOLLOW_SCROLL` 够用）
- 切页 300ms 滑页、子页签 indicator

## 阅读范围

实际读过：HdsTabs.md（animationDuration / applyHide/Show / HdsAnimationMode）；《底部页签》视觉反馈与跟手隐藏；《动效属性》时长表；SymbolGlyph BounceSymbolEffect；ace_engine `tab_bar_pattern.cpp/.h`、`tab_content_model_ng.cpp` UpdateSymbolEffect、`component_material_interaction.cpp`；Spatialization `ImmersiveLightView.ets`。

未读：HdsTabs HSP 里 SCROLL vs CLICK 的时长差、Symbol 渲染器 bounce 振幅。
