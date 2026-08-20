---
id: rules.type.windows-desktop
type: rule
status: active
severity: should
scope: type
when: always
when_to_use: Windows 桌面应用（含 WebView 壳 + 本地后端）
related: [rules.type.client-runtime]
---

# Windows 桌面类型包

与 [client-runtime](../client-runtime/) 叠加：本包补充 Windows 上的验收方式。

## 默认边界

- 窗口、标题栏、导航、密度与系统/设计规范（若项目对标鸿蒙 PC，走 HarmonyOS 文档的 PC 布局，而不是把手机规范硬套过来）。
- 系统工具（如 ADB sidecar）由应用按仓库约定封装，不依赖开发者机器上偶然存在的 PATH。
- 分层按该仓库已声明结构（常见为 UI → 类型化 IPC → 命令薄层 → 领域）。禁止 UI 直达领域内部或 core 引用壳框架。不要把 Android MVVM 生套到非 MVVM 的桌面栈上。

## 质量门禁

- **能启动应用时：** 改 UI 或主路径后，实际启动窗口核对，必要时截图。不要只看组件单测。
- 设备相关功能（连手机、读 logcat）还要满足 Android 侧条件：有设备才宣称联调通过。
- 启动失败时先看应用日志与崩溃文件，再改代码。

## 不要默认引入

- 捆绑本可复用的系统运行时（例如已约定使用系统 WebView2 时再内嵌一套浏览器）。
