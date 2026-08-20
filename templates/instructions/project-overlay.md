---
id: template.project-overlay
type: template
status: active
when_to_use: 为具名仓库新增与类型包的差异覆盖时
---

# 模板：具名项目覆盖

创建 `instructions/rules/by-project/<project-id>/`。

`README.md`：

```markdown
---
id: project.<id>
type: hub
status: draft
scope: project
when_to_use: 正在该仓库工作时
related: [project.<id>.overlay]
---

# <项目名>

- 仓库：
- 类型包：
- 禁区：
```

`overlay.md`：

```markdown
---
id: project.<id>.overlay
type: rule
status: draft
severity: must
scope: project
when: always
when_to_use: 覆盖类型包与公共规则中与本仓库不同的条款
related: [project.<id>]
---

# <项目名> 差异

只写与 `rules/common` 和 `rules/by-type/<type>` 不同的点。

## 覆盖

- 条款 X → 本仓库改为 Y（原因）
```
