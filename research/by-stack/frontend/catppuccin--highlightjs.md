---
id: research.catppuccin-highlightjs
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [SCSS]
  frameworks: [highlight.js, Catppuccin]
also_relevant: [ui-kit]
utilization: [adapt, anti-pattern]
source:
  platform: github
  repo: catppuccin/highlightjs
  url: https://github.com/catppuccin/highlightjs
  head: 79a8acc
  cloned_to: "%TEMP%/YoAgentResearch/catppuccin--highlightjs"
studied_at: 2026-09-11
related: [research.highlightjs-highlight-js, research.synthesis.frontend]
---

# catppuccin/highlightjs

## 入选理由

2025–2026 编辑器配色里，Catppuccin 是少数**浅色也站得住**的社区方案（Latte / Mocha 成对）。它直接给 hljs 类上色，和产品现有引擎对齐，用来对照「工作台能不能用一套非 GitHub、但仍专业的双主题」。

## 项目是什么

`@catppuccin/highlightjs`：四味（Latte / Frappé / Macchiato / Mocha）的 hljs CSS。颜色来自 `@catppuccin/palette`，不是手写 hex 散落。

## 架构

```
sass/_theme.scss
  $highlights: hljs 类 → palette 角色（mauve / green / peach / overlay2 …）
  @mixin highlights($flavor)
    .hljs { color: text; background: mantle }
    .hljs-* { color: 对应角色 }
latte/mocha/… 只选 flavor 再 include
variables 版假定宿主已有 --ctp-*
```

类映射比 Pretty Lights 更细（`title.class_` 与 `title.function_` 分色；`attr` 与 `name` 分开）。产品现在只有 8 个 `--yo-syntax-*`，装下这套要扩角色或合并。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Latte + Mocha 成对 | adapt | 仅当 Markdown 工作台要独立「阅读器身份」，不要跟 GitHub/华为抢皮 |
| `$highlights` 表 | lesson-only | 示范「类 → 语义色」；不要把 mauve/peach 名泄漏进产品 token |
| 整份 CSS 贴进 iframe | anti-pattern | 华为官网实测是琥珀关键字 + 灰字符串，不是pastel |

## 架构设计经验

社区优秀配色可以当**工作台皮肤**，不能当**官网复刻**。文档产品先锁「这篇表面应对齐谁」，再选表。

## 与当前工作

- **能直接用的：** 若将来做「工作台主题包」，Latte/Mocha 是可选项；变量版可挂到已有 appearance。
- **必须改写的：** 缩成现有 8 角色，或扩 testdata 角色表；禁止第三套 hex。
- **明确不要用的：** 华为网页、GitHub iframe。

## 阅读范围

`sass/_theme.scss`、`sass/catppuccin-latte.scss`、README。未读 Tailwind RGB 变体实现细节。
