---
id: playbook.architecture-review
type: playbook
status: active
when: modify
when_to_use: 自底向上做架构审查或按分层修复时
related: [rule.common.architecture, checklist.architecture-review]
---

# 手册：架构审查

1. 读仓库已声明的分层（README、架构文档、目录约定）。未声明则先问，不要发明第二套。
2. 沿一条真实用户路径画出数据链路：入口、状态持有者、变换、副作用终点。
3. 对照 [architecture.md](../instructions/rules/common/architecture.md)：内聚、硬编码、补丁层、越级。
4. 只改这条链路上证明有问题的层；清该范围内的冗余与兼容代码。
5. 实现后再走同一条路径，留下通路证据。
6. 清单：[checklists/architecture-review.md](../instructions/checklists/architecture-review.md)；平台验证仍走类型包。
