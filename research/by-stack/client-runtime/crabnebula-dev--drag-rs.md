---
id: research.crabnebula-dev-drag-rs
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [Rust]
  frameworks: [Win32 OLE, Tauri plugin]
also_relevant: [windows-desktop]
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: crabnebula-dev/drag-rs
  url: https://github.com/crabnebula-dev/drag-rs
  cloned_to: "%TEMP%/YoAgentResearch/crabnebula-dev--drag-rs"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# crabnebula-dev/drag-rs

## 入选理由

Tauri/WebView **没有** Electron 的 `webContents.startDrag`。要从窗口把文件拖到 Explorer，必须自己 `DoDragDrop`。这是目前和 Tauri 2 对齐的 Rust 实现（README 写明测过 wry 0.46 / Tauri 2），也是 Electron 同构 API 的小仓替代（不必浅克隆整个 Electron）。

它证明了「拖出在 WebView 里必须走原生」，也证明了 **它解决不了 Android 文件**：Windows 上只做已存在路径的 `CF_HDROP`。

## 项目是什么

`crates/drag`：跨平台 `start_drag(window, item, icon, callback)`。`crates/tauri-plugin-drag`：前端 `startDrag({ item: ['/abs/path'], icon })`，命令在主线程跑（`DoDragDrop` 是模态循环）。

## 架构

```
JS  dragstart → preventDefault → invoke start_drag
        ▼
plugin  run_on_main_thread
        ▼
drag::start_drag  (Windows)
  OleInitialize
  ILCreateFromPathW → SHCreateShellItemArrayFromIDLists
        → BindToHandler(BHID_DataObject)     // 真文件的 IDataObject
  IDragSourceHelper.InitializeFromBitmap     // 拖影
  DoDragDrop(COPY|MOVE)                      // 阻塞到松手
        ▼
callback Dropped | Cancel + 光标屏坐标
```

Windows 限制（`lib.rs` / `platform_impl/windows/mod.rs`）：

- `DragItem::Files`：路径必须绝对且 **已经在磁盘上**。用 Shell Item Array，本质是 `CF_HDROP`。
- `DragItem::Data`：**不支持**。会拖当前目录的 dummy，并在松手时 Cancel。
- 没有 `FileGroupDescriptor` / `FileContents` / `IAsyncOperation`。
- `DoDragDrop` 阻塞调用线程，所以 plugin 才要 `run_on_main_thread` + Channel 回结果。

对照 Electron：`startDrag({ file, icon })` 同样要求本地已有文件；虚拟文件两边都没做。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 前端 `dragstart` + `preventDefault` + 原生 `DoDragDrop` | adapt | WebView 不能往 `DataTransfer` 里放真文件 |
| 主线程模态 + 完成回调 | adapt | 不能在 tokio worker 里 `DoDragDrop` |
| 拖影用 `IDragSourceHelper` | lesson-only | 有则更好；v1 可系统默认光标 |
| 直接 `startDrag(androidPaths)` | anti-pattern | 路径不在 NTFS 上，Shell Item 会失败或拖出空目标 |
| 先全部 pull 到 temp 再交给 drag-rs | anti-pattern | 等于 CF_HDROP；大文件卡死拖动手势，取消语义烂，正是 ADB-Explorer #240 的根因 |
| 引入整个 `tauri-plugin-drag` 当文件模块 | anti-pattern | 多一个插件权限面；Yohu 只要 Win32 这一段，且要虚拟文件 |

## 架构设计经验

- **拖出的 API 形状可以抄，载荷不行。** `start_drag(window, items, icon)` 对；`items = Vec<PathBuf>` 只适合本机文件。Android 条目必须是「名字 + 延迟流」，即 ADB-Explorer 的 descriptor。
- OLE 拖出是 **同步模态**。进度必须靠已有的 `transfer.progress` 事件，不能指望 `start_drag` 返回后再发进度。
- 与 wry 拖入共存：拖出期间本窗仍是 `IDropTarget`。落到自己身上要丢弃，或只接受 Explorer 来的 HDROP。

## 与当前工作

- 能直接用：交互骨架（行 `draggable`、`dragstart` preventDefault、invoke、Copy 模式）；`DoDragDrop` 必须在 UI/主线程。
- 必须改写：`IDataObject` 换成 `FILEDESCRIPTOR`+`FILECONTENTS`，`GetData` 调 `TransferRunner` pull；目录展开；Windows 非法名过滤；取消不删 Android 源。
- 不要用：把 `drag` crate 当完整方案；`DragItem::Data`；Move 模式；为拖出关掉 wry 的拖入 handler。

## 阅读范围

`crates/drag/src/lib.rs`（`DragItem` 平台限制）、`crates/drag/src/platform_impl/windows/mod.rs`（全文：`IDropSource`、`CF_HDROP` DataObject、`get_file_data_object`）、`crates/tauri-plugin-drag/src/commands.rs`、`examples/wry-dragout/src/main.rs`。未读 GTK/macOS 后端。
