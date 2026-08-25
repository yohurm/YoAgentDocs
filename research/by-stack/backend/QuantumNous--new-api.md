---
id: research.QuantumNous-new-api
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Go, TypeScript]
  frameworks: [Gin, GORM, React]
also_relevant: [llm-app]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: QuantumNous/new-api
  url: https://github.com/QuantumNous/new-api
  cloned_to: "%TEMP%/YoAgentResearch/QuantumNous--new-api"
studied_at: 2026-08-25
related: [research.Wei-Shaw-sub2api, research.BerriAI-litellm]
---

# QuantumNous/new-api

## 入选理由

用户指定参考（NewAPI）。One API 系下一代模型网关：渠道加权、预扣费、多协议 Relay、失败自动封禁与跨组 failover——是「额度耗尽自动切换」的成熟实现样本。

## 项目是什么

统一 AI 模型中枢：把多家 LLM 聚合成 OpenAI / Claude / Gemini 兼容面，带多租户额度、渠道管理与控制台。基于 One API 演进。

## 架构

| 路径 | 作用 |
|------|------|
| `router/relay-router.go` | 对外 Relay 路由 |
| `middleware/distributor.go` | 按模型选渠道 |
| `controller/relay.go` | 预扣费 → 选渠 → 重试环 |
| `service/channel_select.go` | auto-group / 优先级重试 |
| `service/billing.go` | 预扣与结算 |
| `relay/channel/*` | 每上游一个 Adaptor |
| `model/channel.go` | 渠道实体 |
| `constant/channel.go` | ChannelType 枚举 |

多上游：`Channel`（Type + Key/BaseURL/Models/Group/Weight/Priority/Balance/MultiKey）+ `Ability`（group×model → channel）。`GetAdaptor(apiType)` 工厂分发到 `relay/channel/<vendor>/`。

额度与切换：

- 预扣：`PreConsumeBilling` / `SettleBilling`（`service/billing.go`）
- 重试环：`controller/relay.go` — 失败 `shouldRetry` → 换渠；`ShouldDisableChannel` 自动封禁
- 跨组：`service/channel_select.go`（组内 priority 用尽再切下一组）
- 用户模型限流：`middleware/model-rate-limit.go`

双模入口（`router/relay-router.go`）：

- 统一 OpenAI 面：`/v1/chat/completions`、`/embeddings`、`/responses`…
- 原生：Claude `/v1/messages`；Gemini `/v1beta/models/*`；另有 Midjourney / Suno 专用前缀
- 无「任意 path 通用透传」；靠 Adaptor 转换 + `advancedcustom` / Header/Param Override

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Distribute → Relay → Adaptor | reuse-pattern | 请求面与厂商适配解耦 |
| 预扣费 + 失败退款 | reuse-pattern | 用户额度状态机清晰 |
| priority 耗尽跨组 failover | reuse-pattern | 直接对应「没额度自动切换」 |
| ChannelType 整数 switch | anti-pattern | 新渠道要改核心；应改为注册表 |
| 巨型 GetAdaptor | anti-pattern | 扩展点应插件化 |

## 架构设计经验

1. **用户额度与上游健康分两层状态机**：钱包预扣 vs 渠道 auto-ban 不要混成一个字段。
2. **渠道选择可分层**：组内 priority → 跨 auto-group；重试次数与封禁策略可配置。
3. **协议兼容靠 Adaptor**：统一入口只认一种内部 relay DTO，出入都经变换。

## 与当前工作

- **直接用**：预扣/结算、priority failover、Adaptor 边界。
- **必须改写**：ChannelType 枚举 → 可注册上游类型；补上通用 HTTP 透传面（new-api 偏 LLM）。
- **明确不要用**：整仓 fork；把 MJ/Suno 等垂直协议塞进第一版核心。

## 阅读范围

`README.zh_CN.md`；`router/relay-router.go`；`middleware/distributor.go`；`service/channel_select.go`、`billing.go`；`controller/relay.go`（重试环）；`model/channel.go`；`relay/relay_adaptor.go`；`constant/channel.go`。
