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
2. **在正确的层上改。** 优先修问题所在的模块边界与数据流；不要用无关层的权宜代码把现象盖住。架构判断要有数据链路证据。任务若是模块重设计，以该模块的分层为准。
3. **范围守住。** 只动当前任务涉及的模块。不顺手扩范围、不顺手升级依赖。
4. **按类型验证。** 是否算完成，看该项目类型包里的验证方式（有真机的 Android、可启动的桌面应用、纯逻辑的单测等），而不是套同一种检查。
5. **可检索。** 本库文档一篇一个关注点；源码、密钥、临时 clone 不进 YoAgentDocs。
6. **可迁移。** 指令写成普通 Markdown，不绑定某一家 Agent 产品格式。
7. **用户覆盖规则。** `must` 与用户明确要求冲突时停下询问。
