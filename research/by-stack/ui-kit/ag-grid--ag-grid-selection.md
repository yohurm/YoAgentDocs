---
id: research.ag-grid-ag-grid-selection
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [ag-grid]
also_relevant: [frontend]
utilization: [reuse-pattern, anti-pattern, lesson-only]
source:
  platform: github
  repo: ag-grid/ag-grid
  url: https://github.com/ag-grid/ag-grid
  head: a19cdff8973c9462bd786597241001aaec1d4ed4
  cloned_to: "%TEMP%/YoAgentResearch/ag-grid--ag-grid"
studied_at: 2026-09-10
related: [research.ag-grid-ag-grid, research.desktop-log-list-selection]
---

# ag-grid/ag-grid（单元格选区切片）

## 入选理由

Yohu 曾对照 `enableCellTextSelection` 用 `user-select` 锁格，结果从左拖仍吞前列、双击闪一下整行。需要看官方到底把「选字」和「选格」分成了几条互斥链。

## 项目是什么

AG Grid Community。表体默认 **禁止** 原生选字（`.ag-unselectable { user-select: none }`）。两条互斥能力：

1. **Cell Selection**（`cellSelection`）：Excel 式单元格闭区间，剪贴板由网格模型写出。
2. **Cell Text Selection**（`enableCellTextSelection`）：打开后给格子加 `.ag-selectable { user-select: text }`，并关掉网格自己的 clipboard。官方要求 `ensureDomOrder`，且只保证 **格内** 拖选。

文档写明：打开原生选字后，Ctrl+C 只复制浏览器选中的那截字；跨格、聚合、未渲染行都不归这套管。

## 架构

```
默认：user-select:none + rangeSvc.handleCellMouseDown → 模型 Range → clipboardSvc
可选：enableCellTextSelection → 格子 user-select:text → 浏览器 Selection → 网格 clipboard 停用
```

`cellMouseListenerFeature` 在原生选字开启时，仍要 `focusCell`，因为行拖拽会 `preventDefault` 掉 mousedown。它 **没有** 实现「从格子左缘拖选却不吞上一格」——那是浏览器行为，官方用互斥回避，而不是打补丁。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 选字与选格互斥 | reuse-pattern | 日志是 Family A，只走文档 Range |
| 默认关掉 user-select | reuse-pattern | 清单整表 none，高亮自己画 |
| 把 enableCellTextSelection 当日志方案 | anti-pattern | 官方自己说跨格不保证 |
| 引入 ag-grid | anti-pattern | 只要互斥结论 |

## 架构设计经验

成熟网格承认：浏览器跨格选字不可靠。要跨格就自管 Range；要选字就锁在一格且放弃模型剪贴板。Yohu 日志两样都要（格内字段 + 跨行文档），所以必须走模型，不能走 `user-select: text`。

## 与当前工作

- 能直接用：清单 `user-select: none`；复制走模型。
- 必须改写：删掉锁格/升档补丁。
- 不要用：`enableCellTextSelection` 的 CSS 互斥当选区引擎。

## 阅读范围

官方 [Cell Text Selection](https://www.ag-grid.com/javascript-data-grid/cell-text-selection/)、[Cell Selection](https://www.ag-grid.com/javascript-data-grid/cell-selection/)。源码：`gridBodyCtrl.setCellTextSelection`、`theming/core/css/_general.css`、`rendering/cell/cellMouseListenerFeature.ts`、`gridOptionsDefault.ts`。未读 Enterprise clipboard 全文。
