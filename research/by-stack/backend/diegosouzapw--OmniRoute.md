---
id: research.diegosouzapw-OmniRoute
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [TypeScript]
  frameworks: [Next.js, open-sse]
also_relevant: [llm-app]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: diegosouzapw/OmniRoute
  url: https://github.com/diegosouzapw/OmniRoute
  cloned_to: "%TEMP%/YoAgentResearch/diegosouzapw--OmniRoute"
studied_at: 2026-08-25
related: [research.Wei-Shaw-sub2api, research.BerriAI-litellm]
---

# diegosouzapw/OmniRoute

## 入选理由

用户指定参考。本地优先的多 provider AI 网关，强调配额感知调度、熔断与跨厂商自动 failover，文档化了「unknown ≠ exhausted」等对 AIAPICenter 极有价值的配额语义。

## 项目是什么

聚合大量 LLM provider（含免费档）的网关 + 仪表盘：统一 OpenAI 兼容入口、多协议兼容、Combo 路由策略、配额遥测与自动降级。

## 架构

| 路径 | 作用 |
|------|------|
| `src/app/api/` | Next.js API（relay / providers / quota） |
| `src/lib/quota/` | 配额调度与账本 |
| `src/lib/routing/` | adaptive 路由、circuit |
| `src/lib/resilience/` | 失败分类、熔断 |
| `src/domain/quotaCache.ts` | 连接级配额缓存 |
| `open-sse/` | 上游执行器与协议转换 |
| `docs/OMNIROUTE_*.md` | failover / quota / routing 设计 |

多上游：Provider catalog + Connection（凭证实例）+ Combo（多连接与 strategy）。策略含 `priority / weighted / round-robin / reset-aware / headroom / quota-share` 等（`routingStrategies.ts`）。

额度与切换：

- 状态：healthy / approaching / exhausted / unknown（`docs/OMNIROUTE_QUOTA_TELEMETRY.md`；**unknown 不作 exhausted**）
- 请求前可负担性：`quotaScheduler.ts`（`canAffordRequest`，偏 fail-open）
- Failover：失败分类 + adaptive circuit（`OMNIROUTE_PROVIDER_FAILOVER.md`、`failureClassification.ts`）

双模：统一 `/api/v1/relay/chat/completions` 等；另有 Anthropic/Gemini 兼容与 MCP/A2A。偏「统一聊天入口 + 执行器」，不是任意 path 透传网关。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 配额状态机 | reuse-pattern | unknown 中立；exhausted 才剔除 |
| 失败分类再 failover | reuse-pattern | 先分类再决定换号/冷却 |
| reset-aware / quota-share | adapt | 调度思想可迁到通用上游池 |
| 免费档 OAuth/Cookie 抓配额 | anti-pattern | 不要进产品核心 |
| Next.js 巨型 API 面 | anti-pattern | 生产网关应独立热路径进程 |

## 架构设计经验

1. **配额估计与官方计费分离**：本地估计永不冒充上游账单。
2. **熔断与配额正交**：circuit open 与 quota exhausted 都不可选，原因不同。
3. **策略可配置**：Combo 层挂 strategy，执行器层不关心「为什么选它」。

## 与当前工作

- **直接用**：配额四态、失败分类、耗尽剔除规则。
- **必须改写**：从 LLM catalog 扩到任意 HTTP 上游；热路径脱离 Next 页面进程。
- **明确不要用**：把 350+ 免费 provider 目录当默认资产；Cookie 会话当正规认证。

## 阅读范围

`README.md`；`docs/OMNIROUTE_PROVIDER_FAILOVER.md`、`OMNIROUTE_QUOTA_TELEMETRY.md`、`OMNIROUTE_ROUTING_POLICY.md`；`routingStrategies.ts`；`quotaCache.ts`；`quotaScheduler.ts`；`routingBackend.ts`；`PROVIDER_REFERENCE.md` 开头；`src/lib` 目录扫描。
