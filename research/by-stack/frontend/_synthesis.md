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
| [highlightjs/highlight.js](highlightjs--highlight.js.md) | 类高亮 + `github.css` 旧 Pretty Lights | reuse-pattern / anti-pattern（勿整页贴主题 CSS） |
| [wooorm/starry-night](wooorm--starry-night.md) | GitHub Pretty Lights 现行变量与 `pl-*` | reuse-pattern（色表）/ anti-pattern（WASM 引擎） |
| [catppuccin/highlightjs](catppuccin--highlightjs.md) | Latte/Mocha 社区双主题 | adapt（仅工作台身份）/ anti-pattern（勿进官网 iframe） |
| [primer/github-vscode-theme](primer--github-vscode-theme.md) | 官方 Light/Dark Default，Shiki 同源 | reuse-pattern |

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
| MD 高亮 | Prism / highlight.js / Shiki | **配色优先于换引擎。** 现已 hljs + `--yo-syntax-*`。Markdown 绑 Pretty Lights（github-light/dark-default）；网页华为绑官网实测 span；GitHub 表面维持 prettylights。Shiki 仍是文法升级备选，不是换皮前提 |
| 语法配色 | 状态色 / 旧 github.css / 现行 Pretty Lights / Catppuccin / 官网实测 | 见下节「语法配色对照」。禁止 Markdown 用 `--yo-accent/--yo-danger` 冒充语法色 |
| 提示语法 | `:::` / `> [!NOTE]` / 魔法 **Note:** | 转换引擎已产出 GitHub Alert，MD 必须解析它 |
| 公式 | KaTeX / MathJax / MathML | MD=KaTeX；HTML 优先保留 `<math>`，TeX 分隔符才 KaTeX |
| MD 是否 `html:true` | 方便 vs 安全 | `html:false` + 插件；DOMPurify 仅兜底 |
| 标题尺 | 官网 36/24/20 vs GitHub 下划线 | 网页跟实测官网；Markdown 用自己的 token 阶，两边都不抄对方 |

## 语法配色对照（2026-09-11）

产品三表面必须各锁一张表，角色名继续用 `--yo-syntax-*`（8 个），只换绑定。

| 角色 | 华为官网实测 span（introduction-to-arkts） | 华为皮现状 `--hl-*`（抄 hljs github.css） | 现行 Pretty Lights 浅 / 深 | Catppuccin Latte（对照） |
|------|------------------------------------------|------------------------------------------|---------------------------|-------------------------|
| keyword | `let` `#C98208` | `#d73a49` | `#cf222e` / `#ff7b72` | mauve |
| string | `'hello'` `#393939` | `#032f62` | `#0a3069` / `#a5d6ff` | green |
| comment | （本块无） | `#6a737d` | `#59636e` / `#9198a1` | overlay2 |
| constant / attr | `hi` `#5531EB` | `#005cc5` | `#0550ae` / `#79c0ff` | blue / peach |
| entity | （本块无 title） | `#6f42c1` | `#6639ba` / `#d2a8ff` | blue |
| variable / built_in | `string` `#127180` | `#e36209` | `#953800` / `#ffa657` | red / mauve |
| 块底 | `.highlight-div` `#fafafa` | `--code-bg #fafafa`（对） | canvas inset / 深底 | mantle |

结论：

1. **网页（华为）：** 块底已对。token 色抄的是 2021 GitHub 红，不是今天官网。优秀方案 = 把实测 span 锁进 testdata，再绑 `--yo-syntax-*`。不要上暗色 IDE，不要上 Catppuccin。
2. **网页（GitHub）：** 已绑 Pretty Lights，保持。这是 2025–2026 最眼熟的成对方案（GitHub Theme / Shiki default）。
3. **Markdown：** 现状用强调/成功/危险等**状态色**，不是语法色，所以两边都「不够优秀」。应改绑 Pretty Lights 浅/深（跟外观走），与 GitHub 表面同一张表。不要为换皮引入 starry-night / 全量 Shiki。
4. **引擎：** 网页与 Markdown 继续共用 hljs 角色内核。换 Shiki 只解决文法精度，不解决配色；配色在皮肤层一次换绑即可。

官网代码块容器：`pre` 透明，底色在父级 `.highlight-div` `#fafafa`，圆角 12px。

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。

- frontend 类型包可补一条：文档阅读产品若同时有「官网 HTML」与「Markdown」，必须两套正文作用域，禁止共享 `.markdown` / 官方文章 CSS。
- 设计来源：对标华为开发者文档网页时，度量对象是**文章表面**（标题/正文/note/pre/table），不是站点壳。

## 入选与落选备忘

**入选（历史 Top 5 仍有效）**

1. KaTeX — 公式块的事实标准，可 SSR 字符串。
2. Shiki — 文法/双主题备选；配色可直接用其 `github-light-default` / `github-dark-default`（来自 primer/github-vscode-theme），不必先换引擎。
3. markdown-it — 产品已依赖，插件模型能接 Alert/公式。
4. Docusaurus — Markdown 块组件与作用域的对照（含反例：暗色 Prism、不认 GitHub Alert）。
5. Yari — HTML 技术文档阅读器同构。

**入选（2026-09-11 配色专题 Top 4）**

1. highlight.js — 现引擎；`github.css` 是旧 Pretty Lights。
2. starry-night — 现行 Pretty Lights 变量单源对照。
3. primer/github-vscode-theme — 浅/深 Default，业界最眼熟成对方案。
4. catppuccin/highlightjs — 工作台可选身份皮，不进官网。

**落选**

| 仓库 | 原因 |
|------|------|
| MathJax | 更重；KaTeX 已覆盖文档公式 |
| highlight.js 作 MD 默认 | **已改判：** 产品已落地 hljs。本轮补了单篇深研；MD 默认不必换 Shiki，先换配色绑定 |
| Prism / One Dark Pro / Tokyo Night | 暗色 IDE，偏离文档浅块；与 Pretty Lights 重复 |
| primer/primitives 单篇 | Pretty Lights 数值以 starry-night `style/both.css` 与产品 `github-markdown.css` 为准，不另开一篇 |
| remark/rehype/MDX 全家桶 | 已有 markdown-it；换栈无收益 |
| github-markdown-css | 会把 MD 做成 GitHub 皮肤，不是独立设计 |
| cmark-gfm | 规范对照有用，但产品已在 JS 侧解析 |
| VitePress / MkDocs 正文 | 布局已研；本轮只作对照，避免 ui-kit 重复归档 |
