---
id: task.bug-fix
type: task
status: active
when: modify
description: 修复已有缺陷。用户说闪退、对不齐、动画不对、Logcat、截图有问题、联调失败、修 bug 时使用。根因若是分层错误，升级为架构审查。
when_to_use: 闪退、布局不对、动画异常、Logcat、真机/截图复现、修缺陷
triggers: [bug, 闪退, 修复, logcat, 对不齐, 卡顿, 联调, 截图不对]
inputs: [现象, 可选：复现步骤]
outputs: [根因, 正确层上的修复, 验证证据, 全部验收通过或未完成不得收尾]
related: [role.implementer, playbook.bug-fix]
---

# 任务：修复缺陷

**MANDATORY READ**

- [../roles/implementer.md](../roles/implementer.md)
- [../../../playbooks/bug-fix.md](../../../playbooks/bug-fix.md)
- [../../rules/modification/common.md](../../rules/modification/common.md)
- [../../rules/common/architecture.md](../../rules/common/architecture.md)
- [../../rules/common/quality.md](../../rules/common/quality.md)
- [../../rules/common/acceptance.md](../../rules/common/acceptance.md)

叠加平台类型包（Android 已接真机则安装/截图/Logcat）。

## 目标

在正确层修好本轮问题，并用类型包方式验证。不要用无关层的分支把现象盖住。

**未达验收不得收尾、不得结束对话。** 本轮要修的问题全部达到对应验收标准之前，禁止写总结、禁止声称完成、禁止把半成品当对话收尾。未过关就沿该问题相关的完整数据链路继续查、继续修、继续验，直到清单清零或环境客观上无法再推进。

## 输入

- 现象；能复现则复现。缺模块范围时，根据现象定位一条用户路径，不要扫全仓。
- 同一轮里有多条问题：列出清单，逐条对验收，缺一条都不算完。

## 步骤

按手册。沿数据链路定位后：

- 局部逻辑错误 → 行为修复。
- 边界/越级/补丁堆积/兼容层 → 升级 [architecture-review.md](architecture-review.md)。

验证未过：不要停。回到该现象从入口到状态到副作用的完整链路，补证据、修根因、再验同一路径。只改表象或只报「还差一步」不算推进。

## 完成标准

全部满足才允许总结：

- 本轮每条问题的根因说得清（哪一跳）。
- 每条都在正确层修好，且按类型包验收通过（真机则安装/截图/Logcat，桌面则启动对照）。
- 未扩大到无关模块。

环境做不到类型包验证时：写明缺口，且**仍不算完成**；能继续推进的链路分析与代码修复不得停。只有用户明确改验收条件，或环境缺口使验证客观上无法做完时，才可在回复里交代缺口并等待条件，而不是把半成品当成交付。

## 不做

- 不在调用处叠兼容函数或兼容层代替修根因。
- 不用空判断、吞异常、延时/重试、标志位或只改 UI 代替修根因。
- 不把「顺便重构」塞进行为修复。
- 不在问题未全部达标时写收尾总结、列「后续建议」代替继续修。
- 不把「已定位 / 已改一处 / 理论上应该好了」当成完成。
