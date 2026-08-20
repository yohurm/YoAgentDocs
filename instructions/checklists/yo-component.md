---
id: checklist.yo-component
type: checklist
status: active
when: new-work
when_to_use: 新增或重做 Yo 组件收尾
related: [rules.type.ui-kit]
---

# Yo 组件

- [ ] L0–L5 能指到具体文件；没有一层塞进门面类
- [ ] L5 只转发 `XxxImpl`；`api/` 未 import `internal`，未写 Path / 拟合 / 绘制
- [ ] 无「整组件一个文件包办」
- [ ] 不引用其他组件的 internal
- [ ] 数值在 token/常量池；无新增散落硬编码
- [ ] 有内容区则裁剪区内溢出（含 ripple）；未靠改 ripple 尺寸冒充
- [ ] 对外只有 Yo 门面；api/入口无补丁类
- [ ] 动效走共享配方；宿主/模块未复制外观
- [ ] 默认/按压/禁用（及关键空态）可演示；平台验证已做或已声明缺口
