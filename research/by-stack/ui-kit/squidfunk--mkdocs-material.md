---
id: research.squidfunk-mkdocs-material
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [HTML, CSS, TypeScript, Jinja]
  frameworks: [MkDocs, Material Design]
also_relevant: [client-runtime, docs]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: squidfunk/mkdocs-material
  url: https://github.com/squidfunk/mkdocs-material
  cloned_to: "%TEMP%/YoAgentResearch/squidfunk--mkdocs-material"
studied_at: 2026-09-08
related: [research.vuejs-vitepress, research.synthesis.ui-kit]
---

# squidfunk/mkdocs-material（网格分区与双侧栏拓扑）

## 入选理由

MkDocs Material 是全球 Star 27k+ 的技术文档静态站点标杆。其通过 CSS Grid（`md-main__inner md-grid`）将工作台区域划分为：
1. 顶层无缝 Header（`md-header`）
2. 宏观专栏导航侧栏（`md-sidebar--primary`）
3. 核心文章正文容器（`md-content`）
4. 微观本页目录大纲（`md-sidebar--secondary`）

其分区设计将双侧栏在滚动时均保持粘性吸顶（`md-sidebar__scrollwrap`）同时互不挤压正文，且正文内绝无重复的导航条和多头控制按钮，为全量重构文档阅读 UI 提供了顶级的视觉节奏与网格工程规范。

## 项目是什么

MkDocs Material 是构建在 Python MkDocs 之上的旗舰级技术文档主题系统，采用现代 Material Design 与高定制 CSS 自定义属性驱动。

## 架构

```
md-container
  ├── md-header（单一权威全局操作栏：Logo / 搜索 / 模式切换 / 仓库链接）
  └── md-main
        └── md-grid（响应式 CSS Grid / Flex 三栏网格）
              ├── md-sidebar--primary（左侧专栏章节树，独立滚动轨）
              ├── md-content（中央主体阅读流，独立排版语义容器）
              └── md-sidebar--secondary（右侧页内 TOC 导航，独立滚动轨）
```

- **双侧栏独立滚动隔离**：两边侧栏使用 `overflow-y: auto` 与 `position: sticky`，正文长篇滚动时，左侧专栏树与右侧 TOC 停留在视口内随手可得，不会发生整页混乱滚动。
- **信息展示单一源原则**：文章标题、元信息徽标（如版本、更新时间）只在正文顶部（`article.md-content__inner`）出现一次，绝不在顶栏或侧边栏重复显示相同内容。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| `md-grid` 经典三栏自底向上栅格 | reuse-pattern | 左主导航、中正文、右本页 TOC，三者在同一个顶层容器内对齐，避免分块嵌套错位 |
| 侧栏独立滚动轨与粘性停靠 | reuse-pattern | 左右两栏各自独立处理 overflow，不随主视窗失控滚动 |
| 统一的颜色与暗色模式 Token 变量 | adapt | 使用 `--yo-bg-app`、`--yo-line` 等统一度量，杜绝写死内联颜色 |
| 纯 CSS Checkbox 模拟抽屉折叠（input.md-toggle） | anti-pattern | 桌面应用中折叠展开必须使用响应式状态管理（SolidJS Signal），不应退化为 DOM Hack |

## 架构设计经验

1. **绝对消灭“多轨数据源与外壳双层套娃”**：如果外部容器已经有了统一标题栏与工作台工具栏，内部正文区域绝不允许再画一个独立的伪标题栏。内部只保留干净纯粹的 Article Header。
2. **三栏分区必须物理隔离，边界明确**：左侧专栏、中央正文、右侧大纲各自具有清晰的 `border-right` / `border-left` 分割线与独立滚动容器，不可混杂在一层 div 里通过 margin 硬凑。

## 与当前工作

- **能直接用的**：三栏独立滚动模型（独立 scrollable container）；信息唯一性收口（正文 Header 单次渲染）；
- **必须改写的**：将 Jinja 模板转化为 SolidJS 响应式组件，将纯 HTML 跳转转化为本地 Store 驱动。
- **明确不要用的**：HTML Checkbox 折叠 hack 与重度服务端渲染逻辑。

## 阅读范围

- `material/templates/base.html`
- `material/templates/partials/header.html`
- `material/templates/partials/content.html`
