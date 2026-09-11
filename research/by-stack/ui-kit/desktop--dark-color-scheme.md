---
id: research.desktop-dark-color-scheme
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, Rust]
  frameworks: [YoUI]
also_relevant: [client-runtime]
utilization: [adapt, reuse-pattern, anti-pattern]
source:
  platform: other
  url: https://m3.material.io/styles/color/roles
  repos:
    - primer/primitives
    - radix-ui/colors
    - material-foundation/material-color-utilities
  cloned_to:
    - "%TEMP%/YoAgentResearch/primer--primitives"
    - "%TEMP%/YoAgentResearch/radix-ui--colors"
    - "%TEMP%/YoAgentResearch/material-foundation--material-color-utilities"
studied_at: 2026-09-11
related:
  - research.primer-primitives
  - research.radix-ui-colors
  - research.material-foundation-material-color-utilities
  - research.synthesis.ui-kit
---

# 桌面工作台深色色板

## 背景

YoUI 浅色画布早已用鸿蒙雪域灰 `background_secondary` `#F1F3F5`（不是白）。深色却把 `--yohu-bg-base` 映射成 `background_primary` `#000000`，理由是文档正文「深色 Primary/Secondary 默认都为黑」。这是手机 AMOLED 语言。Yohu 交付 Windows / macOS 桌面工作台，启动 GDI、浮层阴影、卡片凹槽都吃这份画布。

鸿蒙同一节还写：深色正文对比谨慎超过 **17.6:1**（推荐 ≤15.7:1）；舒适性要避免刺眼和黑白页对跳；投影感知降低，**用明度表达层级**。表内 `background_secondary` 深色仍是 `#191A1C`（gray_02）。先前禁令只说它**不能当 surface-2**（比卡片还暗），没说它不能当画布。

## 关键结论

行业默认深色页都不是 OLED 纯黑：

| 体系 | 默认深色页 | 卡片/次级 | 纯黑用途 |
|------|------------|-----------|----------|
| Primer dark | `#0D1117`（neutral.1） | `#151B23` / 再抬 | inset / 高对比 |
| Radix gray | `#111111` 或 `#191919` | `#222222` / `#313131` | 不用作 App 底 |
| M3 2021 | surface tone 6 | container 10–22 | lowest 高对比才 tone 0 |
| Fluent 2 Web | `grey[16]` ≈ `#292929`（Background1） | 更深档做凹陷 | 高对比画布 |
| 鸿蒙表 | `background_secondary` `#191A1C` | `#202224` / `#2E3033` | `background_primary` 正文「黑」 |
| macOS 窗 | 材料 + 桌面着色，实色约 `#2A–32` | 分组面更亮 | 不是扁平 `#000` |

浅/深要**同构凹槽**：浅色「灰底托白卡」，深色「微抬灰底托 `#202224` 卡」。禁止「浅色凹槽、深色坠黑洞」。

YoUI 映射（值仍全部来自 `Harmony` primitive）：

```
浅：BgBase = background_secondary #F1F3F5
    Surface = comp_background_primary #FFFFFF
    Surface2 = background_tertiary #E5E5EA

深：BgBase = background_secondary #191A1C   ← 原 background_primary #000
    Surface = comp_background_primary #202224
    Surface2 = background_fourth #2E3033
```

层级仍满足 page < surface < surface-2。`#191A1C`→`#202224` 步进与 Radix 2→3 同量级，分层继续靠 hairline + 明度，不靠黑上叠黑影。

启动画布 `window_boot::CANVAS_DARK` 必须与 `--yohu-bg-base` 同值，否则 GDI 小窗 / Shared fill 会闪一帧纯黑。

## 与当前工作的关系

- **能直接用：** 只改 Semantic 映射与壳画布常量；不新增 token 名、不第三主题。
- **必须改写：** `DarkColors.BgBase`、`theme.css`、`CANVAS_DARK`、契约测试、设计系统表。
- **明确不要：** Ant/M3 算法造阶、Primer/Radix hex、Fluent Mica 进 WebView、`dark-dimmed`、把 `#191A1C` 再映射成 surface-2。

## 来源与阅读范围

- 产品内：`docs/architecture/harmonyos-design-notes.md` §1.4 / §1.6；`tokens/colors.ts`
- 开源：见三篇单仓深研的阅读范围
- Fluent 2 alias 表（Web，本轮 fluentui 工作树仍是 motion sparse-checkout，未读 tokens 源）
- Apple `windowBackgroundColor` / Desktop Tinting 文档（Web）
- 未读：VS Code `dark_plus` json（clone 不完整）、Carbon / Spectrum 色板
