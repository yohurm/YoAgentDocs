---
id: research.wooorm-starry-night
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [JavaScript]
  frameworks: [TextMate, PrettyLights]
also_relevant: [docs]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: wooorm/starry-night
  url: https://github.com/wooorm/starry-night
  head: 78f05cc
  cloned_to: "%TEMP%/YoAgentResearch/wooorm--starry-night"
studied_at: 2026-09-11
related: [research.highlightjs-highlight-js, research.primer-github-vscode-theme, research.sindresorhus-github-markdown-css, research.synthesis.frontend]
---

# wooorm/starry-night

## 入选理由

GitHub.com 代码着色的开源复刻：TextMate 文法 + **Pretty Lights 类名**（`pl-k` / `pl-s`）+ 官方 CSS 变量。产品 GitHub 表面已经把 `--yo-syntax-*` 绑到 `--color-prettylights-syntax-*`，需要这份对照「角色名从哪来、新浅色和 hljs github.css 差在哪」。

## 项目是什么

开源版 GitHub PrettyLights。`createStarryNight(grammars)` 后 `highlight(value, scope)` 产出 hast，类名跟 github.com 一致。主题不进 JS，只在 `style/*.css`。

## 架构

```
vscode-textmate + Oniguruma WASM
  → scope 链（keyword / string / entity.name.tag …）
  → lib/theme.js 的 scopeToClassGithub
  → pl-c / pl-k / pl-s / pl-ent …
style/both.css：:root 浅色 Pretty Lights + prefers-color-scheme 深色
```

浅色现行值（`style/both.css`）：关键字 `#cf222e`、字符串 `#0a3069`、注释 `#59636e`、常量 `#0550ae`、实体 `#6639ba`、变量 `#953800`。  
这与 hljs `github.css` 的 `#d73a49` / `#032f62` **不是同一代**。产品 `github-markdown.css` 已是这一代。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Pretty Lights 变量表 | reuse-pattern | GitHub 表面继续用；Markdown 若要「优秀且眼熟」应绑同一表 |
| scope → `pl-*` 映射 | lesson-only | 产品走 hljs 类，不要并行再出一套 `pl-*` |
| 600+ 文法 + WASM | anti-pattern | WebView 里再叠一套 TextMate，体积与生命周期都重 |
| markdown-it 示例换引擎 | anti-pattern | 已有 syntax 内核 |

## 架构设计经验

「GitHub 配色」有两代：2018–2021 hljs `github.css`，和现行 Pretty Lights（`#cf222e`）。文档站对标 github.com 用新表；对标某年官网截图要用当时那张表，并写进 testdata。

## 与当前工作

- **能直接用的：** 浅/深 Pretty Lights 八个主角色，正好覆盖 `--yo-syntax-keyword|string|comment|constant|entity|entity-tag|variable|regexp`。
- **必须改写的：** 不要把 Pretty Lights 涂到华为 iframe（官网不是这套）。
- **明确不要用的：** 为追 GitHub 高亮精度引入 starry-night WASM。

## 阅读范围

`lib/theme.js`、`lib/index.js`、`style/both.css`、readme「What is PrettyLights / CSS」。未读 600+ `lang/`。
