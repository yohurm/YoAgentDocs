---
id: research.harmonyos-dev-harmonyos-github-action-example
type: project-study
status: active
when: research
stack:
  capability: devops
  languages: [ArkTS, TypeScript]
  frameworks: [hvigor, ohpm]
also_relevant: []
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: harmonyos-dev/harmonyos-github-action-example
  url: https://github.com/harmonyos-dev/harmonyos-github-action-example
studied_at: 2026-08-25
related: [research.synthesis.devops]
---

# harmonyos-dev/harmonyos-github-action-example

## 入选理由

`github/gitignore` 没有 HarmonyOS 模板。该样本的 `.gitignore` 与 OpenHarmony `applications_startup_guide`、Gitee `HarmonyOS-Cases` 高度同构，代表 DevEco / hvigor 工程的事实标准。

## 项目是什么

鸿蒙工程在 GitHub Actions 上构建的示例。根目录 `.gitignore` 是 DevEco 新建工程常见的那一份短名单。

## 架构

忽略清单（样本原文）：

```
/node_modules
/oh_modules
/local.properties
/.idea
**/build
/.hvigor
.cxx
/.clangd
/.clang-format
/.clang-tidy
**/.test
```

对照其他鸿蒙仓库后的差异：

- 多数同样忽略 `oh_modules`、`.hvigor`、`local.properties`、`**/build`
- 部分额外忽略 `*.hap` / `*.har` / `*.hsp`、`.preview/`、`.appanalyzer/`
- 有的忽略 `oh-package-lock.json5`、`build-profile.json5`、`hvigor/hvigor-wrapper.js`、`**/BuildProfile.ets`
- WuKongIM 的 SDK 样本尝试「忽略 `.idea/*` 再放出部分 XML」，与 JetBrains 官方同一思路

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `oh_modules` / `.hvigor` / `local.properties` / `**/build` / `.cxx` | adapt | 写入 harmonyos overlay；点目录部分改由 `.*/` 覆盖 |
| 忽略 `.clang-format` / `.clang-tidy` | anti-pattern | 这是共享代码风格，应入库 |
| 忽略 `oh-package-lock.json5`、`build-profile.json5`、hvigor wrapper | anti-pattern | 锁文件与工程配置是源，不是产物 |
| `**/BuildProfile.ets`、`*.hap`/`*.har` | adapt | 生成物，overlay 里忽略 |
| 部分提交 `.idea` | anti-pattern | 与点目录政策冲突 |

## 架构设计经验

鸿蒙官方/社区模板是「根路径一条条写」，且常把**风格文件、锁文件、工程配置**和**缓存**混在一起忽略。我们拆开：缓存与点目录走 common；`oh_modules` 与包产物走 overlay；配置与 lock 默认跟踪。

## 与当前工作

能直接用：依赖目录、hvigor 缓存、本地 SDK 路径、模块 `build/`、签名包后缀。

必须改写：不要忽略 `.clang-format`；不要忽略 lock 与 `build-profile.json5`。

明确不要用：把 DevEco 生成的短名单当完整模板（缺 OS、密钥、`.github` 例外说明）。

## 阅读范围

该仓库 `.gitignore`；对照 OpenHarmony `applications_startup_guide`、Gitee `HarmonyOS-Cases/CommonAppDevelopment`、`harmonyos-dev/aigc-harmonyos-sample`、`WuKongIM/WuKongIMHarmonyOSSDK`、`Explore-In-HMOS-Wearable/serenity-kit` 的 ignore 原文。未克隆完整工程。
