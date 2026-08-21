---
id: checklist.architecture-review
type: checklist
status: active
when: modify
when_to_use: 架构审查或模块重设计收尾时
related: [rule.common.architecture]
---

# 架构审查

- [ ] 模块职责单一，跨模块只走公开边界
- [ ] 范围内每个文件能说清只服务哪一层；无「模型+视图+策略」堆在同一文件
- [ ] 公开 API / 门面不含实现（无几何、手势、绘制、持久化算法）；未引用 `internal` 或私有实现包
- [ ] 任务范围内无新增散落硬编码（视觉、路径、超时、文案等已进池或配置）
- [ ] 问题链路上无新的补丁/兼容层；该删的冗余已删
- [ ] 已写出设计前的数据链路（入口 → 状态 → 副作用）
- [ ] 已写出实现后的数据通路证据（代码位置或运行结果）
- [ ] 无越级调用；若项目是 MVVM，View / ViewModel / Model 未互相侵入
