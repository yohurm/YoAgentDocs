---
id: task.architecture-review
type: task
status: active
when: modify
description: 审查已有代码的分层与质量。用户说审查架构、硬编码、补丁、兼容层、越级、MVVM、数据链路、高内聚、单一职责、文件堆职责、API 里写了实现、耦合 internal 时使用。
when_to_use: 架构审查、代码审查、硬编码、补丁层、兼容层、越级、MVVM、数据链路、单一职责、API 耦合实现
triggers: [架构审查, 代码审查, 硬编码, 补丁, 兼容层, 兼容性代码, 越级, MVVM, 数据链路, 高内聚, 耦合, 单一职责, 堆在一个文件, API实现, api/]
inputs: [审查范围：模块或用户路径]
outputs: [链路说明, 问题清单（含文件职责、API 越界、兼容层根因）, 若修复则含通路证据]
related: [role.architect, playbook.architecture-review]
---

# 任务：架构审查

**MANDATORY READ**

- [../roles/architect.md](../roles/architect.md)
- [../../../playbooks/architecture-review.md](../../../playbooks/architecture-review.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../checklists/architecture-review.md](../../checklists/architecture-review.md)

改代码时再读 [../../rules/modification/common.md](../../rules/modification/common.md)。需要重画边界则转 [architecture-design.md](architecture-design.md)。审查 Yo 组件或 `api/` 时再读：

- [../../rules/by-type/ui-kit/file-srp.md](../../rules/by-type/ui-kit/file-srp.md)
- [../../rules/by-type/ui-kit/public-api.md](../../rules/by-type/ui-kit/public-api.md)
- [../../rules/by-type/ui-kit/lifecycle.md](../../rules/by-type/ui-kit/lifecycle.md)
- [../../rules/by-type/ui-kit/state-ownership.md](../../rules/by-type/ui-kit/state-ownership.md)
- [../../rules/by-type/ui-kit/README.md](../../rules/by-type/ui-kit/README.md)

## 目标

对给定模块或用户路径做分层审查。除数据链路与越级外，**必须**检查：文件是否单一职责、公开 API 是否夹带实现、是否存在兼容性代码或兼容层。有问题在正确层修，不加包装。发现兼容层则沿全部相关数据链路查根因，重设计完整通路。

## 输入

范围（模块或一条路径）。缺则先问，不扫全仓。

## 步骤

按手册。审查结论里单独列出：

1. 职责混杂的文件（一层以上塞进同一文件）。
2. API / 包入口 / 门面类中的实现代码或对 `internal` 的引用。
3. 兼容性代码 / 兼容层，以及沿相关数据链路查到的根因（若有）。

未要求改代码时只出结论与证据。

## 完成标准

- 审查清单可勾选（含文件职责、API 门面、无兼容层）。
- 未改代码也有设计前链路；已改则设计前/后都有。
- 宣称 API 干净时，能指出门面文件只做契约转发。
- 宣称无兼容层时，能指出相关链路已统一、无双轨残留。

## 不做

- 不把「拆文件」做成无关模块的全仓重排。
- 不在 API 层用转发函数继续调用本文件里的算法来「看起来像门面」。
- 不用删除包装代替重设计；不把兼容层留作「过渡」。
