---
id: research.ChromeDevTools-devtools-frontend
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [Chromium DevTools]
also_relevant: [ui-kit, frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: ChromeDevTools/devtools-frontend
  url: https://github.com/ChromeDevTools/devtools-frontend
  head: b30fcfe292fac6e200cb06f355ac7dc8568d4e55
  cloned_to: "%TEMP%/YoAgentResearch/ChromeDevTools--devtools-frontend"
studied_at: 2026-09-10
related: [research.synthesis.client-runtime, research.desktop-log-list-selection]
---

# ChromeDevTools/devtools-frontend（Console 选区切片）

## 入选理由

Yohu 日志是虚拟列表 + 多列视觉，却要像控制台一样拖选复制。DevTools Console 是「虚拟视口 + 自管选区模型 + 复制走模型文本」的对照实现，不是 CSS grid 上的原生 Selection。

## 项目是什么

Chromium 开发者工具前端。本次只读 `front_end/panels/console/ConsoleViewport.ts`：虚拟行、选区、复制。不读协议/CDP。

## 架构

```
ConsoleViewport
  ├── 视觉：只渲染视口附近行；上下 gap 占位
  ├── SelectionModel { item, node, offset }   ← 锚 / 头，按消息下标
  ├── refresh()：重排 DOM 前 updateSelectionModel，之后 restoreSelection
  └── copy：selectedText() 按 item 区间拼行文本，再按节点偏移切片
```

- 原生 `window.getSelection()` **只当输入传感器**。滚动卸载行之后，选区以 `{item, node, offset}` 为准，再 `setBaseAndExtent` 画回去。
- `selectedText()` 遍历 `start.item … end.item`，每行取 `childTextNodes` 的未截断文本，再用 `textOffsetInNode` 切首尾。未挂载行仍能从 provider 取全文。
- `copy` 事件 `preventDefault` 后只写 `text/plain`。拖出同样走 `selectedText()`，不写 HTML 碎片。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 选区是 (行下标, 节点, 偏移) | reuse-pattern | Yohu 对应 (seq, 文档偏移) |
| 复制按模型行拼接 | reuse-pattern | 与现有 `formatLogLine` 同构 |
| 虚拟列表卸载后仍能切首尾 | reuse-pattern | 中间行用 visible 文档补齐 |
| 把浏览器 Selection 当唯一真相 | anti-pattern | CSS grid 会按 DOM 序吞前列 |
| 引入整份 DevTools / CodeMirror | anti-pattern | 只要模型，不要编辑器 |

## 架构设计经验

虚拟列表上的「看起来像选字」，真相必须是自己的 Range。浏览器 Selection 可以辅助命中，但不能在 grid 单元格之间自由延展。

## 与当前工作

- 能直接用：锚/头模型；copy 只写模型文本；中间未挂载行补全文。
- 必须改写：Yohu 命中要落到 **单元格 → 文档偏移**，不能让 caret 跨格延展。
- 不要用：把日志做成可编辑 Console；gap 元素伪造选区。

## 阅读范围

`front_end/panels/console/ConsoleViewport.ts`（SelectionModel / updateSelectionModel / restoreSelection / selectedText / onCopy）。未读 ConsoleView 过滤与 Prompt 编辑器。
