---
id: root.usage
type: hub
status: active
when_to_use: 装载顺序、冲突、新增任务登记
related: [root.readme, root.catalog, root.agents]
---

# 使用约定

## Agent 怎么开工

1. **MANDATORY READ** [AGENTS.md](AGENTS.md) 与 [CATALOG.md](CATALOG.md)。
2. 匹配一条任务，**MANDATORY READ** 该任务全文（不要只看书名）。
3. 再读任务里的 MANDATORY READ（角色、手册、规则、类型包）。
4. 按任务完成标准收尾。用户未要求则不 commit / push。

工作区是业务仓库时：若能访问本库，同样先读本库 `CATALOG.md`，不要等用户把短提示贴进对话。

## 渐进装载

先目录（名称 + 何时用），再任务正文，再手册与规则。禁止一上来通读 `instructions/`。

## 场景不够明确时

先问：调研 / 设计 / 审查 / 实现 / 修缺陷；能力类型与平台；是否已有 `rules/by-project/<id>/`；验证条件（是否已接真机、能否启动应用）。

## 冲突

| 情况 | 处理 |
|------|------|
| 任务说明与公共规则冲突 | 任务仅覆盖本作业；回复里写明覆盖了哪条 |
| 具名项目与类型包冲突 | 以具名项目为准 |
| `must` 与用户口头要求冲突 | 停下来问 |
| `should` 需要偏离 | 可以，回复里写原因 |

## 新增任务

1. 用 [templates/instructions/task.md](templates/instructions/task.md)。
2. `description` / `when_to_use` 必须同时写清 **做什么** 和 **何时匹配**（含用户可能说的词）。
3. 正文开头用 **MANDATORY READ** 列出必读文件；不要只用「参见」。
4. **登记到 [CATALOG.md](CATALOG.md)**，否则 Agent 匹配不到。
5. 短提示可选，只给人复制；不是调度入口。

其他新增规则见 [instructions/rules/by-type/docs/](instructions/rules/by-type/docs/)。
