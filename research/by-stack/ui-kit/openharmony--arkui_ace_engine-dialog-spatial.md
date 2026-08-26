---
id: research.openharmony-arkui_ace_engine-dialog-spatial
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++]
  frameworks: [arkui, rosen, harmonyos]
also_relevant: []
utilization: [adapt, reuse-pattern, anti-pattern, lesson-only]
source:
  platform: gitcode
  repo: openharmony/arkui_ace_engine
  url: https://gitcode.com/openharmony/arkui_ace_engine
  cloned_to: "%TEMP%/YoAgentResearch/openharmony--arkui_ace_engine"
studied_at: 2026-08-20
related:
  - research.openharmony-arkui_ace_engine-immersive
  - research.harmony-immersive-light-layers
  - research.harmony-sdf-edge-light
---

# openharmony/arkui_ace_engine（Dialog 空间弹出 + 流光）

## 入选理由

API 26 沉浸光感把「材质」和「空间动效」绑在一起。官方文档只写「弹出和消失过程会自动附带形变、流光」；真正的时间轴、锚点、是否回放，只在 ArkUI Dialog 引擎里。YoUI 要把扫光接到 `YoDialog`，并让关闭按打开回放，必须读这份源码，而不是猜 360° SweepGradient。

对照样本：本机鸿蒙文档 `arkts-immersive-light-sense`、API 26 行为变更「Dialog 默认开启沉浸式系统材质」、已克隆的 `graphic_graphics_effect`（`EdgeLight` / `ContourDiagonalFlowLight` 着色器，Dialog 本身不直接调）。

## 项目是什么

方舟 UI 引擎。Dialog 在 `DialogPattern` / `DialogInnerManager` 里编译两套进场：

1. **旧进场**（无沉浸形变）：蒙层 opacity + 内容 `ScaleAnimation(theme.scaleStart → scaleEnd)`。
2. **API 26 光感进场**（有 `systemMaterial` 且算力/强度允许）：跳过旧 scale，layout 后跑 `PlayDistortion` + `PlayFlowLight`。

## 架构

```text
OpenDialogAnimationInner
  NeedDistortion?
    yes → 不跑旧 scale；等 OnDirtyLayoutWrapperSwap
          → PlayDistortion（形变 + 从中下散开）
          → PlayFlowLight（底→顶边缘光）
    no  → OpacityAnimation + ScaleAnimation(theme)

CloseDialogAnimation
  永远是旧路径：opacity SHARP + ScaleAnimation(end → theme.scaleStart)
  不调用 PlayDistortion / PlayFlowLight 的逆过程
```

档位（`DialogManager`）：

| 效果 | HIGH `EXQUISITE` | MID `GENTLE` | LOW `SMOOTH` |
|------|------------------|--------------|--------------|
| 形变 `NeedDistortion` | 强/均衡（THIN/NORMAL） | **关** | 关 |
| 流光 `NeedEdgeLight` | 强/均衡 | 仅强（`GENTLE_THIN`） | 关 |

弱档（THICK transparency）两套都关。自定义 `transition` / `customStyle` / 无材质也关。

### 从中下往外散开（`PlayDistortion`）

常量（`dialog_pattern.cpp`）：

| 符号 | 值 | 作用 |
|------|-----|------|
| `INITIAL_ZOOM_FACTOR` | **0.2** | `ScaleAnimation(0.2 → 1)`，默认 pivot = 几何中心 |
| `TRANSLATEY_RATIO` | **0.5** | 起始 `translateY = height × 0.5`，弹到 0 |
| 弹簧 | `interpolatingSpring(0, 1, 322, 27)` | 质量 1、刚度 322、阻尼 27 |

中心缩放 0.2 再下移半高：小卡片的视觉中心落在最终矩形的下沿中点附近，看起来就是从**中下往外散开**。不是把 pivot 设成 `center bottom`。

形变是 Rosen `DistortionParam` 前景滤镜：底角先内收（`lb=0.1, rb=0.9`），再回矩形，再加一点桶形 `{0.1, 0.1, -0.1, 0}`，delay 120ms 回到单位矩形。这是闭源/引擎滤镜，不是 Canvas stroke。

### 弹出扫光（`PlayFlowLight`）

不是绕轮廓转一圈。是 **EdgeLight 从底边挪到顶边**：

| 阶段 | 时长 / 延迟 | 内容 |
|------|-------------|------|
| 移动 | 568ms，cubic(0.20, 0, 0.83, 0.83) | `BOTTOM` → `TOP`，intensity 0 |
| 高光 | delay 305ms，dur 220ms，LINEAR | TOP，intensity **0.2** |
| 消失 | delay 443ms，dur 221ms，LINEAR | intensity 0，然后 `ResetEdgeLight` |

