---
id: research.material-foundation-material-color-utilities
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, Java, Dart, C++]
  frameworks: [Material 3]
also_relevant: [frontend]
utilization: [lesson-only, anti-pattern, adapt]
source:
  platform: github
  repo: material-foundation/material-color-utilities
  url: https://github.com/material-foundation/material-color-utilities
  head: 5b3618b
  cloned_to: "%TEMP%/YoAgentResearch/material-foundation--material-color-utilities"
studied_at: 2026-09-11
related: [research.desktop-dark-color-scheme, research.synthesis.ui-kit]
---

# material-foundation/material-color-utilities

## 入选理由

Material 3 色角角色与 HCT 色调海拔的**规范实现**。用来确认「深色默认表面是 tone 6 附近，纯黑只给最低容器」，并明确 **不要** 把动态配色算法引进 YoUI。对照官方 [Color roles](https://m3.material.io/styles/color/roles)。

## 项目是什么

Google 的跨语言色库（TS / Java / Dart / C++）。从种子色生成 tonal palette，再映射 `surface` / `surfaceContainer*` / `onSurface`。阅读 HEAD `5b3618b`（2026-08-21），主读 TypeScript `dynamiccolor/`。

## 架构

```
种子 HCT
  → TonalPalette（同一色相/彩度，tone 0–100）
  → DynamicColor（角色 = palette + tone(scheme) + 对比曲线）
  → ColorScheme（light / dark 各一袋角色）
```

`color_spec_2021.ts` 深色默认（contrastLevel=0）：

| 角色 | tone | 含义 |
|------|------|------|
| background / surface | 6 | 默认底，**不是 0** |
| surfaceContainerLowest | 4 | 最凹；高对比才落到 0 |
| surfaceContainerLow | 10 | 低抬 |
| surfaceContainer | 12 | 卡片默认 |
| surfaceContainerHigh / Highest | 17 / 22 | 再抬 |
| onSurface | 90 | 正文；不是 100 |

`color_spec_2025.ts` 把 lowest 在深色收到 tone 0（OLED 底），**默认 surface 仍不落 0**。`color_spec_2026.ts` 默认 surface 用 tone 4，bright 用 18，容器 6/9/12/15 递进。海拔公式（Compose `surfaceColorAtElevation`）是 `surfaceTint` 按 `ln(dp+1)` 叠在 surface 上——深色越抬越亮。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 深色默认 surface ≈ tone 4–6 | lesson-only | 佐证「工作台画布不要 `#000`」 |
| 层级用明度，少靠投影 | adapt | 鸿蒙 §1.6 已写「投影感知降低，改用明度」 |
| HCT / 动态配色 / surfaceTint | anti-pattern | 与「色板锁鸿蒙官方表」冲突 |
| 扩到 5 个 surfaceContainer | anti-pattern | YoUI 三级 + hairline 已够；不要 Material 角色名 |

## 架构设计经验

- 角色（canvas / card / nested）稳定，tone 随主题变。
- 纯黑是「最低容器」选项，不是默认页。
- 深色正文用 tone 90（约 90% 白），与鸿蒙 `font_primary` 90% 同构。

## 与当前工作

- **能用：** 继续三级明度抬升；画布离开 tone 0。
- **必须改写：** 值取鸿蒙 `#191A1C` / `#202224` / `#2E3033`，不跑 HCT。
- **不要用：** `material-color-utilities`、M3 角色名、`surfaceTint` 叠品牌蓝（会脏中性工作台）。

## 阅读范围

- `typescript/dynamiccolor/material_dynamic_colors.ts`
- `typescript/dynamiccolor/color_spec_2021.ts`（surface 段）
- `typescript/dynamiccolor/color_spec_2025.ts` / `color_spec_2026.ts`（tone 表）
- 官方 Color roles / Elevation 页（Web）
- 未读：量化/评分、Android/Dart 绑定、各 Scheme 变体实现细节
