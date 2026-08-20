---
id: experience.android-youi.yo-corner-l5-facade
type: experience
status: active
when_to_use: 审查 YoUI api/ 门面是否塞进实现，或重做 L1 共享能力时
---

# YoCorner：L5 门面不得持有实现

## 背景

Android-YoUI 增加 L1 圆角能力时，`api/common/YoCorner` 被写成带 Rosen `ScaleRadii`、Figma `budget`、`writePath` / `applyClip` 的工具类，并直接 `import widget.common.corner.internal.*`。这违反 ui-kit 的 L5 薄门面。

## 做了什么

自底向上拆开：

| 层 | 文件 | 职责 |
|----|------|------|
| L0 | `CornerDimens` | smoothing / capsule ε |
| L1 | `CornerGeometry` | fitted / capsule / 邻角分账 |
| L1 | `CornerPath` / `ContinuousCorner` / `CornerOutline` / `CornerClip` / `CornerDrawable` | 绘制 |
| L1 | `CornerImpl` | 装配，对标 `RippleImpl` |
| L5 | `YoCorner` | 接口：`Curve`、`Radii` 数据、静态转发 |

`Radii` 只保留公开数据与 `concentric`。拟合算法不出现在 `api/`。

## 可复用决策

- 组件 API 对标 `YoRipple`：接口 + `XxxImpl`，不是 `final class` 工具包。
- `api/` 只允许指向装配类，禁止 `internal`。
- 玻璃 / SDF 继续锁定 `CIRCULAR`。

## 举一反三（同仓扫描，2026-08-20）

| 模块 | 判定 | 处理 |
|------|------|------|
| `YoCorner` | `api/` 写算法 + import `internal` | 本轮已拆 |
| 其余 `YoButton` / `YoTabs` / `YoRipple` 等 | 接口 + `XxxImpl`，未 import `internal` | 合规 |
| `YoBlur.create` → `BackdropBackground.Builder` | 装配类命名不是 `*Impl`，但不是 `internal` | 记：宜收成 `BackdropImpl` |
| `YoMaterialSurface.create` | 门面里 `new MaterialSurface` + `instanceof BackdropBackground`，返回实现类型 | 记：工厂应迁入 `MaterialSurface`，返回 `YoMaterialSurface` |
| `YoImmersiveLight` | `final class`，但只转发 `ImmersiveLightBinding` | 可接受的薄门面 |
| `YoClickEffect` | `api/` 值对象，无绘制 | 可接受 |
| `api/animation/effect/*`（`YoFade`、`YoEasing`、`ShimmerEffect`） | 动效目录约定：配方编轨写在 `api/animation` | **不是** widget 越级；另案审查，本轮不拆 |

未在本轮改 `YoMaterialSurface` / 动效目录，避免借机重写无关模块。

## 是否应回写类型包

已回写：`public-api.md`、`layering.md`、`file-srp.md`、`coupling.md`、`content-region.md`、`checklists/yo-component.md`、`playbooks/yo-component.md`。
