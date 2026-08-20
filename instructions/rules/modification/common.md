---
id: rule.modification.common
type: rule
status: active
severity: must
scope: common
when: modify
when_to_use: 任何修改已有代码的任务
related: [rule.common.development, rule.common.architecture, checklist.before-modify]
---

# 修改已有代码

先判断任务属于哪一种，再选改法。不要把「重做整个仓库」和「修一处行为」混用。

## 行为修复（默认）

- 先读调用方、边界和已有验证，再改。
- 只改完成任务所需的模块与行；禁止无关重排、无关重命名、整文件格式化。
- 根因在哪一层就改哪一层，避免在调用处叠一层掩盖。
- 保持未声明变更的对外契约（API、CLI、文件格式、环境变量）。
- 风格跟周围代码走。

## 模块重设计（用户明确要求架构/分层/重做该模块时）

- 以该模块的目标分层为准，去掉本模块内为迁就旧结构而留的双轨 API 和过渡适配。
- 仍然只动这个模块及其必要调用点，不借机重写无关模块。
- 重设计必须带上数据链路：改前调用链 + 改后通路证据（见 [architecture.md](../common/architecture.md)）。
- 重设计前用几句话说明新边界和影响面；用户已把任务定义成重设计时，不必再等二次批准。

## 共性

- 行为变化按类型包做验证。
- 纯重构不得改变行为；不能保证就不要在行为修复任务里夹带重构。
- 与任务无关的脏代码可以记下来，不要顺手清理。
