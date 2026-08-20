---
id: rules.type.ui-kit.file-srp
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 组织组件源文件时
related: [rules.type.ui-kit.layering]
---

# 文件单一职责

不要把一个组件的模型、策略、视图、样式、测试堆进同一个文件。

## 必须

- **一个文件一件事。** 超过一层职责就拆文件（或 Android 上拆类型）。文件名反映职责：`model` / `policy` / `view` / `chrome`，而不是 `Utils`、`Helper`、`Impl2`。
- **样式与逻辑分离。** 桌面：`YoFoo.tsx` 不管色值字面量，走 CSS 变量；`YoFoo.css` 不管状态机。Android：尺寸进常量池，不在 View 里写魔法 dp（除调用常量）。
- **测试单独文件。** 不把断言写进产品类。
- **公开门面很薄。** `YoFoo` / `api/` 只转发；实现放 `widget/.../XxxImpl` + `internal`，或非导出模块。`api/` 不得 `import …internal`。
- **预设不是新文件树。** 多种外观用配置/枚举，不为「警告对话框」「列表对话框」各拷一套实现。

## 应当

- 单文件超过「一个层的完整实现」就拆。桌面若 `Foo.tsx` 同时含滑动算法、测量、渲染，应抽出 `foo-model.ts` / `foo-gesture.ts`。
- 同组件的内部类型放在该组件目录，不泄漏到别的组件包。

## 反例

- `api/feedback/dialog/` 下堆补丁类。
- `api/common/YoCorner.java` 同时持有 Radii 拟合、Path 转发，并直接引用四个 `internal` 实现类。
- 一个 `DialogContent.java` 同时懂按钮行、列表、输入、动画。
- `@yohu/ui` 里一个 tsx 包含 token 数字、手势、布局、业务回调拼装。
