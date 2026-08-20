---
id: research.microsoft-vscode-table
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [VS Code workbench]
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: microsoft/vscode
  url: https://github.com/microsoft/vscode
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--vscode"
studied_at: 2026-08-20
related: [research.synthesis.ui-kit]
---

# microsoft/vscode（monaco-table 切片）

## 入选理由

桌面效率工具里，VS Code `Table` 把列轨道交给 `SplitView`，标题只是 sash 格子里的文案。这是「区域划分 ≠ 文本」最干净的桌面例。

## 项目是什么

`src/vs/base/browser/ui/table`：表头一行 SplitView + 表体一行 List。列宽变化经 sash 回调同步到每行 cell 的 `style.width`。

## 架构

```
monaco-table
  ├── SplitView（横向，高度 = headerRowHeight）
  │     view = ColumnHeader.element.monaco-table-th  ← 只有 column.label
  │     sash.vertical::before                         ← 分割线，与 th 无关
  └── List
        monaco-table-tr > monaco-table-td[i] { width = splitview.getViewSize(i) }
```

`ColumnHeader` 实现 `IView`：`minimumSize` / `maximumSize` 来自列定义，`layout(size)` 只广播列宽。标题 DOM 是 `$('.monaco-table-th', …, column.label)`，没有 100% 宽的排序按钮。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| sash = 区域，th = 文本 | reuse-pattern | 分割线绝不能画在标题按钮上 |
| 表头宽与表体 cell 宽同一函数 | reuse-pattern | Yohu 已用 `fileColTemplate` 同一套 grid |
| 引入 SplitView 替换 CSS Grid | anti-pattern | 已有 YoColResizer + grid，不必第二套分栏 |

## 架构设计经验

- overflow/ellipsis 加在 `th`/`td` 文本上，sash 在轨道缝上，互不 clip。
- 表头不承担排序高亮几何；排序若需要，是 label 旁的指示，不是铺满 view。

## 与当前工作

- 能直接用：文本节点与分割线分属两层。
- 必须改写：继续 CSS Grid，不移植 monaco-split-view2。
- 不要用：sash 透明底 + 运动过渡那套 VS Code 主题变量。

## 阅读范围

`src/vs/base/browser/ui/table/{tableWidget.ts,table.ts,table.css}`。未读 splitview.ts 实现细节。
