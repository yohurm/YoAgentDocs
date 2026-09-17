---
id: research.desktop-commanded-travel
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
    - "HarmonyOS 动效属性 / 转场动效（本地 yovo-harmonyos-docs）"
    - "HarmonyOS 滚动条.md"
studied_at: 2026-09-16
related:
  - research.desktop-dialog-used-height
  - research.microsoft-fluentui
  - research.radix-ui-primitives
  - research.harmony-motion-system
  - research.harmony-fluent-inline-end-clip
---

# 桌面尺寸行程：命令式、同拍起程、横轴同构

## 入选理由

Yohu 删除确认展开/收回：点下去先停一拍，名单左右（侧轨夺 8vp、展开钮换字）再硬切。需要对照谁在**同一布局回合**起程，以及横轴怎么跟竖轴一起走。

## 读了什么

| 源 | 路径 | 结论 |
|----|------|------|
| Fluent Collapse size atom | `microsoft--fluentui/.../Collapse/collapse-atoms.ts` | 开/关那一拍读 `scrollHeight` / `scrollWidth`，关键帧写死 `toSize`；横竖同一套 atom；**没有 hold 帧、没有 rAF 再写 to** |
| Fluent whitespace atom | 同文件 | 横轴同时插 `padding/margin-inline`，避免只缩内容盒、边距瞬切 |
| Radix Collapsible | `radix-ui--primitives/.../collapsible.tsx` | `useLayoutEffect`（paint 前）挡住 transition → 量盒 → 恢复 transition；关窗先量再让 Presence 播 |
| 鸿蒙动效 | `harmony--motion-system.md` | 持续元素用标准曲线；共享容器插位置/尺寸；容器内内容可以结束时跳到目标 |
| 鸿蒙滚动条 | `设计指南/控件/展示类/滚动条.md` | 4vp 条、距边 4vp；叠在内容上，**不进布局夺列宽** |

## 官方 / 开源怎么做

**尺寸是命令。** Fluent 在意图当拍量一次、写关键帧。Radix 在 layout 阶段量。没有人先画一帧起点、再 `requestAnimationFrame` 才写终点——那会在第一次上屏之后才起程，用户觉得「点了没动」。

**横轴与竖轴同构。** Fluent Collapse 的 orientation 只换 `maxHeight`/`maxWidth`。贴右开合同文件 `harmony-fluent--inline-end-clip`：只插同构长度。侧轨 0↔8vp 必须跟盒高同一 `spatial-panel`，禁止一轴过渡、另一轴瞬切。

**换牌先换字再插宽。** 测宽用布局宽（`offsetWidth`），禁止视觉 rect。量目标发生在字已进树、paint 之前（Solid = `createRenderEffect`），禁止双 rAF 把横轴再推迟两帧。

**滚条不夺列。** 鸿蒙条是叠层。Yohu 为了不挡 Chip 关闭钮才进 8vp 侧轨；轨宽因此是持续空间属性，必须走行程，不能 snap。

## 当时的通路（审查证据）

```
点击展开
  → setExpanded → Solid 画 YoReveal data-open
  → MutationObserver 同步 run
  → apply: write hold（高度仍是 from）
  → 第一次上屏 = 旧高（用户觉得延迟）
  → rAF → write used（这时才插到 to）
  → 侧轨 data-lane 瞬时 0↔8，Chip 网格左右跳
  → YoSwap 先 hold 旧宽，再双 rAF 才插新宽
```

反模式：hold+rAF 冒充「保证起点」；横轴瞬切当「避免卡顿」；换牌用视觉 rect + 双 rAF。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 意图当拍写 used | reuse-pattern | 锁盒已在 from，直接 `data-travel=used` + to |
| 横竖同一 Travel 原语 | adapt | `resolveTravel`；侧轨宽跟盒高同一 spec |
| paint 前量换牌目标宽 | adapt | `createRenderEffect` + `offsetWidth` |
| hold + rAF 再写 to | anti-pattern | 第一帧上屏后才起程 |
| 侧轨 snap / 换牌双 rAF | anti-pattern | 左右生硬 |

## 不要用

- 引进 Fluent react-motion / Radix Collapsible。
- 再给 Dialog.css 叠第三条高度或宽度引擎。
- 把滚条改回叠在 Chip 关闭钮上。
