---
id: rules.by-project.hub
type: hub
status: active
when_to_use: 查找或新增具名项目覆盖时
related: [template.project-overlay]
---

# 具名项目覆盖

默认**不建**具体项目目录。只有某仓库与类型包存在稳定差异时，才增加：

```
instructions/rules/by-project/<project-id>/
  README.md     # 身份、仓库、类型、禁区
  overlay.md    # 只写差异
```

`<project-id>` 用短英文 id（与仓库名可不同）。模板：[templates/instructions/project-overlay.md](../../../templates/instructions/project-overlay.md)。

装载顺序：公共 → 修改（若在改代码）→ 类型包 → 本目录。
