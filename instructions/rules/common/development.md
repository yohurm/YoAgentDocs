---
id: rule.common.development
type: rule
status: active
severity: must
scope: common
when: always
when_to_use: 编写或组织代码时
related: [rule.common.quality, rule.common.architecture, rule.modification.common]
---

# 编码与结构

## 必须

- 只改与当前任务相关的文件；不把「顺便整理」算进任务。
- 沿用仓库已有的命名、目录、模块边界和错误处理方式；没有先例时先问，再新增约定。
- 函数、类型、模块的名字表达意图，让人不用看注释就知道「是什么 / 做什么」；禁止无意义缩写和误导性名称。
- 命名跟仓库已有风格走。类与模块用名词；函数用动词短语；布尔用 `is` / `has` / `can` 一类前缀（或该语言等价写法）；集合用复数。禁止 `data`、`info`、`temp`、`obj`、`flag` 这类空名字，禁止 `handler1` / `handler2` 用编号区分。装配入口允许 `XxxImpl` 这类仓库已有写法，不要为「名称里不能出现技术词」去改。
- 不要臆造不存在的 API、配置项、CLI 参数或环境变量。先查仓库或官方文档。
- 依赖只在任务需要时添加；写明为什么，并与现有包管理方式一致。
- 注释只解释「为什么」或非显而易见的约束；不复述代码字面意思。
- 删除代码时同时删除真正的死引用；不要留下注释掉的大段旧实现。

## 应当

- 优先改现有抽象，而不是平行再做一套。
- 跨模块行为走现有扩展点；不要在无关层加分支把现象盖住。
- 仓库已有设计 token / 常量池时，视觉、时长、路径、超时等数值进池；细则见 [architecture.md](architecture.md)。
- 用户没要的文档、示例、额外 Markdown 不主动新增。本知识库本身除外。
