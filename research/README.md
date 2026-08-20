---
id: research.hub
type: hub
status: active
when: research
when_to_use: 查找联网项目总结或归档新深研时
related: [playbook.online-deep-research]
---

# 联网项目总结

开源深研产出。源码在系统 Temp，这里只有 Markdown。

这里的目录按**能力层**归档（实现了什么）。`android` / `harmonyos` / `windows-desktop` 是平台规则，不是调研分类；平台写在文档 frontmatter 的标签里即可。

**有第一篇再创建** `by-stack/<capability>/`，不要预建空文件夹。

| 能力层 | 含义 | 横向总结 | 文档 |
|--------|------|----------|------|
| agent | 编排、工具、记忆、多 Agent | 有第一篇后建 | — |
| llm-app | RAG、评测、模型接入 | | — |
| ui-kit | 可复用组件与设计系统 | | — |
| frontend | Web / 页面应用 | | — |
| backend | API 与领域 | | — |
| data | 管道与分析 | | — |
| devops | CI 与交付 | | — |
| client-runtime | 桌面 / IDE / CLI / 应用壳 | | — |
| security | 认证、密钥、供应链（仅当这是主贡献） | | — |
| other | 暂放；满 3 篇同类再考虑升格 | | — |

单项目命名：`<owner>--<repo>.md`。模板见 [templates/research/](../templates/research/)。
