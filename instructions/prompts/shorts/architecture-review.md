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

按仓库已声明的分层与类型包层名检查（见 stack-layering.md；禁止一律叫 MVVM）：旧架构不打补丁、高内聚低耦合、文件单一职责、API/门面不含实现且不引用 internal、无散落硬编码、严禁兼容性代码与兼容层、无越级调用。
禁止在旧架构上继续修补或做防御性加固。先严格调研优秀设计，沿全部相关数据链路排查根因，重设计完整通路并清理旧代码，不得只删包装或留过渡。严禁猜测。
Android 页面才检查 View / ViewModel / Model；桌面检查 View → store → IPC → domain；组件库检查 L0–L5。
每项架构判断先给出数据链路，改完再给同一路径的通路证据。
只动当前问题相关链路，不扫全仓无关模块。
多范围则按所有权切开审查；未通过修完后对全部范围复审；同一波次全绿才收口。范围内分层问题与技术债务全部闭合，不得写成非阻塞观察或已知债务。允许分阶段，但每一阶段交付干净切片，严禁残留技术债务，不维持半成品。
```
