---
id: research.JakeWharton-pidcat
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Python]
  frameworks: []
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern, lesson-only]
source:
  platform: github
  repo: JakeWharton/pidcat
  url: https://github.com/JakeWharton/pidcat
  cloned_to: "%TEMP%/YoAgentResearch/JakeWharton--pidcat"
studied_at: 2026-09-11
related: [research.synthesis.ui-kit]
---

# JakeWharton/pidcat

## 入选理由

经典包名过滤彩色 logcat。字母表 `VDIWEF` 与 Yohu `log_levels.json` 同一张。用来对照「级别色画在哪」以及 Fatal 是否另色。

## 项目是什么

把 `adb logcat -v brief` 按包名/PID 过滤后印到终端。Tag 用 LRU 分配 ANSI 色；级别只画三字符徽章。

## 架构

```
LOG_LEVELS = 'VDIWEF'
TAGTYPES = {
  V: 白字黑底, D: 黑字蓝底, I: 黑字绿底,
  W: 黑字黄底, E: 黑字红底, F: 黑字红底
}
KNOWN_TAGS → allocate_color（红/绿/黄/蓝/品红/青轮转）
```

`TAGTYPES['F']` 与 `E` 都是 `bg=RED`。品红只给 Tag，不给 Fatal。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 字母表 VDIWEF | reuse-pattern | 与 domain testdata 同一向量 |
| 级别只占徽章，正文不强行上色 | reuse-pattern | Yohu 默认消息走 `--yohu-fg`；只给 Error/Fatal 消息同 ink |
| Fatal 与 Error 同红底 | anti-pattern | 终端徽章太窄，桌面清单要更深的 Fatal |
| Tag LRU 彩色 | anti-pattern | 违反「级别色单源」；Yohu Tag 跟级别 ink |

## 架构设计经验

级别色与 Tag 色必须分权。pidcat 把彩色预算给了 Tag，级别只剩蓝/绿/黄/红。Yohu 相反：彩色预算给 V–F，Tag 跟 ink。

## 与当前工作

- 能直接用：徽章式级别强调；字母表锁 testdata。
- 必须改写：徽章是 CSS 反色块，不是 ANSI。
- 不要用：按 Tag 换色、ANSI 8 色当 token。

## 阅读范围

`pidcat.py`：`LOG_LEVELS`、`TAGTYPES`、`allocate_color`、主循环印徽章。未读 fork。
