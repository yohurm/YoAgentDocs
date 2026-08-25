---
id: research.synthesis.devops
type: synthesis
status: active
when: research
stack:
  capability: devops
---

# devops 横向总结

## 本层已研项目

| 仓库 | 一句话 | 利用方式 |
|------|--------|----------|
| [github/gitignore](github--gitignore.md) | 官方按语言拆片段，无鸿蒙，编辑器倾向部分入库 | reuse-pattern / adapt / anti-pattern |
| [gitignore.io](toptal--gitignore-io.md) | OS+IDE+语言拼接成一份文件 | reuse-pattern |
| [harmonyos-dev GitHub Action 示例](harmonyos-dev--harmonyos-github-action-example.md) | DevEco 短名单；常误忽略风格文件与 lock | adapt / anti-pattern |

本轮主题是仓库 `.gitignore` 模板，不是 CI 流水线本身。

## 共同架构经验

1. **先公共底、再按栈叠加。** 官方集拆文件，gitignore.io 做拼接。本库类型包已经是这个模型：`common` + android / harmonyos / frontend 等 overlay。
2. **忽略的是产物、缓存、密钥、本机路径，不是工程配置。** `build-profile.json5`、Gradle wrapper、`.editorconfig`、lock 文件应跟踪。
3. **点目录是编辑器/工具缓存的主战场。** `.idea`、`.vscode`、`.gradle`、`.hvigor`、`.kotlin`、`.cxx`、`.cursor` 都是点目录。一条 `.*/` 比维护 JetBrains XML 白名单更稳，也符合「点目录默认不提交」。
4. **反向包含只能针对目录本身。** Git 忽略父目录后不会再进入。需要入库的点目录必须先 `!/.github/` 这类规则；Yarn Berry 的 `.yarn/releases` 同理，在 frontend overlay 里处理。

## 分歧与取舍

| 议题 | 官方/社区 | 我们的选择 |
|------|-----------|------------|
| `.idea` / `.vscode` | 部分共享 | 整目录不提交（被 `.*/` 覆盖） |
| `.github` | 通常提交 | **唯一默认例外**：CI 与 issue 模板是仓库元数据 |
| Python `lib/` | GitHub Python 模板忽略 | 不忽略 |
| 鸿蒙 lock / `build-profile.json5` | 部分样本忽略 | 不忽略 |
| `.clang-format` | 部分鸿蒙样本忽略 | 不忽略 |
| 是否依赖 gitignore.io | 在线生成 | 不依赖；片段放本库离线复制 |

## 对本知识库规则的候选修订

已按用户要求落地到 `templates/gitignore/`，并在 `instructions/rules/common/git.md` 加了一条「应当」指向该目录。不另开独立规则文件。

## 入选与落选备忘

Top 3：官方片段库（规则来源）、gitignore.io（组装模型）、鸿蒙社区样本（官方集缺口）。

落选：把 `github/gitignore` 的 `VisualStudio.gitignore` 整份当桌面模板（过长且含无关生态）；单独再深研 JetBrains 文档（结论已包含在 Global 片段里：部分提交 `.idea`，我们明确不采用）。
