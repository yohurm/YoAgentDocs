---
id: research.harmony-dialog-appear-flow-light
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [cpp, java]
  frameworks: [harmonyos, arkui, rosen]
also_relevant: []
utilization: [adapt, anti-pattern]
source:
  platform: gitcode
  repos:
    - openharmony/arkui_ace_engine
    - openharmony/graphic_graphics_effect
  cloned_to:
    - "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
    - "%TEMP%/YoAgentResearch/openharmony--graphic_graphics_effect"
studied_at: 2026-08-21
updated_at: 2026-08-21
related:
  - research.openharmony-arkui_ace_engine-dialog-spatial
  - research.harmony-sdf-edge-light
  - research.openharmony-graphic_graphics_effect
---

# HarmonyOS Dialog 弹出扫光：沿边缘的白光线，不是粗扫描带

主题笔记。对照 YoUI 把 L7 画成「铺满卡片的粗白带」的观感差距。源码在 `%TEMP%/YoAgentResearch/`（本机实际根是 `E:\System\Temp\YoAgentResearch`），HEAD `arkui_ace_engine` `8626690d`。

## 入选理由

用户观感：HarmonyOS 是「沿着白色光线扫」，YoUI 扫光粗很多。必须回答三件事：官方扫的是哪条几何、`thickness=250` 是不是描边、HDS 双边流光是否同一条效果。

## 1. Dialog 弹出扫光 ≠ HDS 双边流光

两条官方能力不要焊成一套 Canvas：

| | Dialog / Menu 空间弹出 | HDS `DualEdgeFlowLight` |
|--|------------------------|-------------------------|
| 入口 | ArkUI `PlayFlowLight` → `EdgeLightParam` | `@kit.UIDesignKit` `HdsVisualComponent` / `hdsEffect` |
| 何时 | 高算力 Dialog/Menu 打开；系统自动 | 应用显式挂胶囊 / 屏幕边缘 |
| 颜色 | 白 | 任意 `EdgeFlowLightParam.color` |
| 几何 | SDF 边缘光 × FrameGradientMask，位置枚举 BOTTOM→TOP | `startPos`/`endPos` 沿**周长**的比值 |
| 循环 | 一次；结束 `ResetEdgeLight` | 可循环 |

HDS 文档把位置定义成「上边缘中点起、沿容器边缘走、周长归一化」。那才是「一条光线贴边走」。Dialog 没有用这套 API，但**视觉语言相同：能量只在轮廓上**。带背景蒙层的双边流光 GIF 是彩色 aurora 顶条，不是 Dialog 弹出白光，不要拿来当 L7 目标。

## 2. `PlayFlowLight` 实际喂什么

`dialog_pattern.cpp`：

| 符号 | 值 | 含义 |
|------|-----|------|
| `EDGELIGHT_THICKNESS` | **250** | `CalcDimension`，进 SDF **bloom 宽度**，不是 Canvas stroke |
| `EDGELIGHT_LENGTH_RATIO` | **0.4** | `length = height × 0.4` → FrameGradientMask 的 `RectH` |
| `EDGELIGHT_INTENSITY` | **0.2** | 峰值强度；移动阶段是 0 |
| 移动 | 568ms，BOTTOM→TOP，intensity 0 | 先挪位置 |
| 高光 | delay 305ms，220ms，TOP，intensity 0.2 | 到顶再亮一下 |
| 消失 | delay 443ms，221ms，intensity 0，thickness 0 | 然后 Reset |

过期注释仍写「左上→右下画边」；代码是 **BOTTOM → TOP**。

`RosenRenderContext::UpdateEdgeLightFilter`（Dialog 父节点 tag 走 Menu/Dialog 分支）：

| 着色器字段 | Dialog / Menu | 普通 overlay |
|------------|---------------|--------------|
| `Min/MaxBorderWidth` | 10.3 / 9.9 **px** | 同左 |
| Inner bloom | `thickness × 0.1` | `thickness × 0.8` |
| Outer bloom | **`thickness × 0.0`（关掉）** | `thickness × 0.2` |
| BloomFalloffPow | 8.7 | 8.7 |
| LightMaxIntensity | `0.2 × 2.0 = 0.4` | 按 intensity |

所以 Dialog 外晕为 0：光坐在玻璃**内侧贴边**，外面没有一圈雾。细边约 10px。250 × 0.1 ≈ 25px 的内晕被 pow 8.7 收成贴边软光，不是 250dp 填色。

`UpdateEdgeLightFilterWithLightMask`：

- `AxialCoreWidth = 0.3`：亮核只占 mask 的 30%
- `RectH = length / height = 0.4`
- `BoxAngleDeg`：TOP/BOTTOM 都是 0，靠 `positionY` 把 mask 挪到顶或底
- `AxialDirection = (0, 1)`

亮区是一条**有限高度的边缘帽**沿 Y 从底滑到顶。中段只点亮左右轮廓上很短的一段，看起来就是白光线贴边扫过，不是整张卡片被一条粗带刷过去。

## 3. YoUI 为什么会粗

| | 官方 | 改前 YoUI |
|--|------|-----------|
| 载体 | SDF 细边 × mask | 整圈 `Path` 描边 × 垂直 LinearGradient |
| 法线厚度 | ~10px 细边，Dialog 外晕 0 | 内描边取 `APPEAR_SWEEP_WIDTH_DP` **28dp**，再叠 2.4× / 4.2× 外晕 |
| 切向核 | AxialCore 0.3 × length 0.4h ≈ **0.12h** | `flowSpan` 用 0.4×0.5=**0.20h** 且不裁切，整圈都吃到渐变 |
| 峰值 | intensity 0.2 | `APPEAR_SWEEP_PEAK_ALPHA` 0.88 × 四层加色 |

