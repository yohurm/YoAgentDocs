---
id: research.apple-liquid-glass
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Swift, Objective-C, Java]
  frameworks: [uikit, swiftui, appkit, android]
also_relevant: []
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: other
  url: https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass
  repos: []
studied_at: 2026-09-04
related:
  - research.apple-continuous-corners
  - research.harmony-immersive-light-layers
  - research.QWEA0-Liquid-Glass-Android
  - research.Abdullajon1881-LiquidGlass
  - research.BarredEwe-LiquidGlass
---

# iOS 26 Liquid Glass：光学层、容器与契约

主题笔记。Apple 不公开液态玻璃片元；机制以 WWDC 2025 Session 219 / 284、HIG《Materials》、Technology Overviews 与 UIKit/SwiftUI 公开 API 为准。未反编译 UIKit。

## 背景

Liquid Glass 是 iOS 26 / iPadOS 26 / macOS Tahoe 26 起的**数字超材料**（WWDC 219）：实时弯曲、塑形、会聚光线，同时像轻质液体一样随触控与界面状态变形。它不是 iOS 7 `UIBlurEffect` 的换皮，也不是一层半透明叠色。

YoUI 已有鸿蒙对齐的霜面 / SDF 透镜（`Backdrop*` + `LiquidGlassOptics`）。本笔记回答 iOS 多出来的、必须内化进自研背景模块的契约，而不是再抄一套控件名。

## 关键结论

### 1. 它是导航层，不是内容层

WWDC 219 / Adopting Liquid Glass：

- 玻璃构成**浮在内容之上的功能层**（控件、导航）。内容层保持实心。
- **禁止玻璃叠玻璃。** 玻璃上的标签/图标用填充、透明、vibrancy，不要再套一层材料。
- 标准栏、Sheet、Popover、控件用最新 SDK 自动获得材料；自定义背景会盖住玻璃和 scroll edge。
- 自定义玻璃要**省着放**，只给最重要的可交互元素。

Yo 映射：BAR / CONTROL / MENU / DIALOG / SHEET 可以是玻璃。CARD 继续默认关。列表项、整页根布局、视频上方禁止开材质（已有非目标，与 iOS 一致）。

### 2. 光学是多层系统，不是模糊半径

WWDC 219 把材料拆成同时工作的层（不是互相替换）：

| 层 | 行为 |
|----|------|
| Lensing | 主定义手段。动态弯曲/会聚光线，让控件轻、仍与背景分离。旧材料是散射；新材料是塑形。 |
| Frost / scatter | 仍存在，但随尺寸变：大块更「厚」（更深阴影、更明显折射、更软散射）。 |
| Highlights | 环境光打在几何上；部分场景跟设备运动。交互时从触点向内点亮，并漫到邻近玻璃。 |
| Shadows | 不透明度跟背后内容走：压在文字上加深，压在浅实底上变浅。 |
| Adaptive tint / dynamic range | 每层持续按背后内容变。小件（导航/Tab）可 light↔dark 翻转；大件（菜单/侧栏）只调 tint，不整板翻转。 |
| Vibrancy | `UIVisualEffectView.contentView` 上的标签按 `textColor` 自动变活力。 |

进出场：**用透镜强度渐变来物化/消物化**，不要 fade 掉光学完整性（219）。UIKit 284：设 `effect` / `effect = nil`，不要用 alpha 冒充。

### 3. Regular / Clear 永不同簇

公开枚举：`UIGlassEffect.Style.regular`（「Standard glass」）与 `.clear`（「Clear glass」）。iOS 26.0 引入。

| 变体 | 契约 |
|------|------|
| Regular | 默认可自适应。任意尺寸、任意内容、其上可放任何东西。 |
| Clear | **不做**自适应翻转。永久更透。必须有压暗层才能保证符号可读。仅当同时满足：压在富媒体上；内容层能接受压暗；玻璃上的内容粗且亮。 |

**禁止混用。** 着色是「按底下亮度映射的一组色调」，模仿真彩色玻璃；禁止用不透明实心底冒充 tint。

### 4. 容器：共享采样 + 融合 + 统一适应

玻璃**不能采样另一块玻璃**。多块必须进同一容器：

- SwiftUI：`GlassEffectContainer(spacing:)` + `.glassEffect(_:in:)` + `.glassEffectID(_:in:)`
- UIKit：`UIGlassContainerEffect`（`spacing` = 开始融合的距离）包住若干 `UIVisualEffectView(effect: UIGlassEffect)`，子玻璃加在容器的 `contentView`

容器职责（284 + 官方 overview）：

1. **一次采样**，性能。
2. 形状靠近时像水滴 **merge**；重叠动画合成一块。
3. **统一适应**：各块仍跟背景变，但外观一致。
4. 拆开：先无动画叠到同一位置，再一起动画分开。

