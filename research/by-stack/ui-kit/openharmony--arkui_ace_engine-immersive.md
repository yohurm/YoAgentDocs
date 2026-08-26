---
id: research.openharmony-arkui_ace_engine-immersive
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++, ArkTS]
  frameworks: [arkui, rosen]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: gitcode
  repo: openharmony/arkui_ace_engine
  url: https://gitcode.com/openharmony/arkui_ace_engine
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.harmony-immersive-light-layers
---

# openharmony/arkui_ace_engine（沉浸光感）

## 入选理由

ArkUI 开源引擎是**唯一能看到 ImmersiveMaterial 如何变成像素**的公开代码：`materialFilter` LUT、overlay 拆分、阴影 token、算力档短路、闭源 so 挂钩。HDS / `libhdsmaterialimpl.z.so` 本身闭源，必须靠本仓 + 官方 API 对照。

## 项目是什么

OpenHarmony 方舟 UI 引擎。沉浸光感（API 26）在这里是一套 **材质对象 → 滤镜/着色器 → Rosen 节点属性** 的编译器，不是组件自己画玻璃。

对照样本（算法仍在系统，应用只传对象）：

- `%TEMP%/YoAgentResearch/HarmonyOS_Samples--Spatialization`（官方最佳实践案例）
- `%TEMP%/YoAgentResearch/HarmonyOS_Samples--HarmonyOSComponentUXExamples`
- `%TEMP%/YoAgentResearch/SoraLuna--ui-design-kit-hds-immersive-navigation-demo`

本地对照副本：`E:\GithubGallery\_tmp_research\harmonyos-immersive-light`、`harmonyos-bottom-tabs`。

## 架构

### 编译入口

```text
systemMaterial(ImmersiveMaterial)
  → ImmersiveOptions（style / materialColor / colorInvert / applyShadow / interactive / lightEffect）
  → MaterialUtils::GetImmersiveMaterialConfig
       HIGH/MID: key = {level, style, transparency, colorMode} + lightEffect
       LOW(SMOOTH): 早退，只留 materialColor / applyShadow / interactive；不建 FrostedGlass
  → 优先 dlopen system/lib64/libhdsmaterialimpl.z.so
       CreateUiMaterialFilter / CreateUiMaterialShaderECSub / Overlay / SetMaterial
  → 失败则开源 LUT：UiMaterialFilterCreator + RosenEffectConverter
```

`UiMaterialLevel`：`EXQUISITE=HIGH`、`GENTLE=MID`、`SMOOTH=LOW`（默认就是 SMOOTH）。用户「强/均衡/弱」是另一轴 `UiMaterialTransparency`（THIN/NORMAL/THICK），不是算力档。

开源 LUT 里 **EXQUISITE 与 GENTLE 指向同一套 `Gentle_*` 参数**。HIGH 多出来的折射/粒子在闭源 so，开源仓给不出。

Sheet 特例：`LowerGearLevel` 把 `SHEET_PAGE` 从 GENTLE 强制降到 SMOOTH。

### FrostedGlass 内层（materialFilter）

`FrostedGlassParam` 是一张扁平参数表，Rosen `RSNGFrostedGlassFilter` / `RSNGFrostedGlassEffect` 一次吃完。开源能命名的子层：

| 字段 | 层 | 说明 |
|------|----|------|
| `blurParams` `{radius, K}` | 霜模糊 | vp 半径 × 降采样 K |
| `refractParams` + `envLightParams[0]` | 折射/透镜 | EC 路径把 `envLightParams[0]` 当 `refractOutPx` |
| `bgRates/KBS/Pos/Neg` + `bgAlpha` | 捕获后色调 | 对采样背景做增益，不是前景 tint |
| `materialColor` tag | 赋色 | alpha>0 时清掉 emboss；不透明会盖住滤镜 |
| `weightsEmboss` | 浮雕高光 | 可拆到 overlay |
| `weightsEdl` + `edLight*` | **静止边缘光** | 角度默认 75°/120°，方向 `(0,-1)`；不是盒描边 |
| `sd*` | 滤镜内阴影/高光辅助 | 与节点 `shadow` 不是同一层 |
| `samplingScale` | 捕获扩张 | 配合折射外扩 |

HIGH/MID 生效后：节点 `backgroundColor` **恢复透明**，`borderWidth` **恢复无边框**。静止高光只活在滤镜里。

### Overlay 拆分

`needSplitOverlayShader=true` 时：

- **Base shader**（`SetMaterialShader`）：emboss/edl 权重清零，保留模糊/折射/bg tint。
- **Overlay shader**（`appendOverlayShader_[0]`）：emboss+edl 回来，`bgAlpha=0`，叠在材质之上。
- **触点光**（`lightEffect`）另走 `RSNGSDFEdgeLightEffect` → `appendOverlayShader_[1]` → `SetOverlayNGShader`。默认色白；SpreadFactor 47.8，`LightMaxIntensity=2.0×intensity`。

