---
id: playbook.architecture-design
type: playbook
status: active
when: new-work
when_to_use: 产出模块分层方案时
related: [task.architecture-design, rule.common.architecture]
---

# 手册：架构设计

1. 读仓库已有分层文档与目录。对标任务先读 [design-sources.md](../instructions/rules/common/design-sources.md)。
2. 实现手段不明则先走联网深研任务，再回来。
3. 选一条主用户路径，写现状链路（若是新模块则写「将接入的邻层」）。
4. 给出目标分层：模块边界、依赖方向（下层不持有上层）、状态与副作用落点、公开 API；写清持久化 / 领域 / 传输模型在哪一层转换。对照该仓已声明架构与类型包层名（[stack-layering.md](../instructions/rules/common/stack-layering.md)），禁止越级，禁止把另一栈的 MVVM/L0–L5/IPC 词汇硬套过来。方案用文字和关系图；不写排期、人员、进度。
5. 写清不引入什么、与旧代码的关系（替换哪一块，而不是双轨或兼容层）。
6. 用户要求落地时，交 [implement.md](implement.md)；否则停在方案。
