---
id: research.desktop-settings-form-row
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C#, XAML, TypeScript]
  frameworks: [WinUI3, HarmonyOS]
also_relevant: []
utilization: [reuse-pattern, anti-pattern]
source:
  platform: other
  repo: desktop/settings-form-row
  url: https://learn.microsoft.com/en-us/windows/apps/design/app-settings/guidelines-for-app-settings
  cloned_to: "%TEMP%/YoAgentResearch/CommunityToolkit--Windows"
studied_at: 2026-09-10
related: [research.CommunityToolkit-Windows, research.files-community-Files]
---

# 桌面设置行：右槽是簇，不是拉满轨道

## 入选理由

Yohu 设置页路径框悬在行中、不贴「浏览」。需要对照 WinUI 设置规范、SettingsCard 源码、Files 设置页，以及鸿蒙列表「内容左、操作右」。

## 项目是什么

主题笔记，不是单仓。源码对照 CommunityToolkit SettingsCard + Files 设置页。

## 架构

规范（WinUI App settings）：SettingsCard 上 Header / Description 在左，**action control 在卡片右侧**。

SettingsCard 实现：根 Grid `Auto | * | Auto | Auto`，Content 在 Auto 列且 `HorizontalAlignment=Right`。卡片拉满，操作不拉满。

Files `Views/Settings/*.xaml`：设置项把 ComboBox / ToggleSwitch 直接放进 SettingsCard.Content，没有再包一层 stretch 轨道。DevTools 的 IDE 路径 + Browse 是**编辑表单**（竖叠 Stretch），不是紧凑设置行，不能抄到 YoFormRow。

鸿蒙设置项常见 `Row { 标题; Blank(); extra }`：Blank 吃中间，extra 整簇在尾。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 信息 stretch / 操作 hug | reuse-pattern | 与 SettingsCard 列定义同构 |
| 路径+浏览同一簇 | reuse-pattern | 簇内只有 gap，贴行尾 |
| 右槽 stretch + 子级 max-width | anti-pattern | 实测路径盒与浏览空 122px，盒中心在行宽 57% |
| 把编辑表单的 Width=* 抄进设置行 | anti-pattern | Files DevTools 编辑态才 Stretch |

## 架构设计经验

右槽只有一种契约：hug 贴尾。需要「路径够读」时给路径盒自己的上限，不要把槽拉宽再把盒子放进去。Tooltip `block`（`width:100%`）和 `controlFill` 是同一条错误链。

## 与当前工作

- **能直接用：** 删 `controlFill`；路径 Tooltip 不再 `block`；路径盒 `flex: 0 1` + 上限 360。
- **必须改写：** 路径是只读展示，不是可编辑 TextBox。
- **不要用：** 页面再给 `.yohu-text-field` 或路径盒写一行 stretch。

## 阅读范围

SettingsCard.xaml 列定义与 PART_ContentPresenter；Files `GeneralPage.xaml` / `DevToolsPage.xaml`；WinUI 设置指南。未跑 Files 真机。
