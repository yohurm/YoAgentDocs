---
id: research.highlightjs-highlight-js
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [JavaScript]
  frameworks: [highlight.js]
also_relevant: [ui-kit]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: highlightjs/highlight.js
  url: https://github.com/highlightjs/highlight.js
  head: fc3f063
  cloned_to: "%TEMP%/YoAgentResearch/highlightjs--highlight.js"
studied_at: 2026-09-11
related: [research.shikijs-shiki, research.wooorm-starry-night, research.catppuccin-highlightjs, research.synthesis.frontend]
---

# highlightjs/highlight.js

## 入选理由

YoDocPreview 网页解析与 Markdown **已经共用** `engine/syntax` + hljs 角色。要谈「优秀配色」，必须先读它怎么把 token 收成 CSS 类，以及官方 `github.css` / `github-dark.css` 如何把类绑到 Pretty Lights。

## 项目是什么

浏览器/Node 语法高亮器。按语言注册文法，`highlight(code, { language })` 产出带 `hljs-*` 类的 HTML。主题是**另一份 CSS**，引擎本身不带颜色。

## 架构

```
registerLanguage(id, def)
  → highlight({ language, ignoreIllegals })
  → <span class="hljs-keyword|string|comment|…">
src/styles/*.css   只给类涂色，可整份替换
```

- 核心与语言分包：`highlight.js/lib/core` + 按需 `languages/*`（产品已这样用）。
- `src/styles/github.css`（2021-05-15）把每个选择器注释成 `prettylights-syntax-*`。浅色关键字 `#d73a49`、字符串 `#032f62`、注释 `#6a737d`、常量 `#005cc5`、实体 `#6f42c1`、内建 `#e36209`、标签 `#22863a`。
- `github-dark.css` 对应暗色：关键字 `#ff7b72`、字符串 `#a5d6ff`、实体 `#d2a8ff`、内建 `#ffa657`。
- 类表是稳定契约（文档 [css-classes-reference](https://highlightjs.readthedocs.io/en/latest/css-classes-reference.html)）。换皮只改 CSS。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 类 → 角色 → `--yo-syntax-*` | reuse-pattern | 产品已有 `roles.ts` / `theme.css`，不要再写死 hex |
| `github.css` 与 Pretty Lights 的注释对照 | adapt | 作「文档浅色块」基线，但**不要当成华为官网现色** |
| 整页 `import 'highlight.js/styles/xxx.css'` | anti-pattern | 会绕过表面皮肤，GitHub / 华为 / Markdown 糊成一套 |
| 自动语言探测 | anti-pattern | 文档 fence / `codehub` / 扩展名已能定语言 |

## 架构设计经验

高亮引擎出**角色**，配色是**表面策略**。同一份 `hljs-keyword` 在华为文章、GitHub blob、Markdown 工作台可以绑三套 token，禁止第三份 hex 表。

## 与当前工作

- **能直接用的：** 继续 hljs 内核；浅/深 GitHub 主题色值作 Pretty Lights 旧版对照。
- **必须改写的：** Markdown `skin.css` 把语法角色绑到 `--yo-accent-text` / `--yo-ok-text` / `--yo-danger-text`（状态色，不是语法色）。网页华为 `--hl-*` 抄的是 `github.css`，与 2026-09-11 官网实测 span 色不一致。
- **明确不要用的：** 为「更好看」给网页 iframe 套 Atom One Dark / VS2015；官网是浅底文档块。

## 阅读范围

`src/styles/github.css`、`src/styles/github-dark.css`、README 用法段。未读全部 190+ 语言定义。
