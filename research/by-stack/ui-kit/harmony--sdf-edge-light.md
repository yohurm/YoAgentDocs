---
id: research.harmony-sdf-edge-light
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [cpp, sksl, java]
  frameworks: [openharmony, arkui, android]
also_relevant: []
utilization: [adapt, anti-pattern]
source:
  platform: github
  repos:
    - openharmony/graphic_graphics_effect
    - openharmony/graphic_graphic_2d
    - openharmony/arkui_ace_engine
  cloned_to:
    - "%TEMP%/YoAgentResearch/openharmony--graphic_graphics_effect"
    - "%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d"
    - "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.openharmony-graphic_graphics_effect
---

# HarmonyOS SDF 边缘光：「环境光沿边缘柔和流转」

主题笔记。实现 Agent 只对齐公式与分工，不要把第三方源码拷进 YoAgentDocs / Android-YoUI。

本地规范原文（设计《沉浸光感》均衡档，2026-06-12）：**「环境光沿边缘柔和流转，自然勾勒层次」**。这是静止态环境高光，不是手指点光源。官方开发文档：高/中算力走 `materialFilter` + `shadow`；`systemMaterial` 生效后背景变透明、**边框宽度恢复为无边框**。低算力才用 `backgroundColor + borderWidth + shadow`。

## 1. 它是 SDF 场光照，不是 canvas stroke

能量沿 **SDF 法线（有符号距离 d）** 衰减，再乘 **切向/轴向 mask**。没有 `Paint.STROKE` 闭合路径。

两条官方静止相关路径：

| 路径 | 落点 | 何时出现 |
|------|------|----------|
| A. FrostedGlass `edLight` | 材质滤镜内部 | HIGH/MID `ImmersiveMaterial` 的均衡/强档 |
| B. `GESDFEdgeLight` / `RSNGSDFEdgeLightEffect` | overlay 着色器 | `edgeLight` API、Sheet 出现光、交互 `lightEffect` |

设计文案主要指 **路径 A**。路径 B 是同一套「SDF 距离 + mask」语言的 overlay。YoUI 现在的 SweepGradient 描边两边都不像。

### 路径 A（材质滤镜，静止流转）

`GEFrostedGlassEffect`：轮廓带宽 × 对角扇形。

- `edgeBand = EdgeBandAA(sd, width, feather, shift)`：在 SDF=0 附近一条带，宽度被 `min(highLightWidthPx, borderWidth)` 夹住。
- `diagMask = DiagonalFanMask(uv, dir, angleDeg, featherDeg)`：从中心看，沿 `dir` 与 `-dir` 各一个扇叶；角度外 `smoothstep` 熄灭。
- 最终 `edge = edgeBand * diagMask`，再把高光色 mix 进模糊背景。

ArkUI LUT（GENTLE / REGULAR / 浅色，均衡档量级）：

| 字段 | 值 | 含义 |
|------|----|------|
| `edLightParams` | `{0.83, 0.92}` | 带宽 vp、羽化 |
| `edLightAngles` | `{75°, 120°}` | 扇叶半角、羽化角 |
| `edLightDir` | `{0, -1}` | 朝上；顶沿亮、底沿暗 |
| `edLightKBS` | `{1.0, 0.2268, 1.5}` | 高光振色 K/B/S |

ULTRA_THIN 带宽更窄：`edLightParams = {0.62, 0.92}`，同样 75°/120°、dir `(0,-1)`。弱档 LUT 里 `weightsEdl` 与 `edLight*` 常为空——弱档官方收敛光彩，只留背景与边框。

这不是 360° 白描边：底沿扇叶到不了（dir 朝上，半角 75° 盖不住正下方）。

### 路径 B（SDF overlay）

`GESDFEdgeLight` Filter SkSL（算法要点，非法条文原样拷贝）：

