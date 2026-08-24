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
- [../../rules/common/acceptance.md](../../rules/common/acceptance.md)

叠加能力类型包与平台类型包（如 `ui-kit` + `android`）。改已有代码时再读 [../../rules/modification/common.md](../../rules/modification/common.md)。目标是 Yo 组件或组件库时再 **MANDATORY READ** [../../../playbooks/yo-component.md](../../../playbooks/yo-component.md)。

## 目标

把约定能力做完整，通过类型包门禁，而不是只接一个 API。

## 输入

- 功能或组件。对标任务要有规范来源（或本机文档路径）。
- 实现手段不明 → 先 [online-deep-research.md](online-deep-research.md)。
- 模块边界不明 → 先 [architecture-design.md](architecture-design.md)。

## 步骤

按手册。动手前列出本轮可判定验收清单（用户条目 + 类型包门禁）。自驱动补齐状态（默认/按压/禁用/空态等），直到**全部**验收通过。任一未过：不总结、不结束对话，继续做。环境做不到的验证写明缺口，且不算完成。

## 完成标准

全部满足才允许总结、才允许结束对话：

- 本轮每条验收均为通过，并附证据。没有「先交 90%」。
- 行为符合规范或用户任务；关键状态（含空/错/禁用等清单内项）均已覆盖。
- 无新增散落硬编码与越级调用。
- 类型包门禁全部通过。环境缺口已写明且对应条目仍为未通过，本轮未完成。

## 不做

- 不把官方示例工程整份拷进仓库。
- 不在实现里保留兼容性代码、兼容层或「临时兼容」双轨 API。
