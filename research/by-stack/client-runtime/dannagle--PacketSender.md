---
id: research.dannagle-PacketSender
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [cpp]
  frameworks: [qt]
also_relevant: []
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: dannagle/PacketSender
  url: https://github.com/dannagle/PacketSender
  head: 9714b2ef7ad30b5d19b6c8bb5b0c4a56df65e7d3
  cloned_to: "%TEMP%/YoAgentResearch/dannagle--PacketSender"
studied_at: 2026-09-11
related: [research.Alexs784-android-simple-adb, research.usebruno-bruno]
---

# dannagle/PacketSender

## 入选理由

桌面「命名条目 + 一键跑一段序列」最接近 Yohu 命令块的产品形态：面板按钮存一段脚本，行与行之间可写 `delay:秒数`。不是 ADB，但间隔语义和预校验路径可直接对照。

## 项目是什么

Qt 网络发包工具。库里是单个 Packet；面板按钮另存 `script` 字符串，按行引用已存 Packet 名，并插入 `delay:` / `sleep:` / `panel:`。

## 架构

```
PanelButton.script  (多行字符串，落在面板 JSON)
  → PanelGenerator 预扫：空行/注释丢弃；delay/sleep 必须 ≥1 秒；packet 名必须能查到
  → 任一错误 → 对话框，整段不跑
  → QtConcurrent 后台线程逐行：
        delay:/sleep: → QThread::sleep(秒)
        包名 → emit sendPacket + 固定 msleep(100)
```

关键点：

- 间隔是**脚本里的一行**，不是 Packet 字段，也不是「本次运行」配置。
- `delay:` 与 `sleep:` 同义；单位是整数秒，小于 1 直接拒。
- 预校验和执行是两趟：先全部过再开跑。
- 间隔睡眠不可取消（`QThread::sleep`）。
- 每个包后还有写死的 100ms，和用户 `delay:` 叠在一起。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 间隔属于序列本身，跟命名条目一起落盘 | reuse-pattern | Yohu 命令块的间隔应存在命令上，不存在发送栏本次运行 |
| 开跑前整段校验 | reuse-pattern | 未知步、非法间隔、空块在 domain `validate` 拦下，不要跑到一半再报 |
| `delay:N` 迷你 DSL | anti-pattern | Yohu 不要在 template 里解析 `delay:`；间隔是字段，不是命令行 |
| 阻塞 sleep、写死 100ms | anti-pattern | 睡眠必须吃 CancellationToken；禁止第二套魔法间隔 |

## 架构设计经验

序列里的「等一会儿」有三种放法：脚本行、条目字段、本次运行参数。PacketSender 选第一种，编辑灵活，但把控制面和命令行混成一种语言。Yohu 已禁止成功/失败正则和组内 DSL，不应再引入第三种行类型。

## 与当前工作

- 能用：块是叶子命令的身体；间隔跟命令一起保存；保存前全量校验。
- 必须改：间隔用 `gap_ms` 常量集，不用 `delay:秒` 文本。
- 不要用：后台线程里不可取消的 sleep；包后固定 100ms；把组文件夹做成面板脚本。

## 阅读范围

`src/panelgenerator.cpp`（脚本预扫与执行）、`src/panel.cpp`（`script` 落盘）、README 面板 delay 说明。未读 intense traffic / UDP flood 的重发节拍（那是单包重复，不是命令块）。
