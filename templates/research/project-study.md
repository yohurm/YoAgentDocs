---
id: template.project-study
type: template
status: active
when: research
when_to_use: 写单仓库深度调研文档时
---

# 模板：单项目深研

保存为 `research/by-stack/<capability>/<owner>--<repo>.md`。

```markdown
---
id: research.<owner>-<repo>
type: project-study
status: active
when: research
stack:
  capability: agent | llm-app | frontend | backend | data | devops | client-runtime | security | other
  languages: []
  frameworks: []
also_relevant: []
utilization: []  # reuse-pattern | adapt | anti-pattern | lesson-only
source:
  platform: github | gitcode | other
  repo: owner/name
  url:
  cloned_to: "%TEMP%/YoAgentResearch/owner--name"
studied_at: YYYY-MM-DD
related: []
---

# owner/name

## 入选理由

## 项目是什么

## 架构

关键模块、数据流、扩展点。可用短列表或 mermaid，不要贴大段源码。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
|  | reuse-pattern / adapt / anti-pattern / lesson-only |  |

## 架构设计经验

对本类项目可复用的决策（含反例）。

## 与当前工作

能直接用的、必须改写的、明确不要用的。

## 阅读范围

实际读过的路径；未读的不装懂。
```
