---
id: research.ag-grid-ag-grid
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [ag-grid]
also_relevant: [frontend]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: ag-grid/ag-grid
  url: https://github.com/ag-grid/ag-grid
  cloned_to: "%TEMP%/YoAgentResearch/ag-grid--ag-grid"
studied_at: 2026-08-20
related: [research.synthesis.ui-kit]
---

# ag-grid/ag-grid（表头切片）

## 入选理由

Yohu 文件清单表头把「列轨道 / 排序文案 / 拖拽条」叠在同一个 `width:100%` 按钮上，分割线被排序高亮吃掉。AG Grid 把这三层拆成独立 DOM，是表头区域划分的参考实现。

## 项目是什么

AG Grid Community 的表头渲染：`headerRendering/cells/column`。网格引擎本身不进 Yohu。

## 架构

```
ag-header-cell（轨道，role=columnheader，拥有宽度）
  ├── ag-header-cell-resize     拖拽条，兄弟节点，不进标题
  └── ag-header-cell-comp-wrapper
        └── ag-cell-label-container
              └── ag-header-cell-label
                    ├── ag-header-cell-text   标题
                    └── ag-sort-indicator    仅当列可排序才进模板
```

`HeaderCellComp` 的模板只放 `eResize` + `eHeaderCompWrapper`。标题组件 `AgColumnHeader` 另外挂进 wrapper。自定义 `innerHeaderComponent` 只换文案，不重写 resize。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 轨道 / 内容包装 / resize 三节点 | reuse-pattern | 分割线与拖拽属轨道，排序属内容 |
| innerHeader 只换文案 | reuse-pattern | 排序交互不要铺满轨道 |
| 整份 Grid 引擎 | anti-pattern | 不引入 ag-grid；Yohu 已有 CSS Grid + YoColResizer |

## 架构设计经验

- 列宽变化只改轨道 `style.width`，不让标题按钮承担几何。
- overflow 裁的是标题 wrapper，不是轨道；否则负偏移的 resize 会被剪掉。

## 与当前工作

- 能直接用：三节点分层；sort 指示器按需出现。
- 必须改写：Solid + token；不抄 ag- 类名。
- 不要用：列组、floating filter、整表虚拟表头。

## 阅读范围

`packages/ag-grid-community/src/headerRendering/cells/column/{headerCellComp,agColumnHeader,headerCellCtrl}.ts`；`cells/columnGroup/headerGroupCellComp.ts`。未读行渲染与列状态机全文。
