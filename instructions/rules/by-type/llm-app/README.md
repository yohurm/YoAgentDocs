---
id: rules.type.llm-app
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束 LLM 应用（非完整 Agent 回路）时
---

# LLM 应用类型包

## 默认边界

- 模型调用、提示模板、检索/上下文装配、输出解析分开。
- 评测与线上提示变更可追踪；不要只在聊天窗口里改完就算发布。

## 质量门禁

- 说明上下文来源（用户、检索、固定提示）和截断策略。
- 对检索类功能：无结果与低相关结果有明确行为。

## 不要默认引入

- 未证明必要的 Agent 编排层。
- 把密钥写进提示模板仓库。
