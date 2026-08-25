---
id: research.apache-apisix
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Lua]
  frameworks: [OpenResty, etcd]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: apache/apisix
  url: https://github.com/apache/apisix
  cloned_to: "%TEMP%/YoAgentResearch/apache--apisix"
studied_at: 2026-08-25
related: [research.Kong-kong, research.TykTechnologies-tyk]
---

# apache/apisix

## 入选理由

动态、etcd 热更新的通用 API 网关。`ai-proxy-multi` 的 instance 级限流与 `fallback_strategy`（429/5xx/自定义状态）是「多上游耗尽切换」在通用网关里最清晰的开源样本之一。

## 项目是什么

Apache APISIX：基于 NGINX/OpenResty 的云原生 API 网关，插件化流量治理，并带 AI 网关插件族。

## 架构

| 路径 | 作用 |
|------|------|
| `apisix/` | 运行时（router、upstream、plugin、balancer、admin） |
| `apisix/plugins/` | 一等插件（限流、鉴权、AI、traffic-split…） |
| `apisix/balancer/` | LB（roundrobin、chash、priority…） |
| `apisix/schema_def.lua` | Route / Service / Upstream schema |

实体：Route（匹配）→ Service（共享上游+插件）→ Upstream（加权 nodes、priority、健康检查）。插件按 priority 分阶段执行（`apisix/plugin.lua`）。

额度与切换：

- 通用限流：`limit-count` / `limit-req` / `limit-conn`
- AI token 成本：`ai-rate-limiting.lua`（按 instance）
- 多实例 failover：`ai-proxy-multi.lua` + `fallback_strategy`（`rate_limiting`、`http_429`、`http_5xx`、`fallback_http_statuses`）
- 优先级均衡：`balancer/priority.lua`（一档耗尽再下一档）
- 熔断：`api-breaker.lua`

双模：普通 Route→Upstream = 原生透传；`ai-proxy` / `ai-proxy-multi` + `ai-providers/*` = 统一/归一化面。可用 `proxy-rewrite` 改 path/host。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 命名 instance + fallback_strategy | reuse-pattern | 按 429/402 等切下一上游 |
| Route/Service/Upstream 分层 | reuse-pattern | 目录与策略分离 |
| priority balancer | adapt | 主备上游档 |
| token 限流与 RPS 限流分开 | adapt | 成本维度独立 |
| 仅靠权重 LB 表达配额 | anti-pattern | 权重不懂「还剩多少额度」 |

## 架构设计经验

1. **耗尽要用语义化触发器**：状态码 / 本地 instance 限流标记，而不是随机重试。
2. **插件合并作用域**：route / service / global 分层挂策略。
3. **健康检查与配额切换正交**：node 不健康 ≠ 额度耗尽。

## 与当前工作

- **直接用**：instance + fallback_http_statuses 模型；透传与 AI 归一并存。
- **必须改写**：不嵌入整套 OpenResty/etcd；把模式抽到自研控制面。
- **明确不要用**：把 APISIX 当业务额度中心（消费侧钱包仍需自建）。

## 阅读范围

`README.md`；`apisix/schema_def.lua`；`router.lua`、`upstream.lua`、`plugin.lua`；`balancer/priority.lua`；`plugins/limit-count.lua`、`ai-rate-limiting.lua`、`ai-proxy-multi.lua`、`ai-proxy/schema.lua`、`key-auth.lua`、`api-breaker.lua`、`traffic-split.lua`、`proxy-rewrite.lua`；`plugins/ai-providers/`。
