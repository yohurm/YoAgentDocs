---
id: rule.common.design-sources
type: rule
status: active
severity: should
scope: common
when: always
when_to_use: 对标系统设计、交互、动效或组件能力时
related: [rule.common.architecture, task.implement]
---

# 设计与规范来源

实现或审查「应对齐某系统/官方」的能力时：

1. **先读规范，再搜实现，再写代码。** 本机已有官方文档树则读本地（鸿蒙常用 `E:\Dev\Doc\HarmonyOS-Developer-docs`）。
2. 规范管状态与度量（默认/按压/禁用、尺寸、时长），开源仓库管实现手段。不要只用 GitHub README 代替官方定义。
3. 对标多个系统时（如鸿蒙 + iOS），先列状态对照，再设计自己的 API，而不是混抄两套控件名。
4. 找不到规范章节时在回复里写明缺口，不要假装已经对标。
