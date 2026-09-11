---
id: research.android-logcat-v-color
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [C++]
  frameworks: [AOSP liblog]
also_relevant: [client-runtime]
utilization: [reuse-pattern, anti-pattern, lesson-only]
source:
  platform: other
  repo: platform/system/core
  url: https://android.googlesource.com/platform/system/core/+/309d6dde312fc279da11c3b77d93d3e8177e830f/liblog/logprint.cpp
studied_at: 2026-09-11
related: [research.synthesis.ui-kit]
---

# AOSP logcat `-v color`

## 入选理由

Yohu 日志分析对标 Android logcat。平台命令 `adb logcat -v color` 是设备侧官方着色，不是社区主题。读 `liblog/logprint.cpp` 的 `colorFromPri`，用来判断 Fatal 是否必须另开一色。

## 项目是什么

AOSP `liblog` 把优先级映射成 xterm 256 前景，前缀 `\x1B[38;5;<n>m`。帮助文本写在 `logcat/logcat.cpp` 的 `show_format_help`。

## 架构

```
entry.priority
  → colorFromPri(pri)
  → snprintf("\x1B[38;5;%dm", color)
  → 整行同一前景，无字母反色块
```

| 优先级 | 256 色号 | 近似 | 用途 |
|--------|----------|------|------|
| VERBOSE / SILENT / 未知 | 231 | 近白 | 默认正文 |
| DEBUG | 75 | 钢蓝 | 调试 |
| INFO | 40 | 绿 | 信息 |
| WARN | 166 | 橙 | 警告（帮助里写 WARNING） |
| ERROR | 196 | 红 | 错误 |
| FATAL | 196 | 红 | **与 ERROR 同号** |

`YELLOW 226` 已定义，未进 `colorFromPri`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| V 灰 / D 蓝 / I 绿 / W 橙 / E 红 | reuse-pattern | 与鸿蒙 `font_*` / brand / confirm / alert / warning 同构，Yohu 已走这条 |
| Fatal === Error 同色 | anti-pattern | CLI 整行同色，桌面清单扫读不够。AS V2 用更深红底区分 Assert |
| 终端 256 霓虹绿/红 | anti-pattern | 禁止当 YoUI hex；桌面要落鸿蒙语义板 |
| 整行同一前景 | lesson-only | Yohu 只给左条 / 级别 / Tag / 严重消息上色，时间列保持三级字色 |

## 架构设计经验

官方 CLI 的「不同颜色」只有五档色相。Fatal 的区分不在色相，而在优先级字母。桌面清单若 Fatal 与 Error 同色且同处理，扫读会并成一块红。

## 与当前工作

- 能直接用：V 弱化、D 蓝、I 绿、W 橙、E 红；色值只引用鸿蒙 primitive。
- 必须改写：Fatal 要比 Error 更深，并保留反色块（对照 AS，不抄 256 色号）。
- 不要用：xterm 40/75/196 当 token；不要引进社区紫（Material `#9C27B0`）冒充官方。

## 阅读范围

`liblog/logprint.cpp`（`ANDROID_COLOR_*`、`colorFromPri`）；`logcat/logcat.cpp` `show_format_help` 中的 color 示例。未读 logd 采集。
