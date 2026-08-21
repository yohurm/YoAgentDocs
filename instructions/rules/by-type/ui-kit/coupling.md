---
id: rules.type.ui-kit.coupling
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 组件之间、组件与动画/宿主之间的依赖
related: [rules.type.ui-kit.layering, rules.type.ui-kit.state-ownership, rule.common.architecture]
---

# 禁止耦合

高内聚、低耦合。依赖只指向更稳定、更低的层。

## 组件之间

- A 的 `internal` / 非导出文件，B 不得引用。B 只能用 B 的 L5 或 L1 共享能力。
- 列表不内嵌「预览面板」实现；组合发生在页面/模块，不发生在 List 内部。
- 揭示/滑动操作不是 Button 的一种 variant，除非规范把它们定义成同一控件。该能力放 List 策略或独立 action 层。

## 能力与展示

- 动画系统不依赖 Loading 示例页；Loading 只消费动画能力。
- Ripple / 按压 / 光感 / 圆角是 L1。控件只走该能力的 L5（`YoRipple`、`YoCorner`），不把实现抄进去，也不为了「看起来裁对」去改 ripple 尺寸冒充内容区（见 [content-region.md](content-region.md)）。
- L5 只依赖同能力 `XxxImpl`。禁止 L5 点名 L1 `internal`（例如 `YoCorner` → `CornerPath`）。
- 平台资源（监听、动画令牌、Popup）只由该组件生命周期所有者释放，B 不得去 detach A 的内部句柄。见 [lifecycle.md](lifecycle.md)。
- 沉浸光感：材质分层（霜面、透镜等）在光感模块内部；控件只选角色/强度。不要在 Tabs、Dialog 里各写一套着色器参数。

## 宿主与模块

- 应用模块禁止复制组件色值、圆角、阴影、页垫。统一走 token 与 `YoPage` / `YoPanel`（或该仓等价壳）。
- 缺组件就补库，禁止模块私有第二套按钮/页签/面板。

## 数据源

- 同一视觉决策只有一个来源（例如选中底、页垫、面板铬）。发现「只有某一页有阴影」时，把铬收进共享面板，而不是给那一页打补丁。

## 反例

- Dialog 直接依赖 List 的内部 ripple 实现。
- `api/common/YoCorner` 直接 `import widget.common.corner.internal.*`。
- 文件预览写在文件列表组件内部状态机里。
- 动画实验室标注残留在正式 YoLoading API 上。
