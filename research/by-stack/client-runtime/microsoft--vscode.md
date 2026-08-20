---
id: research.microsoft-vscode-keybindings
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [VS Code workbench]
also_relevant: [ui-kit, frontend]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: microsoft/vscode
  url: https://github.com/microsoft/vscode
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--vscode"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# microsoft/vscode（快捷键 when 子句 + Output 面板）

## 入选理由

Yohu 日志面板的核心缺口是 **Ctrl+A / Space / Ctrl+C 作用在整页 WebView，而不是当前面板**。VS Code 用 `when` 子句把同一物理键拆成「列表全选 / 输入框全选 / 终端 / 编辑器」四套语义，是桌面壳里最干净的可抄模型。官方文档：[Keyboard shortcuts — when clause](https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts)。

## 项目是什么

VS Code 工作台：命令 + 键位 + 上下文键。Output / Logs 视图是只读编辑器面板，列表控件另有 `list.selectAll`。本次只读快捷键与 Output 切片，不读整个编辑器。

## 架构

```
keydown
  → KeybindingService 自底向上匹配 { key, when }
  → 第一条 key+when 都成立的规则执行，其余不再看
context keys
  ├── inputFocus          焦点在 input/textarea/contenteditable
  ├── listFocus           焦点在已注册的 List/Tree/Table，且 !inputFocus
  ├── listSupportsMultiselect
  ├── view == workbench.panel.output
  └── OUTPUT_FILTER_FOCUS_CONTEXT  过滤框自己的焦点
```

关键契约：

- `WorkbenchListFocusContextKey = listFocus && !inputFocus`（`src/vs/platform/list/browser/listService.ts`）。**列表全选故意排除输入框**，否则 Ctrl+A 会把过滤栏里的字丢掉、改去选中几千行。
- `list.selectAll` 绑定 Ctrl/Cmd+A，`when: listFocus && listSupportsMultiselect`；处理函数对 `lastFocusedList` 做 `setSelection(range(list.length))`，选的是 **模型下标**，不是 DOM 文本。
- `inputFocus` 由 `contextKeyService` 在 `focusin` 里用 `isEditableElement(activeElement)` 维护，失焦再清。
- Output 面板：编辑器根上 `CONTEXT_IN_OUTPUT.set(true)`；过滤框用独立 `OUTPUT_FILTER_FOCUS_CONTEXT`。Esc **只在过滤框焦点时**清过滤字，不会在看日志时误清。
- 输入框右键菜单的 Select All 走 `document.execCommand('selectAll')`，与列表 `list.selectAll` 是两条命令。
- 终端 Ctrl+A：Windows/Linux **默认不绑** `terminal.selectAll`（避免和 shell 行首冲突）；macOS 才绑 Cmd+A。官方 Terminal 文档写明这一点。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `when` = 焦点上下文，不是「模块已挂载」 | reuse-pattern | 日志模块还在，焦点在设备栏时，Space/Ctrl+A 不得改日志 |
| `listFocus && !inputFocus` | reuse-pattern | 过滤栏 Ctrl+A 选输入字；列表 Ctrl+A 选行 |
| 全选操作模型下标，禁止 `window.getSelection()` | reuse-pattern | 虚拟列表没有把 10k 行放进 DOM |
| 引入 ContextKeyService / 命令注册表 | anti-pattern | 单 WebView 工作台用 `contains(activeElement)` 足够 |
| 把 Ctrl+A 做成全局 `execCommand('selectAll')` | anti-pattern | 这正是当前 WebView 整页反白的原因 |

## 架构设计经验

- 快捷键规则要同时写 **键** 和 **何时**。只绑 `window.keydown` 等于没有 `when`。
- 同一物理键可以有多条规则：输入框 / 列表 / 面板铬 / 全局，按焦点互斥。
- 过滤框是子上下文，不要和面板命令共用一条 `inPanel`。

## 与当前工作

- 能直接用：面板根 `contains(focus)` + `isEditableTarget` 两层；Ctrl+A 在列表上选 `visible` 行 key；Ctrl+C 写剪贴板文本。
- 必须改写：Yohu 没有命令注册表，用纯函数 `matchLogsShortcut(e, ctx)` 即可。
- 不要用：VS Code 主题变量、Output 的隐藏行过滤（Yohu 过滤在消费端重建可见区）、终端「Windows 不绑 Ctrl+A」（Yohu 不是 shell）。

## 阅读范围

`src/vs/platform/list/browser/listService.ts`（`listFocus` / `!inputFocus`）、`src/vs/workbench/browser/actions/listCommands.ts`（`list.selectAll`）、`src/vs/platform/contextkey/common/contextkeys.ts`、`src/vs/platform/contextkey/browser/contextKeyService.ts`（`inputFocus` 维护）、`src/vs/workbench/contrib/output/browser/{outputView.ts,output.contribution.ts}`、`src/vs/workbench/browser/actions/textInputActions.ts`。未读 keybindingResolver 实现细节。
