---
id: root.readme
type: hub
status: active
when_to_use: 人阅读本库地图时
related: [root.agents, root.catalog]
---

# YoAgentDocs

给 Agent 开发用的工具无关知识库。Agent 根据 [CATALOG.md](CATALOG.md) **自行匹配任务并执行**，不依赖用户粘贴提示词。

- Agent 先读 [AGENTS.md](AGENTS.md) → [CATALOG.md](CATALOG.md)
- 人看本文件与 [USAGE.md](USAGE.md)

正文中文；目录与文件名英文。不绑定某一家 Agent 产品。`AGENTS.md` 是开放的短地图，便于多种工具自动发现入口。

## 顶层域

| 目录 | 是什么 |
|------|--------|
| [CATALOG.md](CATALOG.md) | 任务目录（调度） |
| [instructions/](instructions/) | 原则、规则、任务、角色、清单 |
| [playbooks/](playbooks/) | 多步怎么干 |
| [templates/](templates/) | 新增文档骨架 |
| [research/](research/) | 联网深研产出 |
| [experiences/](experiences/) | 自有项目经验 |

## 指令怎么叠加

```
CATALOG 命中的任务
  > 具名项目规则（有才装）
    > 平台类型包（android / windows-desktop / harmonyos…）
      > 能力类型包（ui-kit / frontend / agent…）
        > 修改规则（仅当改已有代码）
          > 全项目公共开发规则
            > 工程原则
```

验证方式在类型包「质量门禁」：例如 Android **已接真机**才走安装、截图、Logcat。

## 源码不进本库

开源深研 clone 到 `%TEMP%\YoAgentResearch\<owner>--<repo>\`，只把 Markdown 写入 `research/`。