SwiftUI 默认 morph 过渡是 `matchedGeometry`。`identity` 玻璃变体（社区 API 目录）只作占位，不画材料。

### 5. 形状、同心、尺寸

- 默认形状胶囊。UIKit 用 `cornerConfiguration`；靠近容器角时半径自动变，以保持 **concentric**（已见 [apple--continuous-corners.md](./apple--continuous-corners.md)）。
- **小块更清、可翻转；大块更不透、不翻转。**
- 硬件圆角决定控件曲率。玻璃 / SDF 锁定 `CIRCULAR`。

### 6. Interactive 与系统控件

- `UIGlassEffect.isInteractive`：点按缩放/回弹。iOS 有；其它平台接受参数无行为。
- 标准按钮用 `.glass()` / `.prominentGlass()` / `.clearGlass()`，不要为每个按钮自建 `UIGlassEffect`。
- 滑块/开关的拇指在交互时才变成玻璃。

### 7. Scroll edge 是邻层，不是玻璃本体

滚动内容从玻璃/栏底下过去时，系统把内容溶进背景，把玻璃抬起来。深色内容触发玻璃转 dark 时，edge 改成压暗。密集浮层可选用 hard edge（接近 iOS 18 实心底）。自定义浮在滚动边上的控件用 `UIScrollEdgeElementContainerInteraction`。

### 8. 无障碍是材料修饰符，不是另一套控件

Reduce Transparency → 更霜、挡住更多背后。Increase Contrast → 偏黑/白 + 对比描边。Reduce Motion → 减弱部分效果、关掉弹性。标准控件自动跟系统；自定义玻璃必须自己接同一信号。

### 9. UIKit 接线（自定义一块）

```text
UIVisualEffectView
  effect = UIGlassEffect(style:)     // regular | clear
  isInteractive, tintColor
  cornerConfiguration
  contentView ← 标签/图标（vibrancy）
  进出场：动画 block 里设 effect / nil
多块：
  外层 UIVisualEffectView(effect: UIGlassContainerEffect)
    contentView ← 若干 内层 UIVisualEffectView(UIGlassEffect)
```

`UIGlassEffect` 继承 `UIVisualEffect`。它与 `UIBlurEffect` 用途不同（284：「distinct from other visual effects, like UIBlurEffect」）。

## 与当前工作的关系

| 点 | 方式 | 说明 |
|----|------|------|
| 导航层 / 禁玻璃叠玻璃 | reuse-pattern | 写入背景模块策略；容器共享采样 |
| Regular / Clear | adapt | Yo 变体枚举，不叫 `UIGlassEffect.Style` |
| 尺寸→厚度 / 小件翻转 | adapt | L3 策略；大 MENU/DIALOG 不整板 light↔dark |
| 物化 = 调透镜 | adapt | 禁止 alpha 冒充材料进出场 |
| 阴影跟背后内容 | adapt | 已有 luminance probe，接到 L4 阴影 alpha |
| 容器 spacing + merge | adapt | 新 L1 并集；实现见 Android 开源 smin，不是 UIKit 私有渲染器 |
| Scroll edge | adapt | **独立 L1**，不是玻璃 overlay |
| 同心圆角 | reuse-pattern | 已有 CONTINUOUS/CIRCULAR；玻璃锁 CIRCULAR |
| RGB 色散 / 重力高光当默认 | anti-pattern | 官方讲 lensing 与环境高光，不规定光谱边纹；重力是「部分场景」 |
| `layer.render` 截图当 L0 | anti-pattern | 官方是系统材料；社区截图层会打破排除自身与 live display list |
| 给内容表/卡片墙铺玻璃 | anti-pattern | 219 表格示例：保持内容层 |
| 用不透明 fill 当 tint | anti-pattern | 284 明确破坏材料 |

## 来源与阅读范围

读过：

- WWDC 2025 219 *Meet Liquid Glass* 全文稿（光学、层、变体、无障碍）
- WWDC 2025 284 *Build a UIKit app with the new design* 自定义玻璃 / Container / scroll edge / 物化段落
- `developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass`
- `developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views`（JS 页摘要 + 检索快照）
- `developer.apple.com/documentation/swiftui/glasseffectcontainer`
- UIKit JSON：`UIGlassEffect`、`UIGlassEffect.Style`（regular / clear）、`UIGlassContainerEffect.spacing`（iOS 26.0）

未读 / 不装懂：UIKit 私有 backdrop 层、Metal 片元、HIG Materials 全文（该页 JS 无法抓取正文，结论以 219 为准）。SwiftUI `Glass` `.identity` 仅见社区 API 目录，不以官方 JSON 引用。
