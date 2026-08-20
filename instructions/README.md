---
id: instructions.hub
type: hub
status: active
when_to_use: 查找原则、规则、任务、清单时
related: [root.catalog, root.usage]
---

# 指令层

**先匹配任务：** 根目录 [CATALOG.md](../CATALOG.md)。本页是规则与清单的地图，不是调度入口。

## 种类

| 目录 | 种类 | 作用 |
|------|------|------|
| [principles/](principles/) | 原则 | 少而稳的工程信念 |
| [rules/](rules/) | 规则 | 必须或应当遵守 |
| [prompts/](prompts/) | 任务、角色、可选短提示 |
| [checklists/](checklists/) | 自检 |

规则：`must` 不可自行放宽；`should` 偏离须说明。

## 作用域

| 路径 | 何时装 |
|------|--------|
| [rules/common/](rules/common/) | 始终（由任务 MANDATORY READ 拉入相关篇） |
| [rules/modification/](rules/modification/) | 改已有代码 |
| [rules/by-type/](rules/by-type/) | 识别到能力或平台后 |
| [rules/by-project/](rules/by-project/) | 该具名仓库有覆盖时 |

叠加顺序见 [README.md](../README.md)。类型包验证（如 Android 真机）仍由类型包规定。
