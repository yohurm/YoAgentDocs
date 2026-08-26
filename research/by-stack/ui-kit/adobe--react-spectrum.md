---
id: research.adobe-react-spectrum
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React, Spectrum]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt]
source:
  platform: github
  repo: adobe/react-spectrum
  url: https://github.com/adobe/react-spectrum
  cloned_to: "%TEMP%/YoAgentResearch/adobe--react-spectrum"
studied_at: 2026-08-20
related: [research.synthesis.ui-kit]
---

# adobe/react-spectrum（Table 表头切片）

## 入选理由

Spectrum 明确把「排序」从「列宽」里拆出去：`useTableColumnHeader` 不再处理 resize；视觉上 `columnResizer::after` 才是 1px 分割线。对应 Yohu 截图里名称高亮贴住「类型」、分割线不像列界。

## 项目是什么

React Spectrum TableView + Spectrum CSS `components/table`。状态在 `@react-stately/table`，拖拽交互在 `@react-aria/table`，绘制在 CSS。

## 架构

```
headCell（轨道，padding 来自 header-padding token）
  ├── headCellContents / headCellButton   文案 + 省略
  ├── sortedIcon   默认 display:none；is-sorted-* 才显示
  ├── columnResizerPlaceholder   占 10px，避免字贴线
  └── columnResizer（absolute，inset-inline-end: -10px）
        └── ::after  1px 分割线
```

可排序时 hover 改的是表头文字色，不是用圆角片去冒充列宽。PR #3295 的目标就是「resize 引用从 useTableColumnHeader 里拿掉」。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 分割线画在 resizer 铬上 | reuse-pattern | 线是区域划分，不是标题的边框 |
| 排序图标未激活不占位 | reuse-pattern | `display:none` 直到 is-sorted |
| placeholder 把字从线推开 | adapt | 用已有 `--yohu-space-sm`，不新增 10px 魔法数 |
| 整份 TableLayout / 虚拟化 | lesson-only | Yohu 已有 grid + VirtualList |

## 架构设计经验

- 文本溢出只发生在 `headCellContents`（content-region）。
- 可 resize 时 headCell `padding: 0`，把内边距交给 contents，避免轨道 padding 和拖拽条抢边。

## 与当前工作

- 能直接用：图标按需出现；内容区裁剪；分割线属铬。
- 必须改写：不引入 React Aria hooks；拖拽继续走 `YoColResizer`。
- 不要用：Spectrum 的 21px 拖拽热区数字、菜单 chevron。

## 阅读范围

`packages/@adobe/spectrum-css-temp/components/table/{index,skin,vars}.css`；aria/stately/spectrum table 包入口。未读 TableView.tsx 全文（稀疏检出未拉实现文件）。
