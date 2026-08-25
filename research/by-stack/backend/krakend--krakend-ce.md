---
id: research.krakend-krakend-ce
type: project-study
status: active
when: research
stack:
  capability: backend
  languages: [Go]
  frameworks: [Lura, Gin]
also_relevant: []
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: krakend/krakend-ce
  url: https://github.com/krakend/krakend-ce
  cloned_to: "%TEMP%/YoAgentResearch/krakend--krakend-ce"
studied_at: 2026-08-25
related: [research.Kong-kong, research.Portkey-AI-gateway]
---

# krakend/krakend-ce

## 入选理由

声明式、无状态的聚合型网关（BFF）。`encoding: no-op` 透传与多 backend 扇出合并，清晰演示「统一聚合面」与「原生透传面」可在同一配置壳下共存——且刻意不做分布式额度账本。

## 项目是什么

KrakenD CE：把 Lura 与限流/熔断/JWT/Lua 等中间件组装成的高性能 API Gateway，GitOps 友好、水平扩展简单。

## 架构

本仓是薄组装层，逻辑多在依赖中：

| 路径 | 作用 |
|------|------|
| `executor.go` | 装配日志、指标、路由、插件 |
| `backend_factory.go` | Backend 管线（OAuth2 client、缓存、RL、CB…） |
| `proxy_factory.go` | Endpoint proxy 管线 |
| `handler_factory.go` | HTTP 层中间件 |
| `plugin.go` | 加载 `.so` 插件 |
| `krakend.json`、`tests/fixtures/` | 示例与 no-op 测试 |

配置：`endpoints[]` → `backend[]`（host、url_pattern、encoding、extra_config）。多 backend 并发扇出合并；单 backend + `no-op` = 原始透传。

额度与切换：路由器/后端层限流与 circuit-breaker；**无**跨节点配额钱包。故障靠 CB/主机列表，非额度感知切换。

双模：聚合变换 endpoint vs `no-op` passthrough endpoint，共享外层鉴权/限流壳。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| no-op vs aggregate 双 endpoint | reuse-pattern | 双模最干净的配置表达 |
| 工厂装饰栈 | lesson-only | 热路径保持可组合 |
| 无状态扩展 | lesson-only | 配额状态必须外置 |
| 指望 CE 存多租户额度 | anti-pattern | 设计上就不做协调 |
| QoS 挂在 backend | adapt | 每上游独立 RL/CB |

## 架构设计经验

1. **双模是配置问题，不是两套网关**：同一外壳，encoding/变换开关不同。
2. **无状态网关把账本外置**：AIAPICenter 的额度服务应独立于代理热路径。
3. **聚合与透传目标不同**：统一 API 常要字段过滤/合并；原生要字节级忠实。

## 与当前工作

- **直接用**：双 endpoint 模式；每 backend QoS；薄核心 + 插件。
- **必须改写**：外挂配额中心与 Selector；支持动态上游池（不只 GitOps JSON）。
- **明确不要用**：把 KrakenD 当额度与自动切换的唯一实现。

## 阅读范围

`README.md`；`go.mod`；`executor.go`；`backend_factory.go`；`proxy_factory.go`；`handler_factory.go`；`plugin.go`；`krakend.json`；`tests/fixtures/krakend.json`。
