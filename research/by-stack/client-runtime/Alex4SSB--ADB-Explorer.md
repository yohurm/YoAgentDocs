---
id: research.Alex4SSB-ADB-Explorer
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C#]
  frameworks: [WPF, Win32 OLE]
also_relevant: [windows-desktop]
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: Alex4SSB/ADB-Explorer
  url: https://github.com/Alex4SSB/ADB-Explorer
  cloned_to: "%TEMP%/YoAgentResearch/Alex4SSB--ADB-Explorer"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# Alex4SSB/ADB-Explorer

## 入选理由

Yohu 要的是 **Windows Explorer ↔ Android 文件/目录双向拖拽**。这是目前最接近的开源产品：WPF 文件管理器、官方 ADB sidecar、对 Explorer 暴露虚拟文件（不是先拷到磁盘再 `CF_HDROP`）。2026-08 仍在提交；issue #240 / #317 把空文件和「Move 取消丢源」钉死成设计约束。

官方 Shell 规范：[Handling Shell Data Transfer Scenarios](https://learn.microsoft.com/en-us/windows/win32/shell/datascenarios)、[Clipboard Formats](https://learn.microsoft.com/en-us/windows/win32/shell/clipboard)（`CFSTR_FILEDESCRIPTOR` + `CFSTR_FILECONTENTS`）。Raymond Chen：[Why is my delay-rendered format being rendered too soon?](https://devblogs.microsoft.com/oldnewthing/20070918-00/?p=25083) — `CF_HDROP` 只能描述**已经存在**的文件。

## 项目是什么

Windows 上的 Fluent ADB 文件管理器（WPF）。浏览、push/pull、剪贴板与 Explorer 拖拽共用同一套 `IDataObject`。Yohu 是 Tauri WebView，不能搬 UI，只搬 **Shell 数据对象契约**。

## 架构

```
Explorer / 剪贴板
        │  GetData(FileGroupDescriptorW / FileContents[lindex])
        ▼
VirtualFileDataObject  (IDataObject + IAsyncOperation)
        │  占位空 descriptor → 后台 PrepareDescriptors → 替换真实组
        ▼
FileClass.PrepareDescriptors
        │  目录递归成 FILEDESCRIPTOR 树
        │  Stream lambda：排队 PullFile → 等完成 → SHCreateStreamOnFileEx(temp)
        ▼
adb pull → %TempDragPath% → IStream (STGM_DELETEONRELEASE)
        ▼
Explorer 自己写目标路径（源不知道 drop 落点）
```

反向（PC → Android）：

```
Explorer IDataObject
  ├── AdbDrop（自家格式，设备内/设备间）
  ├── Shell ID List（ZIP / UNC / DLNA）
  ├── FileDescriptor + FileContents（虚拟文件）
  └── CF_HDROP（本机已有路径）→ VerifyAndPush → adb push
```

关键契约：

- **虚拟文件，不是 HDROP。** `FileGroupDescriptorW` 描述名字/是否目录/时间/大小；`FileContents` 按 `lindex` 给 `IStream`。Explorer 不知道也不需要 ADB 路径。
- **GetData 才拉。** `CreateDescriptorStream` 在 Explorer 真正要内容时才 `adb pull`。拖动过程中只准备描述符。拖拽未进入 `IAsyncOperation` 时拒绝 `TYMED_ISTREAM`，避免「刚按下就 pull」。
- **IAsyncOperation 恒为异步。** Explorer 可在后台抽数据，窗口不冻死。剪贴板没有 `EndOperation`，改听 `CFSTR_PERFORMEDDROPEFFECT` / `PasteSucceeded`。
- **占位 descriptor。** 先登记空 `FileDescriptor`/`FileContents`，文件夹树在后台算完再替换。若立刻给齐，Explorer 会把剪贴板内容读进内存。
- **目录 = 多条 descriptor。** 文件夹一项 `IsDirectory=true`，子项相对路径用 `\`。
- **默认 Copy。** `GiveFeedback` 在未按 Shift 时把 Move 改成 Copy。#317：Move 在 pull 完成后删 Android 源，Explorer 取消写目标后源已没了。
- **Windows 非法文件名直接不给内容。** 冲突名、回收站、非法字符 → `includeContent=false`。
- **自识别格式 `AdbDrop`。** 应用内/多窗口拖拽走这条，不经过 Explorer 虚拟文件。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `FILEDESCRIPTOR` + `FILECONTENTS` + `IAsyncOperation` | adapt | Android 文件在 drop 前不在本机；这是 Shell 规定的虚拟文件协议 |
| GetData 才 pull，拖动中不传字节 | adapt | 大文件、取消、只划过窗口都不能提前占带宽 |
| 默认 Copy，Move 要等目标确认再删源 | adapt | #317 是产品级缺陷，Yohu v1 只做 Copy |
| 目录展开成 descriptor 树 | adapt | `adb pull` 目录与 Explorer 文件夹拖入是同一模型 |
| 用窗口标题嗅探 Explorer 当前路径再直接 pull 到目标 | anti-pattern | 作者自己承认不可靠，已改回「temp + 流」；Yohu 不要做 |
| `STGM_DELETEONRELEASE` 当事务 | lesson-only | Explorer 取消后临时文件可能被一并删掉，不能当回滚 |
| 整份 WPF / Vanara / 多窗口 IPC | anti-pattern | Yohu 是单 WebView + IPC，只抄数据对象 |

## 架构设计经验

- Explorer **不会告诉源「用户把文件放到了哪个文件夹」**。源只能交出流，由目标写盘。想自己 `adb pull -a dest` 就必须另开保存对话框，那已经不是拖拽。
- `CF_HDROP` 在 hover 时就会被问到（用来决定 Paste 是否可用）。用它做「稍后 pull」会得到 0 字节文件（#240）。
- 拖出与拖入是两条 `IDataObject` 角色：源实现 `IDropSource`+虚拟内容；目标读 `CF_HDROP` / Shell ID List。不要合成一个「万能拖拽模块」。
- 取消、失败、Move 的删除时序必须写进契约，不能留给 OLE 默认行为。

## 与当前工作

- 能直接用：拖出协议（描述符树 + 延迟 `IStream` + 异步抽取）；拖入对 `CF_HDROP` 只取本机路径再走现有 `files.push`；v1 只允许 Copy。
- 必须改写：OLE 放在 `yohu-app` 命令薄层（或一小段 Win32），`GetData` 回调进已有 `TransferRunner`（进度事件、取消、`SafetyRoot`）。不要在 Solid 里解析 FORMATETC。临时目录用 `%LOCALAPPDATA%\YohuAdbTools\`，不要另起一套。
- 不要用：Explorer 路径侦测、应用内 `AdbDrop` 私有格式（Yohu 单窗）、APK 拖到窗口即 `adb install`（那是 scrcpy 的投屏语义，文件模块不是安装器）、Move 删除源。

当前 Yohu 缺口：`TransferRunner` `is_file()` 拒绝目录；UI 只有对话框，没有 OLE 源。拖出必须补虚拟 `IDataObject`，不能只接 `drag-rs`。

## 阅读范围

`ADB Explorer/Services/AppInfra/LowLevel/VirtualFileDataObject.cs`（`PrepareTransfer`、`GetData` 拒绝未开始的 ISTREAM、`IAsyncOperation`）、`FileDescriptor.cs` / `DataFormats.cs`、`Models/File/FileClass.cs`（`PrepareDescriptors`、`CreateDescriptorStream`）、`CopyPasteService.cs`（`UpdateDrag` 格式优先级、`AcceptDataObject` 四路）、`NativeMethods/DragDropNative.cs`（`DoDragDrop`、`GetComStreamFromFile`）、issue #240 / #317。未读归档压缩与多窗口 `IpcService` 细节。
