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
3. 先判断是行为修复还是模块重设计；给出拟改范围与契约影响（可短），再动手。
4. 行为修复保持最小必要改动；重设计只拆该模块的错误分层，不扩到无关模块。
5. 按平台类型包验证后走 [checklists/before-finish.md](../instructions/checklists/before-finish.md)。
