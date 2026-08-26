---
id: rules.by-type.hub
type: hub
status: active
when_to_use: 按项目类型装载基础规则时
related: [rules.hub, template.type-pack]
---

# 项目类型基础规则

类型包分两套，**可叠加**：

- **能力层**：这类软件做什么（与 `research/by-stack/` 目录名对齐）
- **平台层**：跑在哪、怎么验收（有设备才走真机等）

识别到之后都装。分层**用词**以 [common/stack-layering.md](../common/stack-layering.md) 为准：Android 页面、桌面 IPC、组件库 L0–L5 不是同一套。没有对应目录就只用公共规则。新建用 [templates/instructions/type-pack.md](../../../templates/instructions/type-pack.md)。

## 能力层

| 类型 | 目录 | 放什么 |
|------|------|--------|
| agent | [agent/](agent/) | 多 Agent、规划、工具调用、记忆、编排 |
| llm-app | [llm-app/](llm-app/) | RAG、评测、提示词产品化、模型接入 |
| frontend | [frontend/](frontend/) | Web UI、状态、页面 |
| ui-kit | [ui-kit/](ui-kit/) | 可复用组件库（Yo 组件等） |
| backend | [backend/](backend/) | API、领域、持久化 |
| data | [data/](data/) | 管道、仓、分析 |
| devops | [devops/](devops/) | CI、部署、可观测 |
| client-runtime | [client-runtime/](client-runtime/) | 桌面、IDE、CLI、应用壳 |
| docs | [docs/](docs/) | 文档站与本知识库 |

分类词与 `research/by-stack/` 对齐（`research/README.md`）。`security`（认证、密钥、供应链，仅当这是主贡献）与 `other`（暂放）是预留分类，尚无独立类型包时按公共规则。

## 平台层

| 类型 | 目录 | 典型验证 |
|------|------|----------|
| android | [android/](android/) | **已接真机则安装 + 截图 + Logcat**；未接设备则不假装实机通过 |
| harmonyos | [harmonyos/](harmonyos/) | 官方文档为设计基准；有真机则实机对照 |
| windows-desktop | [windows-desktop/](windows-desktop/) | 启动桌面应用核对界面与主路径 |

某仓库与类型包不同的点写到 [../by-project/](../by-project/)。例如「只适配某一型号」属于具名项目，不写进 `android/` 公共包。
