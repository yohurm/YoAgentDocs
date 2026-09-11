---
id: research.sindresorhus-github-markdown-css
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [CSS]
  frameworks: []
also_relevant: []
utilization: [adapt]
source:
  platform: github
  repo: sindresorhus/github-markdown-css
  url: https://github.com/sindresorhus/github-markdown-css
  cloned_to: "%TEMP%/YoAgentResearch/sindresorhus--github-markdown-css"
studied_at: 2026-09-11
related: [research.github-markup, research.synthesis.frontend]
---

# sindresorhus/github-markdown-css

## 入选理由

官方推荐的 GitHub Markdown 皮肤。README 写明：容器 class 必须是 `markdown-body`，宽度约 980px。

## 项目是什么

从 GitHub.com 抽出来的生成 CSS，带 light/dark 与 `prefers-color-scheme` 自动切换。

## 架构

无运行时。样式挂在 `.markdown-body` 上。作者建议配 GitHub `/markdown` API 或 starry-night 高亮。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `.markdown-body` 作用域 | adapt | GitHub 方言阅读框单独用这份 CSS，不进华为/通用文章皮肤 |
| 当网页阅读通用皮 | anti-pattern | 与 Yari/华为铬冲突 |

## 架构设计经验

GitHub Markdown 是第三条阅读表面，不是把通用 `y-article` 再调参。

## 与当前工作

拷贝 `github-markdown.css` 进 `engine/github/`，只给 GitHub blob 的 Markdown 表面用。

## 阅读范围

`readme.md`、`github-markdown.css` 头部与 `.markdown-body` 规则。
