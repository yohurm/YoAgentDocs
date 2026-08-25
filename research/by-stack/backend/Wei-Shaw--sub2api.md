---
id: research.Wei-Shaw-sub2api
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Go, TypeScript]
  frameworks: [Gin, Vue3, Ent, PostgreSQL, Redis]
also_relevant: [llm-app]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: Wei-Shaw/sub2api
  url: https://github.com/Wei-Shaw/sub2api
  cloned_to: "%TEMP%/YoAgentResearch/Wei-Shaw--sub2api"
studied_at: 2026-08-25
related: [research.QuantumNous-new-api, research.diegosouzapw-OmniRoute]
---

# Wei-Shaw/sub2api

## 入选理由

用户指定参考项目。把多上游订阅账号的配额分发给下游 API Key，覆盖鉴权、计费、智能选号与协议别名——直接对应 AIAPICenter 的「额度 + 自动切换 + 双模入口」。

## 项目是什么

AI API 网关：管理 Anthropic / OpenAI / Gemini / Grok 等订阅或 API Key 上游账号，向用户签发平台 Key，在网关侧做转发、token 计费、并发与限流。Go 后端 + Vue 管理台。

## 架构

关键分层：

| 路径 | 作用 |
|------|------|
| `backend/internal/server/routes/` | HTTP：gateway / admin / auth |
| `backend/internal/service/` | 调度、计费、配额抓取、渠道 |
| `backend/internal/domain/` | 平台与账号类型常量 |
| `backend/pkg/pluginapi/` | 外置 gRPC 插件协议 |
| `frontend/` | 管理台 |

多上游模型：

- **Platform**：`anthropic / openai / gemini / … / composite`（`domain/constants.go`）
- **Account**：上游凭证实例（oauth / apikey / upstream …）+ 优先级、并发、限流
- **Channel**：面向下游的计费产品面（模型定价、映射），与 Account 分离
- **Group**：用户 Key 绑定分组 → 决定可用账号池；`composite` 按模型解析目标平台

主请求路径：鉴权 → Group/Channel → `gateway_scheduling.go` 选号 → 转发 → 用量结算（`gateway_usage_billing.go`）。

额度与切换：

- 调度：粘性会话、优先级、排除已失败账号 `excludedIDs`（`service/gateway_scheduling.go`）
- 换号重试：`service/gateway_service.go`（如 429 后 NextAccount）
- 上游配额抓取：`openai_quota_service.go`、`gemini_quota.go`、`grok_quota_*` 等

双模入口（`server/routes/gateway.go`）：

- 统一/兼容：`/v1/messages`、`/v1/chat/completions`、`/v1/responses`
- 原生/特化：`/v1beta/models/*`（Gemini）、`/antigravity/v1/*`、Codex 别名路径

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Account ≠ Channel | reuse-pattern | 上游凭证池与下游计费面必须拆开 |
| excludedIDs 换号 | reuse-pattern | 耗尽/失败后同请求内切换上游 |
| 多入口别名 | adapt | 客户端习惯路径可多挂，调度内核保持单一 |
| 订阅 OAuth 配额抓取 | adapt | 可作插件；合规与 ToS 风险自担 |
| 平台特判爆炸 | anti-pattern | 按厂商堆专用路径会失控，应收敛为适配器注册表 |

## 架构设计经验

1. **上游账本与下游钱包分两层**：上游看订阅/厂商配额，下游看用户额度与渠道定价。
2. **选号内核单一**：协议面可以胖，调度状态机要瘦（粘性 + 优先级 + 排除集）。
3. **插件边界清晰**：outbound transport 用 gRPC 插件扩展，不改核心转发环。

## 与当前工作

- **直接用**：Account/Channel/Group 三分法；耗尽换号；双协议入口思路。
- **必须改写**：从「订阅拼车」产品叙事扩成通用 HTTP API 管理；平台常量改为可注册上游类型。
- **明确不要用**：把商业化支付/拼车运营当成核心；无审查地复制 OAuth/Cookie 抓配额灰产路径。

## 阅读范围

`README_CN.md`；`backend/internal/domain/constants.go`；`backend/internal/server/routes/gateway.go`；`service/gateway_scheduling.go`、`account.go`、`channel.go`；`docs/PLUGIN_DEVELOPMENT.md`；配额相关 `*_quota*` service 文件名扫描。
