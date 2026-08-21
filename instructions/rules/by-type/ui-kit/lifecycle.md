---
id: rules.type.ui-kit.lifecycle
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 新增或重做 Yo 组件、attach/detach/destroy、资源泄漏、列表回收、浮层显隐时
related: [rules.type.ui-kit.layering, rules.type.ui-kit.state-ownership, rules.type.ui-kit.coupling]
---

# 组件生命周期

实例从创建到销毁必须分阶段、有所有者。禁止把 Activity / Fragment / 页面生命周期写进控件，禁止用第二套动词或兼容层「补一次 bind」。

运行时数据归谁见 [state-ownership.md](state-ownership.md)。

## 阶段（必须能指到层）

| 阶段 | 谁做 | 做什么 |
|------|------|--------|
| create | L5 门面 + `XxxImpl` 装配 | 建对象，不注册窗口/滚动/广播 |
| bind / 配置 | L3 → L4 | 写入模型与视图；可重复调用 |
| attach / 入树 | L4 | 订阅窗口与 L1 能力；重复 attach 先 detach |
| present | L1 Presence / Swap | 谁还在树上；出场结束再卸节点 |
| update / 再 bind | L3 | 区分首次与后续；回收项必须复位 |
| detach / 出树 | L4 | 只取消窗口绑定的工作，允许再 attach |
| destroy | Impl 统一入口 | 永久释放；之后公开操作快速失败 |

列表项 ViewHolder：`bind` 是回收入口；`destroy` 只解除该行监听与按压/动效，不是整表销毁。

## 必须

- **一套动词。** 实例生命周期用 `attach` / `detach` / `destroy`。不要平行再发明 `close` / `release` / `dispose` 当销毁。浮层对宿主的呈现回调另用 `onWillAppear` → `onDidAppear` → `onWillDisappear` → `onDidDisappear`（鸿蒙对齐），不算第二套销毁 API。
- **资源有主、释放对称。** 监听、Handler/`post`、动画令牌、Popup/Dialog、模糊缓冲、滚动源，谁 `own` 谁释放。注册逆序释放；`destroy` 幂等。attach 申请的，detach 必须还；永久资源只在 `destroy`。
- **destroy 后即死。** 公开 API `requireAlive`；晚到回调靠代际或 destroyed 检查丢弃，不操作新 View。销毁过程不向宿主发业务回调。
- **detach ≠ destroy。** `onDetachedFromWindow` / unmount 只停窗口工作，不得清空重挂载仍需要的监听与 Controller。Activity 销毁、浮层 Host 移除必须接到 `destroy` 或等价提交，不能只靠 GONE。
- **再 bind 无旧态。** 列表回收、重复 `show` / `setItems` 必须清按压、ripple、选中残留与旧动画。首次 `bind` 不是「状态变化」（例如选中 bounce 只在 id 真正改变时）。
- **进出场与卸载分权。** 布尔 Presence 只服务 Dialog/菜单/模块；内容换牌用 keyed Swap。控件禁止内嵌 Presence 状态机。关闭：出场（含透明度）结束后再卸节点，不要把半透明缩放停在屏幕上。
- **外部源成对。** 滚动、窗口焦点、配置变更、广播：`bind` 必有 `unbind`，落在同一对象。禁止 `setOnScrollChangeListener` 覆盖调用方已有监听。
- **动效跟生命周期走。** detach / destroy / 范围关闭时取消 L1 配方并释放 lease；禁止 `View.animate()` 与脱离生命周期的后台动画。页面进后台：有限反馈可跑完，装饰与无限动画 pause 或 cancel。
- **L5 不写生命周期算法。** 门面只转发 `destroy` / 呈现回调；观察者与释放顺序在 Impl。

## 应当

- 能画出该组件 create → destroy 的数据链路（谁注册、谁释放、晚到回调怎么丢）。
- Reduce Motion 时跳过进出场视觉，仍走完整 bind / unbind / destroy。
- 平台完成事件只有一个来源（例如 Dialog 只以 `onDismiss` 提交完成）。

## 反例

- `postDelayed` 不保存 Runnable，destroy 无法移除。
- 重复 `show` / `setItems` 泄漏内部动画与 Recycler 绑定。
- Dialog 无 View detach / Activity destroy 观察，销毁后仍 `isShowing`。
- 首次 `bind` 就弹选中 bounce；回收后仍亮着上一项 ripple。
- 用 GONE 冒充 detach，动画还在跑。
- 在 `api/YoXxx` 里写 `onAttachedToWindow` 或释放顺序。
- Toast/Sheet 不用 `destroy`，另留一套 `close` 且不释放。
