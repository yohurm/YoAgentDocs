---
id: research.primer-primitives
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [Style Dictionary]
also_relevant: [frontend, client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: primer/primitives
  url: https://github.com/primer/primitives
  head: f48bc06
  cloned_to: "%TEMP%/YoAgentResearch/primer--primitives"
studied_at: 2026-09-11
related: [research.primer-react, research.desktop-dark-color-scheme, research.synthesis.ui-kit]
---

# primer/primitives

## 入选理由

GitHub 工作台的**色值仓**（`@primer/primitives`），不是控件仓。上一轮只读了 `primer/react` 的消费面；深色方案要看这里的 base 灰阶、功能 token 覆盖表、`dark` / `dark-dimmed` 分板。产品与 Yohu 同属开发者桌面，不是手机 AMOLED 皮。

## 项目是什么

MIT。`src/tokens/` 用 Style Dictionary 编成 CSS 变量。色分三层：`base/`（不进组件）、`functional/`（`bgColor` / `fgColor` / `borderColor`）、`component/`。浅/深是两套 base；无障碍与色觉是 `overrides`，不是另造算法。阅读 HEAD `f48bc06`（2026-09-09）。

## 架构

```
base/color/{light,dark}/*.json5     原始灰阶 / 色相（禁止组件直读）
        │
functional/color/*.json5            bgColor.default|muted|inset|emphasis
        │  org.primer.overrides     只写相对主模式的差量
        ▼
dist/css/functional/themes/dark.css
dist/css/functional/themes/dark-dimmed.css
```

深色默认板（`src/tokens/base/color/dark/dark.json5`）不是纯黑：

| 阶 | hex | 角色（经 `bgColor.json5` 覆盖） |
|----|-----|--------------------------------|
| black / 0 | `#010409` | 仅 inset / 高对比；默认页不用 |
| 1 | `#0D1117` | **dark `bgColor.default`**（页面） |
| 2 | `#151B23` | dark `bgColor.muted`（次级/侧栏） |
| 3 | `#212830` | dimmed 默认页 |
| 4–6 | `#262C36`…`#2F3742` | 抬升表面 |

`bgColor.default` 浅色指向 `neutral.0`（白），深色 **override 到 `neutral.1`**，明确躲开 `black`。`dark-dimmed` 再抬到 `neutral.3`，给长时间编码减对比。文档写：对比度按 **muted**（更差的那块底）验收，不是只对 default。

`fgColor.json5` 规则：禁止组件写裸黑/裸白，忽略主题与无障碍板。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 深色默认页 ≠ base.black | reuse-pattern | 功能名共享，深色把 default 抬到下一档灰 |
| 对比度对 muted 验收 | adapt | YoUI 已测 Fg/Surface；应补 Fg/canvas 与舒适上限 |
| `overrides` 只写差量 | reuse-pattern | 对齐现有 `Colors` / `DarkColors` 双板，不要算法 |
| `dark-dimmed` 第三主题 | anti-pattern | YoUI 只有 light/dark；不要为舒适再开一板 |
| 引进 Primer 蓝灰 `#0D1117` | anti-pattern | 色值锁鸿蒙表；只抄「抬离纯黑」的映射 |

## 架构设计经验

- 深色不是浅色取反。浅色 default=最亮，深色 default=次暗，最暗留给 inset。
- 工作台长时间阅读优先「微抬画布」，高对比才回到更黑。
- 第三主题（dimmed）是产品偏好，不是 token 架构刚需。

## 与当前工作

- **能用：** 深色 `--yohu-bg-base` 应对齐浅色，用 `background_secondary`（凹槽灰），不要 `background_primary` 纯黑。
- **必须改写：** hex 用鸿蒙 `#191A1C`，不是 Primer `#0D1117`。
- **不要用：** Style Dictionary 进 `@yohu/ui`；不要 `dark_dimmed`；不要 `--bgColor-*` 前缀。

## 阅读范围

- `src/tokens/base/color/dark/dark.json5`
- `src/tokens/base/color/dark/dark.dimmed.json5`（结构；未逐色相核对）
- `src/tokens/functional/color/bgColor.json5`
- `src/tokens/functional/color/fgColor.json5`（规则段）
- `scripts/themes.config.ts`（经官方文档交叉）
- 未读：component token、语法高亮板、完整 dimmed 色相表
