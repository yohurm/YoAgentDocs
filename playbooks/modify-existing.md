---
id: playbook.modify-existing
type: playbook
status: active
when: modify
when_to_use: 修改已有实现时
related: [rule.modification.common, checklist.before-modify]
---

# 手册：修改已有代码

1. 走 [checklists/before-modify.md](../instructions/checklists/before-modify.md)。
2. 在公共规则之上叠加 `rules/modification/common.md` 与类型修改规则。
3. 先判断是行为修复还是模块重设计。拟议改动若是在旧架构上打补丁、叠兼容或做防御性加固，按重设计处理。给出拟改范围与契约影响（可短），再动手。
4. 行为修复保持最小必要改动。重设计先调研优秀设计、重画完整数据链路，再替换该模块错误分层并清理旧代码；不扩到无关模块。本阶段相关通路整理到干净，严禁残留技术债务。严禁猜测。
5. 按平台类型包验证后走 [checklists/before-finish.md](../instructions/checklists/before-finish.md)。
