---
id: research.synthesis.ui-kit
type: synthesis
status: active
when: research
stack:
  capability: ui-kit
---

# ui-kit 横向总结

## 本层已研项目

| 仓库 / 主题 | 一句话 | 利用方式 |
|-------------|--------|----------|
| [openharmony/arkui_ace_engine（沉浸光感）](openharmony--arkui_ace_engine-immersive.md) | ImmersiveMaterial → 滤镜/着色器 → Rosen 节点 | adapt 档位短路与 LUT 接线 |
| [arkui_ace_engine Dialog 空间弹出](openharmony--arkui_ace_engine-dialog-spatial.md) | API 26 Dialog：0.2 缩放+半高平移从中下散开；流光底→顶；关闭不回放 | adapt 打开映射；anti-pattern：360° sweep、关闭走旧 scale |
| [HarmonyOS Dialog 弹出扫光](harmony--dialog-appear-flow-light.md) | 白光线贴 SDF 边扫；Dialog 外晕 0；250 是 bloom 不是 stroke | adapt 细帽+clip；anti-pattern：28dp 填色 |
| [HarmonyOS 沉浸光感分层](harmony--immersive-light-layers.md) | 滤镜 / overlay / 组件自叠的层所有权 | reuse-pattern：层不许越级 |
| [openharmony/graphic_graphics_effect](openharmony--graphic_graphics_effect.md) | SDF 边缘光与 FrameGradientMask 的 RuntimeShader | adapt 公式；anti-pattern：闭合描边 |
| [HarmonyOS SDF 边缘光](harmony--sdf-edge-light.md) | 「沿边流转」= SDF 带宽×扇形；overlay 另用 SDF×mask | 白底 occlusion，禁止加粗 stroke |
| [openharmony/graphic_graphic_2d 点光源](openharmony--graphic_graphic_2d-point-light.md) | 半径 `≈1.731×z`、灯/受光配对、IlluminatedType | reuse-pattern：自照明 + 出界灯 |
| [HdsTabs 点光源](harmony--hdstabs-point-light.md) | `lightColor` → Rosen 点光源色；HDS 闭源 | anti-pattern：当 rim / SDF overlay |
| [HdsTabs 动效](harmony--hdstabs-motion.md) | 切页默认 0；bounce DOWN；栏显隐 228/30 | reuse-pattern：通道分权；anti-pattern：切页滑页 |
| [HdsTabs 项切换](harmony--hdstabs-item-switch.md) | 仅选中 id 变化弹 DOWN；色瞬时；bind 须先 cancel | reuse-pattern：bounce 叠出血；anti-pattern：整项 scale |
| [bindSheet 全量](harmony--bindsheet-full.md) | SIDE 无档无条；同面板返回 ≠ 再弹一层 | reuse-pattern：关闭关整棵；anti-pattern：FAQ 800ms 自写滑页 |
| [arkui_ace_engine Tabs 动效](openharmony--arkui_ace_engine-tabs-motion.md) | TabBarPattern 弹簧 / Symbol 触发 / duration=0 | reuse-pattern：listItemSwipeSpring |
| [Spatialization HdsTabs 隐藏](HarmonyOS_Samples--Spatialization.md) | 滚动方向调 applyHide/Show，曲线在 HDS | lesson-only：应用不自写藏栏 |
| [microsoft/fluentui react-motion](microsoft--fluentui.md) | atom → Presence 工厂 → Fade；enter/exit 默认同时长 | reuse-pattern：换牌原语与按钮分层 |
| [radix-ui/primitives Presence](radix-ui--primitives.md) | 布尔进出场状态机，不管视觉 | reuse-pattern：Presence ≠ Swap |
| [HarmonyOS 圆角参数](harmony--corner-radius.md) | 4/8/16/20/32vp 层级；圆弧 RRect；clip 与半径分权 | reuse-pattern：同几何 fill+clip；Rosen 比例缩放 |
| [Rosen RoundRect](openharmony--graphic_graphic_2d-roundrect.md) | 四角 XY 半径 + ScaleRadii | reuse-pattern：邻角抢边按比例缩 |
| [iOS 连续圆角](apple--continuous-corners.md) | G2 continuous；concentric = parent − padding | adapt：CIRCULAR 默认，CONTINUOUS opt-in |
| [phamfoo/figma-squircle](phamfoo--figma-squircle.md) | 每角两 cubic + 一弧；邻角分账 | adapt 公式，不进 npm |
| [racra smooth-corner](racra--smooth-corner-rect-android-compose.md) | s=0 走 RRect；胶囊回退 circular | reuse-pattern：曲线是策略不是 token |
| [ag-grid 表头](ag-grid--ag-grid.md) | 轨道 / 标题 wrapper / resize 三节点 | reuse-pattern：分割线不进排序钮 |
| [Spectrum Table](adobe--react-spectrum.md) | resizer::after 画列界；排序图标未激活不占位 | reuse-pattern：文本区裁剪 |
| [VS Code 表格 sash](microsoft--vscode-table.md) | SplitView 做列轨道，标题只是 sash 格子里的文案 | reuse-pattern：区域划分 ≠ 文本 |
| [HarmonyOS 静态/动态模糊样本](HarmonyOS_Samples--FuzzySceneOptimization.md) | 转场前一次性 createEffect；动画帧上 blur 会掉帧 | reuse-pattern：先有板再开弹簧 |

