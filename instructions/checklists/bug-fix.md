---
id: checklist.bug-fix
type: checklist
status: active
when: modify
when_to_use: 修缺陷收尾
related: [task.bug-fix]
---

# 修缺陷

- [ ] 已复现或写明无法复现的原因
- [ ] 根因落在明确的一层
- [ ] 修复在该层，无新的绕过包装、兼容性代码或兼容层
- [ ] 无空判断掩盖上游、无吞异常、无延时/重试/标志位躲时序、无只改 UI 藏错数据
- [ ] 已按类型包验证或声明环境缺口；回复区分已做检查与未验证项
- [ ] 未扩大到无关模块
