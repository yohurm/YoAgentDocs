---
id: rules.type.docs
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 编辑 YoAgentDocs 或同类文档知识库时
related: [root.usage]
---

# 文档 / 知识库类型包

本仓库自身也走这套规则。

## 必须

- 目录与文件名英文；正文中文；技术名词与仓库名原文。
- 一篇一个关注点；用 frontmatter 的 `id` / `type` / `scope` / `when` / `related` 做检索，不靠超长文件名。
- 源码、clone、构建产物不进本库；调研只写 Markdown 总结。
- 新增任务必须写入 [CATALOG.md](../../../../CATALOG.md) 的表与触发词，否则 Agent 无法自行匹配。
- 任务正文用 **MANDATORY READ** 点名必读文件。
- 不为某一家 Agent 产品写特化格式；`AGENTS.md` 仅作短地图。不在本库预写 `.mdc`。
- 空的 `research/by-stack/<capability>/` 不预先创建；有第一篇再建模。
- 具名项目覆盖只写差异。

## 应当

- 新增指令先套 `templates/`。
- 联网调研结论升格为规则前先问用户。
