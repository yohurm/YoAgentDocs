---
id: research.harmony-bindsheet-full
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [ArkTS, C++]
  frameworks: [harmonyos, arkui]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  repo: HarmonyOS bindSheet / SheetOptions
  url: https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-universal-attributes-sheet-transition
studied_at: 2026-08-24
related:
  - research.harmony-hdstabs-motion
---

# HarmonyOS bindSheet 全量（主题笔记）

对照本地 `半模态转场.md`（2026-07-28）、设计《半模态面板》、FAQ「如何在 bindSheet 中实现页面切换」。开源 ArkUI `SheetPresentationPattern` 管手势与档位；HDS 半模态标题栏是 `HdsNavigationTitleMode.MODAL`，不是另一套 Sheet 引擎。

## 入选理由

YoSheet 要对齐 `SheetOptions` 契约与设计里的两种页面跳转，而不是再接一套 Android BottomSheet。

## 形态与约束

| preferType | 高度 | detents / 控制条 | 关闭手势 |
|------------|------|-------------------|----------|
| BOTTOM | LARGE / MEDIUM / FIT_CONTENT / Length | 多档才显示 dragBar | 下滑 |
| CENTER | 默认 560vp，min 320，max 短边 90% | 无档 | 无滑关 |
| POPUP | 同 CENTER | 无档；不下滑关闭 | 点外（默认可交互） |
| SIDE | **始终全屏高** | **无档、无控制条** | 侧滑（镜像相反） |
| CONTENT_COVER | 全屏 | 无档、无铬 | `modalTransition` |

SIDE 不支持：height、detents、dragBar、scrollSizeMode、mode/uiContext 换层、hover、floatingDragBar。CONTENT_COVER 不支持铬、外层可交互、自定义宽。`mode` 显示期间不可改。OVERLAY 默认。

`enableOutsideInteractive` 未设时：底/中/侧拦截，POPUP 放行。为 true 时无蒙层，`maskColor` 无效。

## 同面板 vs 多层面板

设计：

- **同面板：** 面板不动，内容自右往左；标题栏左侧是层级返回；系统返回逐级 pop；**关闭按钮关掉整棵面板**。
- **多层面板：** 新 sheet 从底升起；返回先关上层。

FAQ 用 Stack + `translateX = 面板宽` + EaseOut 800ms 是应用侧示意，不是系统 Navigation。YoSheet 应走自研 ContentSwitch SLIDE，并在无自定义 leading 且 depth>1 时出示层级返回。

## API 26+ 仍非 Android 目标

`showInSubWindow`、`enableHoverMode` / `hoverModeArea`、`radiusRenderStrategy`、`uiContext`：N/A。`systemMaterial` 映射已有 BackdropMode，不另造材质 API。

## 与 YoSheet

**已有：** 档位/height 互斥、preferType、blur、mask、title、outsideInteractive 默认、willDismiss / springBack、宽高/类型回调、边框、mode、scrollSizeMode、键盘、圆角、placement、effectEdge、floatingDragBar、modalTransition、push/pop SLIDE、硬件返回先 pop。

**要补：** 同面板层级返回（不覆盖用户 leading）；SIDE 编译为单档全高并关掉控制条；显式多档 / height / 可见控制条对 SIDE 拒绝。

**不要抄：** FAQ 800ms 自写 translate；用第二个 YoSheet 冒充同面板 push。
