---
id: task.implement
type: task
status: active
when: new-work
description: 按规范与已有分层实现功能或组件。用户说开发、对齐鸿蒙/iOS、做完整、全量实现、加上某组件时使用。路径不明则先深研，分层未定则先架构设计。
when_to_use: 开发功能、对齐官方设计、全量实现、做组件
triggers: [开发, 实现, 对齐, 全量, 做组件, 加上, 完整功能]
inputs: [要做的功能, 可选：对标文档]
outputs: [实现, 类型包验证证据]
related: [role.implementer, playbook.implement]
---

# 任务：实现功能

**MANDATORY READ**

- [../roles/implementer.md](../roles/implementer.md)
- [../../../playbooks/implement.md](../../../playbooks/implement.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../rules/common/design-sources.md](../../rules/common/design-sources.md)
- [../../rules/common/quality.md](../../rules/common/quality.md)

叠加能力类型包与平台类型包（如 `ui-kit` + `android`）。改已有代码时再读 [../../rules/modification/common.md](../../rules/modification/common.md)。目标是 Yo 组件或组件库时再 **MANDATORY READ** [../../../playbooks/yo-component.md](../../../playbooks/yo-component.md)。

## 目标

把约定能力做完整，通过类型包门禁，而不是只接一个 API。

## 输入

- 功能或组件。对标任务要有规范来源（或本机文档路径）。
- 实现手段不明 → 先 [online-deep-research.md](online-deep-research.md)。
- 模块边界不明 → 先 [architecture-design.md](architecture-design.md)。

## 步骤

按手册。自驱动补齐状态（默认/按压/禁用/空态等），直到门禁满足或环境不具备（须写明）。

## 完成标准

- 行为符合规范或用户任务。
- 无新增散落硬编码与越级调用。
- 类型包验证已做或已声明缺口。

## 不做

- 不把官方示例工程整份拷进仓库。
- 不在实现里保留「临时兼容」双轨 API。
