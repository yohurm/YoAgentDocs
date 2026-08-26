---
id: research.radix-ui-primitives
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React]
also_relevant: [frontend]
utilization: [reuse-pattern, lesson-only]
source:
  platform: github
  repo: radix-ui/primitives
  url: https://github.com/radix-ui/primitives
  cloned_to: "%TEMP%/YoAgentResearch/radix-ui--primitives"
studied_at: 2026-08-20
related: [research.microsoft-fluentui, research.synthesis.ui-kit]
---

# radix-ui/primitives（Presence）

## 入选理由

Yohu `YoPresence` 已按 Radix 的 `data-state` 思路做了布尔进出场。源码确认：Presence **不管视觉**，只延迟卸载。文案换牌不能再复用「单节点 visible」模型，需要 keyed 组（Fluent PresenceGroup），否则会把换牌状态机塞进 Button。

## 项目是什么

无头组件库。`@radix-ui/react-presence` 是内部工具：`present` 布尔值 + 状态机 `mounted / unmountSuspended / unmounted`，用计算样式里的 `animation-name` 判断是否真的在播出场。

## 架构

- JS：状态机 + `animationend`；CSS 写在消费方。
- 不提供「key 从 A 换成 B 时双节点重叠」。
- 读 `animation-name` 变化来决定能否 UNMOUNT，避免没有关键帧时干等。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Presence ≠ 视觉配方 | reuse-pattern | `YoPresence` 继续只服务 Dialog/模块；换牌另立 `YoSwap` |
| 无动画则立刻卸 | reuse-pattern | `shouldSkipMotion` / 无 swapping 时不要双节点 |
| 单 present 布尔覆盖换牌 | anti-pattern | 用两个独立 Presence 叠在按钮里会闪、会对不齐 |

## 架构设计经验

布尔 Presence 和 keyed Swap 是两种原语。强行用前者模拟后者，就会在控件文件里长出测量宽度、双 face、generation 计时器。

## 与当前工作

- 能直接用：出场结束再卸、测试环境 skip。
- 不要用：把 Button 做成「两个 YoPresence」。

## 阅读范围

`packages/react/presence/src/{presence,use-state-machine,index}.tsx`。
