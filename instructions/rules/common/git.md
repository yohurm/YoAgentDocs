---
id: rule.common.git
type: rule
status: active
severity: must
scope: common
when: always
when_to_use: 涉及 git、提交、分支、远程时
related: [rule.common.security, templates.gitignore.hub]
---

# 版本与提交

## 必须

- 用户没明确要求时，不 `commit`、不 `push`、不改 git config。
- 不使用 `--force` / `--force-with-lease` 推 `main` 或 `master`；其他强制推送也必须用户明确要求。
- 不跳过 hooks（`--no-verify`、`--no-gpg-sign` 等），除非用户明确要求。
- 不把密钥、`.env`、凭证文件纳入提交；发现用户要求提交这类文件时先警告。
- 不改与任务无关的 git 历史；不用 `rebase -i` 等需要交互的命令。
- 提交说明写「为什么」，1～2 句；不要只列文件名。

## 应当

- 提交前看 status、diff、近期 log，使说明与仓库已有风格一致。
- 用户要求提交且改动跨多个模块时，按模块或功能分别提交，而不是一个杂包。
- 未要求时不开 PR、不新建长期分支。
- 远程操作前确认当前分支与跟踪关系，避免推错仓库。
- 远程默认 SSH，不用 GitHub HTTPS。多账号时 `push` 前用 `ssh -T git@github.com` 确认登录名。
- 新建或补全仓库 `.gitignore` 时，从 [templates/gitignore](../../../templates/gitignore/) 拼接：先 `common.gitignore`（点目录默认不提交），再叠类型包 overlay。不要整份拷贝 GitHub 的 `VisualStudio.gitignore` 或 JetBrains 的「部分提交 `.idea`」。
