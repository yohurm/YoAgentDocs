---
id: rules.type.data
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束数据管道 / 分析项目时
---

# 数据类型包

## 默认边界

- 摄入、变换、存储、消费分开描述；任务契约（字段、粒度、迟到数据）写清楚。
- 批与流不混用同一套「看起来能跑」的脚本当生产。

## 质量门禁

- 可重跑或可补数；失败有现场（日志、位点、分区）。
- 个人与密钥字段不进明文下游。

## 不要默认引入

- 未评估的新仓、新调度器。
