---
id: rules.type.backend
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束后端 / API 项目时
related: [rule.modification.backend]
---

# 后端类型包

## 默认边界

- 传输层、应用/领域、持久化分开；新功能进现有边界，不按「这次任务」另起一层同名抽象。

## UI 与实现分层

无桌面 View。handler / 传输层只译协议；领域规则与持久化分别在应用/领域层与存储层。不要把这套叫做 MVVM。对照 [stack-layering.md](../../common/stack-layering.md)。
- 鉴权、多租户、审计跟现有中间件，不在单个接口上发明私有认证。

## 质量门禁

- 接口有明确的输入校验与错误语义。
- 数据变更可追踪（迁移、版本或等价机制），禁止只改运行中的库表「先用着」。

## 不要默认引入

- 第二套 ORM、消息队列、或与现网不一致的部署单元。
