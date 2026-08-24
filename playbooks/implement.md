---
id: playbook.implement
type: playbook
status: active
when: new-work
when_to_use: 按已定分层实现功能时
related: [task.implement, rule.common.quality, rule.common.acceptance]
---

# 手册：实现功能

1. 确认规范来源与目标分层。缺一则先深研或架构设计。
2. 装载能力类型包 + 平台类型包；改旧代码则叠加修改规则。
3. 按正确层实现；数值进 token/常量池；公开命名跟仓库约定（如 `Yo`）。
4. 列出本轮验收清单（任务条目 + 类型包门禁）。自驱动补齐关键状态，直到**每一条**通过。任一未过则继续改，不要总结、不要结束对话。没有「先做主流程再欠空态」。
5. 环境做不到的验证写明缺口，该条记未通过，本轮不算完成。收尾用 [checklists/before-finish.md](../instructions/checklists/before-finish.md) 与 [acceptance.md](../instructions/rules/common/acceptance.md)。
