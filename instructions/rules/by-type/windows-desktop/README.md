---
id: rules.type.windows-desktop
type: rule
status: active
severity: should
scope: type
when: always
when_to_use: Windows 桌面应用（含 WebView 壳 + 本地后端）
related: [rules.type.client-runtime, rule.common.stack-layering]
---

# Windows 桌面类型包

与 [client-runtime](../client-runtime/) 叠加：本包补充 Windows 上的验收方式。

## 默认边界

- 窗口、标题栏、导航、密度与系统/设计规范（若项目对标鸿蒙 PC，走 HarmonyOS 文档的 PC 布局，而不是把手机规范硬套过来）。
- 系统工具（如 ADB sidecar）由应用按仓库约定封装，不依赖开发者机器上偶然存在的 PATH。

## UI 与实现分层

本栈**不是** Android MVVM。已声明的桌面工作台（Yohu 一类）分层为：

```
View（Solid / 页面）
  → store（会话投影、乐观 UX、把事件交给 IPC）
  → 类型化 IPC 门面（@yohu/api：类型 + invoke 转发）
  → commands（薄：鉴权/在线校验 + 转发）
  → domain / 服务 crate（填充、判定、编排、校验、安全根、采集、传输、更新检查）
```

- View 只展示并把用户事件交给 store；不填命令模板、不判定成败、不持有更新检查状态机。
- store 不编排多设备并行、不写安全根规则；那些在 domain。
- commands 不写 SettingKey 校验、HTTP、占位符替换。
- 禁止 UI 直达领域内部或 core 引用壳框架。
- 点击路径上的纯函数镜像（选择解析、过滤匹配）必须 testdata 对齐，见 [stack-layering.md](../../common/stack-layering.md)。

不要把这套改写成 ViewModel / Repository，也不要用 Activity 生命周期去套 Tauri 窗口。

## 质量门禁

- **能启动应用时：** 改 UI 或主路径后，实际启动窗口核对，必要时截图。不要只看组件单测。
- 设备相关功能（连手机、读 logcat）还要满足 Android 侧条件：有设备才宣称联调通过。
- 启动失败时先看应用日志与崩溃文件，再改代码。

## 不要默认引入

- 捆绑本可复用的系统运行时（例如已约定使用系统 WebView2 时再内嵌一套浏览器）。