## 共同架构经验

- 高/中算力清掉 `borderWidth`，高光在 `materialFilter`。低算力才用 1vp 边框。
- 边缘高光是 **SDF 距离场上的光照**，不是 canvas stroke。法线衰减与切向 mask 必须拆开。
- 白底可见性来自接触压暗，不是更白或更粗的描边。官方 overlay 合成是加色。
- Bloom 内侧远大于外侧（ArkUI overlay 0.8 / 0.2）；高 falloff 幂让晕贴边。
- 组件（Tabs/Sheet）只叠官方允许溢出的那一层，不要在应用侧再画发丝线圈。
- Tabs 指尖光是 **同一节点上的 Rosen 点光源**（BORDER_CONTENT，z=80vp），不是 Sheet/Menu 的 SDF `edgeLight` overlay。`lightColor` 只进 L6。
- **HdsTabs 动效五通道分权：** 切页时长、图标 bounce、栏显隐、材质按压/光、MiniBar。底部页签切页默认 **0ms**；选中反馈是 Symbol `BounceSymbolEffect` DOWN，不是整栏 scale。栏显隐的对象是 **TabBar 节点 `setTabBarTranslate`**（ArkUI `UpdateTransformTranslate`），和 interactive / 点光源不是一条链。`setTabBarOpacity` 是独立 API，不能用淡出玻璃冒充整栏离场。

### Dialog 空间弹出（2026-08-20 增补）

- **打开语言**是「中心 scale 0.2→1 + translateY = 0.5×height→0」，视觉原点在最终矩形中下，不是 Sheet 居中的 0.86。
- **扫光**是 EdgeLight 底边→顶边的**细边缘帽**（Dialog 外晕 0、细边 ~10px），不是绕 clip 中心转一圈，也不是铺满卡片的粗扫描带。合成是 OverlayNG **加色**（`image.rgb + light.rgb`），不是 SRC_OVER 半透明描边；L7 若拆到 sibling View，必须把 ADD 一起带走，否则白玻璃上看不见。
- **打开时钟**挂在 AfterLayout，不在「第一帧霜面编译完」。
- **关闭**官方是 opacity→0 + FORWARDS，最后一帧透明后才卸节点；不要把不透明 0.2 缩放停在屏幕上。
- 弹簧 stiffness 322 / damping 27 可换成 perceptual response；关闭官方不回放形变，YoUI 若回放打开语言，收回末帧仍须收到 alpha 0。
- **关闭**在引擎里仍是旧缩放淡出。要对齐打开语言，visibility 必须 1→0 走同一映射。
- 形变滤镜（四角塌陷/桶形）仅 HIGH 且闭源；Android 用 scale+translate 表达散开，不抄 DistortionParam。

### 动效分层（2026-08-20 增补）

- **atom / 配方 / 控件**必须拆开。Fade 只做 opacity；谁还在树上（PresenceGroup / Swap）是另一层；Button 只引用配方。
- 同类元素交叉淡入淡出：**同时、同时长**（鸿蒙《转场动效》；Fluent Fade `exitDuration = duration`）。不要入场 160、出场 200 错开，更不要对工具栏按钮补间 width。
- 布尔 `Presence` 只覆盖 Dialog/菜单/模块。内容换牌用 **keyed Swap**（Compose `AnimatedContent` 默认 `TopStart` 对齐、SizeTransform 可关）。
- 简单透明度（文案替换）按鸿蒙「只靠颜色/透明度」走 **100ms**，不是面板级 200–350ms。
- 控件文件禁止内嵌测量 `getBoundingClientRect` 的宽度状态机。

### 圆角（2026-08-20 增补）

- **半径 token** 跟鸿蒙层级（同层统一、浮层更大）。**曲线**是第二轴：CIRCULAR = Rosen/Skia 圆弧；CONTINUOUS = UIKit G2（Figma s≈0.6）。
- fill / clip / outline / ripple **必须同一条 Path**。`borderRadius` 不管 clip；内容区另裁（鸿蒙 `clip(true)` ≡ `content-region.md`）。
- 邻角半径之和超过边长：circular 用 Rosen 比例缩放；continuous 用 Figma 邻角分账。不要只 `min(r, half)`。
- 玻璃 / SDF 锁定 CIRCULAR。连续路径与 `sdRoundBox` 混用会漏光。
- 胶囊（半径 ≥ 短边一半）即使请求 CONTINUOUS 也回退 CIRCULAR。
- 叠层 AA 黑边：同色或同心缩小，不加粗 stroke。

### 表头轨道与文本（2026-08-20 增补）