1. `lightMaskValue = lightMask.eval(p).r`
2. SDF 解码（Filter）：`d = (encoded * 2 - 1) * spreadFactor`；Shape 先按 `(x+63.5)/127.5` 映到 f16，避免量化。Shader 路径直接读 `sdfSample.a`。
3. 细边（法线）：`edgeThickness = mix(minBorderWidth, maxBorderWidth, smoothstep(0,1,intensity))`；`thinBorder` 是 `|d|` 在 thickness 内的平滑帽（Filter 版还区分内外符号）。
4. Bloom（法线）：`dNorm = |d| / (d>0 ? outerBloomWidth : innerBloomWidth)`；`falloff = max((1-dNorm) / (1+dNorm)^bloomFalloffPow, 0)`；`bloom = maxBloomIntensity * falloff`。仅当 `intensity` 越过 `bloomIntensityCutoff` 才开。
5. `b = intensity * maxIntensity * (thinBorder + bloomPart)`
6. 输出 `rgb = lightColor * b`；Filter 再与原图 **RGB 相加**。

Shader 路径：`lightMaskValue < 1e-5` 直接丢弃；只画外接矩形减内接矩形的环，避免整卡着色。

切向没有单独的指数衰减。切向明暗全部来自 `lightMask`。mask=1 时整圈等亮——那才是发丝线圈，官方用 mask 禁止这种情况。

## 2. lightMask / FrameGradientMask：流转与「顶亮底暗」

`lightMask` 是 `GEShaderMask`。边缘光只读它的 **R 通道当强度**。没有 mask，Filter 直接失败返回原图。

### FrameGradientMask（ArkUI `UpdateEdgeLightFilterWithLightMask`）

圆角盒 SDF 带宽 × 三次 Bezier × 轴向包络。

- 带宽：`innerGradient` / `outerGradient` 归一化后取 min；带外为 0。
- 轴向：`t = clamp(dot(localPos, axialDir) * invSpan + 0.5, 0, 1)`；`envelope = smoothstep(0, riseEnd, t) * (1 - smoothstep(fallStart, 1, t))`；`gradient *= 1 + strength*(envelope-1)`。

ArkUI overlay 默认：

| 参数 | 值 | 作用 |
|------|----|------|
| `AxialDirection` | `(0, 1)` | 沿局部 Y |
| `AxialFeatherStrength` | `1.0` | 轴向外完全压暗 |
| `AxialCenter` | `0.5` | 包络中心 |
| `AxialCoreWidth` | `0.3` | 亮核很窄 |
| `InnerFrameWidth` | `1000` | 几乎整块内部都算「内带」 |
| `OuterFrameWidth` | `0.1` | 外溢极薄 |
| `RectPos` | `(0, 0.1)` | 条带略偏 |
| `BoxAngleDeg` | 由 `EdgeLightPosition` | TOP=0，把亮核转到顶边 |

结果：一条**有限弧长的亮帽**沿轮廓走，不是闭合线圈。旋转 `BoxAngleDeg` 即「流转」到顶/左/右/底。

Sheet 出现光不用 FrameGradient，改用 `RadialGradientMask`（中心约 `(0.5, 0.05)`）：顶沿一点扫开，同样不是 360°。

路径 A 的 `DiagonalFanMask` 是静止材质的等价物：dir `(0,-1)` + 75°/120° = 顶部锥。

## 3. 参数表

### GE 结构体默认（算法仓，不是产品 LUT）

| 字段 | GE 默认 | 夹取 |
|------|---------|------|
| `sdfSpreadFactor` | 64 | 0–4096 |
| `bloomIntensityCutoff` | 0.1 | Shader 侧 0–1 |
| `maxIntensity` | 1 | |
| `maxBloomIntensity` | 1 | |
| `bloomFalloffPow` | 2 | |
| `minBorderWidth` | 2 | |
| `maxBorderWidth` | 5 | |
| `innerBorderBloomWidth` | 30 | |
| `outerBorderBloomWidth` | 30 | |
| `lightMask` | 必填 | |
| `sdfImage` / `sdfShape` | 二选一，image 优先 | |

### ArkUI overlay 默认（`RosenRenderContext::UpdateEdgeLightFilter`，已从源码核对）

