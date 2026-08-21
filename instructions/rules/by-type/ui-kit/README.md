---
id: rules.type.ui-kit
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 自研组件库（YoUI、@yohu/ui 及同等 Yo* 组件层）
related: [rules.type.ui-kit.layering, rules.type.ui-kit.file-srp, rules.type.ui-kit.lifecycle, rules.type.ui-kit.state-ownership, rules.type.ui-kit.coupling]
---

# 组件库类型包

来源：YoUI、YoADBTools（`@yohu/ui`）会话与落地结构的共性，不是某一仓库的文件清单。

页面业务、模块编排不放组件库。库只提供：token、可复用交互能力、按层拆开的控件、薄的对外 API。

## Agent 必读

做或改 Yo 组件时 **MANDATORY READ**：

- [layering.md](layering.md) 每个组件的层
- [file-srp.md](file-srp.md) 文件单一职责
- [lifecycle.md](lifecycle.md) 创建 / attach / detach / destroy 与资源释放
- [state-ownership.md](state-ownership.md) 内容、配置、交互状态、绘制瞬态归谁
- [coupling.md](coupling.md) 禁止耦合
- [content-region.md](content-region.md) 内容区与裁剪
- [public-api.md](public-api.md) 对外 API

## 默认边界

- 公开组件名 `Yo` 前缀；样式命名空间可与组件名不同（如 `yohu-*`），但必须全库一致。
- 参考鸿蒙 / iOS / 开源时内化成本库 API，不在对外名字里留「某系统拷贝」。
- 组件库不承载页面级状态机（Android ViewModel、桌面模块 store、鸿蒙页面状态层都不进库）。
- 宿主与模块**只用**库组件与 token；缺能力就在库里自底向上补，禁止在模块里再做一套外观。

## 质量门禁

- 每个改动的组件覆盖默认 / 按压 / 禁用 / 关键空态（示例页或宿主）。
- 能指出 attach / detach / destroy 与资源对称释放；destroy 幂等；再 bind / 回收无旧态。
- 平台验证走叠加的平台包。
- 对照官方时核对视觉与交互状态，不是「能编译」。

## 不要默认引入

- 第二套 UI 框架、第二套主题、双轨组件、模块内私有 token 数字、第二套生命周期动词。
