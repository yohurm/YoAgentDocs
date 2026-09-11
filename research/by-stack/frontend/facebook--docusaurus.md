---
id: research.facebook-docusaurus
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript, CSS]
  frameworks: [Docusaurus, Infima, MDX, Prism]
also_relevant: [docs, ui-kit]
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: facebook/docusaurus
  url: https://github.com/facebook/docusaurus
  head: 1434f56a07d83b6d52c6db2c7ef6403221747256
  cloned_to: "%TEMP%/YoAgentResearch/facebook--docusaurus"
studied_at: 2026-09-09
related: [research.mdn-yari, research.shikijs-shiki, research.synthesis.frontend]
---

# facebook/docusaurus（正文块，不含站点壳）

## 入选理由

对照「Markdown 文档工具怎么画标题/代码/提示/表格/图片/公式」。只读 classic 主题的内容组件与 Infima `.markdown` 作用域，不读布局（布局已有 VitePress / Material 深研）。

## 项目是什么

MDX 文档站。正文：remark 插件链 → MDX → `MDXProvider` 把 `pre/code/admonition` 换成主题组件 → Infima `.markdown` 排版。公式与 Shiki **都不是核心默认**（KaTeX 站点自配；高亮是 Prism palenight）。

## 架构

```
MD 源
  → remark：directive 提示块、标题 slug、GFM、图片/链接变换
  → rehype：默认空；站点可加 rehype-katex
  → <div class="markdown"> + Admonition / CodeBlock
TOC 在 DocItem 兄弟栏，不在 .markdown 内
```

内容块要点：

| 块 | 做法 |
|----|------|
| 提示 | `:::note/tip/warning/danger` → Infima `alert--*`；**不解析** `> [!NOTE]` |
| 代码围栏 | Prism 运行时，默认**暗色 palenight**，带 title / 行高亮 / 复制 |
| 行内代码 | Infima 底色+细边，不把字涂成品牌色 |
| 图片 | `max-width:100%`，**无卡片阴影** |
| 表格 | `display:block; overflow:auto` + 斑马纹 |
| 公式 | 站点级 remark-math + rehype-katex，可换成 MathJax |
| 标题 | h1 不进 TOC；h2+ 有 `id` 与 hover `#` |

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 正文 CSS 必须挂在独立作用域类上 | reuse-pattern | `.markdown` 不污染壳 |
| 提示类型枚举（note/tip/warning/danger） | adapt | 映射我们转换出的 `[!NOTE]`/`[!TIP]`/`[!WARNING]`，不抄 Infima 色板 |
| 图片不加阴影 | reuse-pattern | 现行 `.yo-md img { box-shadow }` 应去掉 |
| 默认暗色 Prism 当「所有文档代码块」 | anti-pattern | 网页阅读要对齐官网浅底；Markdown 也该跟工作台外观，而不是抄 palenight |
| Infima `table { display:block }` 套到华为表 | anti-pattern | 会打乱 `.tablenoborder` |
| 把 Docusaurus 的 `:::` 当唯一提示语法 | anti-pattern | 我们的 MD 源是 GitHub Alert |

## 架构设计经验

Markdown 主题是**作用域排版 + 块组件表**，不是「整页 GitHub CSS」。提示块的作者语法与皮肤要分开：语法跟我们的转换引擎，皮肤跟我们的 token。

## 与当前工作

- **能直接用的：** 作用域类、提示四态、图片无阴影、公式可插拔。
- **必须改写的：** 不引入 Infima / MDX / Prism-react-renderer；不把 hash-link `#` 画进标题（大纲已在工作台右侧）。
- **明确不要用的：** palenight 暗色代码当网页皮肤；在文章里再做一个 TOC。

## 阅读范围

`packages/docusaurus-mdx-loader/src/processor.ts`、`packages/docusaurus-theme-classic/src/theme/Admonition/`、`CodeBlock/`、`DocItem/Layout`、`MDXContent`。Infima 样式在 npm 依赖里，未进本 clone。
