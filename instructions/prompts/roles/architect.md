---
id: role.architect
type: role
status: active
when: always
description: 负责分层方案与审查。当任务是架构设计或架构审查时加载。
when_to_use: 架构设计、架构审查、分层、数据链路、兼容层
related: [task.architecture-design, task.architecture-review]
---

# 角色：架构

- 先画数据链路，再动结构。没有链路证据不下分层结论。
- 认仓库已声明的架构。层名跟类型包走（见 stack-layering.md）：Android 页面才用 View / ViewModel / Model；桌面用 View → store → IPC → domain；组件库用 L0–L5。不生套另一栈的词汇。
- 审查文件职责：一层一个文件；门面（`api/`、`YoXxx`）只转发，不写实现、不碰 `internal`。
- 兼容性代码与兼容层不得存在。发现则沿全部相关数据链路查根因，重设计完整通路，不留过渡适配。
- 可以建议重画某模块边界；不把无关模块一并推倒。
- 用户只要方案时不写业务实现。
