---
id: research.harmony-hdstabs-point-light
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++, ArkTS]
  frameworks: [harmonyos, arkui, hds, rosen]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: other
  repo: HarmonyOS HdsTabs + Rosen point light
  url: https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ui-design-hdstabs
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d"
studied_at: 2026-08-20
related:
  - research.openharmony-graphic_graphic_2d-point-light
  - research.openharmony-arkui_ace_engine-immersive
  - research.harmony-immersive-light-layers
---

# HarmonyOS HdsTabs 点光源（主题笔记）

HDS / UIDesignKit **无公开实现仓**。GitCode / GitHub / AtomGit 搜 `HdsTabs`、`hdsMaterial` 只有官方样本与第三方 Demo。结论：**组件层只接线，算法在 Rosen + ArkUI 开源引擎。** 不要假装读过 HDS 私有仓。

对照 clone：

| 路径 | 角色 |
|------|------|
| `%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d` | 半径、配对、着色器 |
| `%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine` | `.pointLight` 落地、`lightEffect` 跟手 |
| `%TEMP%/YoAgentResearch/SoraLuna--ui-design-kit-hds-immersive-navigation-demo` | 只用 `systemMaterialEffect`，未设 `lightColor` |
| `%TEMP%/YoAgentResearch/HarmonyOS_Samples--HarmonyOSComponentUXExamples` | HdsTabs 布局样本，无 `lightColor` |
| `E:\GithubGallery\_tmp_research\harmonyos-immersive-light` | 本地 Rosen/docs 副本 |
| `E:\Dev\Doc\HarmonyOS-Developer-docs` | HdsTabs / 底部页签 / hdsEffect / 点光源系统接口 |

## 入选理由

回答 YoTabs 六问：光是灯还是斑、半径、IlluminatedType、`lightColor` 接到哪、跟手出界、能抄什么。

## 1. 点光源是灯，不是贴在组件上的光斑

官方系统接口（`ts-universal-attributes-point-light-style-sys.md`）：`lightSource` **照亮周围被标 Illuminated 的组件**。hdsEffect 同一模型：一盏灯最多照 12 个受光件；灯坐标初始化在组件中心，**不跟滚动位移**。

Rosen：`RSPointLightManager` 配对光源节点与受光节点。绘制在受光节点的 `RSCoverageNGShaderDrawable` 上，用圆角矩形 **描边/填充**，不是独立 overlay 贴图。

**HdsTabs 用哪一种：** 开源等价物是 **同一节点既挂灯又挂受光**（自照明胶囊），不是邻项互照。ArkUI `ControlInteractionBase`：

- `InitLightEffect`：本节点 `IlluminatedType = 3`（BORDER_CONTENT），边宽 0.5vp
- 按下/移动：本节点 `LightPosition(localX, localY, 80vp)` + `intensity = 3` + `lightEffect.color`

HdsTabs 闭源，但 API 语义与 ArkUI 26 `Tabs.barFloatingStyle.systemMaterial.lightEffect` 对齐；设计指南「指尖位置被定义为动态光源……照亮容器边缘」。

## 2. positionZ → 照射半径

见 [openharmony--graphic_graphic_2d-point-light.md](./openharmony--graphic_graphic_2d-point-light.md)。`radius ≈ 1.731 × z_px`。沉浸触点光写死 **z = 80vp → ~138vp**。hdsEffect `options.height` 对应这根 z（示例 150）。

## 3. IlluminatedType 对 Tabs 意味着什么

| 类型 | Tabs 含义 |
|------|-----------|
| BORDER | 只亮胶囊描边，符合「沿边缘照亮」字面 |
| CONTENT | 只亮填充，边缘弱 |
| BORDER_CONTENT = 3 | **开源 Immersive `lightEffect` 实际值**：边 + 内容 |
| BLOOM_* | Rosen 开源绘制无分支，Tabs 不要用 |
| FEATHERING_BORDER（Rosen=9） | 羽化边；HDS 公开枚举是 `DEFAULT_FEATHERING_BORDER=20`，闭源映射 |
| BLEND_* | OVERLAY 混合，按压阴影那条，不是 Tabs 默认 |

设计文案要边缘光晕；开源实现同时填内容（强度 ×0.3）。YoTabs 应对齐 **BORDER_CONTENT**，不要只画 CONTENT 光斑。

`hdsEffect.pointLight` 支持的组件列表 **不含 Tabs**。HdsTabs 走的是材质 `lightEffect`，不是开发者再 `.visualEffect(pointLight)`。

## 4. HdsTabs.lightColor 接到哪一层

`HdsTabsFloatingStyle.lightColor`：页签栏光效颜色。默认浅 `#33FFFFFF`、深 `#33E5E5E5`。设计：改色用低透明、高亮度。

