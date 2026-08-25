---
id: templates.gitignore.hub
type: hub
status: active
when_to_use: 新建仓库或补全 .gitignore 时
related: [rule.common.git, research.synthesis.devops]
---

# .gitignore 模板

复制后拼成仓库根目录的 `.gitignore`。**先 `common.gitignore`，再叠与类型包对应的 overlay。** 不要从 GitHub 整份 `VisualStudio.gitignore` 或 gitignore.io 在线生成结果当唯一文件。

点目录（`.idea`、`.vscode`、`.gradle`、`.hvigor`、`.kotlin`、`.cxx`、`.cursor` 等）由 common 的 `.*/` 默认忽略。`.gitignore` / `.editorconfig` / `.gitattributes` 是文件，不受 `.*/` 影响。

`.github/` 是唯一默认反向包含：CI 与 issue 模板是仓库元数据。不需要 GitHub/Gitee 工作流时删掉 common 里那两行。

## 怎么拼

| 仓库类型 | 拼接顺序 |
|----------|----------|
| Android | common + [android](android.gitignore) |
| HarmonyOS | common + [harmonyos](harmonyos.gitignore) |
| Windows 桌面（Yohu 一类） | common + [windows-desktop](windows-desktop.gitignore)；有 WebView UI 再加 frontend；有 Cargo 再加 rust |
| Web 前端 | common + [frontend](frontend.gitignore) |
| Python（backend / agent / llm-app / data） | common + [python](python.gitignore) |
| Go | common + [go](go.gitignore) |
| Rust 库或 crate | common + [rust](rust.gitignore) |
| 本知识库 / 文档站 | common + [docs](docs.gitignore) |

多类型叠加：common 只出现一次，overlay 按上表追加。重复的 `build/`、`node_modules/` 无害。

## 片段

- [common.gitignore](common.gitignore) — 必选底：点目录、OS 垃圾、密钥、日志
- [android.gitignore](android.gitignore)
- [harmonyos.gitignore](harmonyos.gitignore)
- [windows-desktop.gitignore](windows-desktop.gitignore)
- [frontend.gitignore](frontend.gitignore)
- [python.gitignore](python.gitignore)
- [go.gitignore](go.gitignore)
- [rust.gitignore](rust.gitignore)
- [docs.gitignore](docs.gitignore)

## 不要默认忽略

- Gradle / hvigor wrapper、`build-profile.json5`、`oh-package-lock.json5`、`package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`
- `.clang-format`、`.clang-tidy`、`.editorconfig`
- 源码目录名叫 `lib/` 的树（GitHub Python 模板会误伤）

需要跟踪某个点目录时，写在 common **之后**：

```
!.yarn/
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/sdks
!.yarn/versions
```

Git 忽略父目录后不会再进入；必须先反向包含该目录本身。frontend overlay 已带 Yarn Berry 这一段。
