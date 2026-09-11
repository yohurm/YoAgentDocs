---
id: research.microsoft-WinUI-Gallery
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C#, XAML]
  frameworks: [WinUI3]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt]
source:
  platform: github
  repo: microsoft/WinUI-Gallery
  url: https://github.com/microsoft/WinUI-Gallery
  head: b1730cb
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--WinUI-Gallery"
studied_at: 2026-09-08
related: [research.desktop-workbench-command-band, research.desktop-winui-titlebar-chrome]
---

# microsoft/WinUI-Gallery

## 入选理由

官方 WinUI 3 壳样本：`TitleBar` 与 `NavigationView` 分两行，Gallery 自己也把搜索放进 TitleBar.Content。用来对照「规范允许什么」和「文档阅读器该不该照抄」。

## 项目是什么

WinUI 控件画廊。主窗口：Mica + TitleBar + NavigationView + Frame。

## 架构

`MainWindow.xaml`：

```
Grid
  Row0 TitleBar（Title、Back、PaneToggle、Content=AutoSuggestBox）
  Row1 NavigationView（IsBackButtonVisible=Collapsed，IsPaneToggleButtonVisible=False）
        Frame
```

Fluent 把 **返回、窗格开关** 算标题铬（与 NavigationView 绑定），把 **页面命令** 留在 Frame 里的 CommandBar。Gallery 把「搜控件」放进 TitleBar.Content，那是应用级搜索，不是文档会话工具。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| TitleBar 与 NavigationView 分 Grid 行 | reuse-pattern | 壳铬 ≠ 页面内容 |
| 页面 CommandBar 在 Frame 内 | reuse-pattern | 原貌/排版/导出属于页面 |
| Gallery 搜索进 TitleBar.Content | lesson-only | 样本应用的全局搜；文档 URL 不是全局搜 |

## 架构设计经验

官方 TitleBar 槽位是身份、返回、窗格、可选全局 Content。把会话级工具（视图模式、导出）塞进 TitleBar 会和 caption / 拖拽区抢命中。

## 与当前工作

- **能直接用：** 设置页返回键留在 TitleBar（对标 Back）。
- **必须改写：** 文档 URL 对标 Files 地址栏，不抄 Gallery 把搜索放进 caption 行。
- **不要用：** 预览模块再往 TitleBar.actions 堆分段器。

## 阅读范围

`WinUIGallery/MainWindow.xaml`、`Samples/TitleBar/TitleBarPage.xaml`。未读全部 NavigationView 代码后置。
