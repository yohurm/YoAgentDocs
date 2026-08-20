---
id: research.amir20-dozzle
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript, Go]
  frameworks: [Vue 3]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: amir20/dozzle
  url: https://github.com/amir20/dozzle
  cloned_to: "%TEMP%/YoAgentResearch/amir20--dozzle"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# amir20/dozzle

## 入选理由

现代 Web 日志查看器（虚拟列表、级别过滤、复制）。用来对照 **Ctrl+F 检索 vs Ctrl+K 命令面板** 的分层，以及「复制当前可见日志」不依赖 DOM 选区。Yohu UI 设计系统已规划 `Ctrl+K` 命令面板，日志 Ctrl+F 必须给它让路。

## 项目是什么

Docker / Swarm / K8s 实时日志 Web UI。前端 Vue 3 + 虚拟滚动，后端 Go。默认快捷键写在设置页。

## 架构

快捷键按 **组件挂载范围** 注册（`onKeyStroke`），不是单一全局表：

| 键 | 注册位置 | 语义 |
|----|----------|------|
| Ctrl/Cmd+K | `layouts/default.vue`（壳） | 打开容器模糊搜索（命令面板） |
| Ctrl/Cmd+F | `components/Search.vue`（日志检索组件挂着才有效） | 打开检索条并 focus；条内再按则 `stopPropagation` + 关闭 |
| Ctrl+Shift+L | `ViewerWithSource.vue`（当前日志视图） | 清当前流 |
| 行级 Copy | `LogActions.vue` 悬停菜单 | `clipboard.write` 单条 raw |
| 工具栏 Copy logs | `ContainerActionsToolbar.vue` | **fetch 过滤后的全文** 再写入剪贴板，不经过选区 |

`Search.vue` 在检索 input 上单独绑 Ctrl+F：`stopPropagation` 后关闭检索。壳级 Ctrl+K 与日志 Ctrl+F 互不抢。清屏用 **Ctrl+Shift+L**，避开浏览器地址栏 Ctrl+L（Dozzle 跑在真浏览器里）。Yohu 是 Tauri WebView，Ctrl+L 仍可用，但必须面板作用域，免得焦点在别的模块时误清。

Dozzle **没有**列表 Ctrl+A。复制「全部可见」走工具栏，复制一条走行菜单。这是产品取舍：流式日志上全选 10 万行会卡，他们选择「复制过滤结果」而不是「选中再复制」。

Yohu 缓冲上限默认 10k，全选 + 复制可见区是可行的，应对标 AS 的选区复制，同时保留导出按钮给全量环。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 壳命令与面板命令分层注册 | reuse-pattern | Ctrl+K 将来归壳；Ctrl+F/L/A 归日志面板 |
| 检索条打开时 Ctrl+F 换语义 | reuse-pattern | 已在关键字框里则 select 内容，不要再 prevent 掉原生 |
| 复制走数据模型 / API，不走 innerText | reuse-pattern | 与 AS `CopyMessageTextAction` 同构 |
| 无选区、只靠工具栏复制全部 | adapt | Yohu 要补选区；工具栏导出已存在 |
| 正则检索覆盖 | anti-pattern | 需求「无正则框」 |
| Ctrl+Shift 修饰清屏 | lesson-only | WebView 里 Ctrl+L 可留，但必须 `inPanel && !inEditable` |

## 架构设计经验

- 快捷键的作用域 = **注册该监听的组件是否在树上**。Dozzle 一页一个容器视图，组件卸载即失效。Yohu 模块切换会卸日志页，但侧栏设备列表仍在——所以还要面板 `contains`，不能只靠「模块已挂载」。
- 「复制全部」和「全选」不是一回事。全选是给随后的 Ctrl+C / 删除准备的；复制全部可以没有选区。Yohu 两者都要：Ctrl+A 选可见行，Ctrl+C 复制选中（无选中则不要把整页 HTML 放进剪贴板）。

## 与当前工作

- 能直接用：Ctrl+F 只在日志页；检索框内放行/再聚焦；复制拼模型文本。
- 必须改写：补列表多选与 Ctrl+A；清屏保持 Ctrl+L（面板内）。
- 不要用：SQL/DuckDB 查询、正则、Docker 语义、每行悬停动作菜单（Yohu 用行选中 + 快捷键）。

## 阅读范围

`assets/layouts/default.vue`、`assets/components/Search.vue`、`assets/components/LogViewer/{ViewerWithSource.vue,LogActions.vue}`、`assets/components/ContainerViewer/ContainerActionsToolbar.vue`、`assets/pages/settings.vue`（快捷键说明）。未读 Go 后端与 SQL engine。
