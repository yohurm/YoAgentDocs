---
id: research.Portkey-AI-gateway
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [TypeScript]
  frameworks: [Hono]
also_relevant: [llm-app]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: Portkey-AI/gateway
  url: https://github.com/Portkey-AI/gateway
  cloned_to: "%TEMP%/YoAgentResearch/Portkey-AI--gateway"
studied_at: 2026-08-25
related: [research.BerriAI-litellm, research.apache-apisix]
---

# Portkey-AI/gateway

## 入选理由

轻量 TS 网关：统一 OpenAI 兼容 handler + 配置驱动的 fallback/loadbalance + 通配 `/v1/proxy/*` 透传。展示「热路径保持小」与策略树递归试靶，对照额度账本缺失的边界。

## 项目是什么

开源 AI Gateway（可跑 Node / Workers）：请求级 Config 描述 provider 与 targets 策略，厂商差异收敛到 ProviderConfig。

## 架构

| 路径 | 作用 |
|------|------|
| `src/index.ts` | 路由入口 |
| `src/handlers/*Handler.ts` | 各 endpoint |
| `src/handlers/handlerUtils.ts` | `tryTargetsRecursively` 策略引擎 |
| `src/handlers/retryHandler.ts` | 单 target 重试（含 429 Retry-After） |
| `src/handlers/proxyHandler.ts` | 通配透传 |
| `src/providers/*` | 每厂商 Config |
| `plugins/` | Guardrail 插件 |

多上游：请求 Config 中 `provider` + `targets[]` + `strategy.mode` ∈ `single | fallback | loadbalance | scientist`（`types/requestBody.ts`）。`Providers` 注册表在 `src/providers/index.ts`。

额度与切换：开源核心几乎无用户账单；靠 FALLBACK 顺序试下一 target、LOADBALANCE 权重与 Redis 限流（`rateLimiter.ts`）。**没有**订阅配额耗尽换号的领域模型。

双模：统一 `/v1/chat/completions` 等；透传 `POST/GET/DELETE /v1/proxy/*` 与兜底 `/v1/*` → `proxyHandler`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| tryTargetsRecursively 策略树 | reuse-pattern | nested targets + 继承 retry/cache |
| ProviderConfig 收敛厂商差异 | reuse-pattern | handler 保持瘦 |
| 统一 handler + 通配 proxy | reuse-pattern | 双模壳层 |
| 请求级 config 持久化为池 | adapt | 升级为平台级 upstream pool |
| 无额度账本只抄路由壳 | anti-pattern | AIAPICenter 核心在配额切换 |
| 热路径体量控制 | lesson-only | 证明网关不必巨石 |

## 架构设计经验

1. **策略是数据**：fallback/loadbalance 用配置描述，代码只解释。
2. **透传作兜底**：未知路径进 proxy，而不是为每个厂商加路由。
3. **计费可外置，但产品定位要诚实**：开源只做路由时，不要假装有完整额度中心。

## 与当前工作

- **直接用**：策略树、ProviderConfig、proxy 兜底。
- **必须改写**：补持久化上游池 + 配额账本；请求级 header config 改为管理面配置。
- **明确不要用**：以为 Portkey OSS 已解决「没额度自动切换」。

## 阅读范围

`README.md`；`src/index.ts`；`handlerUtils.ts`（含 `tryTargetsRecursively`）；`retryHandler.ts`；`proxyHandler.ts`；`providers/index.ts`；`types/requestBody.ts`；`plugins/README.md`。