| 字段 | 值 | 备注 |
|------|----|------|
| `SpreadFactor` | **47.8** | 不是 GE 的 64 |
| `BloomIntensityCutoff` | 0 | 任意强度都可开 bloom |
| `LightMaxIntensity` | **2.0 × intensity** | |
| `MaxBloomIntensity` | **19.3 × intensity** | |
| `BloomFalloffPow` | **8.7** | 远高于 GE 的 2，晕贴边 |
| `MinBorderWidth` | **10.3** | |
| `MaxBorderWidth` | **9.9** | 小于 min：更亮则细边略收 |
| `InnerBloomWidth` | `thickness × 0.8` | Menu/Dialog 改为 ×0.1 |
| `OuterBloomWidth` | `thickness × 0.2` | Menu/Dialog 改为 ×0.0 |
| `Color` | 参数 RGB / 255 | alpha 丢弃 |

Sheet 出现光是另一套：Spread 64，内外 bloom 30/20，强度 1，pow 1——动画扫光，不是静止材质。

### Inner 0.8 / Outer 0.2 对白底意味着什么

比例乘在 **thickness（px）** 上，不是 GE 的 30px 绝对宽度。

- 80% 能量往玻璃**内侧**渗；外侧只留 20% 薄晕。
- 合成是 **加白**。白霜 + 白晕对比度≈0。
- Menu/Dialog Outer=0：官方连那 20% 外晕都关掉。
- **白底要看见边缘：先做接触变暗 / occlusion / 背景振色压暗，再加高光。** 只加白或加粗白 stroke 是 anti-pattern。

路径 A 的可见性同样靠滤镜里的背景压暗（`bgKBS`）和折射，不是 1vp 白边。

`MaxBorderWidth 9.9 < Min 10.3`：`thickness = 10.3 + intensity×(-0.4)`。mask 越亮，细边略变窄、bloom 相对更主导——亮帽柔软，避免变成硬线圈。

## 4. 为什么官方禁止发丝 border

算力枚举：`EXQUISITE`（高）/ `GENTLE`（中）/ `SMOOTH`（低）。

`ViewAbstract::SetImmersiveConfigs`：

- **SMOOTH（低）**：主题写入 1vp 半透明边框 + 实色背景 + 阴影。无 `materialFilter`。
- **GENTLE / EXQUISITE（中/高）**：先 `RemoveBorderAndBackgroundEffect`——已有 `borderWidth` **清成 0**，背景改透明——再挂 `materialFilter` / EC shader。

官方 API 文档原话：高/中算力 `systemMaterial` 生效后，已设 `backgroundColor` 恢复透明，已设 `borderWidth` **恢复为无边框**。高光在滤镜里。再画 1px 白描边会：

1. 与 SDF 带宽抢同一条等值线，变成双线；
2. 360° 闭合，直接违反「沿边流转 / 顶亮底暗」；
3. 白底上看成发丝线圈，正是设计禁止的。

弱档 / 低算力才允许 1vp 边框代替滤镜高光。HIGH 学低档加描边 = 档位用反。

## 5. YoUI 现状与官方差距

| | 官方 | YoUI 现在 |
|--|------|-----------|
| 静止高光载体 | HIGH/MID：`materialFilter` 内 SDF 带宽×扇形；overlay 另用 SDF+mask | `GlassLensProgram`：`mix(white)` × sky；失败则 `BackdropChrome.drawRim` SweepGradient stroke |
| 法线衰减 | `f(\|d\|)`，内外 bloom 不对称 | 透镜有 optical/glue；画布 rim 是固定 stroke 宽 |
| 切向 / 流转 | `DiagonalFanMask` 或 `FrameGradientMask` 轴向包络 | SweepGradient 绕几何中心扫角；注释已写「不是 Rosen SDF」 |
| 无边框 | HIGH/MID 强制 `borderWidth=0` | HIGH/MID 已 0；LOW 才 1dp |
| 白底 | 滤镜压暗 + 再加高光 | 透镜有 `LENS_OCCLUSION`；画布 rim 另画一层 `RIM_OCCLUSION_COLOR` stroke，仍是描边 |
| 点光源 | `lightEffect` → `RSNGSDFEdgeLightEffect` | 跟手径向光斑，另一条管线 |

