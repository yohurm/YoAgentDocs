---
id: rules.type.client-runtime
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束桌面 / IDE / CLI / 应用壳时
related: [rules.type.windows-desktop]
---

# 客户端运行时类型包

壳与进程模型。Windows 桌面验收叠加 [windows-desktop](../windows-desktop/)；Android 壳叠加 [android](../android/)。

## 默认边界

- 壳（窗口、命令、权限）与内部业务 UI/逻辑分开。
- 自动更新、文件权限、后台进程跟平台惯例；不把 Web 页面约定硬套到原生壳上。

## 质量门禁

- 说明最低系统版本与安装/卸载影响的文件位置。
- CLI 要有非交互用法，便于脚本与 Agent 调用。
- 壳层改动：在对应平台包的条件下启动一次（桌面窗口或设备上的宿主 App）。

## 不要默认引入

- 未要求的常驻后台服务、全局 PATH 污染、开机自启。