四层粗 stroke + 高 alpha + 不裁切的全路径 = 「一块白布从下往上推」，不是光线。

## 4. 对 YoUI L7 的改写（adapt）

1. **只画轮廓上的一条帽。** 用水平 clip 跟 `edgeY(progress)` 走，核高 = `APPEAR_FLOW_LENGTH_RATIO × APPEAR_FLOW_CORE_RATIO`。
2. **细边用 `TOUCH_LIGHT_EDGE_WIDTH_DP`。** 禁止把 28dp / 250 当 stroke。
3. **Dialog 外晕 = 0。** 内晕 `APPEAR_INNER_BLOOM_DP`，clip 在板内。
4. **峰值对齐可见能量，不是只抄 LightMax 0.4。** `LightMax = 0.2×2.0`，但着色器还乘 `MaxBloom = 0.2×19.3 ≈ 3.86`。Canvas 没有 SDF bloom 时，轮廓实线要用满白 ADD，否则在霜面上等于没扫光。
5. **不要抄 HDS 彩色双边流光当 Dialog 打开。** 那是另一条产品能力。

anti-pattern：加粗 stroke / 提高 alpha 来「更像官方」。

## 5. 为什么「改细」之后启动扫光完全看不见（2026-08-21 补读）

用户现象：Dialog 打开没有扫光，不是「还是太粗」。上一轮只改了几何（细帽 + 峰值 0.40 SRC_OVER），没有改合成。源码里扫光能被看见，靠的是加色叠在霜面上，不是半透明白描边盖上去。

### 5.1 官方合成是 OverlayNG 加色，不是 SRC_OVER

`graphic_graphics_effect` `GESDFEdgeLight::MakeImageMerger`：

```
return vec4(imageColor.rgb + composeImageColor.rgb, imageColor.a);
```

光的 RGB **加到** 霜面 RGB 上，板的 alpha 不动。`PlayFlowLight` 把 `RSNGSDFEdgeLightEffect` 挂在 Dialog 列节点的 `SetOverlayNGShader`（`rosen_render_context.cpp`），是节点上的 overlay 着色器，不是一张 SRC_OVER 的 sibling View。

着色器里亮度是 `mask × LightMaxIntensity × (细边 + bloom)`。Dialog 峰值 `intensity 0.2 × LightMax 2.0 = 0.4` **加色**。白霜上加 0.4 白会顶到高光；同样 0.4 做 SRC_OVER 盖在已接近白的玻璃上，对比度接近 0。

YoUI L6 触点光已经用 `PorterDuff.Mode.ADD`（对齐 Rosen 点光）。L7 弹出光在独立 `ImmersiveAppearOverlay` 上仍走默认 SRC_OVER。L7 拆到 sibling 是为了 EXPAND 时不重录 L0–L4 DisplayList，但拆走以后没有把 OverlayNG 的加色一起带走。

### 5.2 时钟：位置和强度是两条动画

`PlayFlowLight`（`dialog_pattern.cpp`）不是「progress 一个数同时管位置和亮度」：

| 动画 | 何时 | 写入 |
|------|------|------|
| 移动 | 0–568ms | BOTTOM→TOP，**intensity 0** |
| 高光 | delay 305ms，220ms | TOP，intensity **0.2** |
| 消失 | delay 443ms，221ms | intensity 0，thickness 0，然后 Reset |

可见段是「帽已经走到中上之后才亮，在顶边闪一下再灭」。YoUI 用 `4p(1-p)` 把强度绑在同一根 0→1 弹簧上：两端为 0，中间才亮。静止 progress=1 时 L7 本来就该是 0，所以截静帧证明不了扫光。再叠 SRC_OVER 细描边，中间那几帧也看不出来。

anti-pattern：把描边加粗、或给 sibling overlay 挂 hardware ADD 层来「做出效果」。官方可见能量来自 SDF × MaxBloom，不是铺满卡片。

adapt：

1. L7 绘制与 overlay 合成都走 **ADD**（对齐 `MergeImage` / OverlayNG）。sibling overlay 要用 layer paint 的 ADD，否则只在透明 buffer 里加，再 SRC_OVER 到父节点，加色还是丢了。
2. 几何保持细帽；Canvas 没有 SDF 把内部打成 0，所以只描左右轮廓上的一段实线。L7 必须画在霜面同一张 Canvas 上再 ADD（对齐 OverlayNG 同节点），不要给 sibling overlay 挂 hardware ADD 层——真机上会把整卡加出一条粗色带。
3. 静止 intensity=0 是对的。要验收必须在 **打开过程中** 截帧，不能等 progress=1。

## 阅读范围

- `dialog_pattern.cpp` `PlayFlowLight`、常量
- `rosen_render_context.cpp` `ParseEdgeLightPosition` / `UpdateEdgeLightFilter` / `UpdateEdgeLightFilterWithLightMask`
- `edgelight_property.h`
- `menu_pattern.cpp` `PlayLightAnimation`（对角 TOP_LEFT↔BOTTOM_RIGHT，同一套 EdgeLightParam）
- 本地文档：`沉浸光感.md` 空间动效；`hdsEffect.md` `EdgeFlowLightParam`；`自带背景的双边流光.md`
- YoUI：`ImmersiveAppearLight.java`、`ImmersiveAppearOverlay.java`、`ImmersiveDimens`
- `graphic_graphics_effect` `ge_sdf_edge_light.cpp` `MakeImageMerger`（`rgb + rgb`）
- `graphic_graphic_2d` `rs_overlay_ng_shader_drawable.cpp` OverlayNG 画在节点之上

未打开闭源 `libhdsmaterialimpl`；未并排录 HarmonyOS 真机 Dialog。HDS 双边流光无开源着色器。
