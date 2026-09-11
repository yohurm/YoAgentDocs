---
id: research.KaTeX-KaTeX
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript, JavaScript, SCSS]
  frameworks: [KaTeX]
also_relevant: [docs]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: KaTeX/KaTeX
  url: https://github.com/KaTeX/KaTeX
  head: cd5ec4fb23803a6645ba90cccca2eeb7bc7be8dd
  cloned_to: "%TEMP%/YoAgentResearch/KaTeX--KaTeX"
studied_at: 2026-09-09
related: [research.markdown-it-markdown-it, research.mdn-yari, research.synthesis.frontend]
---

# KaTeX/KaTeX

## 入选理由

文档正文里的公式要单独成块，不能靠浏览器碰巧支持 MathML。KaTeX 是文档站（Docusaurus / VitePress 插件）的默认选择：同步排版、可 `renderToString`、自带字体，不必上完整 TeX 引擎。本轮主题是「网页 iframe 与 Markdown 工作台各怎么画公式」。

## 项目是什么

自包含的 TeX **子集** 排版库：Lexer → MacroExpander → Parser → HTML/MathML。不是 TeX 发行版，没有 `\input` / 宏包。输出确定，适合在解析 Markdown 时直接生成 HTML。

## 架构

```
katex.render / renderToString
  → Settings（displayMode / throwOnError / trust / maxExpand）
  → parseTree → buildHTML + buildMathML → span.katex
contrib/auto-render.renderMathInElement
  → 扫 DOM 分隔符（$$、\\[ \\]、AMS 环境；默认不开 $）
```

- 行内：`displayMode: false`（`\textstyle`）。块级：`true`（`span.katex-display`）。
- 自托管合同：`katex.min.css` 与同级 `fonts/`（woff2）。桌面应用禁止默认走 CDN。
- `trust: false` 挡住 `\url` / `\href` / `\includegraphics` / HTML 属性注入。
- `throwOnError: false` 时把坏公式画成红色源码，不炸整篇文章。
- 一份 `macros` 对象就是一个信任边界；不能做成进程级全局。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `renderToString` + `displayMode` | reuse-pattern | Markdown 工作台：插件解析 `$`/`$$` 后同步出 HTML |
| 自托管 CSS + fonts | reuse-pattern | 与 iframe / 工作台共用一套字体 |
| `renderMathInElement` 扫 iframe 正文 | adapt | 仅当网页 HTML 里出现分隔符；`ignoredTags` 含 `pre/code` |
| 默认开启 `$...$` | adapt | 必须排在 `$$` 之后，避免切错 |
| `trust: true` / 进程级 `macros` | anti-pattern | 打开 HTML 注入；跨文档串宏 |
| 把 Parser 抄进业务仓 / 上 MathJax 全家桶 | anti-pattern | 用 npm 包即可 |

## 架构设计经验

公式是**独立内容块**：解析（找分隔符）与排版（KaTeX）必须拆开。Markdown 走解析插件 + `renderToString`；网页 HTML 才允许事后 DOM 扫描。两种表面共用字体，不共用「何时跑 JS」的策略。

## 与当前工作

- **能直接用的：** `renderToString` 合同、`throwOnError: false`、自托管字体、分隔符顺序。
- **必须改写的：** 桌面壳不能 CDN；iframe 内跑 auto-render，不要在宿主扫 sandboxed 文档。
- **明确不要用的：** 黄金样本目前没有公式时，不要无条件把 `katex.js` 打进每篇文档；有公式再启用。不要用 KaTeX 去「美化」官方已经是 `<math>` 的片段（那种走 MathML）。

## 阅读范围

`katex.ts`、`src/Settings.ts`、`src/parseTree.ts`、`src/buildTree.ts`、`contrib/auto-render/auto-render.ts`、`docs/api.md`、`docs/autorender.md`、`docs/font.md`、`docs/security.md`。
