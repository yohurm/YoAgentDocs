---
id: task.bug-fix
type: task
status: active
when: modify
description: 修复已有缺陷。用户说闪退、对不齐、动画不对、Logcat、截图有问题、联调失败、修 bug 时使用。根因若是分层错误，升级为架构审查。
when_to_use: 闪退、布局不对、动画异常、Logcat、真机/截图复现、修缺陷
triggers: [bug, 闪退, 修复, logcat, 对不齐, 卡顿, 联调, 截图不对]
inputs: [现象, 可选：复现步骤]
outputs: [根因, 正确层上的修复, 验证证据]
related: [role.implementer, playbook.bug-fix]
---

# 任务：修复缺陷

**MANDATORY READ**

- [../roles/implementer.md](../roles/implementer.md)
- [../../../playbooks/bug-fix.md](../../../playbooks/bug-fix.md)
- [../../rules/modification/common.md](../../rules/modification/common.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../rules/common/quality.md](../../rules/common/quality.md)

叠加平台类型包（Android 已接真机则安装/截图/Logcat）。

## 目标

在正确层修好当前现象，并用类型包方式验证。不要用无关层的分支把现象盖住。

## 输入

- 现象；能复现则复现。缺模块范围时，根据现象定位一条用户路径，不要扫全仓。

## 步骤

按手册。沿数据链路定位后：

- 局部逻辑错误 → 行为修复。
- 边界/越级/补丁堆积 → 升级 [architecture-review.md](architecture-review.md)。

## 完成标准

- 根因说得清（哪一跳）。
- 修复后按类型包验证，或写明环境缺口。
- 未扩大到无关模块。

## 不做

- 不在调用处叠兼容函数代替修根因。
- 不把「顺便重构」塞进行为修复。
