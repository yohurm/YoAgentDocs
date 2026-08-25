---
id: research.BerriAI-litellm
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Python]
  frameworks: [FastAPI]
also_relevant: [llm-app]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: BerriAI/litellm
  url: https://github.com/BerriAI/litellm
  cloned_to: "%TEMP%/YoAgentResearch/BerriAI--litellm"
studied_at: 2026-08-25
related: [research.Portkey-AI-gateway, research.QuantumNous-new-api]
---

# BerriAI/litellm

## 入选理由

统一 OpenAI 格式 SDK + 自托管 Gateway；明确拆出 **统一面** 与 **厂商原生透传**（`/anthropic/{path}` 等），并有 Router cooldown/fallback 与预算过滤——最贴近 AIAPICenter「双模访问」需求。

## 项目是什么

LiteLLM：把 100+ provider 收成 OpenAI 兼容调用；Proxy 提供虚拟 Key、预算、负载均衡与 guardrails。

## 架构

| 路径 | 作用 |
|------|------|
| `litellm/router.py` | Router：deployment、fallback、cooldown |
| `litellm/router_strategy/` | least-busy / lowest-cost / TPM-RPM / budget_limiter |
| `litellm/llms/` | 每 provider 变换层 |
| `litellm/proxy/` | Gateway：auth、hooks、endpoints |
| `litellm/proxy/pass_through_endpoints/` | 原生透传 |
| `ARCHITECTURE.md` | 请求流 |

多上游：**Deployment** = 逻辑 `model_name` 组 + `litellm_params`（真实 model/key/base/tpm/rpm）。组内选路，支持 tag / routing_groups。

额度与切换：

- `allowed_fails` → cooldown；`default_fallbacks` / model fallbacks
- 预算过滤：`router_strategy/budget_limiter.py`（先健康再滤预算）
- Proxy：virtual key spend、`proxy/hooks`（max_budget、并行限流）

双模：

- 统一：`/v1/chat/completions`、`/v1/messages`、embeddings…
- 原生：`llm_passthrough_endpoints.py` — `/anthropic/{path}`、`/bedrock/{path}`、`/vertex_ai/...`
- 配置型透传：yaml `pass_through_endpoints` 指向任意 target URL

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 逻辑模型组 vs 物理 deployment | reuse-pattern | 统一名与真实上游解耦 |
| 统一面 + `/provider/{path}` | reuse-pattern | 双模访问的标杆拆法 |
| 预算作 filter | adapt | 可套到多上游选路 |
| 配置任意 URL 透传 | adapt | 支撑「任意第三方 API」 |
| 单文件巨石 router.py | anti-pattern | 策略必须拆包，勿整文件复制 |

## 架构设计经验

1. **SDK 变换层 ≠ Proxy 治理层**（ARCHITECTURE 两层模型）：前者管协议，后者管 Key/预算/钩子。
2. **Cooldown 与 Fallback 分层**：短暂冷却不等于永久下线。
3. **透传与转换并存**：原生路径跳过 body 归一，只走鉴权与计量壳。

## 与当前工作

- **直接用**：双轨路由设计；deployment 组；透传 endpoint 模式。
- **必须改写**：Python 巨石 → 自研模块边界；通用 API 不只 LLM passthrough 前缀。
- **明确不要用**：依赖整棵 litellm 作为运行时内核。

## 阅读范围

`README.md`；`ARCHITECTURE.md`；`router.py` 头部与 fallback/cooldown 相关逻辑；`budget_limiter.py`；`pass_through_endpoints.py`；`llm_passthrough_endpoints.py`；`proxy/` 目录扫描。
