---
id: task.architecture-design
type: task
status: active
when: new-work
description: 为新模块或重画边界产出分层方案。用户说自底向上设计、先出架构、怎么分层、要不要新开模块、Plan、先别写代码时使用。
when_to_use: 新模块分层、自底向上设计、先方案后代码、是否新开模块
triggers: [架构设计, 自底而上, 怎么分层, 新模块, 先出方案, Plan, 先别写代码]
inputs: [要设计的能力或模块, 可选：对标系统]
outputs: [分层方案, 数据链路, 对现有模块的影响]
related: [role.architect, playbook.architecture-design]
---

# 任务：架构设计

**MANDATORY READ**

- [../roles/architect.md](../roles/architect.md)
- [../../../playbooks/architecture-design.md](../../../playbooks/architecture-design.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../rules/common/design-sources.md](../../rules/common/design-sources.md)

再按工作区叠加能力/平台类型包。

## 目标

给出可执行的分层：边界、依赖方向、状态放哪、数据从哪进到哪出。默认先方案；用户明确要求落地再写代码。

## 输入

- 能力或模块名。对标系统设计时带上规范来源。
- 缺范围先问。

## 步骤

按手册执行。实现路径完全不明时，先切 [online-deep-research.md](online-deep-research.md)，再回到本任务。

## 完成标准

- 有目标分层与「设计前/设计后」数据链路。
- 写明不做什么、不引入什么。
- 未要求实现则不改业务代码。

## 不做

- 不在方案阶段加补丁式兼容层「先跑通」。
- 不把调研源码拷进业务仓库或本知识库。
