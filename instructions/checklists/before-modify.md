---
id: checklist.before-modify
type: checklist
status: active
when: modify
when_to_use: 修改已有代码动手前
related: [rule.modification.common]
---

# 改代码前

- [ ] 已读入口、调用方和现有测试或等价验证
- [ ] 已区分「行为修复」还是「该模块重设计」，并叠加对应修改规则
- [ ] 已叠加平台类型包的验收方式（有真机 / 能启动应用等）
- [ ] 对外契约（API / CLI / 数据格式）是否会变，已明确；会变则已征得同意或给出兼容方案
- [ ] 计划的 diff 仅覆盖任务所需模块
- [ ] 任务本身不是重设计、但准备重写模块时，已说明理由并得到确认
- [ ] 若是分层/架构任务：将使用 [architecture-review](architecture-review.md) 清单，并准备数据链路证据
