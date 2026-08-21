---
id: rule.common.stack-layering
type: rule
status: active
severity: must
scope: common
when: always
when_to_use: 架构审查、架构设计、跨技术栈描述 UI 与实现边界时
related: [rule.common.architecture, rules.by-type.hub]
---

# 技术栈分层词汇

公共规则只要求三件事：认仓库已声明的层、禁止越级、副作用落在正确层。

**层的名字跟类型包走。** 禁止把所有带界面的工程都叫做 MVVM，也禁止用 Android 的 View / ViewModel / Model 去描述 Tauri 桌面、组件库或 HTTP 后端。

审查或设计时：先读仓库架构文档，再读对应类型包的「UI 与实现分层」。描述用该包的层名；需要对照时可以写映射，不要替换成另一套。

## 各栈分层（摘要）

| 类型包 | UI / 展示 | 实现 / 副作用终点 | 禁止 |
|--------|-----------|-------------------|------|
| [android](../by-type/android/README.md) 有界面的应用模块 | View（Activity / Fragment / 自定义 View） | ViewModel 持状态与决策；Model / Repository 取数与持久化 | View 直访数据库/网络/文件；业务状态进 View |
| [android](../by-type/android/README.md) 组件库 / [ui-kit](../by-type/ui-kit/README.md) | L4 视图 + L5 `YoXxx` 门面 | L0 token、L1 共享能力、L2 模型、L3 策略 | 给每个控件硬套页面 ViewModel；在 `api/` 里画 Path |
| [harmonyos](../by-type/harmonyos/README.md) 页面 | ArkUI 页面与自定义组件 | 页面状态层（仅当工程采用官方推荐的状态分离） | 用 Android 控件习惯替换 ArkUI 分层且不写映射 |
| [frontend](../by-type/frontend/README.md) | 页面组件 | 仓库已有的 store / service；网络在 API 客户端 | 组件里散落 fetch 与持久化；未采用 MVVM 时仍把 store 叫做 ViewModel |
| [windows-desktop](../by-type/windows-desktop/README.md) + [client-runtime](../by-type/client-runtime/README.md) | View（页面）→ store（投影/会话）→ 类型化 IPC 门面 | commands 薄转发 → domain / 服务 crate | UI 填充/判定/校验/编排；core 引用壳框架；把这套叫做 MVVM |
| [backend](../by-type/backend/README.md) | 无桌面 UI；handler 只译协议 | 应用/领域 → 持久化 | handler 里写领域规则或 SQL |

组件库分层细则在 [ui-kit/layering.md](../by-type/ui-kit/layering.md)（L0–L5）。那是**控件**分层，不是页面 MVVM，也不是桌面 IPC 分层。

## 越级（所有栈共用，层名不同）

- 展示层直达数据源或领域内部。
- 门面（`api/`、`YoXxx`、IPC 命令层、HTTP handler）夹带算法。
- 把本应在实现层的填充、判定、校验、编排、安全根放在 UI。

## 允许的契约孪生（不是兼容层）

点击或按键路径上不能等 IPC 时，允许 UI 与 domain **各有一份纯函数镜像**（选择解析、过滤匹配、面包屑夹紧、首屏设置默认值）。必须：

- 同一份 testdata / fixture 锁死；
- 禁止第三份实现；
- 禁止语义分叉（文案、未知值、默认根）。

语义一旦分叉，按 [architecture.md](architecture.md)「严禁兼容层」沿链路重设计，不要再加适配器。
