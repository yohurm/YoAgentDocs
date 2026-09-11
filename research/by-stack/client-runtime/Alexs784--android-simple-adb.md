---
id: research.Alexs784-android-simple-adb
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [python]
  frameworks: [kivy]
also_relevant: []
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: Alexs784/android-simple-adb
  url: https://github.com/Alexs784/android-simple-adb
  head: 8841ca020032c840ebfe3884b7319d163c70f8fe
  cloned_to: "%TEMP%/YoAgentResearch/Alexs784--android-simple-adb"
studied_at: 2026-09-11
related: [research.dannagle-PacketSender]
---

# Alexs784/android-simple-adb

## 入选理由

少有的「ADB 命令脚本是一等对象」的开源桌面实现：Script 含有序 Step，可拖拽重排、保存、复制、对当前设备一键跑。用来对照 Yohu「命令里的块」而不是「组文件夹」。

## 项目是什么

Kivy + SQLite 的 Python 桌面应用。用户从预置步骤表挑命令（含占位符），组成 UserScript，在已选设备上顺序执行。

## 架构

```
UserScript (id, name)
  → UserStep[] (position, command_id, PickleType Command)
        Command { value, is_adb }
  → Run: for step in steps:
        adb?  subprocess `adb -s SERIAL {value}`
        else  subprocess host 行（含 `sleep N`）
        退出码 ≠ 0 则 time.sleep(2) 再试，最多 2 次
```

关键点：

- Script 是容器，Step 是可复用命令模板的一次引用；一步内部还可以是多条 `Command`（例如按 view id 点击先 dump 再 pull 再 tap）。
- 间隔不是 Script 字段。预置步骤 `COMMAND_SLEEP` 的 `is_adb=False`，值是主机 `sleep {秒}`。
- 失败重试另写死 2 秒，和用户 sleep 步不是一条语义。
- 执行在按钮回调里同步 for 循环；`shell=True` 拼命令行。
- 2024-01 后未见活跃。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 可执行单位是「脚本/块」，不是文件夹 | reuse-pattern | Yohu 组继续只当目录；多行+间隔挂在命令叶子上 |
| 步骤有序、可增删重排 | reuse-pattern | 命令管理第三栏编辑 steps，而不是用换行拼 template |
| Sleep 当成一条「命令」 | anti-pattern | 结果区会多出假输入行；Yohu 间隔不是 ADB/主机命令 |
| 失败重试 + 写死 2s | anti-pattern | 违反 ADR-v6-009 与「不因失败中断」；禁止做成败分支 |
| `shell=True` 拼串 | anti-pattern | 继续走现有 `split_command_line` + `Runner` |

## 架构设计经验

ADB GUI 里「等一会儿」常被做成步骤表里的一种命令。这样编辑器简单，但把控制面泄漏成可执行行。Yohu 的 IO 块是输入/输出对，sleep 没有设备输出，不应进 `>>>`。

一步展开成多条底层命令（dump/pull/tap）说明：块的持久化单位应是「用户看见的一行 template」，不要在 UI 再藏一套子进程脚本。

## 与当前工作

- 能用：叶子命令拥有有序 steps；占位符在开跑前填一次。
- 必须改：间隔是块级 `gap_ms`，不是 `sleep` 步；执行在 domain，可取消。
- 不要用：退出码重试、主机 sleep 步、SQLite 逐步表、在 View 里 for 循环 exec。

## 阅读范围

`storage/database/model/{user_script,user_step,command}.py`、`script_editor/editor_screen.py`（`run_script`）、`storage/database/database_manager.py`（预置 sleep 步）、`comands/commands_utils.py`。未读步骤选择器全部预置命令表。
