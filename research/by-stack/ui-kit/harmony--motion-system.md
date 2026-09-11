---
id: research.harmony-motion-system
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [ArkTS, C++]
  frameworks: [harmonyos, arkui, rosen]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://developer.huawei.com/consumer/cn/doc/design-guides/design-animation-overview-0000001750985338
  repos:
    - openharmony/window_window_manager
    - openharmony/arkui_ace_engine
  cloned_to:
    - "%TEMP%/YoAgentResearch/openharmony--window_window_manager"
studied_at: 2026-09-08
related:
  - research.apple-motion-system
  - research.openharmony-window_window_manager
  - research.harmony-hdstabs-motion
---

# HarmonyOS 动效体系（主题笔记）

## 背景

Yohu 启动交接把 HWND 矩形当动画对象，每帧 `SetWindowPos`，图标抖动。要扔掉那套，先读鸿蒙**系统级**动效：本地设计指南 `D:\A_yoprogram\Learn\yovo-harmonyos-docs\设计\设计指南\通用设计基础\动效\`（概述 / 设计原则 / 动效属性 / 转场动效 / 手势动效），开发侧 `@ohos.curves` / `animateTo` / 窗口 `RSTransitionEffect`。

官方路径不是「改布局矩形」，而是：

```
原则（引力 / 过场义务）
  → 属性（时长 · 曲线 · 帧率）
    → 元素分类（进场 / 出场 / 持续 / 静止）
      → 编排（共享元素 · 共享容器 · 共享动势 · 淡入淡出）
        → 引擎（Rosen 表面 Scale/Translate/Opacity，不是窗口布局补间）
```

## 关键结论

### 分层

| 层 | 官方说什么 | 不是什么 |
|----|------------|----------|
| L0 | 自然流畅、过场义务、60fps 交互、可打断弹簧、隐藏等待 | 为动而动 |
| L1 | 时长随行程：100 / 150 / 200 / 300 / 350ms；曲线分标准 / 减速 / 加速 / 弹簧 | 16 档 Material duration |
| L2 | 属性动画：`animateTo` / 隐式 `animation` / `Animator` 补间 **transform 与 opacity** | 每帧改组件布局宽高冒充运动 |
| L3 | 转场配方：共享容器、geometryTransition、pageTransition、窗口 `RSTransitionEffect` | 跨屏共用一个几何补间 |
| L4 | 控件 / 窗口只接线配方 | 控件里自写第二套毫秒 |

### 曲线与元素类型（设计《动效属性》《转场动效》）

- **标准** `cubic-bezier(0.40, 0.00, 0.20, 1.00)`：运动前后都在视线里（图片缩放、Tab、开关）。
- **减速** `cubic-bezier(0.00, 0.00, 0.40, 1.00)`：新出现（弹框）。
- **加速**：视线里的物体出场（窗口出场、卡片删除）。
- **interpolatingSpring** 设计默认 Stiffness 128 / Damping 12 / Mass 1 / Velocity 0；`springMotion` 等价 Response 0.555 / DampingFraction 0.53。弹簧时长由物理参数算，**指定 duration 不生效**。跟手、可打断、继承速度。微振荡只给小面积。
- 元素四类决定曲线：进场减速、出场加速、持续标准、静止不动。
- **电脑 / 大屏层级转场 = 淡入淡出（弹簧）**，不要套手机左右位移。

### 一镜到底

- **共享元素**：转场前后都在的焦点（如搜索框）。
- **共享容器**：有明确边界的一组；容器补间 **位置 / 尺寸 / 圆角**，容器内用淡入淡出或共享元素。内容属性可以在结束时跳到目标值（OpenHarmony `SharedTransitionEffectType.Exchange` 文档写明 fontSize 等不跟）。
- **共享动势**：没有中间布局属性时，抽位移 / 缩放 / 旋转。

### 启动页（窗口子系统，见 window_window_manager）

启动页是 **leash 上的独立 starting surface**，不是把应用窗口从 480×300 拉到全屏：

1. 主窗口布局已经是最终矩形。
2. starting surface 画静态图。
3. 应用首帧就绪 → `SetAlpha(start→end)`（默认 1→0，时长配置默认 200ms）。
4. finish 回调里 `RemoveChild(startingWinSurfaceNode_)`。

窗口进出场默认 `RSTransitionEffect`：`Scale(0.7)` + `Opacity(0)`，作用在 **surfaceNode / leashWinSurfaceNode**，不是 `SetWindowRect` 插值。

### 开发 API 分层（ArkUI）

1. 属性：`animateTo` / 隐式 animation（80% 场景）。
2. 出现/消失：`transition(TransitionEffect.*)`。
3. 共享几何：`geometryTransition` + `animateTo`（同场景）；跨 Router 用 `sharedTransition`。
4. 逐帧：`Animator` / 画布。优先 transform，避免布局驱动。

窗口动画曲线 API 20+：`INTERPOLATION_SPRING` 参数 `[velocity, mass, stiffness, damping]`。

## 与当前工作的关系

| 用 | 改写 | 不要 |
|----|------|------|
| L0–L1 已在 `@yohu/ui` `motion.ts`（时长/标准/减速/加速） | 原生启动交接改成 **表面 Scale+Opacity**，主窗一次落到最终矩形 | 每帧 `SetWindowPos` 改 HWND 宽高 |
| 同屏：共享容器（小窗快照矩形 → 主窗外框） | 异屏：电脑层级 = 出场加速淡出 + 进场减速淡入，**禁止跨屏共享几何** | 把主窗从 0.92 倍 HWND 尺寸插值到全尺寸（WebView2 会拉伸上一帧） |
| 过场义务、`SPI`/系统关动画则时长 0 | 启动交接等 hydrate 后播（隐藏等待） | 第二套毫秒表 |
| 投屏占用已走 `IDCompositionAnimation` | 启动交接与占用共用「合成器属性」层 | GDI 每帧 StretchBlt 活 Logo 跟窗口抢尺寸 |

## 来源与阅读范围

- 本地：`动效/概述.md`、`设计原则.md`、`动效属性.md`、`转场动效.md`、`手势动效.md`。
- 在线：华为《弹簧曲线》《@ohos.curves》《@ohos.animator》《WindowAnimationCurveParam》。
- 源码：`%TEMP%/YoAgentResearch/openharmony--window_window_manager` 的 `animation_config.h`、`window_controller.cpp` `UpdateWindowAnimation`、`starting_window.cpp` `SetStartingWindowAnimation`。
- 未整仓再克隆 `arkui_ace_engine`（体积大）；共享元素行为以 OpenHarmony docs `ts-transition-animation-shared-elements.md` 与既有 ace_engine 主题笔记为准。
