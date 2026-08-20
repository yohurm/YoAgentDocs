---
id: task.online-deep-research
type: task
status: active
when: research
description: 从 GitHub、GitCode 等检索并 clone 到 Temp 做源码级深研，产出写入 research/。用户说调研、深研、搜开源、拉仓库研究、先看别人怎么做时使用。
when_to_use: 开源调研、GitHub、GitCode、Temp clone、实现路径不明要先看项目
triggers: [调研, 深研, GitHub, GitCode, clone, 优秀项目, 先看别人]
inputs: [调研主题, 可选：当前项目约束, 关注能力层, Top N]
outputs: [候选表, N 篇单项目深研, 1 篇能力层横向总结, 已更新的 research Hub]
related: [role.online-researcher, playbook.online-deep-research]
---

# 任务：联网深度调研

**MANDATORY READ**

- [../roles/online-researcher.md](../roles/online-researcher.md)
- [../../../playbooks/online-deep-research.md](../../../playbooks/online-deep-research.md)
- [../../rules/common/design-sources.md](../../rules/common/design-sources.md)

## 目标

围绕主题自动挑选 Top N（默认 3～5）个项目做源码级深研，写入 `research/`，按能力层归类并更新横向总结。

## 输入

- **必填：** 调研主题。缺则先问。
- **选填：** 当前项目约束、能力层、Top N、必须包含/排除的仓库。

## 步骤

按手册。有官方文档则先读规范再搜实现。自动入选 Top N，入选理由写入文档。Clone 到 `%TEMP%\YoAgentResearch\<owner>--<repo>\`。

## 完成标准

- N 篇单项目深研符合模板；Hub 与 `_synthesis.md` 已更新。
- 源码不在 YoAgentDocs 内。

## 不做

- 不写用户业务代码。
- 不把结论自动写入 `instructions/rules/`。
