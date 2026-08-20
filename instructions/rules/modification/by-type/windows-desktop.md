---
id: rule.modification.windows-desktop
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改 Windows 桌面应用的界面或主路径时
related: [rule.modification.common, rules.type.windows-desktop]
---

# 修改 Windows 桌面

- 壳层布局、导航、标题栏、模块主路径改完后，在能启动的前提下打开应用核对。
- 依赖本机 sidecar 或外部工具的功能，按仓库约定的封装路径查，不默认用户装了全局工具。
- 连设备的功能同时遵守 Android 类型包：没有手机不要宣称设备联调完成。
