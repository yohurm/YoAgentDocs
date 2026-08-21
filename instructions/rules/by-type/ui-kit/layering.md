---
id: rules.type.ui-kit.layering
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 新增或重做任一 Yo 组件时
related: [rules.type.ui-kit.file-srp, rules.type.ui-kit.lifecycle, rules.type.ui-kit.state-ownership]
---

# 组件分层

每个组件自下而上拆层。禁止在一个类/文件里同时承担多层。

```
L0  数值单源     token / 常量池（色、字号、圆角、间距、时长、控件尺寸）
L1  共享能力     涟漪、按压、光感、Presence、圆角几何、裁剪、图标…
L2  模型         纯数据与不变式，不碰 View / DOM
L3  策略         宽度、隐藏、选中、手势阈值等规则
L4  视图         铬层（边框/阴影/光感）与内容区（可裁剪的孩子）
L5  对外 API     YoXxx 门面：创建、配置、事件、destroy；无布局 / 绘制 / 释放顺序
```

## 落地对照

| 层 | Android YoUI（示意） | 桌面 `@yohu/ui`（示意） |
|----|----------------------|-------------------------|
| L0 | `core/constants` | `tokens/`（Primitive → Semantic → 组件只消费后两层） |
| L1 | `animation/`、公共 ripple / immersive / corner | `motion/`、交互配方 |
| L2 | `widget/.../internal/model` | `*-model.ts` |
| L3 | `internal/controller`、`config` | 独立 policy/binder，不写进 JSX 大函数 |
| L4 | `internal/view`、chrome | `Foo.tsx` 视图 + `Foo.css` |
| L5 | `api/.../YoXxx.java` | `index.ts` 导出的 `YoXxx` |

## 必须

- 先定层再写代码。新增能力问：它属于 L1 共享，还是该组件 L3 策略。按压缩放、涟漪、光感默认进 L1，不要复制进每个按钮。运行时谁持有状态、何时 attach/destroy，见 [state-ownership.md](state-ownership.md) 与 [lifecycle.md](lifecycle.md)。
- L4 必须能指出**内容区**边界（见 [content-region.md](content-region.md)）。铬层效果（阴影、描边、光感）不要和内容测量混在一个函数里。
- 圆角：半径 token 在 L0；拟合 / Path / Outline / Drawable 在 L1（`widget/common/corner`）。组件只调 L5 `YoCorner`，禁止私自 `addRoundRect` / `GradientDrawable.setCornerRadius`。玻璃 / SDF 锁定 `CIRCULAR`。
- 动效：组件只引用配方名 / 语义时长，不写裸 `ms` 或第二套曲线。
- 共享壳（页、面板、标题栏传送区）是 L1/L4 基础设施。模块分区走统一面板，禁止每个页面自己画阴影描边。

## 反例

- 在 `api/` 里写测量、手势、绘制，或把 L1 几何类嵌进 `YoXxx`。
- Dialog 为每种预设复制一整套实现类，而不是 L3 配置 + 同一套 L4。
- 列表项把 ripple、揭示区、文字、业务点击全写在一个 View 里且无法指出内容区。