`GlassLensProgram` 的 sky 项（`n.y`）方向对，但仍是 **加白**，没有官方那条 `thinBorder(d)+bloom(d)` 的法线剖面，也没有独立 `lightMask`。`drawRim` 的 200° 锥是角域近似，能量从中心辐射，不是沿轮廓法线；span 一大，圆角处仍像线圈。

## 6. 白底可见边缘的正确做法

**学 occlusion + fresnel / 接触压暗，不要加粗 stroke。**

顺序：

1. HIGH/MID 保持 `borderWidth=0`。
2. 在 SDF 零等值线内侧做接触暗（YoUI 已有 `LENS_OCCLUSION` / `RIM_OCCLUSION_COLOR` 方向对，应进透镜场，而不是第二圈 stroke）。
3. 再沿法线加高光：细边 ~10px 量级、bloom 内 80% 外 20%、pow≈8.7。
4. 用扇形或轴向 mask 关掉底沿；禁止 `span≥360`。
5. 合成用加色或有限 mix，但高光色不要假定「白底上加白可见」。

加粗 `drawRim` stroke / 提高 `RIM_COLOR` alpha = 发丝线圈，官方 HIGH 专门清掉的东西。

## 7. GlassLensProgram / BackdropChrome.drawRim 五条改写建议

给后续实现 Agent，不在本任务改 Android-YoUI。

1. **载体对齐档位。** HIGH：只在 AGSL 里画 rim（SDF 法线剖面 × 顶部扇形）；MID 软件回退用同一公式采样 SDF，不要用 SweepGradient stroke 冒充。LOW 才允许 1dp 边框。`drawRim` 在透镜已安装且 `restEdgeLight>0` 时必须继续不叠第二圈。
2. **拆成「距离 × mask」。** 法线：`thinBorder(|d|) + inner/outer bloom`；切向：`edLightDir=(0,-1)` 的 75°/120° 扇形，或轴向 envelope。禁止用绕中心的 SweepGradient 当唯一空间函数。
3. **先暗后亮。** 白底：occlusion / 接触暗写进同一 SDF 场（透镜 `occ` 项），高光加在压暗之后。删掉「只靠 `mix(white)` / 加粗白 halo」作为可见性手段。
4. **Bloom 比例抄 ArkUI overlay，不要抄 GE 30/30。** Inner 0.8、Outer 0.2、FalloffPow ~8.7、细边 min/max ≈10.3/9.9。外晕加宽 ≠ 更像官方。
5. **分清静止与 overlay。** 静止 = 路径 A 扇形，跟手指无关。`lightEffect` / 出现扫光才是路径 B + FrameGradient/径向 mask。不要把点光源半径画成静止闭合描边。

## 阅读范围

- 设计：`E:\Dev\Doc\HarmonyOS-Developer-docs\设计\...\沉浸光感.md`；开发：`沉浸光感.md`、`@ohos.arkui.uiMaterial`、FAQ。
- graphics_effect：见 [openharmony--graphic_graphics_effect.md](openharmony--graphic_graphics_effect.md)。
- graphic_2d：`rs_render_shader_def.in` 的 `RSNGSDFEdgeLightEffect` 属性袋；`rs_ui_shader_base.cpp` 工厂。
- arkui_ace_engine：`rosen_render_context.cpp` `UpdateEdgeLightFilter` / `UpdateEdgeLightFilterWithLightMask`；`view_abstract.cpp` `RemoveBorderAndBackgroundEffect`；`ui_material_filter_creator.cpp` edLight LUT；`sheet_edge_light.cpp`；`ui_material_theme.cpp` 低档 1vp 边框。
- YoUI（只读对照）：`GlassLensProgram.java`、`BackdropChrome.drawRim`、`LiquidGlassOptics.edgeLight`、`ImmersiveDimens`。

未打开闭源 `libhdsmaterialimpl.z.so`。EXQUISITE 的滤镜创建走该 so；OpenHarmony 开源 LUT 覆盖 GENTLE，公式仍以 graphics_effect 为准。
