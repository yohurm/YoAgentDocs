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

- 不顺手改全局 token、主题轴和组件分层。任务就是重做某组件时，在该组件边界内一次做完，避免双轨 API。
- 交互（按压、涟漪、内容区裁剪）优先复用已有能力，不在单个控件里再写一套。
- 视觉验收叠加平台包：Android 有真机看动画；桌面则启动组件预览或宿主应用。
