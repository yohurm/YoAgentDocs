---
id: rule.modification.frontend
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改已有前端项目时
related: [rule.modification.common, rules.type.frontend]
---

# 修改前端

- 不顺手改设计 token、全局主题、路由结构和状态管理方案。
- 复用已有组件与样式来源；不平行造一个视觉不一致的按钮/表单。
- 用户可见文案变更保持产品语气；不把调试语句和占位符留在界面上。
- 交互改动要说明在哪些断点或关键路径上目视/手工看过；无设计稿时跟随周围页面。
- 不把业务逻辑塞进无关的展示组件，也不把样式逻辑拆进与本次无关的全局文件。
