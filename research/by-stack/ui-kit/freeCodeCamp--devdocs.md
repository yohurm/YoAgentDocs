---
id: research.freeCodeCamp-devdocs
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [JavaScript, Ruby, CSS]
  frameworks: [DevDocs App, Custom MVC]
also_relevant: [client-runtime, frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: freeCodeCamp/devdocs
  url: https://github.com/freeCodeCamp/devdocs
  cloned_to: "%TEMP%/YoAgentResearch/freeCodeCamp--devdocs"
studied_at: 2026-09-08
related: [research.vuejs-vitepress, research.synthesis.ui-kit]
---

# freeCodeCamp/devdocs（多文档流与工作台分栏分区体系）

## 入选理由

DevDocs 是 GitHub 上 Star 高达 39k+ 的经典快速离线技术文档浏览器与阅读工作台。它面对的核心痛点与我们高度一致：如何在同一个桌面级 Web 界面内整合来自华为、MDN、Go、Rust 等不同技术来源的大量异构文档，并且保持“极速切换、分栏清晰、视觉统一、绝无多头展示”。其将页面严格拆分为全局壳（Menu / Resizer）、侧边多层分类（Sidebar）、单向路由内容宿主（Content）的架构，为解决工作台“多轨杂乱、分区分散”提供了极佳的范式。

## 项目是什么

DevDocs 是一个面向程序员的全功能 API 文档工作台。它将上百种异构官方文档清洗提取为标准结构，并在客户端以极其干净的经典分栏架构提供即时搜索、快捷键盘导航与统一阅读体验。

## 架构

```
app.views.Document（总控视窗）
  ├── app.views.Menu（全局左侧极速导航与菜单）
  ├── app.views.Sidebar（左侧专栏/分类树：搜索框 + 文档拾取器 + 列表折叠）
  ├── app.views.Resizer（可拖拽分栏轨道：处理左右视图物理宽度）
  └── app.views.Content（中央唯一阅读宿主）
        ├── EntryPage / TypePage / SettingsPage（按路由单一激活）
        └── 统一滚动与锚点状态机（scrollMap / historyStack）
```

- **单向生命周期与视图互斥激活**：`app.views.Content` 作为容器，内部维护严格的当前激活子视图（`this.view`）。切换文档时严格调用前任的 `deactivate()` 并挂载新视图的 `activate()`，杜绝多个内容源并行悬挂导致的混乱。
- **物理分栏与宽度持久化**：通过 `Resizer` 明确划分 Chrome 控制区与 Content 阅读区，保证左右边界明确，侧边栏折叠与展开有固定动画与边界状态机。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 容器级 Single Active View 状态机 | reuse-pattern | 中央阅读区只能存在一个权威正文渲染器，杜绝两套 HTML/IFrame 并行共存 |
| 边界明确的 Resizer 分栏隔离 | reuse-pattern | 侧边栏与阅读区之间通过明确的分栏边界切分，禁止按钮跨区乱塞 |
| 统一的全局滚动与锚点锁（scrollToTarget） | adapt | 文档更新后由集中机制统一平滑滚动或还原位置，而不是组件内部到处打补丁 |
| 古典全局命名空间挂载（app.views.*） | anti-pattern | 现代桌面应用应基于 TypeScript 模块化与 SolidJS 细粒度响应式 Store，不可退化为全局对象 |

## 架构设计经验

1. **绝对禁止“内容区域自作主张又画一套顶栏”**：DevDocs 的 Content 区域只负责展示经过标准化清洗的文档正文，绝不在内部重复渲染一遍侧边栏已有的标题、版本选择器或搜索框。所有控制元信息一律收敛于工作台的外壳控制栏。
2. **单一权威状态机（Single Active View）**：正文模式（原貌网页 vs Markdown）必须是互斥渲染的单一状态流，切换时必须完整卸载旧交互并接管焦点。

## 与当前工作

- **能直接用的**：工作台分栏分区拓扑；单向内容渲染宿主职责；消除正文内与外壳重复的控制栏。
- **必须改写的**：DevDocs 采用无框架原生 DOM 与 Ruby 构建流，我们使用 SolidJS 细粒度响应式 + @yohu/ui 高性能组件。
- **明确不要用的**：基于 Cookie 与原生 EventBus 的弱类型状态同步机制。

## 阅读范围

- `assets/javascripts/views/layout/document.js`
- `assets/javascripts/views/content/content.js`
- `assets/javascripts/views/sidebar/sidebar.js`
