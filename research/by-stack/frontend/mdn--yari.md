---
id: research.mdn-yari
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript, SCSS]
  frameworks: [Yari, React, Prism, Cheerio]
also_relevant: [docs]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: mdn/yari
  url: https://github.com/mdn/yari
  head: 3d6a8ade29a3e2d324341f7ab56b75cb59a67def
  cloned_to: "%TEMP%/YoAgentResearch/mdn--yari"
studied_at: 2026-09-09
related: [research.facebook-docusaurus, research.KaTeX-KaTeX, research.synthesis.frontend]
---

# mdn/yari（HTML 文章表面）

## 入选理由

网页阅读面对的是**已经是 HTML 的技术文档**，不是 Markdown。Yari 是 MDN 的文章引擎：把 HTML 正文放进 `<article>`，外壳（侧栏/大纲/顶栏）全部在文章外。这与华为文档 iframe 同构。

## 项目是什么

构建期用 Cheerio 改 HTML，产出 `doc.body` 段列表；客户端 `main-page-content` 注入 prose。Markdown 源会先 `m2h` 变成同一套 HTML，再走同一管道。

## 架构

```
KumaScript HTML 或 markdown/m2h
  → wrapCodeExamples / formatNotecards / wrapTables
  → extractSections（抽走首个 h2/h3）
  → <article class="main-page-content">
       header h1（元数据标题）
       .section-content { 危险地 innerHTML }
TOC / Sidebar = 兄弟铬，不在 article 内
```

| 块 | Yari |
|----|------|
| 代码 | `pre.brush` → `.code-example` 语言条 + **浅色** Prism token（CSS 变量，不是暗色主题表） |
| 提示 | `.notecard` 左强调 + 行内 `<strong>Note:</strong>`；构建期压平内部 h4，避免进大纲 |
| 表格 | `figure.table-container` 横向滚动；保留官方表 class |
| 图片 | 居中、`height:auto`、细边；暗色下白底托透明图 |
| 公式 | 原生 `<math>` + 按需 MathML polyfill；**不用 KaTeX** |
| 标题 | 页标题来自 JSON；正文 h2 margin-top 很大；permalink `#` 在标题上 |

`m2h` 把 `> [!NOTE]` / `**Note:**` 收成 notecard。这只发生在 **MD→HTML**，不是反过来解析官网 HTML。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 文章根 vs 工作台铬分离 | reuse-pattern | iframe 只装文章；专栏/大纲留在壳 |
| 浅色文档代码块 + 语言条 | reuse-pattern | 对齐华为 `pre.TypeScript` + codehub，而不是 IDE 暗色 |
| 提示压成 callout、禁止变成标题 | reuse-pattern | 与现有 `normalizeArticleHtml` 藏 PNG、用 `::before` 标签一致 |
| 宽表外包滚动容器 | adapt | 保留 `.tablenoborder` |
| 有 `<math>` 再加载公式运行时 | reuse-pattern | 按内容门控 |
| 用 GitHub/Infima CSS 去套官方 HTML | anti-pattern | 两套皮肤互毁 |
| 在 iframe 里再长一个 TOC | anti-pattern | 工作台已有大纲 |
| 给整篇文章加卡片阴影 | anti-pattern | 文章表面不是组件 |

## 架构设计经验

HTML 技术文档阅读器：**规范化 HTML + 一篇文章 CSS**。不要把 Markdown 主题（`.vp-doc` / `.markdown` / `.md-typeset`）铺上去。代码块家族是「浅底文档块」，与 Markdown 工作台的双主题高亮是两条产品决定。

## 与当前工作

- **能直接用的：** 文章根选择器、浅色代码、提示非标题、表滚动包裹、公式门控、铬/文分离。
- **必须改写的：** 华为标题层级仍以 `testdata/huawei-headings.json` 为准，不要抄 MDN 的「剥 h2 再 React 重插」。permalink `#` 不要做（大纲点击已滚动）。
- **明确不要用的：** MDN 顶栏/侧栏/Baseline；Prism 语言动态 import 全表；把 `dangerouslySetInnerHTML` 用在工作台 Markdown（那条走 markdown-it）。

## 阅读范围

`client/src/document/index.tsx`、`ingredients/prose.tsx`、`code/syntax-highlight.tsx`、`build/wrap-tables.ts`、`build/format-notecards.ts`、`markdown/m2h/handlers/code.ts`、`ui/molecules/notecards/index.scss`。
