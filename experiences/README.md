---
id: experiences.hub
type: hub
status: active
when_to_use: 沉淀或查阅经验时，先区分公共方法与具名项目对照
---

# 经验

分两摊，禁止混写、禁止反向引用。

| 目录 | 是什么 | 谁可以引用它 | 它可以引用 |
|------|--------|--------------|------------|
| [public/](public/README.md) | 可复用方法与审查节奏；不出现未公开仓库名、产品名、本机路径、账号 | `instructions/`、`playbooks/`、`research/`、其它 `public/` | 仅公共指令与公共经验 |
| [private/](private/README.md) | 具名项目事实、模块表、路径、对照已修 | **无人**：公共指令与 `public/` **禁止**链到这里 | 可以引用公共指令与 `public/` |

公共篇讲「怎么做」；私有篇只保留「这个仓库当时怎么对上」。方法一旦写进 `instructions/` / `playbooks/`，私有篇删重复方法，只留项目对照。

升格为规则须经用户确认，见本库 `USAGE.md`。
