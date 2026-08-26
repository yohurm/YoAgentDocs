---
id: research.microsoft-fluentui
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React, WAAPI]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: microsoft/fluentui
  url: https://github.com/microsoft/fluentui
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--fluentui"
studied_at: 2026-08-20
related: [research.radix-ui-primitives, research.synthesis.ui-kit]
---

# microsoft/fluentui（react-motion 切片）

## 入选理由

Yohu 已有 Presence / Collapse，缺的是「内容换牌」分层。Fluent 把 **atom（fade）→ Presence 工厂 → Fade 配方 → 产品组件** 拆开，且 Fade 的 enter/exit **默认同一时长**。这正好对照我们把换牌逻辑堆进 `YoButton`、还用 160ms 入 / 200ms 出的失败尝试。

## 项目是什么

Fluent UI v9 的动效包：`@fluentui/react-motion`（工厂与 PresenceGroup）+ `@fluentui/react-motion-components-preview`（Fade / Scale / Collapse 等配方）。产品控件（Dialog、Menu）只引用配方，不写关键帧。

## 架构

```
motionTokens（时长/曲线数字）
  → fadeAtom（只改 opacity）
    → createPresenceComponent(fadePresenceFn)
      → Fade / FadeSnappy
  → PresenceGroup（按 child key 合并映射：进入 appear、离开 visible=false，exit 完再删 key）
```

- `Fade`：`exitDuration` **默认等于** `duration`（对称交叉）。Snappy 变体用 `durationFast`（150ms）。
- `fadeAtom`：opacity 关键帧 + `fill: both`；默认曲线 linear（透明度不是空间运动）。
- `PresenceGroup`：不管关键帧，只管家「哪些 key 还挂在树上」。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| atom / 配方 / 控件三层 | reuse-pattern | 换牌原语进 `motion/`，`YoButton` 只消费 |
| Fade enter=exit | adapt | 同类文案交叉淡入淡出必须同时、同时长 |
| PresenceGroup keyed 映射 | adapt | `YoSwap` 按 `keys` 保留 outgoing 直到出场结束 |
| WAAPI 作为唯一引擎 | anti-pattern | 我们 CSS-first；不引入 react-motion |
| durationNormal=200 当默认 Fade | lesson-only | 数值走鸿蒙：控件内简单透明度用 100ms |

## 架构设计经验

- 透明度是 **effects atom**，不要和 width 绑在同一个 JS 测量循环里。
- 列表/换牌的「还在不在树上」与「怎么动」必须拆开（Group vs Fade）。
- 产品组件禁止内嵌一套自己的 Presence 状态机。

## 与当前工作

- 能直接用：对称 Fade、keyed 残留 outgoing、Button 变薄。
- 必须改写：Solid + CSS `@keyframes`，不用 WAAPI 工厂。
- 不要用：Fluent 16 档时长、把 Fade 默认 200ms 抄进工具栏按钮。

## 阅读范围

`packages/react-components/react-motion/library/src/{index,motions/motionTokens,factories,components/PresenceGroup,utils/groups}`；`react-motion-components-preview/library/src/{atoms/fade-atom,components/Fade}`。未读整个 Fluent 组件库。

## 表头增补（2026-08-20）

同仓 `react-table`：`TableHeaderCell` 拆 `root`（轨道 / columnheader）+ `button`（排序）+ `sortIcon`（仅 `sortDirection` 有值）+ `aside`（resize 兄弟）。hover 画在 **root 矩形底**上，因此 button 可以 `width:100%`。Yohu 排序用圆角 `.yohu-interactive`，不能抄满宽按钮，否则高亮片会冒充列区域。`aside` 与 AG Grid `eResize` 一样是兄弟，不进标题。
