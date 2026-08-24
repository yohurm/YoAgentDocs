---
id: playbook.bug-fix
type: playbook
status: active
when: modify
when_to_use: 修复已有缺陷时
related: [task.bug-fix, rule.modification.common]
---

# 手册：修复缺陷

1. 列本轮问题清单（用户点名的现象；连带暴露且同链路的一并纳入）。每条对应一条验收：类型包门禁 + 该现象消失。
2. 复现：按平台类型包（真机截图/Logcat、启动桌面应用、或可跑测试）。
3. 沿该现象的**完整数据链路**定位到一层（入口 → 状态持有 → 变换 → 副作用终点），不要只看报错点。
4. 局部逻辑 → 最小必要改动修在该层。分层损坏或发现兼容层 → 升级架构审查，不要在错误层打补丁。禁止空判断/吞异常/延时重试/只改 UI 当修复。
5. 再走同一路径验证。未过关：回到第 3 步，继续审查同一条完整链路并修。**不要总结、不要结束对话。**
6. 清单上每条都验收通过后，才写根因、修改点、验证证据。有未验证项就继续做；环境缺口须写明且不声称完成、不结束对话。
7. [checklists/bug-fix.md](../instructions/checklists/bug-fix.md) 与 [checklists/before-finish.md](../instructions/checklists/before-finish.md) 全部勾上再收尾。
