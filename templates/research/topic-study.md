---
id: template.topic-study
type: template
status: active
when: research
when_to_use: 写跨仓库主题笔记（对标系统设计、动效、几何等）时
related: [template.project-study, playbook.online-deep-research]
---

# 模板：主题深研

保存为 `research/by-stack/<capability>/<topic>.md`。`<topic>` 沿用 `<owner>--<repo>` 文件名形式；`owner` 可以是系统名（`harmony`、`apple`、`HarmonyOS_Samples`），`repo` 是主题描述。

与 [project-study.md](project-study.md) 的区别：**不是单仓库赏析**，而是围绕一个主题（对标系统能力、动效、几何、交互契约）整合官方文档与多个开源实现的结论。官方规范由 `source.url` 指向；参考实现仓库用 `repos:` 列表记录，不假装读过 HDS 一类私有仓。

```markdown
---
id: research.<topic-slug>
type: topic-study
status: active
when: research
stack:
  capability: agent | llm-app | ui-kit | frontend | backend | data | devops | client-runtime | docs | security | other
  languages: []
  frameworks: []
also_relevant: []
utilization: []  # reuse-pattern | adapt | anti-pattern | lesson-only
source:
  platform: github | gitcode | other
  url:  # 官方文档 / 规范来源（必填）
  repos: []   # 可选：参考实现仓库 owner/name
  cloned_to: []  # 可选：对应 %TEMP%/YoAgentResearch/<owner>--<repo>；未克隆则省略
studied_at: YYYY-MM-DD
updated_at: YYYY-MM-DD  # 可选：追加结论时更新
related: []
---

# <主题>

## 背景

主题是什么、为什么研、官方规范来源（本地文档优先）。

## 关键结论

能直接引用的机制、参数、状态定义。可用短列表或 mermaid，可含必要代码引用；不要贴大段源码。

## 与当前工作的关系

能直接用的、必须改写的、明确不要用的（对标 Yo 组件 / 模块名写清楚）。

## 来源与阅读范围

官方文档路径、clone 的仓库路径、实际读过的文件；未读的不装懂。
```
