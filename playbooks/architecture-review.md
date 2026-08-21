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
3. 对照 [architecture.md](../instructions/rules/common/architecture.md)：内聚、硬编码、**严禁兼容层**、越级、**文件单一职责**、**API 不含实现**。
4. 打开范围内每个公开门面（`api/`、`YoXxx`、包入口、IPC 命令层）：只应看到类型与转发。出现算法、绘制、手势或 `import …internal` 记为越界。
5. 发现兼容性代码或兼容层：沿其全部相关数据链路查根因，重设计完整通路，不得只删包装。只改相关链路上证明有问题的层；混职责文件按层拆开；范围内不留双轨与兼容代码。
6. 实现后再走同一条路径，留下通路证据。
7. 清单：[checklists/architecture-review.md](../instructions/checklists/architecture-review.md)；平台验证仍走类型包。
