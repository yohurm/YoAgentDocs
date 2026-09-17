---
id: research.desktop-dialog-used-height
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, CSS]
  frameworks: [SolidJS, fluentui, radix]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: mixed
  repos:
    - microsoft/fluentui
    - radix-ui/primitives
  url: https://github.com/microsoft/fluentui
  cloned_to:
    - "%TEMP%/YoAgentResearch/microsoft--fluentui"
    - "%TEMP%/YoAgentResearch/radix-ui--primitives"
  official:
    - "HarmonyOS 弹出框.md（本地 yovo-harmonyos-docs）"
    - "openharmony arkui_ace_engine CloseDialogAnimation"
studied_at: 2026-09-16
related:
  - research.harmony-fluent-inline-end-clip
  - research.microsoft-fluentui
  - research.radix-ui-primitives
  - research.openharmony-arkui-ace-engine-dialog-spatial
  - research.harmony-motion-system
---

# 桌面 Dialog 用后高：命令式行程 × 关窗锁盒

## 入选理由

Yohu 删除确认是 fit Dialog：预览 Chip + `YoReveal` 其余名单。用户看到两件事：

1. 展开/收起时**窗口高度变化动画没了**。
2. 从未展开就点取消，**内容区仍折叠一下**。

需要对照谁把「内容变高」做成可插值尺寸，以及官方弹出框关闭时改不改内容布局。

## 读了什么

| 源 | 路径 | 结论 |
|----|------|------|
| Fluent Collapse size atom | `%TEMP%/YoAgentResearch/microsoft--fluentui/packages/react-components/react-motion-components-preview/library/src/components/Collapse/collapse-atoms.ts` | `scrollHeight` **量一次** → `maxHeight` 关键帧 `out → measured → unset`；行程中 `overflow: hidden`；**没有 ResizeObserver** |
| Radix Collapsible | `%TEMP%/YoAgentResearch/radix-ui--primitives/packages/react/collapsible/src/collapsible.tsx` | 关之前 `useLayoutEffect` 先挡住 transition 再 `getBoundingClientRect`，写成 `--radix-collapsible-content-height`；**Presence 延迟卸节点**；量高发生在 `context.open` 变 false 时，不是卸完之后 |
| 鸿蒙弹出框 | `Learn/yovo-harmonyos-docs/设计/设计指南/控件/容器类/弹出框.md` L121 / L154 | 电脑最大高 = `0.9 * 窗口内容层`；居中。内容区必选。关闭不是内容重排 |
| ArkUI CloseDialogAnimation | YoAgentDocs `openharmony--arkui_ace_engine-dialog-spatial.md` | 关闭 = 最后一盒的 **opacity + scale**，`FillMode FORWARDS`，不回放、不改子树固有高 |

## 官方 / 开源怎么做

**尺寸动画是命令，不是观察。** Fluent 在打开那一拍读 `scrollHeight`，关键帧写死 `toSize`。Radix 在开/关意图上量一次，把 px 交给 CSS 变量。两边都禁止「盒自己在插值时再触发重测」。

**关窗锁最后一盒。** 鸿蒙 / ArkUI 关闭走整盒淡出缩放。Radix 关 Collapsible 时先量再让 Presence 播完。没有人在 Presence 还在出场时把 height 清掉，也没有人把内容区临时改成 `flex: 1 1 0` 去吃剩余高。

**量高不要带变换。** Presence 入场 `scale` 时 `getBoundingClientRect` 是视觉盒（偏小）。Fluent 用 `scrollHeight`；布局锁应用 `offsetHeight`（不含 transform）。

## Yohu 当时的通路（审查证据）

```
删除确认 open
  → YoPresence recipe=dialog（scale-in）
  → YoDialog fit + bindHugTravel（isOpen 寿命）
      ResizeObserver 盯 .yohu-dialog__main / .yohu-reveal
      行程中 data-travel → 主槽被 flex 拉高 → RO dirty → finish 重测 → 行程被吃掉
  → 取消：setDeleteOpen(false)（名单/展开不在这一拍清）
      isOpen false → onCleanup stop() → write({}) 清 height
      data-box=exit → body/main flex 1 1 0
      Reveal 仍关（出流、内容绝对定位）→ main 固有高只剩预览
      锁高没了 + fill-flex → 内容区折一下
  → Presence scale-out 还在播
```

反模式：用 RO 盯正在插值的盒；用 `isOpen` 当 hug 寿命；关窗 `write({})`；exit 复用 fill 的剩余高契约；`getBoundingClientRect` 当出场锁。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 量一次、关键帧写死 px | reuse-pattern | hug-travel `from/to` 只在意图上算 |
| 意图 = 属性/子树，不是尺寸 | adapt | `MutationObserver`：`data-open` / `data-layout` / `childList` |
| 关窗锁最后一盒 | reuse-pattern | hug 冻到 Presence 卸节点；exit 不再 fill-flex |
| 布局高 ≠ 视觉高 | adapt | `offsetHeight`，禁止 rect 当目标 |
| RO 盯插值盒 | anti-pattern | 反馈环，行程消失 |
| 出场清 height / 内容重排 | anti-pattern | 未展开点取消也会折 |

## 不要用

- 引进 Fluent react-motion / Radix Collapsible 包。
- 再给 Dialog.css 叠第三条高度引擎。
- 把 Collapse `0fr/1fr` 请回 fit Dialog。
