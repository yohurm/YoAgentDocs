---
id: research.primer-react
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React]
also_relevant: [frontend, client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: primer/react
  url: https://github.com/primer/react
  head: 4dc92ecc5014f4c5f7675ec64d11cd0e0d5aa83e
  cloned_to: "%TEMP%/YoAgentResearch/primer--react"
studied_at: 2026-09-10
related: [research.ant-design-ant-design, research.synthesis.ui-kit]
---

# primer/react

## 入选理由

GitHub 设计体系的 React 实现，产品是 **开发者工具工作台**，和 Yohu 同族，不是中后台表单站。补 Ant/Arco 覆盖不到的：ActionList / ActionMenu 复合菜单、AnchoredOverlay 锚点浮层、焦点陷阱。Token 已迁到 CSS 变量（`--fgColor-*`），JS `ThemeProvider` 标 deprecated。

## 项目是什么

`@primer/react`（MIT）。monorepo：`packages/react` 主库，另有 styled-react / mcp / doc-gen。视觉 token 主要来自外部 `@primer/primitives`，本仓消费 CSS 变量。阅读 HEAD `4dc92ec`。

## 架构

```
@primer/primitives（本仓外）→ CSS 变量 --fgColor-* / --bgColor-* / --borderColor-*
        │
ThemeProvider（deprecated JS theme）
  └── next/ThemeProvider   只灌 context + data-* ，不再发 JS 色值

Overlay          Portal + useOverlay + useFocusTrap
AnchoredOverlay  Overlay + useAnchoredPosition（@primer/behaviors）
ActionMenu       AnchoredOverlay + ActionList
ActionList       复合组件：List/Item/Group/LeadingVisual/TrailingAction/Divider
```

- **复合菜单，不是 items[] 一张表。** `ActionList = Object.assign(List, { Item, Group, Divider, Description, LeadingVisual, TrailingVisual, TrailingAction, Heading })`。旧的 `items` 输入已 deprecated。
- **ActionMenu 只编排开合手势。** `onClose` 手势枚举：`anchor-click | click-outside | escape | tab | item-select | arrow-left | close`。子菜单走 `arrow-left`。列表样子全交给 ActionList。
- **AnchoredOverlay** 把「锚点渲染 / 外来 ref」分成两个类型分支：`renderAnchor` 有值，或 `renderAnchor: null` 且必须给 `anchorRef`。定位在 `@primer/behaviors`，不在 CSS。
- **Overlay** 自带焦点陷阱与 Portal。进场位移向量按 `anchorSide` 算，时长写死 `animationDuration = 200`。Portal 挂到 `[data-portal-root]` 下的 `#__primerPortalRoot__`，找不到再退 `document.body`（`Portal/portalRoot.ts`）。
- **ActionList 按容器推断 ARIA。** 在 ActionMenu 里：单选 `menuitemradio`、多选 `menuitemcheckbox`、否则 `menuitem`（`ActionList/Item.tsx`）。同一套 List 皮肤，多种角色。
- **不要把 YoContextMenu 改成 ActionMenu 组件。** v6 右键仍是壳唯一 Host + 场景表。学的是 List 槽位、键盘（Arrow/Home/End/typeahead/Tab）和 Overlay 关闭手势，不是每处自挂一份菜单。
- **主题：** `colorMode: day | night | light | dark | auto`。legacy `applyColorScheme` 用 `deepmerge` 把 scheme 叠进 JS theme；新路径要 CSS 变量。组件样式已是 CSS modules + `var(--fgColor-danger)`。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| ActionList 复合槽（Leading/Trailing/Description） | adapt | 对照 `YoMenuItem`：图标、快捷键、危险项应是槽，不是一个 icon 字符串 |
| ActionMenu 与 List 分权 | reuse-pattern | 开合/定位 = Host；条目样子 = List。已有 `YoContextMenuHost` + scene 表，条目渲染可向 ActionList 靠 |
| AnchoredOverlay 的「自绘锚点 vs 外来 ref」 | adapt | YoSelect 是自绘触发钮；右键是外来坐标。两种锚点不要合成一个糊 API |
| 关闭手势枚举 | adapt | 右键/下拉的 dismiss 应对齐：外点、Esc、选中、Tab |
| Portal 挂到 `data-portal-root` | adapt | 比写死 `document.body` 更适合 WebView 滚动容器；Host 仍唯一 |
| List 按容器推断 menuitem / option | reuse-pattern | 同一套条目皮肤给右键、Select、Tab overflow |
| 键盘：Arrow / typeahead / Tab 关菜单 | adapt | 现有右键主要只有 Esc，这是工作台缺口 |
| 焦点陷阱在 Overlay，不在每个菜单 | reuse-pattern | Dialog 已有；菜单若焦点循环，进同一 Overlay 原语 |
| CSS 变量功能色 `--fgColor-danger` | adapt | YoUI 已有 tone；工具栏危险动作用语义色，不要再开 GitHub 绿/紫 |
| JS ThemeProvider + deepmerge | anti-pattern | 官方自己弃用；YoUI 继续 emit CSS |
| Overlay `animationDuration = 200` 写死 | anti-pattern | 必须 `MotionSpec` |
| 引进 `@primer/react` / Octicons / primitives | anti-pattern | 视觉是 GitHub，不是鸿蒙工作台 |
| PageLayout / SplitPageLayout 当壳 | lesson-only | 工作台分栏已有自己的壳，不要再包一层 Primer 布局 |

## 架构设计经验

- **工作台菜单是「列表皮肤 + 锚点浮层」，不是 Select。** Ant 的 Dropdown 仍偏中后台；Primer 把命令列表做成 ActionList，菜单只是把它挂到锚点上。YoUI 右键已经是这条，缺的是同一套 List 给工具栏溢出菜单、页眉 overflow。
- **Token 外置。** Primer 本仓不再定义色板。YoUI 色板已在 `tokens/`，不要把组件仓再变成第三份色。
- **废弃复合 API 比废弃色值更痛。** `deprecated/ActionList` 仍在。YoUI 公开面从第一天用 `Yo*` 槽位，不要先做 `items: []` 再拆。

## 与当前工作

**能直接用**

- 菜单 = Host（开合/定位/焦点）+ List（项/组/分割/前后视觉）。
- 锚点两种入口：自绘触发 vs 外来矩形。
- 功能色走语义 token，不用 Primer 名。

**必须改写**

- Solid + 已有 `context-menu/` / `popover-place` / `YoPresence`。
- 动效用 `MotionSpec`，不用 200ms 常量。
- 项类型对齐现有 `YoMenuItem`，不要抄 React slot marker。

**明确不要用**

- `@primer/react`、`@primer/primitives`、Octicons。
- GitHub day/night 色与 `sponsors` / `done` 功能色。
- styled-components 遗留路径（`packages/styled-react`）。

## 阅读范围

`%TEMP%/YoAgentResearch/primer--react`，HEAD `4dc92ec`：

- `README.md`、`packages/react/README.md`
- `packages/react/src/index.ts`
- `packages/react/src/ThemeProvider.tsx`
- `packages/react/src/next/index.ts`
- `packages/react/src/ActionList/index.ts`
- `packages/react/src/ActionMenu/ActionMenu.tsx`
- `packages/react/src/AnchoredOverlay/AnchoredOverlay.tsx`
- `packages/react/src/Overlay/Overlay.tsx`
- `packages/react/src/Portal/portalRoot.ts`
- `packages/react/src/ActionList/Item.tsx`（角色推断）
- `packages/react/src/legacy-theme/ts/color-schemes.ts`（确认 CSS 变量回退）

未读：Dialog 全文、PageLayout、TextInput、TooltipV2 实现、`@primer/primitives` / `@primer/behaviors` 独立仓。
