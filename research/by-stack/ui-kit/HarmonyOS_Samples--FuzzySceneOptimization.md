---
id: research.harmonyos-samples-fuzzy-scene-optimization
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [arkts]
  frameworks: [arkui, harmonyos]
also_relevant: []
utilization: [reuse-pattern, anti-pattern]
source:
  platform: gitcode
  repo: HarmonyOS_Samples/FuzzySceneOptimization
  url: https://gitcode.com/HarmonyOS_Samples/FuzzySceneOptimization
  cloned_to: "%TEMP%/YoAgentResearch/HarmonyOS_Samples--FuzzySceneOptimization"
studied_at: 2026-08-20
related:
  - research.openharmony-arkui_ace_engine-dialog-spatial
---

# HarmonyOS_Samples/FuzzySceneOptimization

## 入选理由

YoDialog 三区每次 `create()` 都在弹出弹簧前新建霜面；官方把「动态模糊 vs 静态模糊」写成了可跑样本。要回答「为什么预持有的模糊对话框不卡、现场 create 的卡」，必须读这份样本和配套指南，而不是猜 GPU。

对照：本机鸿蒙文档《对比动态模糊与静态模糊》（`ui-dynamic-vs-static-blur-examples`）。

## 项目是什么

官方性能样本：全屏模态转场里分别用 `Image.blur`（每帧动态模糊）和 `effectKit.createEffect().blur()`（一次性静态 PixelMap）。

## 架构

```text
点击
  动态：bindContentCover 打开，Image 每帧 .blur(13)
  静态：先 staticBlur() 写出 PixelMap，再 bindContentCover 只显示 Image(pixelMap)
```

指南用 Frame Profiler：同一转场，动态平均 RenderFrame 6.113ms / 108fps，静态 3.357ms / 119.9fps。结论是模糊内容不随帧变时必须一次性处理，禁止在动画帧上实时模糊。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 转场前准备静态模糊板 | reuse-pattern | 预持有 `YoDialog.build()` 的模糊对话框对齐这条 |
| 动画帧上 `.blur()` / live backdrop | anti-pattern | 三区每次 create + 等第一帧霜面打完再 `present()` 就是这条 |

## 架构设计经验

静态模糊是**先有板，再开弹簧**。系统 Dialog 的 `systemMaterial` 在引擎里已经是现成滤镜；应用侧自己做玻璃时，编译 RuntimeShader / CPU stamp rim 必须离开弹出时钟。

## 与当前工作

**能直接用：** 预持有定义；进程内复用 rim 场；layout 之后立刻开 EXPAND，不要等霜面第一帧。

**不要用：** 在 `OnDrawListener` 里堵住 `present()` 直到 L0–L4 建完——那会把「建板」变成用户可感知延迟。

## 阅读范围

- `entry/src/main/ets/pages/StaticBlur.ets`、`MotionBlur.ets`
- README 中文/英文
- 官方指南 `对比动态模糊与静态模糊.md`
