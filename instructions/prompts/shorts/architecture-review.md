---
id: short.architecture-review
type: short
status: active
when: modify
when_to_use: 在对话中要求自底向上审查架构与代码质量时
related: [rule.common.architecture, playbook.architecture-review]
---

# 短提示：架构审查

给人复制用。Agent 应直接匹配 [CATALOG.md](../../../CATALOG.md) 中的「架构审查」。

```
自底向上审查代码架构与质量。请读取 YoAgentDocs：
- instructions/rules/common/architecture.md
- playbooks/architecture-review.md
- instructions/checklists/architecture-review.md

按仓库已声明的分层检查：高内聚低耦合、文件单一职责、API/门面不含实现且不引用 internal、无散落硬编码、严禁兼容性代码与兼容层、无越级调用。
发现兼容层则沿全部相关数据链路排查根因，重设计完整通路，不得只删包装或留过渡。
UI 若是 MVVM，检查 View / ViewModel / Model 边界。
每项架构判断先给出数据链路，改完再给同一路径的通路证据。
只动当前问题相关链路，不扫全仓无关模块。
```
