---
id: research.shikijs-shiki
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript]
  frameworks: [Shiki, TextMate]
also_relevant: [docs, ui-kit]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: shikijs/shiki
  url: https://github.com/shikijs/shiki
  head: 48cd2cc695ed2e3357c3f9c370578ea843d6d9a3
  cloned_to: "%TEMP%/YoAgentResearch/shikijs--shiki"
studied_at: 2026-09-09
related: [research.markdown-it-markdown-it, research.facebook-docusaurus, research.synthesis.frontend]
---

# shikijs/shiki

## 入选理由

Markdown 代码块需要语法着色，且工作台有浅/深外观。Shiki 是 VitePress 的编译期高亮器：TextMate 文法、双主题 CSS 变量、markdown-it 的 `highlight` 钩子现成。用来对照「不要把官网浅色文档块画成 IDE 暗色主题」。

## 项目是什么

基于 TextMate 文法的高亮引擎。语言/主题是你自己加载的 ESM 对象；默认 Oniguruma WASM，浏览器侧可用 JS 引擎避开 WASM。v4 单体包 `createHighlighter` 会把全量语言做成异步块，WebView 里应走细粒度 `@shikijs/core`。

## 架构

```
createHighlighterCore({ engine, langs, themes })
  → codeToTokens → codeToHast → HTML（inline style）
themes: { light, dark }  →  token.color + --shiki-dark
markdown-it 插件把 md.options.highlight 设成同步 codeToHtml
```

- `bundle/full` gzip ~1.3 MB；`core` ~34 KB。文档写明 Web 应用不要直接 `import 'shiki'`。
- 双主题：一份 HTML，靠 `html.dark .shiki span { color: var(--shiki-dark) }` 或 `light-dark()`。
- Transformers：行高亮、diff、focus；**不带 CSS**，调用方自己写。
- `highlight` 若返回以 `<pre` 开头的字符串，markdown-it 不再包一层 `<pre><code>`。
- 高亮器必须单例；`dispose()` 释放 WASM。渲染循环里 `createHighlighter` 是反模式。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 细粒度 langs + JS engine + 双主题 | reuse-pattern | Markdown 工作台：预载 ArkTS/TS/JSON/XML/bash 等 |
| `md.options.highlight` 同步 | reuse-pattern | 先 await 高亮器，再 render |
| 行高亮 / fence `{1,3}` transformers | adapt | 仅当 Markdown 源真有这些标记 |
| 全量 `shiki` bundle / Twoslash / Monaco | anti-pattern | 文档阅读器不需要 TS 语言服务 |
| 对华为 iframe 里现成 `<pre>` 再跑一遍 Shiki | anti-pattern | 会盖掉官网浅色文档块 |

## 架构设计经验

高亮是**表面策略**，不是全局单例皮肤。Markdown 可以用 TextMate 双主题；网页阅读应对齐官网浅底代码块。两套表面不要共用一个「暗色 IDE」主题。

## 与当前工作

- **能直接用的：** 单例、显式语言表、双主题变量、markdown-it `highlight` 契约。
- **必须改写的：** 语言表按华为文档实际 fence（`ArkTS`/`ts`/`json`/`xml`/`bash`/`text`）做 alias，不要导入全语法仓库。
- **明确不要用的：** 把 Shiki 打进网页 iframe 的 `officialSkin`；不要把 VitePress 的复制按钮/语言条抄成第二套 TOC。

## 阅读范围

`packages/core/src/highlight/code-to-html.ts`、`packages/markdown-it/src/core.ts`、`docs/guide/bundles.md`、`docs/guide/dual-themes.md`、`docs/guide/transformers.md`、`docs/guide/best-performance.md`。
