---
id: research.synthesis.client-runtime
type: synthesis
status: active
when: research
stack:
  capability: client-runtime
---

# client-runtime 横向总结

## 本层已研项目

| 仓库 | 一句话 | 利用方式 |
|------|--------|----------|
| [microsoft/vscode](microsoft--vscode.md) | `when` 子句：`listFocus && !inputFocus` 才 Ctrl+A 全选行 | reuse-pattern |
| [JetBrains/android logcat](JetBrains--android.md) | 快捷键绑在日志宿主；过滤栏与编辑器分焦点；复制结构化消息 | reuse-pattern / adapt |
| [logdyhq/logdy-ui](logdyhq--logdy-ui.md) | Space 暂停；INPUT 放行；但 document 级监听不可搬到工作台 | reuse-pattern / anti-pattern |
| [amir20/dozzle](amir20--dozzle.md) | 壳 Ctrl+K vs 面板 Ctrl+F；复制走模型不是 DOM 选区 | reuse-pattern |
| [Alex4SSB/ADB-Explorer](Alex4SSB--ADB-Explorer.md) | Explorer 拖出虚拟文件：`FILEDESCRIPTOR`+`FILECONTENTS`，GetData 才 adb pull | adapt / anti-pattern |
| [Genymobile/scrcpy](Genymobile--scrcpy.md) | 拖入 = 路径入队 + `adb push`；明确不做拖出 | reuse-pattern / anti-pattern |
| [tauri-apps/wry](tauri-apps--wry.md) | Windows 上 wry 接管 `IDropTarget`，HTML5 drop 没有完整路径 | reuse-pattern / anti-pattern |
| [crabnebula-dev/drag-rs](crabnebula-dev--drag-rs.md) | Tauri 拖出骨架：`DoDragDrop`；载荷只认本机已有文件 | adapt / anti-pattern |

## 共同架构经验

### 快捷键焦点作用域

桌面工作台的快捷键是 **焦点作用域**，不是「这个模块还在树上」。四份实现用不同机制表达同一件事：

1. **VS Code**：上下文键（`inputFocus` / `listFocus` / `view == output`）。
2. **Android Studio Logcat**：Swing 焦点宿主 + `registerCustomShortcutSet(editor.contentComponent)`。
3. **Logdy**：`document.keydown` + 跳过 INPUT（单页查看器才够用）。
4. **Dozzle**：监听注册在具体组件上，卸载即失效；壳与面板分层。

Yohu 是「侧栏设备 + 模块页」的单 WebView。正确收缩是：

```
inLogsPanel  = 日志页根 contains(event.target)
inEditable   = INPUT / TEXTAREA / select / contenteditable
inLogsList   = inLogsPanel && !inEditable && 焦点在虚拟列表
```

| 键 | when | 动作 |
|----|------|------|
| Space | `inLogsPanel && !inEditable` | 暂停/继续可见区（采集继续） |
| Ctrl+A | `inLogsList` | 选中当前会话全部 **可见行**（模型 key，preventDefault） |
| Ctrl+A | `inEditable` | 原生选中输入框文字 |
| Ctrl+C | `inLogsList` 且有选中 | 复制列对齐文本 |
| Ctrl+F | `inLogsPanel` | 聚焦关键字（已在框内则选中内容） |
| Ctrl+L | `inLogsPanel && !inEditable` | 清空可见区 |
| Ctrl+T | `inLogsPanel && !inEditable` | 新建会话 |
| Ctrl+W | `inLogsPanel && !inEditable` | 关闭会话 |
| Ctrl+Tab | `inLogsPanel` | 循环会话 Tab |

对话框/重命名打开时，面板命令全部停（Logdy 的 modal 门闩）。

### Windows ↔ Android 文件拖拽

