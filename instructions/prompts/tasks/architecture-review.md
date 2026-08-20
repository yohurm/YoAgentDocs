---
id: task.architecture-review
type: task
status: active
when: modify
description: 审查已有代码的分层与质量。用户说审查架构、硬编码、补丁、越级、MVVM、数据链路、高内聚低耦合时使用。
when_to_use: 架构审查、硬编码、补丁层、越级、MVVM、数据链路
triggers: [架构审查, 硬编码, 补丁, 越级, MVVM, 数据链路, 高内聚]
inputs: [审查范围：模块或用户路径]
outputs: [链路说明, 问题清单, 若修复则含通路证据]
related: [role.architect, playbook.architecture-review]
---

# 任务：架构审查

**MANDATORY READ**

- [../roles/architect.md](../roles/architect.md)
- [../../../playbooks/architecture-review.md](../../../playbooks/architecture-review.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../checklists/architecture-review.md](../../checklists/architecture-review.md)

改代码时再读 [../../rules/modification/common.md](../../rules/modification/common.md)。需要重画边界则转 [architecture-design.md](architecture-design.md)。

## 目标

对给定模块或用户路径做分层审查。有问题在正确层修，不加包装。

## 输入

范围（模块或一条路径）。缺则先问，不扫全仓。

## 步骤

按手册。未要求改代码时只出结论与证据。

## 完成标准

- 审查清单可勾选。
- 未改代码也有设计前链路；已改则设计前/后都有。
