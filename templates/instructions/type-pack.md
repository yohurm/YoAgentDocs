---
id: template.type-pack
type: template
status: active
when_to_use: 新增项目类型基础规则包时
---

# 模板：类型包

放到 `instructions/rules/by-type/<type>/README.md`。

```markdown
---
id: rules.type.<type>
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use:
related: []
---

# <类型>类型包

## 默认边界

- 

## 质量门禁

- 写清**有条件**的验证：例如「已接真机则… / 未接设备则… / 能启动应用则…」。不要把某一平台的验收写成所有项目的必须项。

## 不要默认引入

- 
```

然后在 `instructions/rules/by-type/README.md` 的表中增加一行。
