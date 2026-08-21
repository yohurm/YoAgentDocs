---
id: playbook.bug-fix
type: playbook
status: active
when: modify
when_to_use: 修复已有缺陷时
related: [task.bug-fix, rule.modification.common]
---

# 手册：修复缺陷

1. 复现：按平台类型包（真机截图/Logcat、启动桌面应用、或可跑测试）。
2. 沿该现象的数据链路定位到一层。
3. 局部逻辑 → 最小必要改动修在该层。分层损坏或发现兼容层 → 升级架构审查，不要在错误层打补丁。
4. 再走同一路径验证。
5. [checklists/before-finish.md](../instructions/checklists/before-finish.md)。
