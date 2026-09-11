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
| [microsoft/vscode 选区](microsoft--vscode-selection.md) | 编辑器选区是模型 (line, column)；复制 `getValueInRange` | reuse-pattern |
| [JetBrains/android logcat](JetBrains--android.md) | 快捷键绑在日志宿主；过滤栏与编辑器分焦点；复制结构化消息 | reuse-pattern / adapt |
| [JetBrains/android 文档选区](JetBrains--android-selection.md) | Logcat 字段是 Document span，不是表格格 | reuse-pattern |
| [Chrome DevTools Console](ChromeDevTools--devtools-frontend.md) | 虚拟视口自管 `{item,offset}`；复制拼模型行 | reuse-pattern |
| [logdyhq/logdy-ui](logdyhq--logdy-ui.md) | Space 暂停；INPUT 放行；但 document 级监听不可搬到工作台 | reuse-pattern / anti-pattern |
| [amir20/dozzle](amir20--dozzle.md) | 壳 Ctrl+K vs 面板 Ctrl+F；复制走模型不是 DOM 选区 | reuse-pattern |
| [Alex4SSB/ADB-Explorer](Alex4SSB--ADB-Explorer.md) | Explorer 拖出虚拟文件：`FILEDESCRIPTOR`+`FILECONTENTS`，GetData 才 adb pull | adapt / anti-pattern |
| [Genymobile/scrcpy](Genymobile--scrcpy.md) | 投屏协议源 + 拖入 push；客户端按源码自写，不拉 `scrcpy.exe` | reuse-pattern / anti-pattern |
| [tauri-apps/wry](tauri-apps--wry.md) | Windows 上 wry 接管 `IDropTarget`，HTML5 drop 没有完整路径 | reuse-pattern / anti-pattern |
| [crabnebula-dev/drag-rs](crabnebula-dev--drag-rs.md) | Tauri 拖出骨架：`DoDragDrop`；载荷只认本机已有文件 | adapt / anti-pattern |
| [yume-chan/ya-webadb](yume-chan--ya-webadb.md) | scrcpy 协议 TS 包 + WebCodecs 解码；ADB 重实现不可抄 | adapt / anti-pattern |
| [barry-ran/QtScrcpy](barry-ran--QtScrcpy.md) | 工作台投屏按钮清单；独立 Qt 窗与 FFmpeg 不可搬 | lesson-only / anti-pattern |
| [NetrisTV/ws-scrcpy](NetrisTV--ws-scrcpy.md) | WebView 要独立二进制视频通道；fork server 是反例 | adapt / anti-pattern |
| [openharmony/window_window_manager](openharmony--window_window_manager.md) | 启动面是 leash 上的 surface；Scale/Opacity 在 Rosen，窗口布局已是终态 | reuse-pattern |
| [microsoft/Windows.UI.Composition-Win32-Samples](microsoft--Windows.UI.Composition-Win32-Samples.md) | Win32 HWND 只当 target；动画 Offset/Visual，不改窗口尺寸 | reuse-pattern |
| [lwouis/alt-tab-macos](lwouis--alt-tab-macos.md) | layer.actions = NSNull() 禁掉 frame 隐式动画 | reuse-pattern / anti-pattern |
| [LineageOS Bluetooth snoop](LineageOS--android_packages_modules_Bluetooth.md) | 设备侧 HCI：文件 + 8872 + 隐私环三条出口 | reuse-pattern / anti-pattern |
| [wireshark androiddump](wireshark--wireshark.md) | 本机讲 ADB 协议接 `tcp:8872`，先探端口再列出接口 | adapt / anti-pattern |
| [mauricelam/btsnoop-extcap](mauricelam--btsnoop-extcap.md) | root 后 `tail -F` 受保护文件；属性开关要重启蓝牙 | reuse-pattern / anti-pattern |
| [thelok1s/btsnoop-adb](thelok1s--btsnoop-adb.md) | Rust 流式解析 btsnoop → pcap；运输与解析切开 | reuse-pattern / adapt |
| [无线连接路径（主题）](android--wireless-connect-paths.md) | 无线调试 ≠ 经典 tcpip ≠ 厂商管家；Yohu 缺编排不缺协议 | reuse-pattern / anti-pattern |
| [LineageOS adb Wifi](LineageOS--android_packages_modules_adb.md) | TLS 口 + 配对 + 遗产 `tcpip:`；只调 sidecar | reuse-pattern / anti-pattern |
| [bk138/droidVNC-NG](bk138--droidVNC-NG.md) | 无 ADB 投屏要装包+录屏+无障碍；不当主路径 | lesson-only / anti-pattern |
| [Yohu 双应用 1420 撞车](yohu--desktop-dev-port-collision.md) | 两套 Tauri 共用 `devUrl:1420`；壳身份 ≠ 端口上的前端 | anti-pattern / lesson-only |

