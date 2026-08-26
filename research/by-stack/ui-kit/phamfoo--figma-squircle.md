---
id: research.phamfoo-figma-squircle
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [svg]
also_relevant: []
utilization: [adapt]
source:
  platform: github
  repo: phamfoo/figma-squircle
  url: https://github.com/phamfoo/figma-squircle
  cloned_to: "%TEMP%/YoAgentResearch/phamfoo--figma-squircle"
studied_at: 2026-08-20
related:
  - research.apple-continuous-corners
---

# phamfoo/figma-squircle

## 入选理由

把 Figma 博文公式写成可测的路径参数：每个角输出 `a,b,c,d,p,arcSectionLength`。Android 没有 UIKit continuous，这是可移植的几何，而不是 Compose 封装。

## 项目是什么

TypeScript：输入宽高、四角半径、`cornerSmoothing`，输出 SVG path。

## 架构

1. 四角相同：预算 = `min(w,h)/2`，半径夹到预算，再算一套 params 复用四角。
2. 四角不同：`distributeAndNormalize` 大角先分邻边预算（按半径比例），再各自 `getPathParamsForCorner`。
3. 每角：`p=(1+s)R`；弧心角 `90(1-s)`；两段 cubic + 一段 `a` 圆弧。
4. `preserveSmoothing`：空间不够时 Figma 会降低 s；打开则保持 s、压缩控制点。Yo 跟 Figma 默认（降低 s），避免角互相侵入。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 角参数公式 | adapt | 内化进 Yo `ContinuousCorner`，不依赖 npm |
| 邻角分账 | adapt | 与 Rosen `ScaleRadii` 目标相同、算法不同；circular 用 Rosen，continuous 用分账 |
| 拷贝 SVG 字符串到 Android | anti-pattern | 用 `Path.cubicTo` / `arcTo` |

## 架构设计经验

平滑度是半径之外的第二参数。空间不足时必须先动平滑度或半径，不能让 `p` 超过半边。

## 与当前工作

能直接用：公式与绘制顺序。必须改写：Android `Path`、px、API 30 outline。不要用：把 TypeScript 当运行时。

## 阅读范围

`src/index.ts`、`src/draw.ts`、`src/distribute.ts`。
