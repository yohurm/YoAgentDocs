---
id: research.markdown-it-markdown-it
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript, JavaScript]
  frameworks: [markdown-it]
also_relevant: [docs]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: markdown-it/markdown-it
  url: https://github.com/markdown-it/markdown-it
  head: 924b203442f62cea128b5f2680294697621b416d
  cloned_to: "%TEMP%/YoAgentResearch/markdown-it--markdown-it"
studied_at: 2026-09-09
related: [research.shikijs-shiki, research.KaTeX-KaTeX, research.synthesis.frontend]
---

# markdown-it/markdown-it

## 入选理由

当前产品 Markdown 阅读已经依赖 markdown-it。深研它的规则链，才能正确接入 GFM 表格、围栏、GitHub Alert、公式，而不是 `html: true` 把转换产物当 HTML 回灌。

## 项目是什么

CommonMark 解析器 + 可选 GFM（表格、删除线）。同步、基于 token 流（不是嵌套 AST）。默认 `html: false`，`validateLink` 拒绝 `javascript:` / `file:` 等。

## 架构

```
render(src)
  → core: normalize → block → inline → linkify → typographer
  → renderer.render(tokens)
block 规则序：table, code, fence, blockquote, list, heading, paragraph, …
```

- `options.highlight(str, lang, attrs)` 只给 **fence** 用。返回以 `<pre` 开头则跳过内置包装。
- `html: true` 时 `html_block` / `html_inline` **原样输出**，文档要求必须外接 sanitizer。
- 插件：`md.block.ruler.before(...)` / `md.inline.ruler.after(...)` / 覆盖 `renderer.rules`。
- GitHub `> [!NOTE]`、`$` 公式都不是内置，必须插件。`$` 已是行内终止符（`0x24`），公式插件应 `after('escape')`。
- 没有异步规则。Shiki 必须先初始化，再同步 `highlight`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `html: false` + 插件出 HTML | reuse-pattern | 公式/高亮从 renderer 吐出，不靠 raw HTML |
| fence `highlight` 契约 | reuse-pattern | 对接 Shiki |
| core/block 规则变形 `> [!NOTE]` | adapt | 转换引擎已产出 GitHub Alert 语法 |
| `html: true` 当「能跑华为 HTML」 | anti-pattern | 网页表面根本不该进 markdown-it |
| 在 `highlight` 里 `await` | anti-pattern | API 同步 |

## 架构设计经验

Markdown 阅读器 = **解析器 + 一组块渲染器**。块（标题/代码/表格/提示/公式）各自挂规则，不要先 `render()` 再对 HTML 做正则。网页原文是另一条管道，禁止用同一 parser 往返。

## 与当前工作

- **能直接用的：** 已在 `toc.ts` 的 markdown-it；表格与围栏内置；`validateLink`。
- **必须改写的：** 现行 `html: true` + 事后 DOMPurify。目标改为 `html: false`，Alert/公式走插件；DOMPurify 只作最后兜底。
- **明确不要用的：** 用 markdown-it 解析华为 API HTML；把 `> [!NOTE]` 当普通引用（当前默认行为，转换白做了）。

## 阅读范围

`src/parser_core.ts`、`src/parser_block.ts`、`src/renderer.ts`、`src/rules_block/fence.ts`、`src/rules_block/table.ts`、`docs/architecture.md`、`docs/safety.md`。
