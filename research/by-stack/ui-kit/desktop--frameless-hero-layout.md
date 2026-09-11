---
id: research.desktop-frameless-hero-layout
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, CSS]
  frameworks: [SolidJS, Tauri, Windows11-Fluent, Raycast, Arc]
also_relevant: [client-runtime, frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  url: https://learn.microsoft.com/en-us/windows/apps/design/
  repos:
    - microsoft/fluentui
    - raycast/extensions
    - arc-browser
studied_at: 2026-09-08
related: [research.workbench-layout-multisource-unification, research.synthesis.ui-kit]
---

# 现代 Windows 桌面应用交互规范：初始空态、无边框标题栏三键与单点拓扑

## 背景与核心准则
1. **禁止硬编码默认文档**：顶级桌面生产力工具（VS Code, Arc, Raycast, Linear）严禁在启动时默认填入并自动拉取某具体文档。启动必须是零依赖、TTI < 16ms 的纯粹初始空态（Hero State），由用户掌控意图。
2. **沉浸式标题栏与标准三键**：
   - Windows 11 标准标题栏高度 40px，背景融于亚克力分层，默认 drag，交互元件与三键严格 no-drag；
   - 窗口控制三键（最小化、最大化/还原、关闭）：宽度严格 46px，高度 100%，直角（无圆角，确保甩至右上角可准确触发），关闭按钮悬浮色微软标准红 #e81123，Segoi Fluent 1:1 矢量路径。
3. **初始空态（Hero State）矩阵**：
   - 居中微发光 Command Center 输入框；
   - 剪贴板自感知胶囊（检测到链接一键直达）；
   - Bento Grid 官方技术文档精选预设卡片；
   - 底部极客快捷键清单。
4. **状态自然跃迁**：
   - 未载入文档：Hero 欢迎台全屏展示；
   - 载入文档后：平滑切入三栏工作台（左侧专栏树 + 中央全宽黄金画布 + 伴生大纲列表 + 24px 静默底栏）。
