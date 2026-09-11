---
id: research.github-markup
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [Ruby]
  frameworks: [commonmarker]
also_relevant: [client-runtime]
utilization: [adapt, lesson-only]
source:
  platform: github
  repo: github/markup
  url: https://github.com/github/markup
  cloned_to: "%TEMP%/YoAgentResearch/github--markup"
studied_at: 2026-09-11
related: [research.markdown-it-markdown-it, research.synthesis.frontend]
---

# github/markup

## 入选理由

GitHub.com 把仓库里的 markup 变成 HTML 的第一段官方管道。要做「仓库阅读」必须先认清：Markdown 只是多种 markup 之一，后面还有消毒、高亮、图片 CDN。

## 项目是什么

按扩展名选底层库（`.md` → commonmarker / cmark-gfm）。它**只做 markup→HTML**，消毒与高亮在 GitHub.com 后续滤镜里。

## 架构

```
blob → 选引擎 → HTML →（站内）消毒 / linguist 高亮 / emoji / 任务列表 / 图片 CDN
```

Contents API 的 `Accept: application/vnd.github.html+json` 走的就是这条 Markup 库。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Markdown 允许嵌入 HTML | adapt | OmniRoute README 全是 HTML 块，本地 parser 必须 `html: true` 再消毒 |
| 不在 markup 里做高亮 | reuse-pattern | 代码文件走独立 blob 表面，不把 `.rs` 包进假 Markdown |
| 直接 POST /markdown 当预览 | anti-pattern | 无 token 会撞配额，桌面应本地 GFM |

## 架构设计经验

仓库阅读器的主对象是 **blob**，不是「文档」。先分型再选表面。

## 与当前工作

能直接用：分型 + GFM HTML + 消毒。不要：把整仓当专栏 Markdown 树；不要用 markup gem 进 Rust。

## 阅读范围

`README.md`；官方 Contents / Markdown REST 文档。未读 Ruby 引擎内部。
