---
id: research.desktop-winui-titlebar-chrome
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, CSS]
  frameworks: [WinUI3, VSCode-Workbench, SolidJS, Tauri]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://learn.microsoft.com/en-us/windows/apps/develop/title-bar
  repos:
    - microsoft/vscode
    - vuejs/vitepress
    - freeCodeCamp/devdocs
studied_at: 2026-09-08
related: [research.desktop-frameless-hero-layout, research.workbench-layout-multisource-unification, research.microsoft-vscode]
---

# WinUI 标题栏三键与工作台 Parts 铬层

## 背景

无边框 Windows 桌面应用必须自绘标题栏。常见失败：把最小化/最大化/关闭做成圆角工具按钮、放进带 padding 的 flex 组，导致悬停色块缩进、高度不满、关闭红不铺满。本主题对照 WinUI 3 官方标题栏文档与 VS Code workbench Parts。

## 关键结论

1. **系统三键是窗口铬，不是工具栏按钮。** WinUI `AppWindowTitleBar` 把 caption 区保留在窗口右上角；应用内容可以画在下面，但交互命中区必须是满高直角矩形。自绘时宽 46px、高 = 标题栏高、`border-radius: 0`、父级右侧 padding 为 0。
2. **关闭键颜色不走普通 hover token。** 官方：Close 的 hover/pressed 使用系统定义色，不应用 `ButtonHoverBackgroundColor`。自绘对应 `#c42b1c` 悬停、更深一档按下，图标变白。
3. **Tall 标题栏。** 标题栏里放搜索时用 `PreferredHeightOption.Tall`（文档示例 48px 行）。本仓库取 40px：三键仍拉满行高，搜索/分段器走独立 28px 槽，不要让三键跟着 28px 垂直居中成「浮块」。
4. **VS Code Parts：** `TITLEBAR_PART` / `SIDEBAR_PART` / `EDITOR_PART` / `AUXILIARYBAR_PART` / `STATUSBAR_PART` 分权。标题栏不画文档大纲；状态栏只承载只读指标。VitePress / DevDocs 同样：Chrome 与 Canvas 解耦，左专栏 / 中正文 / 右 TOC。
5. **反模式：** 模块内再实现一套 caption；工具按钮复用 46px caption 类；在 API 门面里对 IPC 做 mock 双轨。

## 与当前工作的关系

- **能直接用的：** 三键贴边满高；关闭系统红；标题栏 Parts 与预览模块分家；状态栏 22px 只读。
- **必须改写的：** Tauri 无系统 caption overlay，必须自绘 `WindowCaptionButtons`，但几何对齐 WinUI。
- **明确不要用的：** 把 caption 放进 `padding: 0 8px` 的 actions 行；`yo-titlebar__btn` 同时服务齿轮和关闭。

## 来源与阅读范围

- https://learn.microsoft.com/en-us/windows/apps/develop/title-bar（Caption buttons、Tall height、Close 色例外）
- VS Code `Parts` 枚举与 Layout 网格（先前 clone 结论，见 client-runtime/microsoft--vscode.md）
- VitePress / DevDocs 三栏分区（见已有 ui-kit 深研）