`length = height × 0.4`，`thickness = 250`，色白。注释里写的对角「左上→右下」是过期注释，代码是底→顶。

着色器侧在 `graphic_graphics_effect` 的 `EdgeLight` / `ContourDiagonalFlowLight`；Dialog 只喂 `EdgeLightParam`。

### 关闭为什么看起来会「停在最后一帧」

`CloseDialogAnimation`（`dialog_inner_manager.cpp` 919–978）不是回放 EXPAND，而是：

1. `OpacityAnimation(opacityEnd → opacityStart)`，`opacityStart_` 默认 **0.0**（`dialog_theme.cpp`）
2. `ScaleAnimation(scaleEnd → scaleStart)`
3. `FillMode::FORWARDS`：结束值一直保持到节点删掉
4. `OnFinish` → `PostDialogFinishEvent` 才卸节点
5. `FinishCallbackType::REMOVED`

最后一帧是**全透明**，不是不透明的缩小卡片。YoUI EXPAND `contentAlpha()` 恒为 1，收回落到 scale 0.2 时仍然不透明；若弹簧尾帧再 `setVisibility(VISIBLE)` 或 `restore()` 把缩放弹回 1，就会钉在最后一帧。

### 打开何时才启动画

`NeedDistortion()` 为真时，`OpenDialogAnimationInner` **直接 return**，不跑旧 scale。真正的 `PlayDistortion` / `PlayFlowLight` 挂在 `OnDirtyLayoutWrapperSwap` 的 `AddAfterLayoutTask`——**layout 之后**，材质已经是系统 `systemMaterial`，不是「霜面第一帧画完再开钟」。


## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 打开 = 中心 scale 0.2→1 + translateY 0.5h→0 | adapt | 同一套 visibility 0→1 映射，关闭 1→0 即回放 |
| 流光 = 底边→顶边的边缘光，不是 360° sweep | adapt | 改 `ImmersiveAppearLight` 几何 |
| 弹簧 stiffness 322 / damping 27 | adapt | 换成 perceptual `response≈0.35s, ζ≈0.75` |
| 档位：形变仅 HIGH；流光 HIGH 或 MID+强 | reuse-pattern | 已有 `TierResolver.allowsAppearLight` |
| 关闭走旧 scale、不回放 | anti-pattern | YoUI 不要抄这条 |
| Rosen `DistortionCollapse` 四角塌陷 | lesson-only | Android 无等价滤镜；本轮用 scale+translate 表达「散开」，不假装有桶形 |

## 架构设计经验

1. **空间弹出和材质 overlay 必须分通道。** 散开是 `ModalScene` 的 presentation 映射（跟自研 clock / visibility）。流光是 L7 overlay，可以平行于 transform，但 progress 必须能反向，关闭才能回放。
2. **不要用 CENTER 的 0.86 缩放冒充 API 26 Dialog。** Sheet 居中仍是 iOS 0.86；Dialog 光感是 0.2 + 半高平移。混在一个 `CENTER` 分支会弄坏 Sheet。
3. **官方消失路径不是契约。** 打开语言一旦定了，关闭应对称；引擎没做不能当「所以我们也不做」。
4. **360° SweepGradient 是错模型。** 能量应从底边亮帽走到顶边，而不是绕几何中心扫角。

## 与当前工作

**能直接用：** 0.2 缩放、0.5 高度平移、底→顶流光、档位门、弹簧参数换算。

**必须改写：** YoUI `ImmersiveAppearLight` 的旋转 sweep；`ModalScene.Presentation.CENTER` 不能承担 Dialog 进场；`prepareAppear` 只向前弹到 1、关闭不回放。

**不要用：** 拷贝 Rosen DistortionParam / `libhdsmaterialimpl`；把 250 thickness 当 Canvas stroke；Sheet 居中改成 0.2 缩放。

## 阅读范围

实际读过：

- `frameworks/core/components_ng/pattern/dialog/dialog_pattern.cpp`（`PlayDistortion` / `PlayFlowLight` / `NeedDistortion` / `NeedEdgeLight`）
- `frameworks/core/components_ng/pattern/dialog/dialog_inner_manager.cpp`（`OpenDialogAnimationInner` 短路、`CloseDialogAnimation`）
- `frameworks/core/components_ng/pattern/overlay/dialog_manager.cpp`（档位）
- `frameworks/core/components_ng/render/adapter/rosen_render_context.cpp`（`ScaleAnimation`、`ParseEdgeLightPosition`）
- 官方：`arkts-immersive-light-sense`「空间动效」；API 26 UX 变更「Dialog 默认沉浸材质」

未读：Rosen EdgeLight 着色器全文、HDS AdvancedDialog 闭源实现、真机 API 26 并排录屏。
