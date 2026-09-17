---
id: principle.engineering
type: principle
status: active
scope: common
when: always
when_to_use: 任何开发或改代码任务的默认信念层
related: [rule.common.development, rule.common.quality, rule.common.architecture]
---

# 工程原则

1. **先对齐再动手。** 目标、范围、项目类型不清楚时先问，不要用猜测填满实现。
2. **旧架构不打补丁。** 解决问题时，禁止在旧分层上继续修补、叠兼容或做防御性加固。先严格调研已验证的优秀设计，再沿完整数据链路重画边界，把问题重构到新通路上，并清掉旧代码。没有出处与链路证据，严禁靠猜测下结论。任务若是模块重设计，以该模块的分层为准。
3. **范围守住。** 只动当前任务涉及的模块。不顺手扩范围、不顺手升级依赖。不借机清无关脏代码。
4. **按类型验证。** 是否算完成，看该项目类型包里的验证方式（有真机的 Android、可启动的桌面应用、纯逻辑的单测等），而不是套同一种检查。
5. **可检索。** 本库文档一篇一个关注点；源码、密钥、临时 clone 不进 YoAgentDocs。
6. **可迁移。** 指令写成普通 Markdown，不绑定某一家 Agent 产品格式。
7. **用户覆盖规则。** `must` 与用户明确要求冲突时停下询问。
