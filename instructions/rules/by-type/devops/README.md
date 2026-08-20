---
id: rules.type.devops
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束 CI / 部署 / 可观测项目时
---

# DevOps 类型包

## 默认边界

- 流水线、环境配置、密钥注入、观测（日志指标追踪）分开；生产与非生产差异显式配置，不靠手工改机器。
- 变更可回滚；没有回滚路径的发布默认不做。

## 质量门禁

- 密钥走现有注入方式，不写进仓库。
- 对生产有影响的步骤默认手动确认或受保护环境，除非用户要求全自动并接受风险。

## 不要默认引入

- 平行的第二套 CI 系统或云账号体系。