快捷键问题是焦点作用域；文件拖拽问题是 **OLE 角色分裂**。四份实现叠成 Yohu 该走的两条链（官方规范：[Shell data scenarios](https://learn.microsoft.com/en-us/windows/win32/shell/datascenarios)；Chen：[`CF_HDROP` 不能延迟生成文件](https://devblogs.microsoft.com/oldnewthing/20070918-00/?p=25083)）：

**拖入（Explorer → 当前文件夹 → `adb push`）** — 薄，对标 scrcpy + wry：

```
Explorer CF_HDROP
  → wry IDropTarget（整窗 paths + position）
  → tauri://drag-drop
  → 命中测试：文件模块 + 列表/面包屑
  → files.push({ serial, local, remote: session.path/name })
  → TransferRunner + SafetyRoot
```

保持 `dragDropEnabled` 默认 true。不要用 HTML5 `ondrop`（Windows 上无完整路径）。目录需要 `TransferRunner` 接受文件夹（今日 `is_file()` 会拒）。

**拖出（列表行 → Explorer → `adb pull`）** — 厚，对标 ADB-Explorer，**不能**只用 drag-rs：

```
dragstart + preventDefault
  → 原生 IDataObject: FileGroupDescriptorW + FileContents[lindex] + IAsyncOperation
  → Explorer GetData 之后才 TransferRunner.pull 到 temp/IStream
  → Explorer 自己写落点（源不知道目标路径）
```

drag-rs 只提供 `DoDragDrop` 骨架和「路径必须已在磁盘」的 `CF_HDROP`。先全量 pull 再拖 = #240 空文件。v1 只做 Copy；Move 在目标未确认时删 Android 源 = #317。

OLE 胶水留在 `yohu-app`；传输/安全根仍在 `yohu-files`。UI 只做命中、拖影、禁止态。

**全选不能走 `document.execCommand('selectAll')`。** 那会把侧栏、页眉、状态栏一起反白。虚拟列表只渲染可视行，必须 `selectedKeys = visible.map(rowKey)`。

**Space 不得与设备栏抢。** 设备栏 option 的 Space 是选设备；日志 `window` 监听会冒泡到暂停。捕获阶段在面板内消化，或 `!inLogsPanel` 直接 return。

官方鸿蒙文档（本机 `E:\Dev\Doc\HarmonyOS-Developer-docs`）只有 HiLog **打印** FAQ，没有日志面板交互规范。产品对标 AS Logcat + VS Code 面板作用域；视觉仍走 Yohu / 鸿蒙 PC token。

## 分歧与取舍

- **暂停语义**：AS / Logdy 会停采集或停推送。Yohu 需求与 ADR-v6-006 规定 Space 只冻 UI。不要抄服务端 pause。
- **日志宿主**：AS / VS Code Output 用编辑器文档，Ctrl+A 免费。Yohu 用 `YoVirtualList`，必须自己做多选/全选/复制（文件模块已有 `selectedKeys` + Shift/Ctrl 点击）。
- **复制全部 vs 全选**：Dozzle 只提供「复制过滤结果」按钮。Yohu 缓冲 10k，可以同时要 Ctrl+A 选可见区 + 导出走 core 环。
- **Ctrl+L**：浏览器里会撞地址栏，Dozzle 改用 Ctrl+Shift+L。Tauri WebView 无地址栏，保持需求中的 Ctrl+L，但必须面板作用域。
- **Ctrl+K**：留给壳命令面板（UI设计系统-v6 §3），日志不要占用。
- **拖入 vs 拖出**：scrcpy 只做前者；ADB-Explorer 两者都做但 UI 是 WPF。Yohu 必须分两期，不能把「拖拽」当成一个 IPC。
- **虚拟文件 vs HDROP**：Shell 规定非文件系统对象用 descriptor+contents。drag-rs / Electron `startDrag` 都停在 HDROP。Android 条目选前者。
- **drop 落点**：Explorer 不回传目标文件夹（ADB-Explorer 放弃窗口标题嗅探）。拖出不能变成「pull 到用户悬停的那个目录」——那是保存对话框的活。
- **本窗 drop**：wry 拖出时自身仍是 `IDropTarget`。落到自己身上应忽略，或只接受 Explorer 的 HDROP。

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。

- windows-desktop 类型包可补：模块快捷键默认 `panel.contains(focus) && !isEditable`；Ctrl+A 在 listbox 上选模型，禁止整页 `selectAll`。
- 实现配方引用本层四篇，不要再从「AS 风格」口头对标跳过作用域。
- windows-desktop 类型包可补：WebView 拖入走壳 `CF_HDROP` 路径事件 + 命中测试，禁止当主路径用 HTML5 `DataTransfer.files`；拖出非本机文件必须 `FILEDESCRIPTOR`/`FILECONTENTS`，禁止先物化成 `CF_HDROP`。默认 Copy。OLE 不进 UI 层。

## 入选与落选备忘

**入选（快捷键 4）**

- VS Code：when 子句与 `list.selectAll`，直接回答 Ctrl+A 作用域。
- JetBrains/android `logcat/`：产品对标的源码，说明「绑在宿主上」和复制消息。
- Logdy UI：架构已引用的暂停/跟随；反例是 document 监听。
- Dozzle：壳 vs 面板快捷键分层，复制不靠 DOM。

**入选（文件拖拽 4）**

- ADB-Explorer：唯一把 Android→Explorer 做成虚拟文件协议的活跃实现，含空文件/Move 丢源的反例。
- scrcpy：拖入最小闭环；用「明确不做拖出」划清边界。
- wry：Yohu 运行时的拖入实现，解释为什么 HTML5 drop 在 Windows 上不可用。
- drag-rs：Tauri 拖出 API 形状；用「只认本机路径」说明不能当 Android 方案。

**落选**

- `klogg`：Qt C++ 大文件查看器，选区模型不可搬到 Solid 虚拟列表。
- `tstack/lnav`：TUI，键位是模态的。
- `intellij-community`：动作总线与 AS logcat 重复，体量过大。
- `logdyhq/logdy-core`：暂停/环已在架构引用；面板键位在 UI 仓。
- `facebook/flipper` Logs：产品形态不同，维护弱于上述四份。
- `barry-ran/QtScrcpy`：拖入是 scrcpy 同构，无拖出。
- `ganeshrvel/openmtp`：macOS + MTP，不是 Windows ADB。
- `T0biasCZe/AdbFileManager`：WinForms 按钮拷贝，无 OLE。
- `gerosyab/ADBCopy`：双栏 + Explorer 拖入，虚拟文件深度不及 ADB-Explorer。
- `electron/electron`：`startDrag` 与 drag-rs 同构且仓太大；只作对照不入库。
