---
id: task.architecture-review
type: task
status: active
when: modify
description: 审查已有代码的分层与质量。用户说审查架构、重构分层、硬编码、补丁、兼容层、旧架构打补丁/加固、越级、MVVM、数据链路、高内聚、单一职责、文件堆职责、API 里写了实现、耦合 internal 时使用。不得在旧架构上继续修补；不得把范围内问题标成非阻塞而留下半成品。
when_to_use: 架构审查、代码审查、重构分层、硬编码、补丁层、兼容层、旧架构打补丁、防御性加固、越级、MVVM、数据链路、单一职责、API 耦合实现、多范围循环复审、分阶段整理到干净
triggers: [架构审查, 代码审查, 重构, 硬编码, 补丁, 兼容层, 兼容性代码, 旧架构, 打补丁, 防御性加固, 越级, MVVM, 数据链路, 高内聚, 耦合, 单一职责, 堆在一个文件, API实现, api/, 循环复审, 多范围, 非阻塞, 半成品, 技术债, 技术债务]
inputs: [审查范围：模块或用户路径]
outputs: [覆盖文件, 设计前链路, 清单逐项证据, 问题清单（级别+文件:行+正确层）, CLEAN 或 HAS_ISSUES, 若修复则含通路证据]
related: [role.architect, playbook.architecture-review, experience.public.architecture-review-waves]
---

# 任务：架构审查

**MANDATORY READ**

- [../roles/architect.md](../roles/architect.md)
- [../../../playbooks/architecture-review.md](../../../playbooks/architecture-review.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../rules/common/stack-layering.md](../../rules/common/stack-layering.md)
- [../../checklists/architecture-review.md](../../checklists/architecture-review.md)

改代码时再读 [../../rules/modification/common.md](../../rules/modification/common.md)。需要重画边界则转 [architecture-design.md](architecture-design.md)。审查 Yo 组件或 `api/` 时再读：

- [../../rules/by-type/ui-kit/file-srp.md](../../rules/by-type/ui-kit/file-srp.md)
- [../../rules/by-type/ui-kit/public-api.md](../../rules/by-type/ui-kit/public-api.md)
- [../../rules/by-type/ui-kit/lifecycle.md](../../rules/by-type/ui-kit/lifecycle.md)
- [../../rules/by-type/ui-kit/state-ownership.md](../../rules/by-type/ui-kit/state-ownership.md)
- [../../rules/by-type/ui-kit/README.md](../../rules/by-type/ui-kit/README.md)

## 目标

对给定模块或用户路径做分层审查。除数据链路与越级外，**必须**检查：文件是否单一职责、公开 API 是否夹带实现、是否存在兼容性代码或兼容层。有问题在正确层修，不加包装。禁止在旧架构上继续打补丁或做防御性加固：先严格调研优秀设计，梳理完整数据链路，再重设计通路、重构问题并清理旧代码。发现兼容层则沿全部相关数据链路查根因，重设计完整通路。严禁猜测。范围内分层问题全部闭合才算干净；允许按所有权分阶段，但每一阶段必须把该切片整理到干净。**严禁残留技术债务**（兼容层、双轨、TODO/FIXME、跳过测试、例外清单、非阻塞挂账）。禁止残留「非阻塞」半成品。

## 输入

范围（模块或一条路径）。缺则先问，不扫全仓。

## 步骤

按手册。审查结论里单独列出：

1. 职责混杂的文件（一层以上塞进同一文件）。
2. API / 包入口 / 门面类中的实现代码或对 `internal` 的引用。
3. 兼容性代码 / 兼容层，以及沿相关数据链路查到的根因（若有）。

多范围时：按所有权切开并行只读；修复员互不改同一所有权；未通过全部落地后再对**全部**范围复审。报告六段与波次规则见手册。

未要求改代码时只出结论与证据，禁止改文件。

## 完成标准

审查清单与本轮验收**全部通过**才允许总结。缺一项就继续查链路，不要先交 90%。

- 审查清单可勾选（含文件职责、API 门面、无兼容层）。
- 未改代码也有设计前链路；已改则设计前/后都有。
- 宣称 API 干净时，能指出门面文件只做契约转发。
- 宣称无兼容层时，能指出相关链路已统一、无双轨残留。
- **多范围：** 同一复审波次内全部范围结论为 CLEAN 才允许结束。不得因单范围通过或自行缩范围而收口。CLEAN 不得附带未修的分层问题，不得用「非阻塞观察」跳过修复环。优先级只排本阶段动手顺序，P1/P2 同样必须闭合。
- 每一波复审重读当前树，不得沿用上一波 CLEAN。
- **分阶段：** 下一刀另声明范围（模块 + 链路 + 可判定验收）；本阶段相关链路清零后再收口。
- **严禁残留技术债务：** 本阶段范围内无兼容层/双轨、无 TODO/FIXME/HACK 替身、无跳过测试、无例外清单扩容、无非阻塞挂账。相关链路上的既有债务已清零。只审不改时存在上列任一项则不得标 CLEAN。

## 不做

- 不把「拆文件」做成无关模块的全仓重排。
- 不在 API 层用转发函数继续调用本文件里的算法来「看起来像门面」。
- 不在旧架构上继续修补、打补丁或做防御性加固。
- 不用删除包装代替重设计；不把兼容层留作「过渡」。
- 不把库缺口记成消费模块违规（消费方自写第二套除外）。
- 不把范围内分层问题写成非阻塞观察、后续建议、已知债务或「不影响收口」。
- 不新增例外清单、跳过测试或 TODO 注释来让本阶段看起来完成。
- 不引用 `experiences/private/` 来写公共结论。
