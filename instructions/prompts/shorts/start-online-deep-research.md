---
id: short.start-online-deep-research
type: short
status: active
when: research
when_to_use: 在对话中粘贴以切换到联网深度调研模式
related: [role.online-researcher, task.online-deep-research, playbook.online-deep-research]
---

# 短提示：开始联网深度调研模式

给人复制用。Agent 应直接匹配 [CATALOG.md](../../../CATALOG.md) 中的「联网深研」，不必等这段粘贴。

把下面一段贴给 Agent。尖括号换成实际情况；不需要的选填行删掉。

```
开始联网深度调研模式。

调研主题：<必填：要解决的问题或要借鉴的能力>
当前项目/约束：<可选>
关注能力层：<可选：agent / llm-app / ui-kit / frontend / backend / data / devops / client-runtime / docs / security / other>
Top N：<可选，默认 3～5>

请读取 YoAgentDocs：
- instructions/prompts/roles/online-researcher.md
- instructions/prompts/tasks/online-deep-research.md
- playbooks/online-deep-research.md

自动挑选 Top N 并 clone 到系统 Temp 做源码研读；入选理由写进文档。
不要写业务代码。不要把第三方源码放进 YoAgentDocs。源码只放 %TEMP%\YoAgentResearch\<owner>--<repo>\。
调研文档按能力层写入 research/by-stack/，并更新该层横向总结与 research/README.md。
```
