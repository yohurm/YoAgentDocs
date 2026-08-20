---
id: rules.type.android
type: rule
status: active
severity: should
scope: type
when: always
when_to_use: Android 应用或 Android 上的组件库
related: [rule.modification.android, rules.type.ui-kit, rule.common.architecture]
---

# Android 类型包

能力层（`ui-kit` / `frontend` 等）照常叠加。本包只约束 Android 工程的构建与验收。

## 默认边界

- 构建用该仓库已约定的 JDK 与 Gradle。同一台机器上多个 Android 工程并行编译时，默认**串行**，或为任务使用独立的 Gradle user home，避免共用 daemon 导致超时后才成功。
- UI 尺寸与官方设计稿对齐观感（鸿蒙文档常用 vp，Android 用 dp/px），单位不同不作为「可以对不齐」的理由。
- 设备差异、板级属性等适配逻辑放在明确的一层，不要散落在各个界面里。
- **MVVM（有界面的应用模块）：** View 不直访数据库/网络/文件；业务状态不进 Activity/Fragment/自定义 View。组件库模块本身不是页面，遵守 [ui-kit](../ui-kit/) 的分层，不要为每个控件硬套 ViewModel。

## 质量门禁

- **已连接真机：** 界面、动画、崩溃、权限与设备行为，以安装后的真机为准：操作目标路径、截图、读 Logcat。单测不能替代这一步。
- **未连接真机：** 可跑单元/仪器测试与静态检查；在回复里写明「未实机」。不要用模拟描述代替截图结论。
- 用户指定了目标机型时，只在该机型上验收，除非任务要求多机适配。

## 不要默认引入

- 第二套构建系统、未约定的 Kotlin/Java 版本、与现有模块平行的 Support 库。
