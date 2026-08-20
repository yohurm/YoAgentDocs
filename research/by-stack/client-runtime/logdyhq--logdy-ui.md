---
id: research.logdyhq-logdy-ui
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [TypeScript]
  frameworks: [Vue 3, Pinia]
also_relevant: [frontend]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: logdyhq/logdy-ui
  url: https://github.com/logdyhq/logdy-ui
  cloned_to: "%TEMP%/YoAgentResearch/logdyhq--logdy-ui"
studied_at: 2026-08-20
related: [research.synthesis.client-runtime]
---

# logdyhq/logdy-ui

## 入选理由

Yohu 架构已点名 Logdy 的「暂停 / 跟随正交」。UI 在本仓库（core 在 `logdyhq/logdy-core`）。用来核对：Space 暂停怎么绑、键事件如何避开输入框、三态跟随不要误抄进采集层。

## 项目是什么

Logdy 的 Vue Web UI：表格浏览、facet 过滤、详情抽屉、暂停/跟随/从光标恢复。嵌入 `logdy-core` 单二进制。官方：[Following logs](https://logdy.dev/docs/explanation/following-logs)、[Keyboard shortcuts](https://logdy.dev/docs/explanation/features/miscellaneous)。

## 架构

```
initKeyEventListeners()  →  document.keydown
  if target is INPUT: return
  if drawer open: Esc 关 / ↑↓ 换行
  if !settingsDrawer && !modal && Space: toggle paused ↔ following
store.receiveStatus: paused | following | following_cursor
  paused           → client.pause()           后端停推
  following        → client.resume()          跟最新
  following_cursor → client.resumeFromCursor() 从暂停点续
```

`key_events.ts` 是整份键盘逻辑。Space 会 `preventDefault()`，因此焦点在按钮/列表时不会滚页。抽屉打开时 ↑↓ 只换详情行，不改跟随状态。

Yohu 已经实现了更合适的两端：`paused` 冻可见区、`following` 由滚动度量驱动、离开底部只加 `pendingCount`。Logdy 的第三态 `following_cursor` 对应「从暂停点续」——Yohu 不需要，因为环没停。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 输入框内忽略面板快捷键 | reuse-pattern | `target.nodeName === 'INPUT'` 不够，还要 textarea / contenteditable |
| 抽屉打开时快捷键换语义 | reuse-pattern | 对话框/重命名打开时，日志 Ctrl+W 不应关会话 |
| Space 切换暂停 | reuse-pattern | 与需求一致；须再加「焦点在日志面板内」 |
| `document.keydown` 无面板作用域 | anti-pattern | 正是 Yohu 现状：模块挂着就全局抢键 |
| 暂停通知后端停推 | anti-pattern | 违反 Yohu「缓冲继续」 |

## 架构设计经验

- 「跳过 INPUT」只是最低门槛。真正缺的是 **哪一块 UI 拥有这组键**。Logdy 是单页查看器，全局抢键可接受；Yohu 是多模块工作台，不可接受。
- 暂停 / 跟随 / 从某行续 是产品三态。Yohu 用 `paused` × `following` 两轴已经覆盖，不要再引入第三种采集状态。

## 与当前工作

- 能直接用：可编辑目标上的快捷键放行；对话框打开时面板命令停。
- 必须改写：监听从 `window`/`document` 收到面板根；Space 不得在设备栏触发。
- 不要用：正则搜索栏、facet、把暂停打到 core。

## 阅读范围

`src/key_events.ts`、`src/store.ts`（`ReceiveStatus` / `changeReceiveStatus`）、`src/components/TopBar.vue`、`src/api.ts`（pause/resume）。未读 facet / breser 查询实现。
