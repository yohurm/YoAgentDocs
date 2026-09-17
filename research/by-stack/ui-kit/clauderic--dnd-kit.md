---
id: research.clauderic-dnd-kit
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [react, solid, vue]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: clauderic/dnd-kit
  url: https://github.com/clauderic/dnd-kit
  head: 4fa5c57
  cloned_to: "%TEMP%/YoAgentResearch/clauderic--dnd-kit"
studied_at: 2026-09-11
related: [research.atlassian-pragmatic-drag-and-drop]
---

# clauderic/dnd-kit

## 入选理由

桌面列表换位的事实参考实现：虚拟列表必须用 DragOverlay；源行与浮层分离；`arrayMove` 在松手提交。Yohu 命令管理 VirtualList 正是定高虚拟列表。

## 项目是什么

无障碍拖放工具包。core / collision / helpers / 各框架绑定。Sortable 是插件，不是业务列表。

## 架构

- `DragOverlay`：浮层挂在 Feedback 插件，跟指针，可做 drop animation；children 禁止再绑 draggable。
- 虚拟列表：源节点可能卸载，浮层必须在滚动容器外或不受 overflow 限制。
- `arrayMove(from, to)`：splice 换位，同位返回原数组。
- 碰撞：closestCenter；插入还看目标中线上下。
- 键盘：独立 SortableKeyboardPlugin。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 浮层与源行分离 | reuse-pattern | 源行占位，副本跟指针 |
| 虚拟列表用 overlay | reuse-pattern | 行可回收 |
| 引进 dnd-kit | anti-pattern | YoUI 不绑第二套拖放运行时 |

## 架构设计经验

换位反馈是三件套：浮层、占位、邻项位移。只画一条线不够。数据在 dragend 提交，预览用 transform。

## 与当前工作

能直接用：overlay + 源行隐藏/淡化 + `arrayMove`。必须改写：自研几何，配方走 token。不要用：把 `@dnd-kit/*` 加进 `@yohu/ui`。

## 阅读范围

`packages/solid/src/core/draggable/DragOverlay.tsx`、`packages/helpers/src/move.ts`、`packages/dom/src/sortable/*`、官方 DragOverlay / Sortable 文档。
