---
id: research.nathangitter-fluid-interfaces
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Swift]
  frameworks: [uikit]
also_relevant: []
utilization: [adapt, lesson-only, anti-pattern]
source:
  platform: github
  repo: nathangitter/fluid-interfaces
  url: https://github.com/nathangitter/fluid-interfaces
  head: da5c66c
  cloned_to: "%TEMP%/YoAgentResearch/nathangitter--fluid-interfaces"
studied_at: 2026-09-08
related:
  - research.apple-motion-system
---

# nathangitter/fluid-interfaces

## 入选理由

WWDC 2018 Designing Fluid Interfaces 的可编译对照。弹簧参数怎么从设计师语言（response / damping）落到 `UISpringTimingParameters`；手势动画动 transform/center，不改约束循环。

## 项目是什么

2018 教学 App：计算器键、弹簧、手电、橡皮筋、加速度暂停、动量抽屉、FaceTime PiP、旋转。Apache-2.0。维护不活跃，但契约清晰。

## 架构

- 每个界面一个 `InterfaceViewController`。
- 弹簧：`UIViewPropertyAnimator(duration: 0, timingParameters: UISpringTimingParameters(...))`。duration 0 因为弹簧自带时长。
- `UISpringTimingParameters` 扩展：`stiffness = (2π/response)²`，`damp = 4π * damping / response`（mass=1）。与鸿蒙 springMotion 的 response / dampingFraction 同一套话。
- PiP：跟手改 `center`；松手算最近锚点 + 速度，弹簧飞过去。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| response/damping 换 stiffness | adapt | 与鸿蒙 128/12 互算；Yohu `motion.ts` 已有 response 0.555 |
| 动 transform/center | reuse-pattern | 启动交接只动快照变换 |
| 跟手弹簧 | lesson-only | 启动交接无手势，用贝塞尔标准/减速/加速即可 |
| 用 Auto Layout 常量做帧动画 | anti-pattern | 本仓也不这么做 |

## 架构设计经验

流体界面 = 可打断弹簧 + 速度继承 + 离散锚点。启动交接不是手势场景；借用的是「动画属性是变换不是布局」。

## 与当前工作

不要把 PiP bounce 抄进 splash。同屏用标准曲线共享容器；异屏出场加速。弹簧留给 YoUI 滑块（已有）。

## 阅读范围

`README.md`、`Spring.swift`（含 timing 扩展）、`Pip.swift` 前半。未逐行动量抽屉与旋转。
