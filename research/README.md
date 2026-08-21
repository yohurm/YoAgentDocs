---
id: research.hub
type: hub
status: active
when: research
when_to_use: 查找联网项目总结或归档新深研时
related: [playbook.online-deep-research]
---

# 联网项目总结

开源深研产出。源码在系统 Temp，这里只有 Markdown。

这里的目录按**能力层**归档（实现了什么）。`android` / `harmonyos` / `windows-desktop` 是平台规则，不是调研分类；平台写在文档 frontmatter 的标签里即可。

**有第一篇再创建** `by-stack/<capability>/`，不要预建空文件夹。

| 能力层 | 含义 | 横向总结 | 文档 |
|--------|------|----------|------|
| agent | 编排、工具、记忆、多 Agent | 有第一篇后建 | — |
| llm-app | RAG、评测、模型接入 | | — |
| ui-kit | 可复用组件与设计系统 | [横向](by-stack/ui-kit/_synthesis.md) | [ace_engine 沉浸光感](by-stack/ui-kit/openharmony--arkui_ace_engine-immersive.md)、[Dialog 空间弹出](by-stack/ui-kit/openharmony--arkui_ace_engine-dialog-spatial.md)、[静态/动态模糊样本](by-stack/ui-kit/HarmonyOS_Samples--FuzzySceneOptimization.md)、[分层主题](by-stack/ui-kit/harmony--immersive-light-layers.md)、[graphics_effect](by-stack/ui-kit/openharmony--graphic_graphics_effect.md)、[SDF 边缘光](by-stack/ui-kit/harmony--sdf-edge-light.md)、[Rosen 点光源](by-stack/ui-kit/openharmony--graphic_graphic_2d-point-light.md)、[HdsTabs 点光源](by-stack/ui-kit/harmony--hdstabs-point-light.md)、[HdsTabs 动效](by-stack/ui-kit/harmony--hdstabs-motion.md)、[ace_engine Tabs 动效](by-stack/ui-kit/openharmony--arkui_ace_engine-tabs-motion.md)、[Spatialization 藏栏](by-stack/ui-kit/HarmonyOS_Samples--Spatialization.md)、[鸿蒙圆角](by-stack/ui-kit/harmony--corner-radius.md)、[Rosen RoundRect](by-stack/ui-kit/openharmony--graphic_graphic_2d-roundrect.md)、[iOS 连续圆角](by-stack/ui-kit/apple--continuous-corners.md)、[Figma squircle](by-stack/ui-kit/phamfoo--figma-squircle.md)、[Android smooth corner](by-stack/ui-kit/racra--smooth-corner-rect-android-compose.md)、[Fluent react-motion](by-stack/ui-kit/microsoft--fluentui.md)、[Radix Presence](by-stack/ui-kit/radix-ui--primitives.md)、[AG Grid 表头](by-stack/ui-kit/ag-grid--ag-grid.md)、[Spectrum Table](by-stack/ui-kit/adobe--react-spectrum.md)、[VS Code monaco-table](by-stack/ui-kit/microsoft--vscode-table.md) |
| frontend | Web / 页面应用 | | — |
| backend | API 与领域 | | — |
| data | 管道与分析 | | — |
| devops | CI 与交付 | | — |
| client-runtime | 桌面 / IDE / CLI / 应用壳 | [横向](by-stack/client-runtime/_synthesis.md) | [VS Code when 子句](by-stack/client-runtime/microsoft--vscode.md)、[AS Logcat 工具窗](by-stack/client-runtime/JetBrains--android.md)、[Logdy UI 暂停键](by-stack/client-runtime/logdyhq--logdy-ui.md)、[Dozzle 壳/面板分层](by-stack/client-runtime/amir20--dozzle.md)、[ADB-Explorer 虚拟文件拖拽](by-stack/client-runtime/Alex4SSB--ADB-Explorer.md)、[scrcpy 拖入 push](by-stack/client-runtime/Genymobile--scrcpy.md)、[wry WebView2 拖入](by-stack/client-runtime/tauri-apps--wry.md)、[drag-rs 拖出骨架](by-stack/client-runtime/crabnebula-dev--drag-rs.md) |
| security | 认证、密钥、供应链（仅当这是主贡献） | | — |
| other | 暂放；满 3 篇同类再考虑升格 | | — |

单项目命名：`<owner>--<repo>.md`。模板见 [templates/research/](../templates/research/)。
