---
id: research.harmony-apple-motion-spec-unification
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, Rust, Swift, C++]
  frameworks: [harmonyos, arkui, swiftui, uikit]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: other
  url: https://developer.huawei.com/consumer/cn/doc/design-guides/animation-attributes-0000001797117229
  repos:
    - openharmony/window_window_manager
    - nathangitter/fluid-interfaces
  cloned_to:
    - "%TEMP%/YoAgentResearch/openharmony--window_window_manager"
    - "%TEMP%/YoAgentResearch/nathangitter--fluid-interfaces"
studied_at: 2026-09-09
related:
  - research.harmony-motion-system
  - research.apple-motion-system
  - research.nathangitter-fluid-interfaces
  - research.openharmony-window_window_manager
---

# 动效规格统一（HarmonyOS 分级 × Apple 弹簧 × Yo MotionSpec）

## 背景

Yohu 控件层已有 `MotionDuration` / `MotionEasing` / `MotionSpec`（`ui/packages/ui/src/tokens/motion.ts`）。原生气（`core/yohu-motion`）原先只公开三个毫秒常量和两条贝塞尔（`ease_decel` 还锁在单测里）。配方层 `PRESENCE_EXIT_DURATION` / `SWAP_DURATION` 再写一遍 `"local"` / `"slow"`，等于三套入口。本笔记对齐官方**属性层**（时长·曲线），给双端同一套规格名。

不重写启动窗口路径（见 [HarmonyOS 动效体系](harmony--motion-system.md) / [Apple 动效体系](apple--motion-system.md)）。

## 关键结论

### 时长只跟范围与复杂度，不跟平台 API 名

本地鸿蒙《动效属性》（`D:\A_yoprogram\Learn\yovo-harmonyos-docs\设计\设计指南\通用设计基础\动效\动效属性.md`，源 <https://developer.huawei.com/consumer/cn/doc/design-guides/animation-attributes-0000001797117229>）：

| 场景 | 官方档 | Yo 名 |
|------|--------|--------|
| 颜色/透明度 | 100ms | `fast` / `effectsFast` |
| 开关图标 | 150ms | `small` / `spatialSmall` |
| 面板展开 | 160ms | `normal` / `effectsEnter` |
| 删一行 / 局部 | 200ms | `local` / `effectsExit` `spatialLocal` `spatialExit` |
| 旋转等复杂 | 300ms | `slow` / `spatialPanel` |
| 全屏打开图 | 350ms | `enter` / `spatialEnter` |

窗口层默认**不是** 350ms：OpenHarmony `AnimationConfig::WindowAnimationConfig` 默认 `timingProtocol_ = 200`、`EASE_OUT`、scale `{0.7, 0.7, 1}`（`wmserver/include/animation_config.h`）。键盘进出场 500ms + cubic `(0.2, 0, 0.2, 1)`。Yohu 启动交接用 `spatialPanel` 300ms 铺满、`spatialExit` 200ms 异屏出场、`effectsFast` 淡出 overlay，与控件侧栏同一规格，不跟窗口默认 200ms 混成第四套。

### 曲线按元素四类，不是按「好不好看」

《转场动效》：进场减速、出场加速、持续标准、静止零运动。官方贝塞尔：

- 标准 `cubic-bezier(0.40, 0.00, 0.20, 1.00)` — 始终在视线内
- 减速 `cubic-bezier(0.00, 0.00, 0.40, 1.00)` — 新出现
- 加速 `cubic-bezier(0.40, 0.00, 1.00, 1.00)` — 消失
- 强调减速（官方「其他类型」`cubic-bezier(0.00, 0.00, 0.00, 1.00)` 的邻档）Yo 用 Material 常见 `cubic-bezier(0.2, 0, 0, 1)` 做折叠高度

ArkUI `@ohos.curves`（OpenHarmony `js-apis-curve.md`）：`interpolatingSpring(velocity, mass, stiffness, damping)` **忽略** `animateTo` 的 duration。设计指南页面级弹簧默认 **Stiffness 128 / Damping 12 / Mass 1 / Velocity 0**，等价 `springMotion` Response **0.555** / DampingFraction **0.53**。Yo `MotionSpring` 已锁这组；CSS 采成 `linear()`。原生 DComp **不**再积一套弹簧，弹簧槽位贝塞尔回退 `ease_standard`。

### Apple 侧：弹簧参数是 response/damping，不是毫秒表

`nathangitter/fluid-interfaces` `Spring.swift`：`UISpringTimingParameters(damping:response:)`，`UIViewPropertyAnimator(duration: 0, timingParameters:)`。时长由物理 settle，duration 传 0。HIG / SwiftUI：`@Environment(\.accessibilityReduceMotion)` **不会**自动关掉 `.animation(.default)`；大位移换成淡入淡出。Yohu 已有 `prefers-reduced-motion` 与 `SPI_GETCLIENTAREAANIMATION`。

换算（与鸿蒙 interpolatingSpring 互通）：`ω₀ = 2π / response`，`stiffness = ω₀² × mass`，`damping = 2ζω₀ × mass`。Yo 页面弹簧 128/12/1 → response≈0.555、ζ≈0.53，与指南一致。指示器 snap（711/40）是更硬的跟手档，不是第二套「Apple 专用表」。

### API 收敛：一层规格，两套消费

```
MotionSpec 名（effectsFast / spatialPanel / …）
  ├─ TS：duration + easing 名 → CSS var + motionSpecMs()
  ├─ 配方：PRESENCE_EXIT / SWAP / INDICATOR 只引用 MotionSpec.*.duration
  └─ Rust：MotionSpec::duration_ms() + ease() → DComp / wait
```

模块只点配方名（`list` / `panel`）。壳原生只点 `MotionSpec::SpatialPanel`，禁止再写 `300` + `ease_standard` 配对。

反模式：配方层再维护平行毫秒表；`yohu-motion` 只导出三个常量；把窗口 `AnimationConfig` 200ms 当成控件 token。

## 与当前工作的关系

- **直接用：** 鸿蒙 100/150/160/200/300/350 表；标准/减速/加速；128/12 弹簧；Reduce Motion 关空间位移。
- **改写：** Rust `MotionSpec` 枚举与 TS 同名；`ease_decel` / `ease_emphasized` 公开；splash / occupancy 走规格名。
- **不用：** 再克隆 `arkui_ace_engine`（窗口/Tabs 已有笔记）；把 SwiftUI `Animation.spring(duration:bounce:)` 当 CSS token；窗口默认 200ms EASE_OUT 覆盖 `spatialPanel`。

## 来源与阅读范围

- 本地：`动效属性.md`、`转场动效.md`（2026-08-10 镜像）。
- 在线：Huawei 动效属性页；OpenHarmony `js-apis-curve.md` / `ts-explicit-animation.md`（Curve 枚举含 `fast-out-slow-in` = 标准 0.4,0,0.2,1）。
- Clone：`animation_config.h`（窗口 200ms / scale 0.7）；`Spring.swift`（response/damping 滑块 + duration 0 animator）。
- 未读：HDS 闭源弹簧实现、SwiftUI 源码、Apple HIG Motion 全文 PDF（结论来自开发者文档 `accessibilityReduceMotion` 与既有 [apple--motion-system](apple--motion-system.md)）。
