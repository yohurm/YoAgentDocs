---
id: research.desktop-workbench-command-band
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, C#, CSS]
  frameworks: [WinUI3, VSCode-Workbench, SolidJS, Tauri]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://learn.microsoft.com/en-us/windows/apps/develop/title-bar
  repos:
    - files-community/Files
    - microsoft/WinUI-Gallery
    - microsoft/vscode
  cloned_to:
    - "%TEMP%/YoAgentResearch/files-community--Files"
    - "%TEMP%/YoAgentResearch/microsoft--WinUI-Gallery"
studied_at: 2026-09-08
related: [research.desktop-winui-titlebar-chrome, research.files-community-Files, research.microsoft-WinUI-Gallery, research.microsoft-vscode]
---

# 桌面工作台：标题铬与命令带分区

## 背景

无边框文档预览器把 URL、原貌/排版/源码、侧栏、大纲、设置全部堆进 40px 标题栏，和系统三键抢行。需要对照 WinUI 标题栏规范、VS Code Parts、以及 Files 这种完整 WinUI 应用，重画区域。先前 [无边框 Hero](desktop--frameless-hero-layout.md) 里的营销空态（徽章、精选卡片、快捷键清单）不再采用。

## 关键结论

1. **Parts 分权（VS Code）。** `TITLEBAR_PART` 是窗口菜单与 caption；`SIDEBAR_PART` / `EDITOR_PART` / `AUXILIARYBAR_PART` / `STATUSBAR_PART` 各管一块。编辑器动作有独立 `EDITOR_ACTIONS_LOCATION`，默认不进标题栏。
2. **WinUI 标题栏是铬不是工具条。** 官方槽位：图标、标题、返回、窗格开关、可选 Content、RightHeader、系统三键。页面命令走 Frame 内 CommandBar。Gallery 把「搜控件」放进 TitleBar.Content，那是应用级搜索，不是会话工具。
3. **Files 把地址栏做成独立 48px 行。** 顶行 Tab/身份，下一行 NavigationToolbar（侧栏开关 + Omnibar），内容区内再一条 Inner Toolbar（复制/新建），底栏 StatusBar。这是文档/文件类应用该抄的带划分。
4. **空态保持安静。** 地址栏始终可输入；内容区不必 Hero 营销、精选卡片或产品徽章。VS Code 空编辑器和 Files 内容区都不靠大标题卖功能。
5. **反模式：** 工具按钮复用 caption 几何；URL 与关闭键同一行；启动读剪贴板；空态堆预设卡片。

```
TITLEBAR     身份 + 拖拽 + 三键
COMMAND BAND 侧栏开关 | URL | 载入 | 视图 | 大纲 | 设置
BODY         空  或  专栏 | 画布(含文档内工具条) | 大纲
STATUSBAR    只读指标
```

## 与当前工作的关系

- **能直接用：** 标题栏只显示当前文档标题（有文档时）和三键；Command band 常驻 URL；画布头保留导出/复制/外链；空内容区无文案堆砌。
- **必须改写：** Tauri 无系统 TitleBar 控件，Command band 自绘；不抄 Files 多标签。
- **明确不要用：** 精选官方文档网格、YOHU 徽章、营销副标题、把分段器放进 `YoTitleBar`。

## 来源与阅读范围

- https://learn.microsoft.com/en-us/windows/apps/develop/title-bar
- https://learn.microsoft.com/en-us/windows/apps/develop/ui/windows-app-sdk-app-structure
- Files `MainPage.xaml` / `NavigationToolbar.xaml`（clone `51bf632`）
- WinUI-Gallery `MainWindow.xaml`（clone `b1730cb`）
- VS Code `Parts` 枚举（先前深研 + layoutService 公开源）
- marktext 因 GitHub 连接中断未克隆；不假装读过其源码