## 共同架构经验

### 多应用共用 devUrl（2026-09-10）

两套 Tauri 若都写 `devUrl: http://localhost:1420`，**壳身份和页身份会拆开**：`tauri dev` 启动的是当前 crate 的 exe，WebView 加载的却是端口上碰巧在跑的那个 Vite。本机实测文档预览壳吃进了 ADB Tools 前端。启动前必须先查端口属主和页面 `document.title`，不能只认 Cursor 工作区。详见 [Yohu 双应用 1420 撞车](yohu--desktop-dev-port-collision.md)。

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

### 日志文档选区（2026-09-10 增补）

Ctrl+C 的载荷是 **模型文档切片**，不是 `window.getSelection().toString()`。AS Logcat / VS Code / DevTools Console 三条链同构：指针或光标先落到文档坐标，再 slice。Dozzle 更极端——几乎不做拖选，复制直接 `rawMessage`。Yohu 要拖选，就必须自管 `DocRange`；CSS grid 原生延展会吞前列，不能当真相。详见 [日志清单文档选区](../ui-kit/desktop--log-list-selection.md)。

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

### Android 投屏（工作台模块，源码实现客户端）

约束修正：安装包 12 MB **不是**本模块硬限制。否决 `scrcpy.exe` 是因为要 **按源码自写客户端**、画面进 `YoPage` 面板，不是因为体积。设备侧仍用 **未修改** 的官方 `scrcpy-server`（隐藏 API / MediaCodec 不重写）。ADB 仍走 sidecar（ADR-v6-008）；scrcpy 线协议在 core 自写。

四份源码叠成 Yohu 该写的链：

```
@yohu/module-mirror (YoPage + canvas + WebCodecs)
  → @yohu/api  (mirror.start/stop/inject + 二进制帧通道)
  → yohu-app commands 薄转发
  → yohu-mirror::MirrorService     # 新 crate，对标 yohu-logsrv CaptureService
        槽位 Empty/Starting/Live/Stopping，generation 必达
        pin 的 scrcpy-server 版本 ↔ 帧头解析器版本 同一常量
  → yohu-adb  push / reverse|forward / app_process 长驻
        （第三种进程：活着 + TCP，不是 run_streaming 泵 stdout）
  → 设备 官方 scrcpy-server（Java，app_process）
```

| 决策 | 采用 | 来源 | 不采用 |
|------|------|------|--------|
| 设备编码器 | 官方 server jar，版本锁死 | scrcpy `server.c` / `develop.md` | fork `1.19-ws8`（ws-scrcpy） |
| 隧道 | reverse 优先，失败改 forward + dummy byte | scrcpy + ya-webadb `AdbScrcpyClient.start` | 设备上听 8886 WebSocket |
| 解码位置 | UI `VideoDecoder`（WebCodecs） | ya-webadb decoder | core FFmpeg 再塞像素进 WebView；SDL 窗 |
| 视频 IPC | 独立二进制通道，禁走 log 批量事件 | ws-scrcpy 的「必须有二进制路」 | MSE 封 MP4、WASM Broadway |
| 会话模型 | 每 serial 一路，掉线停槽 | 对齐 ADR-v6-016 / CaptureService | 每台再开 Tauri/Qt 窗口（QtScrcpy VideoForm） |
| 控制 | v1 可关整条 control socket（只读） | scrcpy `--no-control` | UI 里吞点击但 socket 仍开 |
| ADB | 现有 sidecar | ADR-v6-008 | `@yume-chan/adb` / WebUSB |

