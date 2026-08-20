---
id: research.jetbrains-android-logcat
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Kotlin]
  frameworks: [IntelliJ Platform, EditorEx]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: JetBrains/android
  url: https://github.com/JetBrains/android
  cloned_to: "%TEMP%/YoAgentResearch/JetBrains--android"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# JetBrains/android（Logcat 工具窗）

## 入选理由

Yohu 日志分析明确对标 Android Studio Logcat（需求 §5.3、架构「AS / Logdy」）。官方交互：[View logs with Logcat](https://developer.android.com/studio/debug/logcat)。本仓库 `logcat/` 是 Studio 工具窗的开源实现：Tab、查询栏、Editor 选区、按消息复制、暂停采集。

## 项目是什么

IntelliJ 平台上的 Android 插件。`logcat` 模块 ≈ 120 个 Kotlin 源文件：工具窗工厂、主面板、过滤语言、消息格式化、动作、设备下拉。本次只读面板交互与快捷键，不读 proto 采集。

## 架构

```
LogcatToolWindow（底栏，可关内容）
  └── LogcatMainPanel
        ├── ActionToolbar（清屏 / 暂停 / 重启 / 软折行 / 滚到底）
        ├── FilterTextField（查询语言，焦点独立）
        └── EditorEx(EditorKind.CONSOLE)   ← 日志就是只读编辑器文档
              registerCustomShortcutSet(..., editor.contentComponent)
```

焦点模型是 **Swing 焦点宿主**，不是全局 window 监听：

- 日志视图是 `EditorFactory.createViewer(..., EditorKind.CONSOLE)`。Ctrl+A / Ctrl+C / 方向键走编辑器默认动作，因为焦点在 `editor.contentComponent` 上。过滤栏是另一个组件，Ctrl+A 只选查询字。
- 面板特有动作（Pause / Clear / Toggle format）注册在 keymap 组 `Logcat.LogcatActions`，`UserInputHandlers` 把需要落在编辑器上的快捷键 `registerCustomShortcutSet(..., editor.contentComponent)`。注释写明：绑在 Logcat editor 上的快捷键，焦点跑到文件编辑器后就失效——这是作用域，不是缺陷。
- `uiDataSnapshot` 向动作总线提供 **当前面板** 的 `LOGCAT_PRESENTER_ACTION` 和 `EDITOR`。多 Tab / 分屏时，动作打在焦点所在的那一块，不会清错窗。
- `CopyMessageTextAction`：按选区相交的 `LOGCAT_MESSAGE_KEY` 取出结构化消息，剪贴板只放 **message 正文**（可多条）。编辑器默认 Ctrl+C 仍复制带时间/PID/Tag 的可见文本。两种复制并存。
- 跟尾：`isScrollAtBottom` + `isCaretAtBottom`。点某一行（光标离开底部）就停跟；工具栏「Scroll to the End」再贴底。
- **Pause 会停采集**（`PauseLogcat` 服务事件，`isLogcatPaused`）。这与 Yohu 需求相反：Yohu 的 Space 只冻 UI，环继续写。

官方产品层：Alt+6 打开 Logcat 工具窗；工具窗激活后 Ctrl+A 选日志，焦点在查询栏时 Ctrl+Space 补全。查询 DSL、分屏、软折行本期 Yohu 不做。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 快捷键绑在面板宿主，不绑 window | reuse-pattern | DOM 等价：面板根 contains(target) |
| 过滤栏与日志视图两个焦点域 | reuse-pattern | 与 VS Code `inputFocus` vs `listFocus` 同构 |
| 复制选中的结构化行，而不是 innerText | adapt | Yohu 无 Editor，按 `visible` + selectedKeys 拼列对齐文本 |
| 多会话动作带 presenter 身份 | reuse-pattern | 只操作 `activeSessionId` |
| Pause = 停 logcat 进程 | anti-pattern | 违反 ADR-v6-006 / 需求「暂停冻 UI、缓冲继续」 |
| 把日志做成可编辑 Console 文档 | anti-pattern | 虚拟列表 + 10k 环，不能上 Monaco/textarea |
| 查询 DSL / 分屏 | lesson-only | 需求明确下一期或不做 |

## 架构设计经验

- 「全选日志」能 round-trip，是因为日志视图 **自己就是选区宿主**。Yohu 的宿主是 `YoVirtualList`，必须自己实现 select-all / copy，不能指望浏览器选中页面文本。
- 组件级 `registerCustomShortcutSet(component)` = 作用域。全局 keymap 只用于「打开工具窗」这类壳命令。
- 跟尾看滚动度量，不看暂停按钮；暂停是另一条轴。Yohu 已正交，保持。

## 与当前工作

- 能直接用：面板内才响应 Space/Ctrl+L/F/T/W/A/C；过滤输入走原生全选；复制格式化行；状态行显示已选条数。
- 必须改写：EditorEx → `YoVirtualList` + `selectedKeys`；Pause 保持 UI 暂停。
- 不要用：Logcat Filter 语言、用户在日志里打字（`UserInputHandlers`）、IJ `AnAction` 总线、清屏即 `logcat -c`（Yohu 已拆「清空可见区」与「清设备缓冲」）。

## 阅读范围

`logcat/src/.../LogcatMainPanel.kt`、`UserInputHandlers.kt`、`util/EditorUtils.kt`（`createLogcatEditor`）、`actions/{CopyMessageTextAction,PauseLogcatAction,ClearLogcatAction,ActionExtensions,LogcatKeymapExtension}.kt`、`resources/META-INF/logcat.xml`。未读过滤 PSI、proto collector、MessageBacklog 细节。
