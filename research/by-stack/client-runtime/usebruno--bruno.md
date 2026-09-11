---
id: research.usebruno-bruno
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [javascript]
  frameworks: [electron]
also_relevant: []
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: usebruno/bruno
  url: https://github.com/usebruno/bruno
  head: d28b57e3e01ba843eaffa992c13f64ffc0471c1f
  cloned_to: "%TEMP%/YoAgentResearch/usebruno--bruno"
studied_at: 2026-09-11
related: [research.dannagle-PacketSender]
---

# usebruno/bruno

## 入选理由

开源集合运行器里，「请求之间等一会儿」做成得最清楚：delay 是**这一次 Run 的参数**，不是请求正文的字段。用来对照 PacketSender「间隔写进脚本」——Yohu 必须二选一，不能两套。

## 项目是什么

本地 API 客户端。Collection / Folder / Request 分层；Collection Runner 可设 Delay between requests (ms)，CLI 为 `bru run --delay`。

## 架构

```
UI RunnerResults / RunCollectionItem
  → delay: number | null   （本次运行；可写入 runnerConfiguration）
  → IPC renderer:run-collection-folder(..., delay)
  → 循环每个 request：
        if delay > 0:  Promise.race(setTimeout(delay), abort)
        axios(request)
```

关键点：

- 同一 delay 用在集合里每一对相邻请求上；请求文件本身没有 `delay_ms`。
- 取消与睡眠竞赛（`AbortController`），不是不可打断的 sleep。
- 实现把 delay 放在**每个请求之前**（含第一发）。文案写 between，代码是 before-each。Yohu 不得照抄这个偏差：间隔只发生在步与步之间。
- 另有 mock-server `globalDelay`，与 Runner delay 不是同一条链，不要混进命令库。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 睡眠可取消 | reuse-pattern | domain `select!{ cancel, sleep(gap) }` |
| 单一间隔用于整段序列 | reuse-pattern | 用户说「选择间隔」，块级一个 `gap_ms` 即可 |
| delay 只活在 Run、不进条目 | anti-pattern（对 Yohu） | 产线预设必须可复现；间隔要跟命令一起落盘 |
| before-each 含第一发 | anti-pattern | 第一刀不应先空等 |
| 自由输入毫秒 | lesson-only | Yohu 用「选择」：domain 常量集，不用裸输入框 |

## 架构设计经验

「间隔是运行参数」适合探索/限流；「间隔是条目属性」适合产线一键复现。Yohu 命令库是后者。Bruno 的可取消睡眠仍应搬到 domain，不要在 store 里 `setTimeout`。

## 与当前工作

- 能用：块级单一间隔；取消打断等待；进度仍按步冒泡。
- 必须改：`gap_ms` 存在 `CommandDefinition` 上，随 `commandlib.save` 全量提交。
- 不要用：发送前再弹「本次 delay」；请求级脚本 `setTimeout`；第一刀前等待。

## 阅读范围

`packages/bruno-app/src/components/RunnerResults/index.jsx`、`.../RunCollectionItem/index.js`、`packages/bruno-electron/src/ipc/network/index.js`（约 1868 行）、`packages/bruno-cli/src/commands/run.js` 的 `--delay`。未读 Bru 文件 schema 全文与测试断言引擎。