**Yohu 投屏显示功能清单（调研导出，供架构设计引用）**

P0 基线（模块从 Planned 转正式，源码闭环）：

1. `selectionMode=singleRequired`，页眉显示选中设备；无在线设备空态。
2. 启停投屏：core 槽位 + 任务中心登记；退出序列 3s 强杀进程树。
3. 画面嵌在模块 `YoPanel`：保持设备宽高比、适应面板（fit ≠ 改 `max_size`）。
4. 状态：未开始 / 启动中 / 直播 / 失败（带 server stdout）/ 设备掉线已停止。
5. 只视频、默认只读（`audio=false`，可关 control）。
6. 质量：`max_size` + `video_bit_rate`（设置或页眉，下次 start 生效）。
7. 截图：当前帧 canvas → 保存对话框（对标 QtScrcpy ToolForm / ya-webadb snapshot）。
8. sidecar：pin 官方 `scrcpy-server` 与解析器同版本；cleanup 删设备 jar。

P1 控制与产测常用：

9. 触控点击/滑动（坐标映射到设备像素）。
10. 导航键：Home / Back / App switch / Power / 音量。
11. 熄屏保持镜像、点亮屏幕。
12. 只读开关（关 control socket，不是忽略指针）。
13. 面板内全屏（F11 作用域在投屏页）。
14. 暂停画面（冻解码；采集是否停由产品定，默认对齐日志：冻 UI）。
15. reverse 失败时「强制 forward」可观察开关。

P2 远期：

16. 音频转发（Android 11+）。
17. 编码包 mux 录屏（MP4/MKV，解码前分流，对标 scrcpy recorder）。
18. 多设备并行投屏。
19. 剪贴板双向同步。
20. HID 键鼠、摄像头源、虚拟显示、OTG、游戏键位、撕出独立 OS 窗口。

明确不做：

- 拉起 `scrcpy.exe` / QtScrcpy 当模块。
- 重实现 ADB 协议或改 fork `scrcpy-server`。
- JPEG `screencap` 轮询。
- 投屏页再做 shell / 文件管理 / APK 安装。
- 视频帧走 logcat 风格逐行 invoke。
- MES。

官方 Android 文档管的是 `MediaCodec` / `Surface`（server 已用），没有「桌面投屏工作台」交互规范。产品对标 scrcpy 能力表 + QtScrcpy 工具条；视觉仍走 Yohu / 鸿蒙 PC token。

### Android HCI 日志（实时抓包，不是 logcat）