- **列轨道**拥有宽度、分割线、拖拽条。**标题文本**（含排序图标）是轨道里的内容区，可以 hug 文案。禁止用铺满轨道的排序按钮去冒充列宽。
- 分割线是铬（Spectrum `columnResizer::after`、VS Code sash、AG Grid `ag-header-cell-resize` 旁的列界）。overflow 只裁内容区，不裁轨道，否则负偏移拖拽条被剪掉。
- 排序图标未激活不占位（Spectrum `display:none` 直到 `is-sorted`；Fluent `sortIcon` 仅 `sortDirection` 有值才渲染）。
- Fluent TableHeaderCell 的 button `width:100%` **不要抄**：它的 hover 是单元格矩形底，Yohu 排序走圆角 `.yohu-interactive`，铺满会把高亮片当成列区域，吃掉下一条分割线。
- 鸿蒙 List：`header` 是 CustomBuilder（内容），`divider` 是 List 属性（铬，含 startMargin/endMargin）。同一条「内容 / 分割」分权。
- 不把 YoDataGrid 整表提前落地；先补 `YoColHeader` 轨道原语，模块只提供排序文案。

## 分歧与取舍

- GE 结构体默认（64 / 2.0 / 30/30）≠ ArkUI overlay LUT（47.8 / 8.7 / thickness×0.8/0.2）。产品对齐后者。
- 静止材质用 FrostedGlass `edLight` 扇形；Sheet/API overlay 用 `RSNGSDFEdgeLightEffect`。不要混成一套 SweepGradient。
- EXQUISITE 滤镜创建在闭源 `libhdsmaterialimpl.z.so`；开源 LUT 覆盖 GENTLE，公式以 graphics_effect 为准。

## 对本知识库规则的候选修订

只记录建议，不自动改 `instructions/rules/`。用户确认后才能升格。

- ui-kit 修改规则可补：HIGH/MID 液态玻璃禁止闭合 hairline；白底边缘用 occlusion+SDF 衰减，不用加粗 stroke。
- 实现配方引用本层主题笔记，而不是再从 GitHub README 推断。
- content-region 可补一句：表头轨道是铬，标题/排序是内容区；`.yohu-interactive` 不得铺满轨道来假装列界。
- 底部页签动效：切页默认 0；选中 bounce 是图标 translationY DOWN，禁止整栏 scale；栏显隐用 stiffness 228 / damping 30，不要 `LOCAL` 贝塞尔。
- 半模态：SIDE 无档位/控制条且高度全屏；同面板 push 只换内容，关闭按钮关掉整棵栈；层级返回不得覆盖用户 leading。

已升格（2026-08-20，用户确认）：圆角几何进 L1；L5 门面要薄、禁止 `api/` import `internal`；组件禁止私自 `addRoundRect` / `GradientDrawable.setCornerRadius`；玻璃 / SDF 锁定 CIRCULAR。见 `instructions/rules/by-type/ui-kit/` 的 public-api / layering / file-srp / coupling。

## 入选与落选备忘

必读仓已浅克隆到 `%TEMP%/YoAgentResearch/`。霜玻璃 HIGH 着色器仍闭源；**点光源**在 `graphic_graphic_2d` 开源，已单列。HDS / UIDesignKit 无公开实现仓。

圆角：规范读本地鸿蒙《圆角参数》+ Apple `CALayerCornerCurve`；实现读 Rosen RoundRect 与 figma-squircle。`stoyan-vuchev/squircle-shape` 单 cubic 落选。

表头（2026-08-20）：入选 AG Grid / Spectrum Table / VS Code monaco-table；Fluent TableHeaderCell 只作对照（单元格底 hover ≠ 圆角片）。落选 Carbon DataTable、Fluent Blazor DataGrid（与 Spectrum / Fluent React 结构重复）。

HdsTabs 动效（2026-08-20）：入选 ace_engine TabBarPattern + Spatialization 接线 + 主题笔记（HDS 无仓）。落选 HarmonyOS-Cases customanimationtab、子页签下划线 Demo（那是 SubTabBar，不是底部 HdsTabs）。iHongRen/harmony-study-demo 只调 API，无曲线，不单列。

Dialog 空间弹出（2026-08-20）：入选 `arkui_ace_engine` `PlayDistortion` / `PlayFlowLight`。社区 DialogHub / 自定义 `transition` 是另一套进场，不单列。`graphic_graphics_effect` EdgeLight 着色器只作对照，Dialog 喂的是 `EdgeLightParam`。

Dialog 弹出扫光形态（2026-08-21）：补读 `UpdateEdgeLightFilter` 的 Dialog 分支（inner 0.1 / outer **0**）和 FrameGradientMask `AxialCoreWidth=0.3`。HDS `DualEdgeFlowLight` 是周长光线，产品不同，不单列作 Dialog L7 实现仓。落选把 `thickness=250` 当 Canvas stroke 的社区扫光 Demo。

Dialog 扫光看不见（2026-08-21）：同一主题补读 `GESDFEdgeLight::MakeImageMerger` 与 OverlayNG。入选仍是 ace_engine + graphics_effect + graphic_2d，不另开仓。根因是 YoUI L7 sibling overlay 丢了加色，不是再调粗细。
