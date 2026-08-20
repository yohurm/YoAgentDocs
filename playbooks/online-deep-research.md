---
id: playbook.online-deep-research
type: playbook
status: active
when: research
when_to_use: 执行联网深度调研的操作步骤时
related: [role.online-researcher, task.online-deep-research]
---

# 手册：联网深度调研

主题若对应官方设计体系（如 HarmonyOS、iOS），**先读已有的本地或官方文档**，再搜开源实现。本机鸿蒙文档若存在，路径一般为 `E:\Dev\Doc\HarmonyOS-Developer-docs`。

## Clone 约定

- 根目录：Windows 下为 `%TEMP%\YoAgentResearch\`（PowerShell：`Join-Path $env:TEMP 'YoAgentResearch'`）。
- 每个仓库：`<owner>--<repo>`，例如 `langchain-ai--langgraph`。
- 使用 git clone；不加入 YoAgentDocs 的 submodule，不把源码复制进本库。
- 默认浅克隆即可（`--depth 1`）；需要追历史再加深。
- clone 默认保留，文档中记录路径；不自动 `rm`。

## 筛选

按顺序权衡：

1. 与调研主题的相关度（能回答「对我们有什么用」）
2. 近期提交 / 维护迹象
3. 可读性：README、模块边界、入口清晰
4. Star、生态热度仅作参考

同一主题下避免入选结构几乎相同的重复实现；可留一个作对照并在落选理由里写。

## 阅读顺序

1. 该领域官方文档与设计规范（本地文档优先）
2. README 与官方架构说明
3. 顶层目录与包边界
4. 启动 / 编排 / 主入口
5. 关键抽象（接口、插件、状态机、数据流）
6. 配置、扩展点、测试所固化的契约

同一主题至少深读入选名单中的多个实现，避免只根据单个 README 下结论。

遇到安装即执行的脚本：遵守公共安全规则，未审查不跑。

## 分类

- 文件夹（互斥）：`research/by-stack/<capability>/`，只按能力层（agent、ui-kit、frontend…）。平台（android 等）用标签，不建调研目录。
- 跨层项目：主分类选核心贡献；次要层写在 `also_relevant` 并在另一 Hub 链过去，不复制全文。
- `other/` 暂放；同类满 3 篇再考虑升格新能力层（须用户同意）。

## 落盘

- 单项目：`research/by-stack/<capability>/<owner>--<repo>.md`
- 横向：`research/by-stack/<capability>/_synthesis.md`
- 更新 `research/README.md` 索引表

模板：`templates/research/project-study.md`、`templates/research/stack-synthesis.md`。
