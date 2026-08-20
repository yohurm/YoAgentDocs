---
id: research.openharmony-graphic_graphic_2d-point-light
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [cpp]
  frameworks: [rosen, arkui]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: openharmony/graphic_graphic_2d
  url: https://github.com/openharmony/graphic_graphic_2d
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--graphic_graphic_2d"
studied_at: 2026-08-20
related:
  - research.harmony-hdstabs-point-light
  - research.openharmony-arkui_ace_engine-immersive
  - research.harmony-immersive-light-layers
---

# openharmony/graphic_graphic_2d（点光源）

浅克隆另读：`E:\GithubGallery\_tmp_research\harmonyos-immersive-light\graphic_graphic_2d`（算法同族，Temp 仓更新）。HdsTabs 实现不在本仓。

## 入选理由

点光源的半径公式、光源/受光配对、IlluminatedType 分轨着色器都在 Rosen，不在 HDS。YoTabs / ImmersiveTouchLight 要抄的是这里的几何，不是组件层 API 名。

## 项目是什么

OpenHarmony 2D 图形子系统。点光源是 **全局配对器 + 受光节点上的 RuntimeShader**，不是给组件贴一张光斑贴图。

## 架构

```text
光源节点 RSLightSource { position(x,y,z), intensity, color }
        │  radius = CalculateLightRadius(z)
        ▼
RSPointLightManager（按 logicalDisplay 分实例）
  RegisterLightSource / RegisterIlluminated
  CheckIlluminated：同 InstanceRoot + 圆 vs AABB
  相对坐标：光源局部 → abs → 受光局部；w 存 radius
        ▼
受光节点 RSIlluminated { type, bloom, borderWidth, lightSourcesAndPosMap }
        ▼
RSCoverageNGShaderDrawable::DrawLight
  FEATHERING_BORDER → SDF 圆角边高光
  NORMAL_BORDER_CONTENT → 法线凸起 + 二次衰减
  其余 → Phong
  再按 type 画 border 描边 / content 填充 / 两者
```

配对约束：最多 12 盏灯；只照同一 `InstanceRoot`；脏光源或脏受光才重算。光源 XY 可在受光矩形外，只要圆心到 AABB 距离 ≤ radius 就会入表。绘制始终 `DrawRoundRect(contentRRect_/borderRRect_)`，光不会画到节点形状外。

### positionZ → 半径

单位：ArkUI 把 `positionZ` 从 vp 转 px 再交给 Rosen；公式吃的是 **像素 z**。

```text
cos = (1/255)^(1/8) ≈ 0.5002
tan = sqrt(1 - cos²) / cos ≈ 1.731   （≈ √3）
radius = z * tan                     （z≤0 → 0）
```

| z（vp，密度 1 时等同 px） | 半径 |
|---------------------------|------|
| 80（沉浸材质触点光常量） | ≈ 138 |
| 150（hdsEffect 示例 height） | ≈ 260 |

NORMAL 着色器衰减（`attenuationCoeff = 0.3`）：

```text
nd = length(lightVec) / radius
att = (1 - smoothstep(0.8, 1.0, nd)) / (1 + 0.3 * nd²)
```

CONTENT 填充再乘 0.3（NORMAL_BORDER_CONTENT 乘 0.5）。BORDER 用 Pen 描 `borderRRect_`，默认沉浸路径 `illuminatedBorderWidth = 0.5vp`。

### IlluminatedType（Rosen 枚举）

| 值 | 名 | 绘制 |
|----|----|------|
| 0 | NONE | 不注册受光 |
| 1 | BORDER | 只描边 |
| 2 | CONTENT | 只填内容 |
| 3 | BORDER_CONTENT | 边 + 内容（**Tabs 触点光用这个**） |
| 4–5 | BLOOM_BORDER / BLOOM_BORDER_CONTENT | 开源 `DrawLightByIlluminatedType` **没有分支**，等于不画 |
| 6–8 | BLEND_* | 同 BORDER/CONTENT，额外 OVERLAY + SaveLayer |
| 9 | FEATHERING_BORDER | 羽化边；与 SDF 形状互斥 |
| 10 | NORMAL_BORDER_CONTENT | 凸起法线 + 半径衰减 |

HDS `PointLightIlluminatedType.DEFAULT_FEATHERING_BORDER = 20`，**不是** Rosen 的 9。映射在闭源 `hdsEffect`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 半径 `≈ 1.731 × z`、z=80vp → ~138vp | reuse-pattern | YoTabs 触点光用同一换算，不要拍 200px 光斑 |
| 光源可出界、绘制 clip 在圆角矩形 | reuse-pattern | 跟手出界；不要 `clipPath` 切掉描边 |
| 同节点既是灯又是受光 | reuse-pattern | Tabs 胶囊自照明，不必另建隐形灯 View |
| 二次衰减 + 0.8–1.0 smoothstep 截断 | adapt | AGSL 复刻，不必移植 RuntimeShader 全文 |
| 12 灯全局配对器 | anti-pattern | Android 单栏单指不要上 RSPointLightManager |
| BLOOM_* / hdsEffect=20 | lesson-only | Tabs 不用；不要按枚举名猜发光 |
| SDF EdgeLight overlay | anti-pattern | Sheet/Menu/`edgeLight` 另一条管线，不是点光源 |

## 架构设计经验

1. **灯是场景对象，斑是受光节点的着色。** 移动的是灯坐标，不是一张跟手 Image。
2. **半径由高度导出，强度是另一维。** 沉浸 Tabs 用 z=80、intensity=3（超过文档建议 0–1）。
3. **出界跟手 + 形状内着色** 才能做出「沿边缘照亮」。硬 clip 内沿会切成卡片。

## 与当前工作

**能直接用：** 半径公式、BORDER_CONTENT、0.5vp 边宽、二次衰减、自照明、出界 XY。

**必须改写：** Rosen RuntimeShader → Android AGSL / `ImmersiveTouchLight`；vp→px 按密度；配对器改为栏内单光源。

**不要用：** 把光斑画在 overlay 且不 clip 到胶囊；拷 `RSNGSDFEdgeLightEffect` 当 Tabs 指尖光；给 LOW 档上点光源（ArkUI SMOOTH 会丢掉 `lightEffectOptions`）。

## 阅读范围

实际读过：

- `rosen/modules/render_service_base/include/property/rs_properties_def.h`（`CalculateLightRadius`、`IlluminatedType`）
- `rosen/modules/render_service_base/src/property/rs_point_light_manager.cpp`
- `rosen/modules/render_service_base/src/drawable/rs_coverage_ng_shader_drawable.cpp`
- 对照：`arkui_ace_engine` `component_material_interaction.cpp`、`OnLightPositionUpdate`

未读：闭源 `libhdsmaterialimpl.z.so`、完整 FrostedGlass 着色器、HdsTabs HSP。
