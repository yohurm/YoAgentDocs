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
| [工作台命令带分区](desktop--workbench-command-band.md) | 标题铬与 Command band 分权；空态安静；文档命令留在画布 | reuse-pattern：URL 不下 caption 行 |
| [files-community/Files](files-community--Files.md) | Tab / 地址栏 48px / Inner Toolbar / 侧栏 / 状态栏 | reuse-pattern：地址栏独立行 |
| [CommunityToolkit/Windows SettingsCard](CommunityToolkit--Windows.md) | 设置行四列 Auto/*/Auto/Auto；Content 右对齐 hug | reuse-pattern：右槽不 stretch |
| [桌面设置行右槽簇](desktop--settings-form-row.md) | 路径+浏览同一簇贴尾；stretch+max-width 会空档居中 | anti-pattern：controlFill / Tooltip block |
| [microsoft/WinUI-Gallery](microsoft--WinUI-Gallery.md) | TitleBar 与 NavigationView 分两行；页面命令在 Frame | adapt：返回留标题栏；URL 不抄进 Content |
| [WinUI 标题栏三键](desktop--winui-titlebar-chrome.md) | 46×满高直角 caption；关闭色例外；Tall 栏与工具钮分槽 | reuse-pattern：贴边满高；anti-pattern：padding 缩进三键 |
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
| [ag-grid 选区](ag-grid--ag-grid-selection.md) | 选字与选格互斥；跨格必须自管 Range | reuse-pattern / anti-pattern |
| [日志清单文档选区](desktop--log-list-selection.md) | 指针 → 格 → 文档偏移 → 自画高亮 → 模型复制 | reuse-pattern |
| [Spectrum Table](adobe--react-spectrum.md) | resizer::after 画列界；排序图标未激活不占位 | reuse-pattern：文本区裁剪 |
| [VS Code 表格 sash](microsoft--vscode-table.md) | SplitView 做列轨道，标题只是 sash 格子里的文案 | reuse-pattern：区域划分 ≠ 文本 |
| [HarmonyOS 静态/动态模糊样本](HarmonyOS_Samples--FuzzySceneOptimization.md) | 转场前一次性 createEffect；动画帧上 blur 会掉帧 | reuse-pattern：先有板再开弹簧 |
| [iOS 26 Liquid Glass](apple--liquid-glass.md) | 导航层超材料：lensing、Regular/Clear、容器共享采样、物化调透镜 | adapt 契约；anti-pattern：内容层玻璃、玻璃叠玻璃、alpha 进出场 |
| [QWEA0 Liquid-Glass-Android](QWEA0--Liquid-Glass-Android.md) | View 系 SDF 透镜 + Regular/Clear + 独立 ScrollEdge | adapt 变体与捕获排除；anti-pattern：色散/重力默认开 |
| [Abdullajon1881 LiquidGlass](Abdullajon1881--LiquidGlass.md) | GlassEffectContainer 对应：共享 recorder + smin 并集 | reuse-pattern：一 host 多 consumer；anti-pattern：触点写进透镜 |
| [BarredEwe LiquidGlass](BarredEwe--LiquidGlass.md) | iOS Metal 截图层复现 | anti-pattern：`layer.render` 当 L0 |
| [Android 玻璃数据链路](android--liquid-glass-data-path.md) | QWEA0 pull / Abdullajon push / Kyant layer / Yo Host 六跳对照 | reuse-pattern：Host 推一次；anti-pattern：每板整树 Capture、chainEffect |
| [Kyant0 AndroidLiquidGlass](Kyant0--AndroidLiquidGlass.md) | Compose：一份内容 GraphicsLayer + 玻璃 RenderEffect | adapt 铬与透镜分权 |
| [HarmonyOS 动效体系](harmony--motion-system.md) | 时长/曲线/四类元素/共享容器；窗口动的是 Rosen 表面不是布局矩形 | reuse-pattern：合成器 Scale+Opacity；anti-pattern：HWND 尺寸插值 |
| [Apple 动效体系](apple--motion-system.md) | HIG 可关可取消；CA 跑 transform；同屏 zoom；跨屏无共享几何 | reuse-pattern：主窗一次到位；anti-pattern：应用层 for 循环改 frame |
| [nathangitter/fluid-interfaces](nathangitter--fluid-interfaces.md) | WWDC 2018：response/damping 弹簧；动 center/transform | adapt 参数换算；lesson-only：启动无手势 bounce |
| [动效规格统一](harmony-apple--motion-spec-unification.md) | 鸿蒙时长表 + 四类曲线 + Apple 弹簧换算；双端 MotionSpec 同名 | reuse-pattern：产品只点规格名；anti-pattern：配方/原生再写平行毫秒表 |
| [贴右横向开合](harmony-fluent--inline-end-clip.md) | Fluent `maxWidth`+overflow；WinUI compact 长度；CSS 只插同构轨道 | reuse-pattern：width 裁切贴 end；anti-pattern：`0fr auto`↔`minmax 1fr` |
| [ant-design/ant-design](ant-design--ant-design.md) | Seed→Map→Alias + ConfigProvider + color×variant；cssinjs 默认 | reuse-pattern：三层派生；adapt：两轴按钮/内外缀/单 Host；anti-pattern：cssinjs、自造色板、Ant 动效 |
| [arco-design/arco-design](arco-design--arco-design.md) | Less token → CSS 变量写 body；仓内 Trigger | reuse-pattern：热更新只改 `--*`；adapt：Trigger 单原语；anti-pattern：写 body、lighten、测盒循环 |
| [shopify/polaris](shopify--polaris.md) | token 元数据 + stylelint 禁 hex；React 包已归档 | adapt：lint 只许 `--yohu-*`；anti-pattern：抄 Shopify 皮肤（许可限制） |
| [primer/react](primer--react.md) | 开发者工具：ActionList + AnchoredOverlay；token 已是 CSS 变量 | reuse-pattern：菜单 Host/List 分权；adapt：复合槽与关闭手势；anti-pattern：写死 200ms、引进 Primer |

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

### 系统窗口动效 vs 控件动效（2026-09-08 增补）

- **控件层**（YoUI CSS）继续 atom / 配方 / 接线；时长与标准/减速/加速曲线已对齐鸿蒙。
- **窗口层**（启动交接、占用盒）必须走合成器属性（鸿蒙 `RSTransitionEffect` Scale+Opacity；Apple `CALayer` transform；Windows DComp / 分层 HWND 冻结位图）。禁止把 HWND/`NSWindow` 矩形当补间通道。
- **同屏**用共享容器：启动快照的视觉矩形 morph 到主窗外框。**异屏**用电脑层级淡入淡出：小窗出场加速，主窗一次落到最终矩形再进场；不要跨显示器 matchedGeometry。
- 鸿蒙 starting window：主窗布局已是终态，启动面 alpha 1→0 后从 leash 摘掉。Apple zoom 只适用于同一视觉连续性。
- 落选：WinUIEx SplashScreen（关 splash 再 Activate 主窗，无合成器过渡）；整仓再克隆 `arkui_ace_engine`（已有 Tabs/Dialog 笔记，体积过大）；Lottie（内容动画，不是窗口 presence）。

### MotionSpec 双端同名（2026-09-09 增补）

- **一层规格名**（`effectsFast` / `spatialPanel` / …）。TS 产出 CSS `var(--yohu-motion-*)` 与 `motionSpecMs()`；Rust `yohu_motion::MotionSpec::duration_ms()` / `ease()` 给 DComp。禁止配方层、splash、占用盒再写平行毫秒表。
- 时长锁鸿蒙《动效属性》100/150/160/200/300/350。窗口 `AnimationConfig` 默认 200ms + scale 0.7 只服务系统窗口，**不是**控件 token。
- 曲线按进场减速 / 出场加速 / 持续标准。弹簧 128/12/1 只在 CSS `linear()`；原生弹簧槽位回退 standard，不在 DComp 再积分。
- Apple `UISpringTimingParameters(damping:response:)` 与鸿蒙 interpolatingSpring 可换算；Reduce Motion 要把大位移换成淡入淡出（系统不会自动关掉自定义 animation）。

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

### 设置行右槽（2026-09-10 增补）

- SettingsCard 根 Grid 是 `Auto | * | Auto | Auto`：标题吃剩余，Content 在 Auto 列且右对齐。鸿蒙设置项是 `Blank()` 把 extra 推到尾。
- **右槽只有 hug 贴尾。** 路径+浏览是一簇，簇内只有 gap。把右槽 stretch 再给子级写死 max-width，中间必然空档，盒子看起来居中。
- Files 设置页把开关/下拉直接放进 SettingsCard.Content。DevTools 路径+Browse 的 Width=* 是编辑表单，不是紧凑设置行。

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
- ui-kit layering 可补：液态玻璃背景是 L1 共享能力；容器共享采样。
- 禁止 `api/` 出现 `UIGlassEffect` / `GlassEffectContainer` 类型名；对外用 Yo 变体枚举。
- Reduce Transparency / 高对比必须改材料层（更霜或实色），禁止只改前景字色。

已升格（2026-08-20，用户确认）：圆角几何进 L1；L5 门面要薄、禁止 `api/` import `internal`；组件禁止私自 `addRoundRect` / `GradientDrawable.setCornerRadius`；玻璃 / SDF 锁定 CIRCULAR。见 `instructions/rules/by-type/ui-kit/` 的 public-api / layering / file-srp / coupling。

## 入选与落选备忘

必读仓已浅克隆到 `%TEMP%/YoAgentResearch/`。霜玻璃 HIGH 着色器仍闭源；**点光源**在 `graphic_graphic_2d` 开源，已单列。HDS / UIDesignKit 无公开实现仓。

圆角：规范读本地鸿蒙《圆角参数》+ Apple `CALayerCornerCurve`；实现读 Rosen RoundRect 与 figma-squircle。`stoyan-vuchev/squircle-shape` 单 cubic 落选。

表头（2026-08-20）：入选 AG Grid / Spectrum Table / VS Code monaco-table；Fluent TableHeaderCell 只作对照（单元格底 hover ≠ 圆角片）。落选 Carbon DataTable、Fluent Blazor DataGrid（与 Spectrum / Fluent React 结构重复）。

HdsTabs 动效（2026-08-20）：入选 ace_engine TabBarPattern + Spatialization 接线 + 主题笔记（HDS 无仓）。落选 HarmonyOS-Cases customanimationtab、子页签下划线 Demo（那是 SubTabBar，不是底部 HdsTabs）。iHongRen/harmony-study-demo 只调 API，无曲线，不单列。

Dialog 空间弹出（2026-08-20）：入选 `arkui_ace_engine` `PlayDistortion` / `PlayFlowLight`。社区 DialogHub / 自定义 `transition` 是另一套进场，不单列。`graphic_graphics_effect` EdgeLight 着色器只作对照，Dialog 喂的是 `EdgeLightParam`。

Dialog 弹出扫光形态（2026-08-21）：补读 `UpdateEdgeLightFilter` 的 Dialog 分支（inner 0.1 / outer **0**）和 FrameGradientMask `AxialCoreWidth=0.3`。HDS `DualEdgeFlowLight` 是周长光线，产品不同，不单列作 Dialog L7 实现仓。落选把 `thickness=250` 当 Canvas stroke 的社区扫光 Demo。

Dialog 扫光看不见（2026-08-21）：同一主题补读 `GESDFEdgeLight::MakeImageMerger` 与 OverlayNG。入选仍是 ace_engine + graphics_effect + graphic_2d，不另开仓。根因是 YoUI L7 sibling overlay 丢了加色，不是再调粗细。

### iOS Liquid Glass 与背景模块（2026-09-04 增补）

- **玻璃是导航层材料，不是模糊参数袋。** Regular 自适应；Clear 固定更透且必须压暗。同簇禁止混变体。小件可 light↔dark；MENU/DIALOG 大板只调 tint。
- **Lensing 是主定义，frost 是厚度。** 物化/消物化调透镜带宽，禁止 alpha 冒充 `UIVisualEffectView.effect`。
- **容器拥有采样权。** 玻璃不能采样玻璃；多板 = 一张 backdrop + 场景 SDF（smin）+ 统一 luminance。`spacing` 是开始融化的距离。子板只贡献形状。
- **Scroll edge 是邻层。** 弱/强模糊渐变带或 hard scrim，不进透镜 pass。
- **Android 开源只提供手段。** QWEA0 / Abdullajon 的 SDF、捕获排除、smin、独立 edge view 可 adapt。色散、重力传感器、Compose/RN 壳、`layer.render` 截图不进 Yo。
- **与鸿蒙沉浸光感并存时的分工：** 鸿蒙 LUT/角色编译（`YoImmersiveLight.Role`）仍是产品入口；iOS 补的是背景光学契约（变体、容器、尺寸厚度、物化、edge）。不要第二套控件树。

### Android 数据链路（2026-09-04 源码）

- **H1 必须 push 且共享。** Abdullajon `Provider.dispatchDraw` 录一次、Kyant 一份 `GraphicsLayer`、Yo `BackdropHost` 已是这条。QWEA0 每块 `BackdropCapture.draw` 整棵 source 是 Dialog 未入树的 pull 后备，不能当 BAR 热路径。
- **排除玻璃靠树或注册表，不靠 `View.draw` 标志。** 硬件 `dispatchDraw` 不进 `View.draw`；QWEA0 自己注释 `isCapturingBackdrop` 拦不住，改 `TRANSITION_VISIBILITY`。Yo `OverlayExclusion.hideInside` 同解。
- **H3/H4 在 Motorola 上分 RenderNode，不 `createChainEffect`。** QWEA0/Abdullajon/Kyant 都链；Yo 已拆 sample→blur→body→lens。
- **折射只向内。** 四套 AGSL 都是 `-n`；向外读子输入越界是透明黑。margin 只给模糊。
- **多板融化是同一 H4 的 SDF 并集。** Abdullajon View 路径 `merge=0`；Compose 才 pack 最多 8 形。并排多个 `YoBlur` 不会融化。

Liquid Glass（2026-09-04）：规范读 WWDC 219/284 + UIKit JSON。实现读 QWEA0、Abdullajon、Kyant0 源码链路。BarredEwe 作 L0 反例。落选 Enie（O(r²) 找边）、PrismalAGSL / KMPLiquidGlass（与 Kyant/View 重复）、destefanis 条纹液体、conorluddy 纯目录。

窗口 / 启动动效（2026-09-08）：规范读本地鸿蒙《动效》五章 + Apple HIG Motion / WWDC 2018·2023·2024。实现入选 `openharmony/window_window_manager`、`microsoft/Windows.UI.Composition-Win32-Samples`、`nathangitter/fluid-interfaces`、`lwouis/alt-tab-macos`。落选 WinUIEx splash（无 morph）、JetBrains SplashManager（仓过大）、`arkui_ace_engine` 整仓再克隆。

贴右横向开合（2026-09-09）：Fluent Collapse 横向走测宽后的 `maxWidth` + `overflowX`，闭合可用 compact `outSize`；WinUI SplitView 同一 pane 变宽。CSS Grid 只插**同构轨道**。Yohu 发送栏应对齐 `YoSwap`/`rail` 的 width 裁切，禁止 `0fr auto`↔`minmax(0,1fr) 0fr`。

### 日志清单文档选区（2026-09-10 增补）

- **选字与选格互斥。** AG Grid 默认 `user-select: none` + 模型 Range；`enableCellTextSelection` 只保证格内，并关掉网格 clipboard。跨格选字不能靠浏览器。
- **日志是 Family A 文档。** AS Logcat 字段是 `Document` span；VS Code 复制 `getValueInRange`；DevTools Console 复制拼模型行。Yohu 对应 `formatLogLine` 偏移，不是 grid DOM 序。
- **从左点 Tag 不吞时间**，因为命中落在 Tag span 起点，不是行首。
- **双击不闪**，因为手势直接写模型，禁止先让浏览器选一整行再改回去。
- **禁止** `user-select: text` 跨格、`Selection.toString()` 当载荷、锁格升档补丁、Monaco/textarea。

入选：DevTools Console、AS Logcat 文档、AG Grid 选区互斥、VS Code Selection、主题笔记。落选：继续打 `user-select` 补丁；把日志换成 Editor。

### 企业级设计体系 / YoUI 升级（2026-09-10 增补）

主题：设计师主导的开源 UI 库，服务 `@yohu/ui` 升级，**主参考 Ant Design**。源码浅克隆在 `%TEMP%/YoAgentResearch/`，知识库只有 Markdown。

- **三层 token 是共识，派生手段不是。** Ant：Seed→algorithm→Map→Alias，运行时 cssinjs（v6 才有 `zeroRuntime`）。Arco：Less 编译成 CSS 变量，`ConfigProvider` 只 `setProperty`。Polaris / Primer：浅/深覆盖表 + CSS 变量。YoUI 已是 Harmony Primitive→Semantic→Component + `emit-theme.ts`，对齐 Arco/Polaris 的静态变量，**不要引进 cssinjs**。
- **色板锁鸿蒙官方表。** Ant `@ant-design/colors.generate`、Arco `lighten(±10)` 都是另造梯度。禁止从品牌色算法生成 10 阶。
- **动效不要向 Ant/Arco/Primer 看齐。** Ant `seeds.ts` 自承 Motion Token 未收敛（0.1/0.2/0.3s）；Primer Overlay 写死 200ms。YoUI `MotionSpec` 已锁鸿蒙，继续当唯一时长源。
- **Button 必须两轴。** Ant `color×variant`，Arco `type×status`。现在的 `YoButtonVariant = primary|secondary|ghost|danger` 把形和语义色混在一起。
- **输入复合：盒内 prefix/suffix，盒外 addon，status 一等。** Ant 与 Arco 同构。`YoTextField` 只有 clearable。
- **浮层一个挂载点。** Ant/Arco `getPopupContainer`；Ant Tooltip `UniqueProvider` 共享一个 popup；Primer `AnchoredOverlay` 分自绘锚点 / 外来 ref。对照已有唯一 `YoContextMenuHost` + `popover-place`。缺的是 Tooltip，不是第二套 Trigger。
- **命令式 API 必须能挂回树。** Ant `App` + `useMessage`；Arco 仍用全局 `Message.config`。YoToast `createToaster` 已是正确形态。
- **工作台菜单看 Primer，不看 Ant Dropdown。** `ActionList` 复合槽 + `ActionMenu` 只管开合手势。右键已走这条，工具栏溢出/页眉 overflow 应复用同一 List。
- **lint 看 Polaris，不看它的皮肤。** 白名单从 emit 的 token 名生成（`getThemeVarNames`），不是手写 regex。全局 `--yohu-*` 只许清单里的名字；组件私有变量另前缀。许可是 MIT 变体（独立应用不得长得像 Shopify）。
- **右键键盘是缺口。** Primer ActionList 有 Arrow / typeahead / Tab 关菜单；YoContextMenu 主要只有 Esc。补键盘，不要改成每处自挂 ActionMenu。
- **Arco 暗色是选择器覆写，不是算法。** `body[arco-theme='dark']` 只换 alias。右键用 `alignPoint`，跟已有 `openContextMenu(x,y)` 同构。Arco **没有** 全局 density，不能拿它的 `size` 冒充 YoUI `data-density`。
- **明确不要做的控件。** YoTable / YoForm 引擎 / DatePicker / Upload / Wave / 中文插空格。清单继续 `YoCol*` + `YoVirtualList`；设置继续 `YoFormRow`。

#### 对本知识库规则的候选修订（本主题）

只记录建议，不自动改 `instructions/rules/`。

- ui-kit token：业务与组件 CSS 只许语义/组件层；Primitive/Seed 仅 `tokens/` 与契约测试。
- ui-kit 公开 API：实心控件的「外形」与「语义色」分轴；禁止再把 `danger` 当成第四种 variant 而不给 status。
- ui-kit 浮层：Select / 右键 / Tooltip 共用一个 Portal 政策；禁止组件私写 `z-index` 魔法数。
- ui-kit 菜单键盘：右键/下拉至少 Arrow / Home / End / Esc / Tab 关；禁止只靠指针。
- token lint：白名单从 `emit-theme` 产物生成，禁止模块定义新的 `--yohu-*` 名。
- 禁止把 `@ant-design/cssinjs` / `antd` / `@arco-design/web-react` / `@shopify/polaris` / `@primer/react` 写进 `@yohu/ui`。

#### 入选与落选备忘（本主题）

入选 4：`ant-design/ant-design`（主参考）、`arco-design/arco-design`（CSS 变量对照）、`shopify/polaris`（token/lint，非控件面）、`primer/react`（工作台菜单）。

落选：

- Semi Design / TDesign / Element Plus：与 Ant/Arco 同构中后台全家桶。
- MUI：Material 3 语言与鸿蒙/YoUI 冲突。
- Carbon：与 Polaris 同属西方企业体系；本轮要的是 lint/token 工程，Polaris 更贴。
- shadcn/ui：复制模板，不是设计体系仓。
- Fluent / Spectrum / Radix：已有动效/表头/Presence 切片，不整仓重研。
- ant-design-vue / NG-ZORRO：同一设计语言的框架移植。
- Meta Astryx：2026 Beta，不当前主参考。
- `@primer/primitives` / `@ant-design/cssinjs` 独立仓：本轮结论已够，不另开篇。
