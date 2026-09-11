---
id: research.microsoft-vscode-selection
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [Monaco]
also_relevant: [ui-kit]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: microsoft/vscode
  url: https://github.com/microsoft/vscode
  head: a0a429cd4ac624c0add568597f00b052c225ae56
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--vscode"
studied_at: 2026-09-10
related: [research.microsoft-vscode, research.desktop-log-list-selection]
---

# microsoft/vscode（编辑器选区切片）

## 入选理由

VS Code Output / 编辑器把「看起来像网页选字」做成 **模型坐标**。用来对照：复制永远 `model.getValueInRange(selection)`，视图折行、DOM 节点顺序都不进剪贴板。

## 项目是什么

`src/vs/editor/common/core/selection.ts`：`Selection` 继承 `Range`，带方向（锚 vs 头）。公开 API 全是 `(lineNumber, column)`。视图坐标是内部细节，复制前要换回模型。

## 架构

```
指针 / 键盘
  → 命中视图坐标
  → coordinatesConverter → 模型 Selection
  → 高亮由视图按模型 Range 绘制
  → 复制 = model.getValueInRange(selection)
```

双击选词、三击选行都改 **模型 Selection**，不会先让浏览器选一整段 DOM 再改回去——所以没有「闪一下整行再收回」。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 锚/头 + 方向 | reuse-pattern | 从右往左拖仍切同一闭区间 |
| 复制只问模型 | reuse-pattern | `formatLogLine.slice` |
| 手势直接写模型 | reuse-pattern | 双击不经过 window.getSelection |
| 引入 Monaco | anti-pattern | 虚拟列表 + 列轨道已存在 |

## 架构设计经验

高亮是视图对模型的投影。谁当真相，谁就不会闪。Yohu 旧补丁先让浏览器选、再 lock/reselect，闪动是结构问题。

## 与当前工作

- 能直接用：DocPos / DocRange；双击写字段 span；复制 slice 模型。
- 必须改写：命中列盒后映射到文档 column，不是 view 折行列。
- 不要用：Monaco 视图层、多光标。

## 阅读范围

`src/vs/editor/common/core/selection.ts`、`model/textModel.ts#getValueInRange`。未读 mouseHandler 全文与 Output 面板 UI。
