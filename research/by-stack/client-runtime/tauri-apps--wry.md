---
id: research.tauri-apps-wry
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Rust]
  frameworks: [WebView2, Win32 OLE]
also_relevant: [windows-desktop, frontend]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: tauri-apps/wry
  url: https://github.com/tauri-apps/wry
  cloned_to: "%TEMP%/YoAgentResearch/tauri-apps--wry"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# tauri-apps/wry（Windows 拖入）

## 入选理由

Yohu 的窗口就是 wry + WebView2。拖入能不能做、HTML5 `drop` 为什么在 Windows 上是空的，答案在 `src/webview2/drag_drop.rs`，不在 Solid。Tauri 2 默认打开这套 handler，映射为 `tauri://drag-drop`。不读这份就会在前端接 `ondrop` 然后发现没有路径。

相关：wry #904（原生 drop 无法透传给 WebView2，2023 起未决）、Tauri #14373（`dragDropEnabled` 命名：开 = 接管 OLE，关 = 把 HTML5 还回去）、WebView2 Feedback #501。

## 项目是什么

跨平台 WebView 封装。Windows 上是窗口化 WebView2。文件拖入由 wry 自己实现 `IDropTarget`，**替换** WebView2 的外部 drop。

## 架构

```
Explorer CF_HDROP
        ▼
WebView2 子 HWND（窗口化控件，不是视觉宿主）
        ▼
DragDropController
  EnumChildWindows → RevokeDragDrop + RegisterDragDrop
  SetAllowExternalDrop(false)   // 关掉 WebView2 自己的 drop
        ▼
IDropTarget::DragEnter / Over / Leave / Drop
  GetData(CF_HDROP) → DragQueryFileW（动态长度，不限 MAX_PATH）
        ▼
DragDropEvent { paths, position }   // position = ScreenToClient
        ▼
Tauri  转成 tauri://drag-enter|drag-over|drag-drop|drag-leave
```

约束（源码注释 + builder 文档）：

- **Windows 上 HTML5 DnD 被关掉。** `draggable="true"` / `ondrop` 不再收到文件。handler 返回值在 Windows 上被忽略。
- 事件是 **整窗** 的，不是某个 DOM 节点。要「只落到文件列表」，必须自己用 `position` 或前端 hover 状态做命中。
- 只认 `CF_HDROP`。ZIP 内部、Outlook 附件等虚拟格式 `GetData` 失败，当作「不是文件」忽略（`DV_E_FORMATETC`）。
- cursor 效果写死 `DROPEFFECT_COPY`，没有 Move/Link，也没有「落在非法区域显示禁止」。

Yohu `tauri.conf.json` 未设 `dragDropEnabled`，默认 **true**（走这套，不走 HTML5）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `tauri://drag-drop` 的 `paths` 就是本机绝对路径 | reuse-pattern | 与 scrcpy 的 SDL_DROPFILE 同构；直接 `files.push` |
| 整窗事件 + 坐标 | reuse-pattern | 文件面板用 `position` / `elementFromPoint` 过滤；落在设备栏/日志页则忽略 |
| 关掉 `dragDropEnabled` 改用 HTML5 drop | anti-pattern | Windows 上 HTML5 拿不到完整路径；WebView2 安全模型如此 |
| 自己再 `RegisterDragDrop` 叠一层 | anti-pattern | 子 HWND 已经注册；Tauri #8532 就是踩这个 |
| 指望 wry 把 drop 传成 JS `File` | anti-pattern | #904 未做；也不需要，Yohu 要的是路径不是 Blob |

## 架构设计经验

- WebView2 是 **窗口化** 子控件，OLE drop 注册在它的 HWND 上。父窗口再注册收不到落在画面上的文件。
- 「前端 drop 区」在 Windows+Tauri 里是 **视觉状态**（hover 高亮），数据面永远是壳事件里的路径数组。
- 拖入与拖出抢同一套 OLE。拖出时（`DoDragDrop` 模态循环）窗口自己的 `IDropTarget` 仍在；应用内拖到自己身上要靠命中测试丢掉，或像 ADB-Explorer 用私有格式。

## 与当前工作

- 能直接用：保持默认 `dragDropEnabled: true`；`@tauri-apps/api/event` 听 `tauri://drag-enter|over|drop|leave`；drop 时把 paths 交给文件 store。
- 必须改写：命中测试（当前模块是 files、落点在列表/面包屑、有选中设备）；目录与多文件；SafetyRoot 仍在 core。hover 用现有 token 做 accent 虚线框，不要新做一套 DnD 组件库。
- 不要用：`ondrop` + `e.dataTransfer.files` 当主路径；为了 HTML5 关掉 Tauri drop；在 UI 里读 `IDataObject`。

## 阅读范围

`src/webview2/drag_drop.rs`（全文）、`src/webview2/mod.rs`（`SetAllowExternalDrop(false)`）、`src/lib.rs`（`DragDropEvent`、`with_drag_drop_handler` 平台说明）、`examples/simple.rs`。未读 macOS/Linux 后端（与 Yohu 目标平台无关）。
