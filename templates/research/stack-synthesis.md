---
id: template.stack-synthesis
type: template
status: active
when: research
when_to_use: 更新某能力层的横向总结时
---

# 模板：能力层横向总结

保存为 `research/by-stack/<capability>/_synthesis.md`。已有文件则合并，不要覆盖掉旧结论。

```markdown
---
id: research.synthesis.<capability>
type: synthesis
status: active
when: research
stack:
  capability:
---

# <capability> 横向总结

## 本层已研项目

| 仓库 | 一句话 | 利用方式 |
|------|--------|----------|

## 共同架构经验

## 分歧与取舍

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。须用户确认后再升格。

## 入选与落选备忘

Top N 理由；有代表性的落选及原因。
```
