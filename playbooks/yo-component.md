---
id: playbook.yo-component
type: playbook
status: active
when: new-work
when_to_use: 在组件库中新增或重做 Yo 组件时
related: [rules.type.ui-kit, task.implement]
---

# 手册：Yo 组件

1. **MANDATORY READ** [ui-kit/README.md](../instructions/rules/by-type/ui-kit/README.md) 及其列出的分层、职责、生命周期、状态所有权、耦合文档。
2. 对标任务先读官方文档（[design-sources.md](../instructions/rules/common/design-sources.md)）。路径不明则先联网深研。
3. 写出该组件的 L0–L5 文件清单、生命周期阶段（create → attach ↔ detach → destroy）与状态所有者（模型 / 配置 / 策略 / 视图瞬态），再编码。共享能力已有则引用，不要复制。L5 = 接口/薄转发；几何与绘制进 L1 `XxxImpl` + `internal`。
4. 实现后走 [checklists/yo-component.md](../instructions/checklists/yo-component.md) 与平台类型包验证。资源释放、再 bind、destroy 后失败路径一并验证。
5. 宿主只改调用 Yo API，不在模块里补第二套样式。
