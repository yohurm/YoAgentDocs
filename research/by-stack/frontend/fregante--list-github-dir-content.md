---
id: research.fregante-list-github-dir-content
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [TypeScript]
  frameworks: []
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: fregante/list-github-dir-content
  url: https://github.com/fregante/list-github-dir-content
  cloned_to: "%TEMP%/YoAgentResearch/fregante--list-github-dir-content"
studied_at: 2026-09-11
related: [research.github-markup, research.synthesis.frontend]
---

# fregante/list-github-dir-content

## 入选理由

把 GitHub「列目录」的两条官方 API 写成可对照的实现：Contents（一层）vs Trees recursive（整仓）。

## 项目是什么

小组件：给定 owner/repo/dir，返回该目录下文件路径。

## 架构

- `viaContentsApi`：每个子目录一次请求，适合**大仓里只看一小枝**。单目录最多 1000 项。
- `viaTreesApi`：一次 recursive 整仓，大仓可达数 MB，且 `truncated`。

官方 Contents：`GET /repos/{o}/{r}/contents/{path}?ref=` 只返回**当前层**。这就是 github.com 文件树。

Linguist：`viewable? = text && size ≤ 1MiB`；图单独预览。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 目录一层一层拉 | reuse-pattern | 专栏展开文件夹再 `doc.catalog(treeUrl)` |
| recursive 当默认 | anti-pattern | OmniRoute 级别仓库会再次拖死 |

## 架构设计经验

代码浏览器的树是 **懒目录**，不是文档站点的预构建 TOC。

## 与当前工作

替换「只收 docs/ 的 recursive 子树」。根目录一次 Contents；点开文件夹再拉一层。

## 阅读范围

`readme.md`；GitHub REST Contents / Trees 文档。
