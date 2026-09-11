---
id: research.CommunityToolkit-Windows
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C#, XAML]
  frameworks: [WinUI3]
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: CommunityToolkit/Windows
  url: https://github.com/CommunityToolkit/Windows
  head: 413892f
  cloned_to: "%TEMP%/YoAgentResearch/CommunityToolkit--Windows"
studied_at: 2026-09-10
related: [research.files-community-Files, research.desktop-settings-form-row]
---

# CommunityToolkit/Windows

## 入选理由

Windows 设置页的官方控件实现（SettingsCard / SettingsExpander）。要回答「路径框该不该拉满右槽」必须看 Content 列怎么占位，不能只看 Gallery 壳。

## 项目是什么

Windows Community Toolkit。本篇只读 `SettingsControls` 的 SettingsCard。

## 架构

`SettingsCard.xaml` 根 Grid 四列：`Auto | * | Auto | Auto`。

- 列 0：图标
- 列 1 `*`：Header / Description（吃剩余）
- 列 2 `Auto`：`PART_ContentPresenter`（开关、下拉、路径簇）
- 列 3 `Auto`：可选 ActionIcon

默认：

- 卡片 `HorizontalAlignment=Stretch`
- `HorizontalContentAlignment=Right`
- ContentPresenter `Grid.Column=2` + `HorizontalAlignment=Right`

折行态（RightWrapped）才把 Content 改到下一行并 Stretch。那是窄宽回退，不是默认把固定宽输入悬在中间。

卡片资源里给 ToggleSwitch 默认右对齐，给 ComboBox / TextBox / Slider 只加 `SettingsCardContentMinWidth`，不加 `Width=*`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 内容列 Auto + 右对齐 | reuse-pattern | 右槽 hug；路径+浏览是簇，不是拉满后左放盒子 |
| 标题列吃剩余 | reuse-pattern | 对应 YoFormRow `__info flex:1`、`__control flex:0` |
| 折行才 Stretch | lesson-only | 窄宽回退可以拉满；宽行禁止制造空档 |
| 给 TextBox 写 Width=* 当默认 | anti-pattern | Toolkit 只设 MinWidth |

## 架构设计经验

设置行有两列契约：**信息 stretch、操作 hug**。把操作列改成 stretch、再给子控件写死 max-width，中间必然出现空洞，盒子会看起来居中。

## 与当前工作

- **能直接用：** YoFormRow 右槽永远 hug 贴尾。
- **必须改写：** 路径展示是只读盒，不是 WinUI TextBox；宽走 `--yohu-layout-settings-control-max`。
- **不要用：** `controlFill` / Tooltip `block` 把右槽拉满。

## 阅读范围

`components/SettingsControls/src/SettingsCard/SettingsCard.xaml`、`SettingsCard.Properties.cs`（`ContentAlignment` 默认 Right）。未读 SettingsExpander 项模板全文。
