---
id: rule.modification.android
type: rule
status: active
severity: should
scope: type
when: modify
when_to_use: 修改 Android 工程且涉及界面、崩溃或设备行为时
related: [rule.modification.common, rules.type.android]
---

# 修改 Android

- 布局、动画、触控、进程崩溃：已接真机则改完安装验证，用截图或 Logcat 作为依据；不要只改代码就结束。
- 未接真机时，把「未实机」写进回复，并跑得了的测试。
- 不在业务页里散落新的机型分支；适配进既有设备/配置层。
- 构建失败优先查 JDK/Gradle 占用与 daemon，而不是改业务代码绕过。
