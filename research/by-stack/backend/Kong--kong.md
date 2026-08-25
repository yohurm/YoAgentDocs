---
id: research.Kong-kong
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Lua]
  frameworks: [OpenResty, PostgreSQL]
also_relevant: []
utilization: [reuse-pattern, adapt, lesson-only]
source:
  platform: github
  repo: Kong/kong
  url: https://github.com/Kong/kong
  cloned_to: "%TEMP%/YoAgentResearch/Kong--kong"
studied_at: 2026-08-25
related: [research.apache-apisix, research.krakend-krakend-ce]
---

# Kong/kong

## 入选理由

成熟的 Service→Route→Upstream→Target 实体模型与 PDK 插件体系。`response-ratelimiting` 可消费上游返回的剩余额度头——对「官方/第三方 API」场景尤其关键。

## 项目是什么

Kong Gateway：声明式或 DB 模式的云原生网关，插件生态与 AI/LLM 插件面完整。

## 架构

| 路径 | 作用 |
|------|------|
| `kong/db/schema/entities/` | Route / Service / Upstream / Target / Plugin / Consumer |
| `kong/runloop/` | 代理环、balancer、plugins_iterator、重试 |
| `kong/router/` | 传统与表达式路由 |
| `kong/plugins/` | 内置插件 |
| `kong/llm/` | AI proxy filters / adapters |
| `kong/pdk/` | 插件开发套件 |

实体职责：Service（调用什么）/ Route（客户端怎么进来）/ Upstream+Target（实例池）/ Plugin（策略挂载作用域）。

额度与切换：

- 客户端限流：`plugins/rate-limiting/`
- 响应驱动：`response-ratelimiting/`（按上游响应头累加）
- 健康检查：Upstream `healthchecks` + `runloop/balancer/`
- 重试：Service `retries` + `upstream_retry.lua`（下一 Target，**非**配额钱包切换）
- AI 归一：`plugins/ai-proxy` + `kong/llm` filter 链

双模：无 AI 插件的 Route→Service = 透传；挂 ai-proxy = 统一归一。可对同一 Upstream 挂多条 Route（统一门面 vs 原生路径）。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Service/Route/Upstream/Target | reuse-pattern | 目录建模黄金分割 |
| 策略挂对作用域 | reuse-pattern | consumer/route/service 分层 |
| response-ratelimiting | adapt | 信任厂商 remaining 头 |
| PDK + priority | lesson-only | 扩展安全边界 |
| 仅靠 retries 当多厂商切换 | anti-pattern | 不懂 402/额度语义 |

## 架构设计经验

1. **产品目录 ≠ 入口 URL ≠ 实例池**：四层拆开后管理面才可扩展。
2. **上游报额度优于本地瞎猜**：有 remaining 头就用响应驱动限流。
3. **插件优先级数字化**：避免隐式执行顺序。

## 与当前工作

- **直接用**：四层实体命名；双 Route（统一/原生）共享或分 Service。
- **必须改写**：自建「provider wallet」状态机（Kong OSS 偏 consumer 限流）。
- **明确不要用**：整仓运行时依赖；把 LLM filter 当唯一扩展方式。

## 阅读范围

`README.md`；`db/schema/entities/{routes,services,upstreams,targets,plugins}.lua`；`runloop/{handler,plugins_iterator,upstream_retry}.lua`；`plugins/rate-limiting/handler.lua`、`response-ratelimiting/handler.lua`、`key-auth/handler.lua`、`ai-proxy/handler.lua`；`kong/llm/schemas/init.lua`；`kong/pdk/` 列表。
