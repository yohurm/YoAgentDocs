---
id: research.yohu-desktop-dev-port-collision
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [typescript, rust]
  frameworks: [tauri, vite]
also_relevant: []
utilization: [anti-pattern, lesson-only]
source:
  platform: other
  repo: yohu/desktop-dev-surfaces
  url: file:///D:/A_yoprogram/Github_Gallery
studied_at: 2026-09-10
related: []
---

# Yohu 桌面双应用：dev 端口与启动面

## 入选理由

本机同时跑两套 Tauri 工作台，**都把 Vite 钉在 `1420`**。Agent 只认当前 Cursor 工作区 crate 名去 `tauri dev`，会把别人的前端装进自己的壳。这是启动面调研，不是功能实现。

## 项目是什么

两套独立产品，壳结构几乎同构：

| 层 | YoDocPreview（文档预览） | Yohu ADB Tools |
|----|--------------------------|----------------|
| 仓库 | `Windows-YoWebDocPreview` | `Yovo-Windows-ADBTools-tauri` |
| 产品名 | `YoDocPreview` | `YohuAdbTools` |
| 二进制 | `yohu-docpreview.exe` | `yohu-adbtools.exe` / `YohuAdbTools` |
| identifier | `com.yohu.docpreview` | `com.yohu.adbtools` |
| LocalAppData 产品目录 | `YoDocPreview`（旧名 `YoWebDocPreview`） | `YohuAdbTools` |
| WebView 用户数据 | `com.yohu.docpreview`（旧：`com.yohu.webdocpreview` / `com.yovo.webdocpreview` / `com.yoweb.docpreview`） | `com.yohu.adbtools` / `com.yovo.adbtools` |
| Vite `index.html` 标题 | `YoDocPreview` | `Yohu ADB Tools` |
| `devUrl` | `http://localhost:1335`（原 1420，已让出） | `http://localhost:1420` |
| Vite port | `1335` | `1420` |
| `beforeDevCommand` | 无（要另开 `pnpm --filter @yohu/shell dev`） | 有：`pnpm dev`（cwd `ui`） |

## 架构

```
用户看见的窗口
  ├─ 原生壳（Tauri WebView，身份 = exe / productName / identifier）
  └─ 前端页面（Vite 或 dist，身份 = 谁占用了 devUrl 端口）

本机 2026-09-10 实测：
  Listen 127.0.0.1:1420
    → node …\Yovo-Windows-ADBTools-tauri\ui\apps\shell\…\vite.js
  Chrome DevTools MCP 当前页
    → 标题「Yohu ADB Tools」URL http://localhost:1420/
  Cursor 工作区
    → Windows-YoWebDocPreview
  文档预览 Vite 终端（319788）
    → 仍写「running on 1420」，但 pid 24116 已不在
```

数据链路：

```
设计前（Agent 误判）：
  当前工作区 = Windows-YoWebDocPreview
  → 清 %LOCALAPPDATA%\YoDocPreview + target\release
  → tauri.js dev（cwd app/yohu-docpreview）
  → cargo run target\debug\yohu-docpreview.exe
  → 窗口加载 tauri.conf.devUrl = :1420
  → 1420 实际是 ADB Tools 的 Vite
  → 用户看到的是 ADB Tools 页面（或套在文档预览壳里的 ADB Tools）

设计后应认的启动面（自底而上）：
  1. 端口：谁 Listen 1420（Get-NetTCPConnection → Win32_Process.CommandLine）
  2. 页面：Chrome / iframe 的 document.title（与 index.html 对齐）
  3. 壳：正在跑的 exe Path（yohu-docpreview vs yohu-adbtools）
  4. 数据：%LOCALAPPDATA%\<DATA_DIR_NAME> 与 identifier，旧身份要单列
  5. 产物：该仓库 target\release，不是隔壁仓库
```

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 端口是前端身份，exe 是壳身份 | anti-pattern | 只启动 crate 不够；`devUrl` 撞车时壳和页会拆成两个产品 |
| 清数据按 identifier 族，不按「听起来像」 | lesson-only | 文档预览有 `YoWebDocPreview` / `com.yovo.webdocpreview` 旧目录 |
| Chrome DevTools 页标题优先于工作区名 | reuse-pattern | MCP `list_pages` 已写明当前页是哪个应用 |

## 架构设计经验

1. **两个 Tauri 应用不能共享同一个 `devUrl` 端口。** 后启动的 Vite 要么起不来，要么把先到的挤掉；`tauri dev` 仍会去加载那个 URL。
2. **「dev 窗口」在本机有三条面，必须先选面再启动：**
   - Chrome → `http://localhost:1420/`（历史联调面，看的是端口上的前端）
   - 文档预览原生壳 `yohu-docpreview.exe`（要自己的 Vite，且 1420 空闲或改端口）
   - ADB Tools 原生壳 `yohu-adbtools.exe`（`beforeDevCommand` 会自己拉 Vite）
3. **Release 与系统应用目录跟产品走，不跟 Cursor 工作区走。** 文档预览 Release 在 `Windows-YoWebDocPreview\target\release`；ADB Tools Release 在 `Yovo-Windows-ADBTools-tauri\target\release`，数据在 `YohuAdbTools`。

## 与当前工作

- 能直接用：启动前先查 1420 属主 + `list_pages` 标题。
- 必须改写：不能再假设「当前仓库 = 用户眼前的 dev 窗口」。
- 不要用：在 1420 已被占用时对另一产品执行 `tauri dev`。

## 阅读范围

- `Windows-YoWebDocPreview/app/yohu-docpreview/tauri.conf.json`
- `Windows-YoWebDocPreview/ui/apps/shell/{vite.config.ts,index.html}`
- `Windows-YoWebDocPreview/core/yohu-protocol/src/identity.rs`
- `Windows-YoWebDocPreview/app/yohu-docpreview/src/{lib.rs,paths.rs}`
- `Yovo-Windows-ADBTools-tauri/app/yohu-adbtools/tauri.conf.json`
- `Yovo-Windows-ADBTools-tauri/ui/apps/shell/{vite.config.ts,index.html}`
- 本机端口、进程、`%LOCALAPPDATA%`、Chrome DevTools `list_pages`

未读：ADB Tools 业务模块与文档预览主题实现（与启动面无关）。