HCI 是 Host Controller Interface 二进制记录（btsnoop / RFC 1761 变体），与 `adb logcat` 不是一条流。官方规范：[Verify and debug](https://source.android.com/docs/core/connect/bluetooth/verifying_debugging)。四份源码叠成设备→主机完整链：

```
Controller ↔ Host
  → SnoopLogger::Capture                    # LineageOS/AOSP gd/hal
        persist.bluetooth.btsnooplogmode
        disabled | filtered | full | kernel
        │
        ├─ disabled → 内存 btsnooz（ACL≈14B）→ bugreport 里的残包
        ├─ full/filtered → 文件 /data/misc/bluetooth/logs/btsnoop_hci.log
        └─ 另：仅 debug 构建（本树）bind 127.0.0.1:8872
              新客户端先发 16 字节 "btsnoop\0" 头

主机三档（互斥，按探测结果选）：

A 套接字（无 root，但设备必须在听 8872）
  sidecar: adb -s SERIAL forward tcp:<local> tcp:8872
  → TcpStream 读 16 + 循环 24+N
  禁止抄 androiddump 的 host:transport / tcp:8872 线协议（ADR-v6-008）

B 文件尾随（adb root 或 su）
  adb shell [su -c] 'tail -F -c +0 /data/misc/bluetooth/logs/btsnoop_hci.log'
  → spawn_long_lived + 二进制泵（不能 stream_lines）
  先 dd/读 8 字节 magic，再开流

C 快照（无 root 保底，非实时）
  adb bugreport → zip 内 FS/data/misc/bluetooth/logs/btsnoop_hci.log
  未开 snoop 时只有截断的 btsnooz
```

| 决策 | 采用 | 来源 | 不采用 |
|------|------|------|--------|
| 运输 | sidecar `adb` + forward 或 shell | ADR-v6-008；extcap/btsnoop-adb | androiddump 自讲 5037 |
| 解析 | 独立 `BtsnoopReader`，运输可换 | btsnoop-adb / btsnoop-rs | 当 logcat 行 |
| 实时默认 | 先探测再承诺 | androiddump 查 `/proc/net/tcp` `22A8` | 假设每台都有 8872 |
| 文件路径 | `/data/misc/bluetooth/logs/...` | AOSP ParameterProvider | SafetyRoot / `yohu-files` |
| 会话模型 | 每 serial 一路，槽位对标 CaptureService | ADR-v6-016 | 塞进 logcat 同一环 |
| 展示 | v1 浅表（时间/方向/HCI 类型/长度）+ 导出 btsnoop | 工作台 | 内嵌 Wireshark；重做协议树 |
| 打开 snoop | `setprop` + 重启蓝牙，并提示会断 BT | extcap `BtsnoopLogSettings` | 静默改属性 |

**市售 `user` 机（项目真机 motorola edge 60 pro 这类）**：本树源码下 8872 不监听；`/data/misc` 无 root 拉不下来。**实时默认不可用**。能做的是：探测后说明原因；提供 bugreport 导出；userdebug/已 root/未来 Android 17 socket 开关再升到实时。

**Yohu 若做（调研导出，供架构设计引用）**

P0 能力探测 + 非实时保底：

1. 读 `getprop persist.bluetooth.btsnooplogmode` 与 `ro.build.type`。
2. 试 `adb forward` + 连 8872，或 `adb shell` 看蓝牙进程是否听 `22A8`。
3. `id -u` / `adb root` 探测文件档。
4. 三态：实时套接字 / 实时文件 / 仅 bugreport。空态写清缺哪一档。
5. `adb bugreport` 抽 `btsnoop_hci.log`，另存，不经 SafetyRoot。

P1 实时（仅探测通过时）：

6. 新 crate（对标 `yohu-logsrv` 槽位），**禁止**复用 `stream_lines`。
7. 解析 16+24+N，时间戳减 `0x00dcddb30f2f8000`。
8. 批量事件（二进制或已解码浅表），禁逐包 invoke。
9. 页眉可 `setprop full` + 重启蓝牙（明确警告）。

明确不做：

- HCI 当 logcat 过滤或同一 `LogLine` 环。
- 为 `/data/misc` 放开 SafetyRoot。
- 重实现 ADB 协议或捆绑 Wireshark。
- 把未开 snoop 的 btsnooz 残包当成全量 HCI。

### 无线连接（运输，不是新模块）

「无线调试」和「不开无线调试也能投屏」不是同一条需求。规范与源码叠成六条路，Yohu 只该做前三条里的运输编排：

```text
A USB ADB          现状。USB 调试
B 遗产 adb tcpip   USB 一次 → tcpip 5555 → connect。不要「无线调试」开关
C Android 11+ TLS  必须开无线调试；pair 口 ≠ connect 口；sidecar host:pair
D 厂商管家         BLE + Wi-Fi Direct + 系统签名。第三方抄不了
E Miracast         系统投屏，只看。不进 yohu-mirror
F 第三方 APK       MediaProjection + 无障碍。另一个产品
```

目录已经能解析 `tcp:`（`DeviceInfo.connection` / `is_tcp_connection` → wifi 编码 + 默认 forward）。源码没有 `adb pair` / `tcpip` / `connect`。设备一旦 Online，文件 / 日志 / 终端 / 投屏共用同一条 ADB，不必为无线再开模块。

| 决策 | 采用 | 来源 | 不采用 |
|------|------|------|--------|
| 不开无线调试的无线 | B：USB 一次 `tcpip` | scrcpy `server.c`；AOSP adb_wifi.md 遗产步骤 | 假装能开机常驻（未 root 做不到） |
| 不插 USB | C：用户开开关 + sidecar pair/connect | AOSP `commandline.cpp`；ya-webadb `wireless.ts` | 自己实现 SPAKE2 / A_STLS |
| 发现 | 先 `adb mdns check`，失败用手输 | 官方排障；Windows 常哑 | 假设局域网一定能看见 |
| 隧道 | `tcp:` 默认 forward | scrcpy `adb_tunnel.c`；Yohu 已有 | 无线上死磕 reverse |
| 零开发者选项 | 不做，或另开 Miracast 接收入口 | 权限模型；droidVNC 对照 | 逆向管家；主路径发 APK |

**Yohu 若做（调研导出）**：P0 设备栏 tcpip 向导；P1 配对码 + `adb pair`；P2 Android 17 受信网自动连。明细见 [主题笔记](android--wireless-connect-paths.md)。

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。

- windows-desktop 类型包可补：模块快捷键默认 `panel.contains(focus) && !isEditable`；Ctrl+A 在 listbox 上选模型，禁止整页 `selectAll`。
- 实现配方引用本层四篇，不要再从「AS 风格」口头对标跳过作用域。
- windows-desktop 类型包可补：WebView 拖入走壳 `CF_HDROP` 路径事件 + 命中测试，禁止当主路径用 HTML5 `DataTransfer.files`；拖出非本机文件必须 `FILEDESCRIPTOR`/`FILECONTENTS`，禁止先物化成 `CF_HDROP`。默认 Copy。OLE 不进 UI 层。
- windows-desktop 类型包可补：工作台投屏自写客户端（官方 scrcpy-server + core 隧道/帧头 + UI WebCodecs）；禁止以 `scrcpy.exe` 黑盒窗口代替模块；视频用独立二进制通道，不走 log 批量 IPC；禁止 fork server。
- windows-desktop 类型包可补：窗口 presence（启动交接、占用过渡）只动画合成器属性（DComp / 分层 HWND 冻结位图的 scale·opacity）。禁止每帧 `SetWindowPos` 改 HWND 宽高。主窗布局一次落到最终矩形。同屏共享容器；异屏淡入淡出，禁止跨屏共享几何。
- windows-desktop 类型包可补：设备 HCI 是独立二进制流（btsnoop），不是 logcat。实时分套接字档与 root 文件档，先探测再开采；无能力时只提供 bugreport 快照。运输走 sidecar adb，禁止自讲 ADB 协议。HCI 路径不进 SafetyRoot。
- windows-desktop 类型包可补：无线 ADB 是运输，不是新模块。优先 USB 一次 `adb tcpip`（不开「无线调试」开关）；Android 11+ 配对走 sidecar `adb pair`/`connect`/`mdns`，禁止重实现 TLS。`tcp:` 默认 forward。禁止把厂商管家或 Miracast 接到 scrcpy 槽。

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

**入选（投屏 4）**

- scrcpy：协议、隧道、只读、版本锁；`server.c` 是客户端编排蓝本。
- ya-webadb：协议/ADB/解码分家；WebCodecs 渲染；reverse 失败切 forward。
- QtScrcpy：工作台按钮与启动参数清单；反例是独立原生窗 + FFmpeg。
- ws-scrcpy：WebView 必须有二进制视频通道；反例是 fork server、PATH adb、模块耦死。

**落选**

- `klogg`：Qt C++ 大文件查看器，选区模型不可搬到 Solid 虚拟列表。
- `tstack/lnav`：TUI，键位是模态的。
- `intellij-community`：动作总线与 AS logcat 重复，体量过大。
- `logdyhq/logdy-core`：暂停/环已在架构引用；面板键位在 UI 仓。
- `facebook/flipper` Logs：产品形态不同，维护弱于上述四份。
- `barry-ran/QtScrcpy`：拖入主题下落选（与 scrcpy 同构）；**投屏主题已入选**（见上）。
- `Creeeeeeeeeeper/rust-ws-scrcpy`：Rust+WS 同构 ws-scrcpy，15 star，锁死单一 server 小版本。
- `bilbospocketses/ws-scrcpy-web`：ws-scrcpy 后继，生态不如 NetrisTV 原仓稳定，作对照不入库。
- `ganeshrvel/openmtp`：macOS + MTP，不是 Windows ADB。
- `T0biasCZe/AdbFileManager`：WinForms 按钮拷贝，无 OLE。
- `gerosyab/ADBCopy`：双栏 + Explorer 拖入，虚拟文件深度不及 ADB-Explorer。
- `electron/electron`：`startDrag` 与 drag-rs 同构且仓太大；只作对照不入库。

**入选（窗口动效 3，2026-09-08）**

- window_window_manager：系统启动页 / 窗口进出场接到 Rosen 表面。
- Windows.UI.Composition-Win32-Samples：Win32 合成器 Visual 动画。
- alt-tab-macos：桌面壳必须关掉 layout 隐式动画。

**落选（窗口动效）**

- WinUIEx SplashScreen：关 splash 再 Activate，无合成器过渡。
- JetBrains SplashManager：仓过大，且仍是 AWT 窗体尺寸，不是目标架构。
- `openharmony/arkui_ace_engine` 整仓再克隆：已有 Tabs/Dialog 笔记；本主题窗口层以 WMS 为准。

**入选（HCI 实时 4，2026-09-10）**

- LineageOS Bluetooth：设备侧唯一生产者；文件 / 8872 / btsnooz 三出口与 `user` 构建门闩。
- wireshark androiddump：无 root 套接字客户端；先探 `22A8`；反例是自讲 ADB 协议。
- btsnoop-extcap：root 文件尾随 + 属性开关；`tail -F` 不是 `cat`。
- btsnoop-adb：Rust 解析与运输分离，格式单测可搬。

**落选（HCI）**

- `mauricelam/btsnoop-rs`：格式库，已作为 extcap 依赖阅读，不单独立篇。
- AOSP `btsnooz.py`：本稀疏树未检出脚本；官方文档只把它当 bugreport 文本提取，不是实时。
- `devk-op/btsnoop-parser`：离线 Python，运输层为零。
- 把 HCI 并进现有 `JetBrains/android` logcat 篇：采集源不同，不能共用。

**入选（无线连接 3 + 1 主题，2026-09-10）**

- 主题笔记：把「无线调试 / tcpip / 管家 / Miracast / APK」拆开，直接回答 Yohu 能不能不开无线调试投屏。
- LineageOS/AOSP adb：TLS 与遗产口的协议源；`host:pair` 形状。
- scrcpy `--tcpip`（补进已有篇）：USB 一次开 5555，不开「无线调试」开关。
- droidVNC-NG：第三方无 ADB 投屏的真实代价（装包 + 每次录屏）。

**落选（无线连接）**

- 小米 / 华为 / OPPO 电脑管家：闭源，无协议可复用；只在主题笔记按权限模型对照。
- `saleehk/adb-wifi` 等二维码包装：只是 `adb pair` 的壳，不如直接读 AOSP + Studio 的 `WIFI:T:ADB` 格式。
- miraclecast / 自建 Miracast 栈：Linux sink；Windows 应走系统 `MiracastReceiver` 或「投影到此电脑」，且只看不控，不进 `yohu-mirror`。
- 把无线并进已有 scrcpy 投屏篇当「又一种编码」：运输问题，不是画质问题。
