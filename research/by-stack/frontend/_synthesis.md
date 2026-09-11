---
id: research.synthesis.frontend
type: synthesis
status: active
when: research
stack:
  capability: frontend
---

# frontend 横向总结

## 本层已研项目

| 仓库 | 一句话 | 利用方式 |
|------|--------|----------|
| [KaTeX/KaTeX](KaTeX--KaTeX.md) | TeX 子集同步排版；`renderToString` 给 MD，auto-render 给 HTML | reuse-pattern / adapt |
| [shikijs/shiki](shikijs--shiki.md) | TextMate 高亮 + 双主题；只给 Markdown 工作台 | reuse-pattern / anti-pattern（勿进华为 iframe） |
| [markdown-it/markdown-it](markdown-it--markdown-it.md) | CommonMark token 流；`html:false` + 插件出块 | reuse-pattern |
| [facebook/docusaurus](facebook--docusaurus.md) | MD 作用域排版与提示四态；默认暗色 Prism 勿抄到网页 | adapt / anti-pattern |
| [mdn/yari](mdn--yari.md) | HTML 文章表面：浅色代码、notecard、表滚动、铬文分离 | reuse-pattern |
| [github/markup](github--markup.md) | 仓库 blob 先分型再 markup→HTML | adapt |
| [sindresorhus/github-markdown-css](sindresorhus--github-markdown-css.md) | `.markdown-body` GitHub Markdown 皮 | adapt |
| [fregante/list-github-dir-content](fregante--list-github-dir-content.md) | Contents 一层目录 vs recursive 整仓 | reuse-pattern / anti-pattern |

对照（本层不重复全文）：VitePress / MkDocs Material 的**布局**在 ui-kit；本轮只借它们的正文块（`> [!NOTE]`、浅/深代码策略）。

## 共同架构经验

1. **网页 HTML 与 Markdown 是两条渲染器。** Yari 从不把官网 HTML 再送进 remark；markdown-it 从不声称能往返复杂 HTML。共用的是主题 token / 标题层级 testdata / 大纲 id 契约，不是同一份 CSS。
1b. **GitHub 仓库是第三条表面：blob 浏览器。** 主对象是文件，不是专栏文档。目录用 Contents 懒加载；Markdown 用 GFM+`html:true`+`markdown-body`；代码/图/二进制各走自己的预览。禁止 recursive 整仓树当默认。
2. **每个块有自己的解析点。** 标题、围栏、表格、提示、公式分别挂钩子；禁止 `render()` 后再用正则「修」HTML（华为 HTML 规范化除外，那是 HTML→HTML）。
3. **提示块不能变成标题。** Yari 压平 notecard 内 h4；华为 PNG 标签改成 `::before` 文案。大纲只收真实 h2–h4。
4. **代码块有两种合法皮肤。** 技术官网：浅底文档块 + 语言/来源条。Markdown 工作台：可跟外观走双主题 TextMate。不要用 Docusaurus palenight 覆盖官网 `pre`。
5. **公式按内容门控。** 无 `$`/`$$`/`<math>` 不加载运行时。MD 用 KaTeX `renderToString`；HTML 有 MathML 走原生，有 TeX 分隔符才 auto-render。
6. **TOC 是铬。** 所有入选项目都把大纲放在文章外。
7. **图片是流内替换元素。** `max-width:100%`；文章不要卡片阴影（Infima/Docusaurus 正文图也无阴影）。

## 分歧与取舍

| 议题 | 选项 | 本主题取舍 |
|------|------|------------|
| MD 高亮 | Prism / highlight.js / Shiki | Markdown 用细粒度 Shiki 双主题；网页用浅色 token 或保留原文，不上 Shiki |
| 提示语法 | `:::` / `> [!NOTE]` / 魔法 **Note:** | 转换引擎已产出 GitHub Alert，MD 必须解析它 |
| 公式 | KaTeX / MathJax / MathML | MD=KaTeX；HTML 优先保留 `<math>`，TeX 分隔符才 KaTeX |
| MD 是否 `html:true` | 方便 vs 安全 | `html:false` + 插件；DOMPurify 仅兜底 |
| 标题尺 | 官网 36/24/20 vs GitHub 下划线 | 网页跟实测官网；Markdown 用自己的 token 阶，两边都不抄对方 |

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。

- frontend 类型包可补一条：文档阅读产品若同时有「官网 HTML」与「Markdown」，必须两套正文作用域，禁止共享 `.markdown` / 官方文章 CSS。
- 设计来源：对标华为开发者文档网页时，度量对象是**文章表面**（标题/正文/note/pre/table），不是站点壳。

## 入选与落选备忘

**入选（Top 5）**

1. KaTeX — 公式块的事实标准，可 SSR 字符串。
2. Shiki — Markdown 代码块质量与双主题。
3. markdown-it — 产品已依赖，插件模型能接 Alert/公式。
4. Docusaurus — Markdown 块组件与作用域的对照（含反例：暗色 Prism、不认 GitHub Alert）。
5. Yari — HTML 技术文档阅读器同构。

**落选**

| 仓库 | 原因 |
|------|------|
| MathJax | 更重；KaTeX 已覆盖文档公式 |
| highlight.js 作 MD 默认 | 架构 v2 曾写；MD 改走 Shiki。网页浅色高亮仍可考虑它，不必再开一篇 |
| remark/rehype/MDX 全家桶 | 已有 markdown-it；换栈无收益 |
| github-markdown-css | 会把 MD 做成 GitHub 皮肤，不是独立设计 |
| cmark-gfm | 规范对照有用，但产品已在 JS 侧解析 |
| VitePress / MkDocs 正文 | 布局已研；本轮只作对照，避免 ui-kit 重复归档 |