| 层 | 是不是 lightColor |
|----|-------------------|
| Rosen `RSLightSource.color` / ArkUI `lightEffect.color` | **是（推断 + 开源等价）** |
| SDF `RSNGSDFEdgeLightEffect` overlay | **否。** 那是 `edgeLight` / Sheet / Menu。`UpdateEdgeLightFilter` 与 `ControlInteractionBase` 不是同一函数 |
| 静止 rim（FrostedGlass `edLight`） | **否。** 那是 L3，色不来自 lightColor |
| `gradientMask` / VEIL | **否。** 蒙层默认 `#CCF1F3F5` / `#99000000` |

HdsTabs **没有** `interactive` 字段。ArkUI 26 把形变和光拆开：

- `ImmersiveMaterial.interactive` → 形变（scale/offset，出界归一化到 ±3×半轴）
- `ImmersiveMaterial.lightEffect.color` → 点光源色
- HDS 悬浮栏按设计指南默认带光感形变；光色用 `lightColor`

`systemMaterialEffect` 是霜玻璃对象，不是光颜色。

**SMOOTH/LOW：** `GetImmersiveMaterialConfig` 早退时 **丢掉 `lightEffectOptions`**。低算力官方触点光开源路径不生效。形变 `interactive` 仍会拷进 config。

## 5. 手指按下：灯能否出组件、照明是否仍在组件内

能出界。`UpdateLightPositionAndColor` 用 **未钳制的 local XY**。`GetLocalLocation()` 在手势仍捕获时可 <0 或 >宽高。配对用圆 vs AABB，灯在矩形外只要距离 ≤ radius 仍照亮。

照明只画在受光节点圆角矩形上（Pen 描边 / Brush 填充）。光不会画到胶囊外的页面上——除非别的节点也标了 Illuminated。

官方跟手：

- 光：每帧写灯坐标，松手 `intensity=0` 并 Reset position
- 形变：按下用 spring（228/16，1000ms），移动直接写，松手弹回；归一化钳在 ±3
- 多指：直接 Cancel
- `adaptToHandedness` 是 **整栏左右搬家**，不是指尖光

hdsEffect 文档：灯不跟滚动。Tabs 触点光是 touch handler 每帧改 position，与「静态 pointLight 不跟滚动」不矛盾。

## 6. 对 YoTabs / ImmersiveTouchLight

**抄（reuse-pattern）：**

- 自照明：灯与受光都是胶囊，不要每项一盏灯
- z=80vp，半径 ~1.73z（约 138vp @1x）
- intensity 量级用「按下 3、松开 0」，不要死守文档 0–1
- 色：浅 `#33FFFFFF`、深 `#33E5E5E5`；业务改色保持低 alpha
- BORDER_CONTENT：边 0.5vp 高光 + 弱内容填
- 二次衰减，0.8R 外 smoothstep 灭
- 灯 XY 跟手可出界；绘制 clip 在胶囊圆角
- 形变与光分通道；REDUCE/LOW 关光

**改写（adapt）：**

- RuntimeShader → AGSL；无 `RSPointLightManager`
- vp→px 用 Android density
- HDS 闭源默认「始终 interactive」→ YoUI 用档位显式开 L5/L6

**不要（anti-pattern）：**

- 把 `lightColor` 当 L3 静止 rim 或 1vp 盒描边
- 用 SDF EdgeLight / Sheet 流光冒充 Tabs 指尖光
- `clipPath` 切在描边内沿（硬卡片）
- 灯画出胶囊、照亮页面内容
- 移植 BLOOM_*、hdsEffect=20、12 灯配对
- 滚动列表上挂静态 `.pointLight`（官方不建议）
- dlopen `libhdsmaterialimpl.z.so`

## 架构设计经验

材质是霜；指尖光是 **按压期间挂在同一节点上的真点光源**。HDS 只暴露 `lightColor` + `systemMaterialEffect`。ArkUI 26 把同一机制拆成 `lightEffect` + `interactive`。分层主题笔记里「L6 = SDF overlay[1]」描述的是 **另一条** `edgeLight` 管线；Tabs 开源触点光走 Rosen point light drawable。

## 阅读范围

实际读过：上文 clone 表；本地 `HdsTabs.md`、`底部页签.md`、`hdsEffect.md`、点光源系统接口；ArkUI `Tabs.md` FloatingTabBarStyle；`component_material_interaction.cpp`；`rosen_render_context.cpp` `OnLight*Update` / `UpdateEdgeLightFilter`。

未读：HdsTabs HSP、`libhdsmaterialimpl.z.so` 是否额外叠 SDF overlay、真机 HdsTabs 松手曲线是否等于 228/16 spring。
