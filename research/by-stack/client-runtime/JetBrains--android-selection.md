---
id: research.JetBrains-android-selection
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Kotlin]
  frameworks: [IntelliJ EditorEx]
also_relevant: [ui-kit]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: JetBrains/android
  url: https://github.com/JetBrains/android
  head: 21cdb67fc9c20d6af9e198fca7c502b5ce4dab6d
  cloned_to: "%TEMP%/YoAgentResearch/JetBrains--android"
studied_at: 2026-09-10
related: [research.JetBrains-android, research.desktop-log-list-selection]
---

# JetBrains/android（Logcat 文档选区切片）

## 入选理由

Yohu 日志对标 Android Studio Logcat。先前深研只读了快捷键与复制动作。本次补「字段为什么能单独选中」：Logcat **不是表格**，是一份 Console `Document`，字段范围是文档偏移。

## 项目是什么

Android Studio Logcat 工具窗。日志视图 = `EditorFactory.createViewer(..., EditorKind.CONSOLE)`。

## 架构

```
LogcatMessage
  → MessageFormatter.accumulate(timestamp / pid-tid / tag / app / level / message)
  → Document 连续文本
  → RangeMarker + LOGCAT_MESSAGE_KEY
  → EditorEx 选区 = (line, column) 文档坐标
```

`FormattingOptions.getTagRange()` / `getAppIdRange()` / `getLeveRange()` 用各字段 **width() 累加** 得到文档闭区间。双击、点选、过滤提示都问「这个 offset 落在哪个字段」，不问 DOM 格子。

两种复制并存：

1. 编辑器默认 Ctrl+C：剪贴板 = 选中的可见文档（含时间/PID/Tag）。
2. `CopyMessageTextAction`：按选区相交的 `LOGCAT_MESSAGE_KEY` 只取 `message` 正文。

没有「CSS grid 单元格 + 原生 Selection」。列开关改变的是 **文档里有没有那一段**，不是另一套几何。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 字段 = 文档 span | reuse-pattern | `formatLogLineParts` 已是同一代数 |
| 点选问 offset 落在哪一段 | reuse-pattern | 从左点 Tag = Tag span 起点，不会吞时间 |
| 默认复制可见文档 | reuse-pattern | Yohu 已是 `formatLogLine` |
| 把日志做成 EditorEx / Monaco | anti-pattern | 10k 虚拟列表，需求禁止 |
| Pause 停采集 | anti-pattern | 已在旧篇记录 |

## 架构设计经验

「从左边选 Tag 不带时间」在编辑器里是免费的：光标落在 Tag 的文档列。Yohu 用 grid 画列，必须 **自己把指针映射回同一套文档列**，而不是让 Chromium 按 DOM 序延展。

## 与当前工作

- 能直接用：字段 span、复制文档切片、双击选字段/词。
- 必须改写：Editor caret → 指针命中单元格再 `mapLogCellOffsetToDoc`。
- 不要用：把行渲染成 textarea；兼容旧的 `user-select` 锁格。

## 阅读范围

`logcat/.../util/EditorUtils.kt`、`messages/{FormattingOptions,MessageFormatter,DocumentAppender}.kt`、`actions/CopyMessageTextAction.kt`。未读 PSI 过滤与 proto 采集。
