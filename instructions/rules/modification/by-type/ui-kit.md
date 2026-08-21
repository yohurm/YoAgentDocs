---
id: rule.modification.ui-kit
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改自研组件库时
related: [rule.modification.common, rules.type.ui-kit]
---

# 修改组件库

- 不顺手改全局 token、主题轴和组件分层。任务就是重做某组件时，在该组件边界内一次做完，避免双轨 API 与第二套生命周期动词。
- 交互（按压、涟漪、内容区裁剪）优先复用 L1，不在单个控件里再写一套。
- 改绑定或显隐时同时改 attach/detach/destroy；不留只 bind 不 unbind 的监听与动画。
- 不把新逻辑堆进已过厚的单文件；按 [file-srp.md](../../by-type/ui-kit/file-srp.md) 拆层。
- 视觉验收叠加平台包。收尾用 [checklists/yo-component.md](../../../checklists/yo-component.md)。
