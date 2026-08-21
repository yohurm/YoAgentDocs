---
id: rules.type.harmonyos
type: rule
status: active
severity: should
scope: type
when: always
when_to_use: HarmonyOS / OpenHarmony 应用或需对标鸿蒙设计的实现
related: [rules.type.android, rules.type.ui-kit]
---

# HarmonyOS 类型包

## 默认边界

- 设计与交互以官方文档为基准。本机若已有文档树（团队常用 `E:\Dev\Doc\HarmonyOS-Developer-docs`），**先读本地全文相关章节**，再补网上过期摘要。
- 组件能力、动效时长与状态（默认 / 按压 / 禁用 / 内容区裁剪等）按规范实现，而不是只接同名 API。

## UI 与实现分层

ArkUI 页面与自定义组件是展示层。仅当该工程采用官方推荐的状态与视图分离时，状态提升到**页面状态层**（可用官方用语，不要默认改名为 Android ViewModel）。自定义组件不直访数据源。

组件能力走 [ui-kit](../ui-kit/) L0–L5。禁止用 Android 控件习惯直接替换 ArkUI 分层而不说明映射。对照 [stack-layering.md](../../common/stack-layering.md)。

## 质量门禁

- 有真机或可运行设备时：安装对照官方示例与设计稿，用截图核对关键状态。
- 无设备时：对照文档列出已实现 / 未实现的状态，并标明未实机。

## 不要默认引入

- 用 Android 控件习惯直接替换 ArkUI 分层，而不说明映射关系。
