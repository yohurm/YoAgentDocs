---
id: rules.type.ui-kit.state-ownership
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 组件状态归谁、selected/enabled 写在哪、Impl 是否复制状态、默认/按压/禁用/空态时
related: [rules.type.ui-kit.layering, rules.type.ui-kit.lifecycle, rules.type.ui-kit.file-srp]
---

# 运行时状态所有权

分层（L0–L5）管文件边界；本篇管**运行时谁持有哪份数据**。两套必须对齐。生命周期阶段见 [lifecycle.md](lifecycle.md)。

## 所有权

| 数据 | 层 | 约束 |
|------|----|------|
| 内容 | L2 模型 | 唯一真源（title、items、icon）。不与配置重复存一份 |
| 样式与策略 | L0 token + L3 不可变配置 | 构建后不变；数组防御性复制 |
| 长期交互状态 | L3（Controller + 只读快照） | `selected` / `enabled` / `visible` / `value` 的唯一写入口 |
| 绘制瞬态 | L4 | 动画进度、触摸坐标、bitmap 缓存；不是业务真源 |
| 平台资源 | 见生命周期 | 监听、Handler、Popup、动画令牌 |

`XxxImpl` 只装配并把 L5 调用转成模型/配置/策略输入，**不保存第二份** enabled / selected / value / visible。

Android YoUI 把上表叫 MVCS+；桌面 `@yohu/ui` 用 model / policy / 视图文件表达同一所有权，不要再发明第三套。

## 必须

- **用户事件一条链：** View 收事件 → L3 提交 → 不可变快照 → L4 按快照绘制。外部回调从同一次提交派生，不从 View 字段再读一遍。
- **内容变更按稳定 ID。** 增删改不得按旧索引猜测新选中项；选择、展开、滚动位置跟 ID 走。
- **交互状态一等公民。** 规范有的默认 / 按压 / 禁用 / 选中 / 空态 / 加载都要能指到所有者：按压与 ripple 是 L1 瞬态（回收必须复位）；禁用与选中是 L3；空态是内容区契约，不是盖一层半透明。
- **禁用同时关掉输入与视觉。** 仍可滑、仍可点，只把透明度调低，算没做禁用。
- **状态色与动效走 token/配方。** 禁止在 item view 里 if-else 写死色值或第二套曲线。选中变化可走共享状态动效；首次 bind 不当作变化（见生命周期）。
- **无状态工厂不要假分层。** Press / Ripple 一类：Builder → 不可变配置 → 绘制物即可，不编造空 Model/Controller。

## 应当

- 每个 L3 状态机有：正常转换、边界归一化、模式迁移、destroy 后拒绝写入。
- 动态主题若出现，先定义可注入快照再整体迁移；禁止再复制一套常量。

## 反例

- Impl 与 Controller 各存一份 `selected`。
- Model 和 Config 都存 title。
- 按压缩放写在业务 State 里，回收后带给下一行。
- `removeItem(i)` 之后用 `i` 当选中索引。
- 只有默认能点；禁用态仍响应手势。
- 为凑齐类名保留未接入的 Model/Controller。
