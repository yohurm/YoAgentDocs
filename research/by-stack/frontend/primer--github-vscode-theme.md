---
id: research.primer-github-vscode-theme
type: project-study
status: active
when: research
stack:
  capability: frontend
  languages: [JavaScript]
  frameworks: [VS Code, Primer]
also_relevant: [ui-kit]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: primer/github-vscode-theme
  url: https://github.com/primer/github-vscode-theme
  head: cd78e5e
  cloned_to: "%TEMP%/YoAgentResearch/primer--github-vscode-theme"
studied_at: 2026-09-11
related: [research.wooorm-starry-night, research.shikijs-shiki, research.synthesis.frontend]
---

# primer/github-vscode-theme

## 入选理由

GitHub 官方编辑器主题。Shiki 的 `github-light-default` / `github-dark-default` 来自这里。用来确认：现行「最眼熟、浅深成对」的优秀方案就是 Pretty Lights 的 TextMate 版，而不是再发明一套。

## 项目是什么

VS Code 扩展。`src/theme.js` 从 Primer primitives 取 `scale` / `color.fg`，`lightDark(light, dark)` 生成多主题（default / dimmed / high contrast / colorblind）。

## 架构

```
getColors(theme) → Primer 色阶
tokenColors[]：TextMate scope → foreground
  comment → gray
  keyword / storage → red[5] / red[3]
  string → 蓝系深色（浅）/ 浅蓝（深）
  entity.name.function → purple
  entity.name.tag → green
  constant → blue
```

与 starry-night Pretty Lights、产品 `github-markdown.css` 是同一家族。Shiki 双主题只要加载这两个 JSON，不必自己配色。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Light Default + Dark Default 成对 | reuse-pattern | Markdown / GitHub 表面的首选「优秀配色」 |
| colorblind / high-contrast 变体 | adapt | 仅当产品做无障碍外观 |
| 把整份 VS Code `colors`（标题栏/按钮）抄进工作台 | anti-pattern | 壳层已有 YoUI token |

## 架构设计经验

优秀语法配色是**色阶 + scope 表**，不是散落 hex。产品已把 hljs 类收成 8 角色，缺的是把角色绑到这张色阶，而不是换引擎。

## 与当前工作

- **能直接用的：** 浅 `#cf222e` / `#0a3069` / `#59636e` / `#0550ae` / `#6639ba` / `#953800`；深 `#ff7b72` / `#a5d6ff` / `#9198a1` / `#79c0ff` / `#d2a8ff` / `#ffa657`（与 `style/both.css`、产品 GH 皮一致）。
- **必须改写的：** Markdown 表面改绑这张表（或华为官方实测表），不要绑状态色。
- **明确不要用的：** 把 VS Code 工作台铬颜色当文章皮肤。

## 阅读范围

`src/theme.js`（`getTheme`、`tokenColors` 前几组）、README。未读 colorblind 全部分支。