交互形变 `interactive` 是 **前景滤镜** `RSNGDistortionCollapseFilter`（四角塌陷 + barrel），不是 materialFilter 内层。

### 阴影（节点层，滤镜外）

`MaterialUtils::GetImmersiveShadow`：半径 **26vp**、Y 偏移 **8vp**、色 `#14050505`。`applyShadow=true`（默认）时优先于通用 `shadow`。投影必须允许溢出 clip。

### LOW（SMOOTH）为什么没有滤镜高光

开源路径在 `GetImmersiveMaterialConfig` 遇到 SMOOTH **直接 return**，不查 `MATERIAL_PARAM_MAP`，不创建 FrostedGlass。官方 API：低算力只改 `backgroundColor` / `borderColor` / `borderWidth` / `shadow`。1vp 描边与实色填充的具体色值在闭源 `SetMaterial`。设计上用边框代替滤镜 edLight，所以 LOW 看起来是「色块 + 发丝边 + 阴影」，不是玻璃。

`style` 对 LOW **不生效**（官方 ImmersiveOptions 写明仅高/中算力）。`colorInvert` 在 SMOOTH 上强制 false。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 层序：捕获→折射→霜→tint→edLight 在滤镜内；阴影/触点光/形变在滤镜外 | reuse-pattern | YoUI L0–L7 应对齐这条所有权，不要把 rim 画成 border |
| LUT `blurParams` / `edLightParams` `{0.62,0.92}` 与 `{0.83,0.92}` | adapt | 已进 YoUI ImmersiveLut；禁止再发明一圈白描边 |
| 阴影 26/8/`#14050505` | reuse-pattern | 所有算力档同一 token |
| HIGH/MID 清边框 | reuse-pattern | 滤镜高光替代描边 |
| LOW 早退、无 FrostedGlass | reuse-pattern | fill+1vp border+shadow，关 L1/L3 |
| EXQUISITE 开源表 = GENTLE | lesson-only | 不要假装开源 HIGH 比 MID 更「真折射」 |
| `libhdsmaterialimpl.z.so` | anti-pattern | 禁止拷 so、禁止 JNI 调华为符号 |
| HDS 只传 `systemMaterialEffect` 对象 | lesson-only | 见主题笔记；组件不重写滤镜 |

## 架构设计经验

1. **材质是对象，算法在系统。** 应用构造 `ImmersiveMaterial` 或 HDS 的 `{materialType, materialLevel}`，引擎按设备档 + 用户透明度编译。业务组件不得再叠一层自绘毛玻璃。
2. **滤镜内 vs 节点外必须切开。** 静止光学（折射、霜、tint、edLight）进 `materialFilter`；阴影、触点 SDF 光、按下 barrel 是 overlay / 前景滤镜。混在一个 Canvas stroke 里会把 rim 画成发丝线。
3. **算力档是短路，不是调参。** LOW 不是「半径更小的玻璃」，是整条滤镜管线不建。
4. **闭源挂钩是一等公民。** 开源仓先 `dlopen` 再 fallback LUT。Android 移植只能吃 LUT + 自研近似，不能声称像素对齐。

## 与当前工作

**能直接用：** 层所有权、LUT 半径/K、edLight 方向与角度、阴影 token、HIGH/MID 无边框、LOW 无滤镜。

**必须改写：** Rosen 滤镜 → Android `RenderEffect` / AGSL；闭源 SDF 触点光 → `ImmersiveTouchLight`；HDS 组件接线见主题笔记。

**不要用：** `libhdsmaterialimpl.z.so`、把 `HdsTabs.lightColor` 当静止 rim、给 VEIL/FLUSH 加材质阴影、在 HIGH/MID 画 1vp 盒描边。

## 阅读范围

实际读过：

- `frameworks/core/components/common/properties/ui_material.{h,cpp}`
- `frameworks/core/components_ng/render/adapter/ui_material_filter_creator.cpp`
- `frameworks/core/components_ng/render/adapter/rosen_effect_converter.{h,cpp}`
- `frameworks/core/components_ng/render/adapter/rosen_render_context.cpp`（overlay shader、SDF edge light、distortion）
- `interfaces/inner_api/ace_kit/include/ui/properties/ui_material{,_structs,_enums}.h`
- 官方：`arkts-apis-uimaterial`、`arkts-immersive-light-sense`、FAQ、`native_material.h`、设计指南《沉浸光感》
- 样本：Spatialization `MaterialUtil.ets`；UXExamples `HdsUtil.ets`

未读：完整 Rosen 霜玻璃着色器实现、HDS 组件闭源实现、`SetMaterial` 内 LOW 填色表。
