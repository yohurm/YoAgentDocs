---
id: root.agents
type: hub
status: active
when_to_use: 任何打开本库或被要求按 YoAgentDocs 工作的 Agent 先读
---

# YoAgentDocs — Agent 入口

本库是任务说明书，不是等人粘贴的提示词本。用户用自然语言下任务即可。

1. **MANDATORY READ** [CATALOG.md](CATALOG.md)（任务目录与触发条件）。
2. 用用户原话匹配一条任务；读该任务全文，再读它列出的 **MANDATORY READ**。
3. 只装载匹配任务所需的规则与类型包。不要把整个仓库塞进上下文。
4. 不要等用户粘贴 `prompts/shorts/`。短提示只给人用。
5. 主题、范围或项目类型缺失时先问；`must` 与用户口头要求冲突时先问。

叠加顺序、验证方式和冲突处理见 [USAGE.md](USAGE.md)。人看地图用 [README.md](README.md)。
