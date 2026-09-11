---
id: research.desktop-log-list-selection
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [SolidJS, Chromium]
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: other
  url: https://developer.android.com/studio/debug/logcat
  repos:
    - JetBrains/android
    - ChromeDevTools/devtools-frontend
    - microsoft/vscode
    - ag-grid/ag-grid
  cloned_to:
    - "%TEMP%/YoAgentResearch/JetBrains--android"
    - "%TEMP%/YoAgentResearch/ChromeDevTools--devtools-frontend"
    - "%TEMP%/YoAgentResearch/microsoft--vscode"
    - "%TEMP%/YoAgentResearch/ag-grid--ag-grid"
studied_at: 2026-09-10
updated_at: 2026-09-10
related:
  - research.JetBrains-android-selection
  - research.ChromeDevTools-devtools-frontend
  - research.microsoft-vscode-selection
  - research.ag-grid-ag-grid-selection
---

# 桌面日志清单的文档选区

## 背景

Yohu 要把日志选区做成 Android Studio Logcat 那种：从字段拖到列间空白也能选中。旧实现把一行拆成 CSS grid 单元格，空白是列宽剩余，不是字符。

## Logcat 完整数据链路（源码）

读自 `JetBrains/android` `logcat/`（HEAD `21cdb67`）：

```
LogcatMessage
  → MessageFormatter.formatMessages
       TimestampFormat.format      文案自带尾空格（width 含空格）
       ProcessThreadFormat.format  "%-5d " / "%5d-%-5d "（空格在字符串里）
       TagFormat.format            tag.padEnd(maxLength + 1)
                                   短 tag 后面的「空白背景」= 真实空格
       LevelFormat                 " X " + 额外 " "
       message
  → TextAccumulator.stringBuilder  一份连续 String
  → DocumentAppender.insertString  写入 Editor Document
  → MarkupModel.addRangeHighlighter(EXACT_RANGE)
       颜色是文档上的着色，不拆盒、不另开格子
  → EditorEx 选区 = Document 偏移
  → 默认 Ctrl+C = 选中的文档切片（含 pad 空格）
  → CopyMessageTextAction = 只要 message 正文（第二条动作，不是选区引擎）
```

结论（不是猜测）：

1. Logcat **没有表格格**。列宽 = 字段 `width()` / `padEnd`。
2. 能选中空白，是因为空白是 `padEnd` / 格式串里的 `' '`。
3. 着色不参与几何。选区只问 Document。
4. 双击选词走编辑器，不是先选 DOM 再改。

## Yohu 旧链路（已废弃）

```
LogLine
  → formatLogLineParts（pid/uid 有 padStart；tag 无 padEnd）
  → YoColTrack 网格 + YoColCell(logLineCellText)   ← 文案不含列间空格
  → 列宽剩余 = CSS padding/轨道，不是字符
  → selection.ts 按格命中、只给格内文本画高亮
  → copy 再映射回 formatLogLine
```

所以选区只能落在各列「有字的地方」。这与 Logcat 不是同一条链。

## 目标链路（替换，不兼容格选区）

```
LogLine + 显示列 + 列宽/ch
  → formatLogDocParts：每个元数据字段 clip/pad 到列宽对应的字符数
       短字段后面/前面的空白是 ' '，进文档
  → 一行一个 white-space:pre 文档（span 只着色，不拆成 grid 格）
  → 原生 Selection 走这一份 DOM 文本（=== 文档）
  → 复制：选区偏移切文档；虚拟列表未挂载的中间行用同一 formatLogDoc 补齐
  → formatLogLine / domain testdata 只给导出，不驱动清单选区
```

禁止：

- 行再用 `YoColTrack` / `YoColCell` 当选区表面
- `user-select: none` 后再自绘格内高亮
- `pointerdown.detail` 当双击计数
- `Selection.toString()` 当跨行唯一载荷（中间行要补文档）

## 来源与阅读范围

`MessageFormatter.kt`、`TextAccumulator.kt`、`DocumentAppender.kt`、`TagFormat.kt`、`ProcessThreadFormat.kt`、`TimestampFormat.kt`、`LevelFormat.kt`、`EditorUtils.kt`、`CopyMessageTextAction.kt`。
