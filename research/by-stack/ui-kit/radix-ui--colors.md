---
id: research.radix-ui-colors
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: []
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: radix-ui/colors
  url: https://github.com/radix-ui/colors
  head: dbdb854
  cloned_to: "%TEMP%/YoAgentResearch/radix-ui--colors"
studied_at: 2026-09-11
related: [research.desktop-dark-color-scheme, research.radix-ui-primitives, research.synthesis.ui-kit]
---

# radix-ui/colors

## 入选理由

把「深色灰阶每一档干什么」写死成 12 步契约，而不是给一条 darken 公式。用来检验 YoUI 现有三级表面（canvas / surface / surface-2）是否过粗、画布该落在哪一档。不引进 Radix 控件。

## 项目是什么

`@radix-ui/colors`。`src/light.ts` 与 `src/dark.ts` 是两套手调色板，另有 alpha / Display P3。阅读 HEAD `dbdb854`（2025-12-17）。官方阶义见 [Understanding the scale](https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale)。

## 架构

每色相 12 步，**浅/深步义相同、hex 分表**：

| 步 | 用途 |
|----|------|
| 1 | App 画布 |
| 2 | 微抬表面（卡/侧栏） |
| 3–5 | 控件底 / hover / 选中 |
| 6–8 | 细分隔 → 控件边 → hover 边 |
| 9–10 | 实心底 / 实心 hover |
| 11–12 | 低对比字 / 高对比字 |

深色中性（`src/dark.ts` `grayDark`）：

| 步 | hex | 约等于鸿蒙 |
|----|-----|------------|
| 1 | `#111111` | 比 `#000` 抬一档 |
| 2 | `#191919` | **`background_secondary` `#191A1C`** |
| 3 | `#222222` | **`comp_background_primary` `#202224`** |
| 5 | `#313131` | **`background_fourth` `#2E3033`** |

文档写明：浅色画布可以是白，深色画布必须落到 gray 1 或 2，用可变别名，禁止「浅色白、深色也用步 1 的公式反相」。步 11/12 对步 2 底保证 APCA Lc 60 / 90。步 9 实心品牌色浅/深常同一 hex——上下文是实心底自己，不是周围表面。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 步义固定、hex 手调双表 | reuse-pattern | 对齐 YoUI Primitive→Semantic，不要 generate() |
| 深色画布 = 步 1/2 不是 `#000` | reuse-pattern | `#191A1C` 就是步 2 |
| 三级表面对应 2 / 3 / 5 | adapt | YoUI 现有 canvas/surface/surface-2 够用，不必扩到 12 步 |
| 引进 Radix hex / P3 / APCA | anti-pattern | 门禁继续鸿蒙 §1.6（深正文 ≥5:1，上限 17.6:1） |
| 品牌实心浅深同 hex | lesson-only | 鸿蒙深色品牌已是 `#317AF7`，保持官方表 |

## 架构设计经验

- 深色要单独校准，不能 `invert(light)`。
- 桌面工作台 3 档表面已能讲清「凹槽 / 卡片 / 次级」；缺的是画布落在步 2，不是再造 12 阶。
- 字色对比以「常用阅读底」为锚（Radix 用步 2；YoUI 用 surface，并应看一眼 canvas）。

## 与当前工作

- **能用：** `DarkColors.BgBase = Harmony.backgroundSecondary.dark`（`#191A1C`）。
- **必须改写：** 不抄 `gray1` `#111111`；不把 `--yohu-*` 改成 `--gray-*`。
- **不要用：** `@radix-ui/colors` 依赖、P3 双轨、自造 10/12 阶。

## 阅读范围

- `src/dark.ts`（`grayDark` / `grayDarkA` / `mauveDark` 开头）
- `src/index.ts` 导出面
- 官方阶义页（Web）
- 未读：全部色相、website 生成器、Themes 的 surface/contrast 附加 token
