---
id: research.github-gitignore
type: project-study
status: active
when: research
stack:
  capability: devops
  languages: []
  frameworks: []
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: github/gitignore
  url: https://github.com/github/gitignore
studied_at: 2026-08-25
related: [research.synthesis.devops]
---

# github/gitignore

## 入选理由

GitHub 官方模板集，创建仓库时的 chooser 数据源，是业界默认起点。要做「我们自己的一套」必须先看它怎么分层，以及哪些规则不能整份照抄。

## 项目是什么

按语言 / 框架拆开的 `.gitignore` 片段库，不是生成器：

- 根目录：主流语言与框架（`Android.gitignore`、`Node.gitignore`、`Python.gitignore`、`Go.gitignore`、`VisualStudio.gitignore`、`Rust.gitignore` 等）
- `Global/`：OS 与编辑器（`Windows` / `macOS` / `Linux` / `JetBrains` / `VisualStudioCode`）
- `community/`：尚未进入主流的专用模板

许可为 CC0，可直接改编。

## 架构

片段互不拼接。GitHub 网页让人「选一个主流模板」；OS / IDE 默认建议进全局 gitignore，而不是每个仓库都合并。

JetBrains 官方片段**不**忽略整个 `.idea/`，只排除用户态 XML。VS Code 片段忽略 `.vscode/*` 再反向放出 `settings.json` / `launch.json`。语言模板里 `.idea/` / `.vscode/` 多为注释，交给 Global。

`Android.gitignore` 覆盖 Gradle、`local.properties`、`captures/`、`.cxx/`、APK/AAB、keystore、`google-services.json`。未写 Kotlin 官方要求忽略的 `.kotlin/`。没有 HarmonyOS / hvigor / `oh_modules` 模板。

`Python.gitignore` 含 `lib/`、`lib64/`，会误伤源码目录。`VisualStudio.gitignore` 很长，含 Silverlight、BizTalk 等与当前桌面栈无关的条目。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 根目录 / Global / community 三分 | reuse-pattern | 我们改成 common + 类型包 overlay，对应关系见横向总结 |
| Android / Node / Python / Go / Rust / VS 的产物名单 | adapt | 只取构建产物与密钥，不整文件拷贝 |
| JetBrains / VS Code 部分提交 `.idea` / `.vscode` | anti-pattern | 与「点目录默认不提交」冲突，不采用 |
| Python 忽略 `lib/` | anti-pattern | 不写入我们的 python overlay |
| 无 HarmonyOS | lesson-only | 必须自建 overlay |

## 架构设计经验

官方集按**语言**切，不按产品类型切。多栈仓库（桌面 WebView + Rust crate + Node UI）需要自己拼接，不能只丢一份 `VisualStudio.gitignore`。

编辑器目录是否入库，官方采取「部分共享」。团队若统一「点目录不提交」，应用一条 `.*/`，而不是维护 JetBrains 那份 XML 白名单。

## 与当前工作

能直接用：各语言的构建产物、密钥、OS 垃圾文件名单。

必须改写：点目录策略；HarmonyOS；去掉 Python `lib/`；桌面栈用精简 MSBuild/Cargo 而不是整份 VS 模板。

明确不要用：把 `Global/JetBrains.gitignore` 当默认；忽略 `oh-package-lock.json5` / `build-profile.json5`（那是工程源，不是官方 GitHub 模板的问题，见鸿蒙样本）。

## 阅读范围

官方 raw：`README.md`、`Android.gitignore`、`Node.gitignore`、`Python.gitignore`、`Go.gitignore`、`Java.gitignore`、`Rust.gitignore`、`VisualStudio.gitignore`、`Global/Windows.gitignore`、`Global/macOS.gitignore`、`Global/Linux.gitignore`、`Global/JetBrains.gitignore`、`Global/VisualStudioCode.gitignore`。未读完 `community/` 下全部专用模板。本机 `git clone` 因 `ca-bundle.crt` 失败，未落到 Temp 工作树。
