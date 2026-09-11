---
id: research.JetBrains-android-logcat-colors
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [Kotlin, XML]
  frameworks: [IntelliJ Color Scheme]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: JetBrains/android
  url: https://github.com/JetBrains/android
  cloned_to: "%TEMP%/YoAgentResearch/JetBrains--android"
studied_at: 2026-09-11
related: [research.JetBrains-android, research.synthesis.ui-kit]
---

# JetBrains/android（Logcat V2 色板）

## 入选理由

Yohu 对标 Android Studio Logcat。已有笔记只读面板交互。本次读 V2 色板：级别字母是色块，消息另有前景；Fatal 在源码里叫 Assert。

## 项目是什么

`logcat/resources/colorSchemes/` 两份方案：`LogcatColorSchemeDefault.xml`（浅）与 `LogcatColorSchemeDarcula.xml`（深）。`LogcatColors.kt` 把 `LogLevel` 映射到 `LOGCAT_V2_LEVEL_*` 与 `LOGCAT_V2_MESSAGE_*`。`LevelFormat` 给字母左右各垫一空格再上 LEVEL 键。设备 `FATAL` 在 `LogcatProtoShellCollector` 收成 `LogLevel.ASSERT`。

## 架构

```
LogLevel
  → LEVEL_KEYS（字母色块 FOREGROUND+BACKGROUND）
  → MESSAGE_KEYS（消息前景，Assert 与 Error 同色）
Tag → ColorPaletteManager 动态前景（与级别无关）
```

浅色字母块：

| 级别 | 字 | 底 |
|------|----|----|
| V | `#000000` | `#d6d6d6` |
| D | `#000000` | `#cfe7ff` |
| I | `#414d41` | `#e9f5e6` |
| W | `#000000` | `#f5eac1` |
| E | `#ffffff` | `#cf5b56` |
| Assert | `#ffffff` | `#7f0000`（更深红） |

浅色消息：V 黑、D `#389FD6`、I `#59A869`、W `#645607`、E/Assert `#cd0000`。

深色 Assert 底 `#8b3c3c`，Error 底仍是 `#cf5b56`。消息 E/Assert 都是 `#ff6b68`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Assert 比 Error 更深，不是紫 | reuse-pattern | 官方 V2 不用社区紫。Yohu Fatal ink 用 warning 压黑，禁止 Material `#9C27B0` |
| 字母是色块，消息另开一键 | adapt | Yohu 虚拟列表不做六档色块。只给 Fatal 反色块；V–E 走 ink |
| Tag 动态调色板 | anti-pattern | 需求是级别色，不是按 Tag 换色。`--yohu-tag` 仍是徽章 |
| 社区 Darcula 紫 Assert | anti-pattern | SO / DEV 文章不是 Studio 默认。以仓内 XML 为准 |

## 架构设计经验

级别色与消息色是两张表。Yohu 把「色相」收进 `--yohu-level-*`，把「怎么画」（反色 / 消息同色）收进模块 paint，不要再让 `--yohu-level-f` 表示白字、`--yohu-level-f-bg` 表示色相。

## 与当前工作

- 能直接用：Fatal 深于 Error；反色块用 `font_on` 叠在 Fatal ink 上。
- 必须改写：色值只派生鸿蒙 `warning`，不抄 `#7f0000`。
- 不要用：IntelliJ `TextAttributesKey`、Tag 调色板、六档字母底。

## 阅读范围

`LogcatColors.kt`、`LevelFormat.kt`、两份 `LogcatColorScheme*.xml`、`LogcatProtoShellCollector` 的 FATAL→ASSERT。未读 `logcat-tags-palette.json`。
