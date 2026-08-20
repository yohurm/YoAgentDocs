---
id: rule.modification.agent
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改已有 Agent / 编排项目时
related: [rule.modification.common, rules.type.agent]
---

# 修改 Agent 系统

- 不悄悄改工具列表、系统提示、记忆写入策略或终止条件；这些是行为契约。
- 改 prompt 或规则时说明影响的场景，并避免把一次性任务说明写进常驻规则。
- 保持编排边界：规划、工具执行、结果汇总不要揉进同一层「万能函数」，除非原项目已经是这种结构且任务就是拆它。
- 新增工具要声明副作用与失败语义；默认只读，写操作需与现有权限模型一致。
- 不把联网深研的源码路径、Temp clone 或未确认的第三方流程写进运行时提示。
