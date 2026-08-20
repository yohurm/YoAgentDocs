---
id: rule.modification.backend
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改已有后端项目时
related: [rule.modification.common, rules.type.backend]
---

# 修改后端

- 不擅自改表结构、消息格式、鉴权方式和对外错误码；需要时先给出迁移/兼容方案。
- 保持现有分层：不要为了图快把领域逻辑写进 handler 或把 SQL 散落到无关模块。
- 幂等、超时、重试、并发控制跟现有约定走；没有约定时新增要写进该服务的说明，而不是只写在代码注释。
- 日志级别与字段与周围接口一致；不把请求体全量打到日志。
- 删除或改名接口时，先搜仓库内外的调用点（含脚本与文档）。
