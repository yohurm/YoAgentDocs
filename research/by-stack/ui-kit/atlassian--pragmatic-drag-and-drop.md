---
id: research.atlassian-pragmatic-drag-and-drop
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [react]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt]
source:
  platform: github
  repo: atlassian/pragmatic-drag-and-drop
  url: https://github.com/atlassian/pragmatic-drag-and-drop
  cloned_to: "%TEMP%/YoAgentResearch/atlassian--pragmatic-drag-and-drop"
studied_at: 2026-09-11
related: [research.clauderic-dnd-kit]
---

# atlassian/pragmatic-drag-and-drop

## 入选理由

Atlassian 官方拖放设计规范把「插缝线」定义清楚：相对放置用线，不是钉在行顶。与 Apple HIG 列表插缝一致。

## 项目是什么

替换 react-beautiful-dnd 的头less 拖放。核心包 + hitbox + drop-indicator。本机浅克隆因 Windows 长路径未能完整 checkout，规范以 [Design guidelines](https://atlassian.design/components/pragmatic-drag-and-drop/design-guidelines) 为准。

## 架构

- closest-edge：指针相对目标中线，得到 before / after。
- Drop indicator：2px selected 边、可选 8px 端点；表示插入缝，不表示「当前悬停行」。
- 源行可降透明度；列表不必 live-mutate。
- 空列表或无法相对放置时不画线。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 最近边 → 插入缝 | reuse-pattern | `insertIndex = round(rel / rowHeight)` |
| 2px accent 线 | adapt | 用 `--yohu-stroke-accent` / `--yohu-accent` |
| 引进 Atlaskit | anti-pattern | 不进 YoUI |

## 架构设计经验

线只回答「插在哪条缝」。谁被拖走要靠浮层。两者缺一都会让用户猜。

## 与当前工作

Yohu v2.61 只把条钉在目标行顶，是错误缝。应改为 0..count 缝坐标，原槽不画线。

## 阅读范围

Atlassian Design guidelines（Drop indicator / closest-edge）；clone 工作树因路径过长未完整检出。
