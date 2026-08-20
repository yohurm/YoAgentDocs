---
id: template.task
type: template
status: active
when_to_use: 新增完整任务时
---

# 模板：任务

新建后必须把一行登记进仓库根目录 `CATALOG.md`。

```markdown
---
id: task.<name>
type: task
status: draft
description: 做什么，以及用户哪些说法时应匹配（含关键词）。
when_to_use:
triggers: []
inputs: []
outputs: []
related: []
---

# 任务：<名称>

**MANDATORY READ**

- 角色
- 手册
- 必要规则

## 目标

## 输入

## 步骤

按手册。需要时串联 CATALOG 中的其他任务。

## 完成标准

## 不做
```
