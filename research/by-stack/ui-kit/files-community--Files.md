---
id: research.files-community-Files
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C#, XAML]
  frameworks: [WinUI3]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: files-community/Files
  url: https://github.com/files-community/Files
  head: 51bf632
  cloned_to: "%TEMP%/YoAgentResearch/files-community--Files"
studied_at: 2026-09-08
related: [research.desktop-workbench-command-band, research.desktop-winui-titlebar-chrome]
---

# files-community/Files

## 入选理由

Windows 上最完整的开源 WinUI 3 工作台之一。主窗口把 **标题铬、地址栏、文件命令栏、侧栏、内容、详情窗、状态栏** 拆成独立 Grid 行，直接回答「功能不要全挤进标题栏」。

## 项目是什么

开源文件管理器。无边框窗口 + 自绘标题区，内容是典型生产力壳。

## 架构

`MainPage.xaml` 顶层三行：

1. **TabBar**（窗口身份 / 多页）
2. **NavigationToolbar**（高 48：侧栏开关、前进后退、Omnibar 路径）
3. **SidebarView** 内容列，内部再拆：
   - **Inner Toolbar**（复制/新建等，作用在当前目录，不在窗口标题栏）
   - 文件列表
   - **StatusBar**（只读计数）
   - 可选 Info pane

地址栏是独立 `UserControl`，背景用 `App.Theme.AddressBar`，与 caption 分离。侧栏开关在地址栏左侧，不在系统三键行。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 地址/命令带独立成行 | reuse-pattern | URL、视图切换、窗格开关走 Command band |
| 文档操作留在内容区内工具条 | reuse-pattern | 导出/复制/外链 = Inner toolbar |
| 空内容区不塞营销 Hero | adapt | Home 可以安静；地址栏始终可输入 |
| 把 Omnibar 塞进 40px caption 与三键抢行 | anti-pattern | Files 刻意拆开 |

## 架构设计经验

窗口铬（拖拽、三键）和导航命令（路径、后退、侧栏）不是同一 Part。内容命令（复制、新建）又是第三 Part，贴着正在操作的对象。

## 与当前工作

- **能直接用：** 标题栏只留身份+三键；其下一行 Command band 放 URL 与窗格开关；画布头放导出/复制。
- **必须改写：** Files 有 TabBar，本应用单会话，不抄多标签。
- **不要用：** 把侧栏/大纲/原貌切换画进 `YoTitleBar`。

## 阅读范围

`src/Files.App/Views/MainPage.xaml`、`UserControls/NavigationToolbar.xaml`（高 48、三列：导航 / Omnibar / 右侧动作）、`Views/Shells/ModernShellPage.xaml` 中 StatusBar。2026-09-11 增补：`Files.App.Controls/Omnibar/{Omnibar.xaml,Omnibar.cs,Omnibar.Events.cs}`——整条是带边框 TextBox 铬；`GotFocus` 主动 `SelectAll`（Yohu 不要抄）；建议选中后 `Select(length, 0)` 光标在末尾；模式钮热区固定 46px。未读完整 ViewModel。
