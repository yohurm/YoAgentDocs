---
id: research.arco-design-arco-design
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, Less]
  frameworks: [React]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: arco-design/arco-design
  url: https://github.com/arco-design/arco-design
  head: c2b050d9c7ce94bebba94f616a0721344231caac
  cloned_to: "%TEMP%/YoAgentResearch/arco-design--arco-design"
studied_at: 2026-09-10
related: [research.ant-design-ant-design, research.synthesis.ui-kit]
---

# arco-design/arco-design

## 入选理由

和 Ant Design 同属中后台全家桶，但主题落地是 **Less token → CSS 变量写到 `body`**，比 Ant 的 CSS-in-JS 更接近 YoUI 的 `emit-theme.ts`。Trigger 是仓内一等组件（不是 `@rc-component/trigger`），Select / Tooltip / Dropdown 都挂它。用来对照：CSS 变量主题怎么热更新、浮层要不要自研。

## 项目是什么

字节跳动的 Arco Design React 实现（`@arco-design/web-react`，MIT）。60+ 组件，主题可走 less-loader 或 Design Lab。阅读 HEAD `c2b050d`（2026-08-24）。

## 架构

```
components/style/theme/default.less   全局 less（字号、时长、z-index、曲线）
        │
components/*/style/token.less         组件 token（引用全局 @size-* / @radius-*）
        │  编译成 --arcoblue-6 等 CSS 变量
        ▼
ConfigProvider.setTheme()             document.body.style.setProperty
        │
ConfigContext                         size / prefixCls / getPopupContainer / componentConfig
        │
Trigger（仓内）                       getPopupStyle + Portal + CSSTransition
        └── Select / Tooltip / Dropdown / DatePicker
```

- **主题不是 Seed→Map 算法。** `ThemeConfig` 只是 `Record<string, any>`。`setTheme` 认 `primaryColor` / `successColor` / `warningColor` / `dangerColor` / `infoColor`，写到 `--arcoblue-6` 等；未给 Hover/Active 时用 `lighten(color, ±10)`。挂在 **`document.body`**，不是子树。暗色主路径是 `body[arco-theme='dark']` 覆写 `--color-bg-*` / `--color-text-*`（`style/theme/css-variables.less`），不是再跑一套 JS 算法。无全局 density：尺寸只靠组件 `size` + 固定 `@size-*` px。
- **组件 token 仍是 Less。** `Button/style/token.less` 把高度绑 `@size-mini`…`@size-large`，水平 padding 却写死 11/15/19px。
- **ConfigProvider** 与 Ant 同形：`getPopupContainer`、`size`（mini/small/default/large）、`componentConfig` 按组件覆盖默认 props、`zIndex`、`focusLock`（Modal/Drawer）、`renderEmpty`。另有 `effectGlobalNotice` / `effectGlobalModal` 去改静态 Message/Modal——和 Ant 同一类陷阱。
- **Trigger 一等公民。** `interface.ts`：`position`、`alignPoint`、`autoAlignPopupWidth`、`mouseEnterDelay=100`、`duration` 默认 200（可拆 enter/exit）。`getPopupStyle.ts` 自己算相对 root 的 left/top，处理 `boundaryDistance`。实现是 class 组件 + `ResizeObserver` + `throttleByRaf`。Dropdown 在 `trigger==='contextMenu'` 时打开 `alignPoint`（跟鼠标，不是跟触发钮）。Select 是 Trigger + `_class/select-view` + VirtualList，Modal/Drawer 走独立 Portal + FocusLock，不进 Trigger 栈。
- **Button**：`type`（外形：primary/secondary/dashed/text/outline）× `status`（warning/danger/success）。比 Ant 的 color×variant 更接近现在的 `YoButtonVariant`，但已经把「形」和「语义色」拆开。
- **Input**：`prefix`/`suffix` 在盒内，`addBefore`/`addAfter` 在盒外；`status=error|warning`；`error` 布尔已废弃。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 主题改 CSS 变量，不改 JS 样式表 | reuse-pattern | 与 `setTheme` + `theme.css` 同构；运行时只 `setProperty` |
| Trigger 作为唯一浮层原语 | adapt | YoSelect / 右键 / 未来 Tooltip 应对齐「一个 place + 一个 Portal」 |
| 右键 = `contextMenu` + `alignPoint` | reuse-pattern | 已有 `openContextMenu(x,y)`；不要把右键做成跟触发钮对齐的 Dropdown |
| 暗色只覆写 alias 变量 | reuse-pattern | 与 YoUI `[data-theme]` 同构；组件 CSS 不动 |
| Button `type`×`status` | adapt | 印证 Ant 的两轴；YoButton 的 danger 应是 status，不是第四种外形 |
| Input 内外缀 + status | adapt | 与 Ant 同构，命名是 `addBefore` 不是 `addonBefore` |
| `componentConfig` 改默认 props | lesson-only | YoUI 体量小，全局默认用 token，不必再做 props 袋 |
| `setTheme` 写 `document.body` | anti-pattern | 没有子树隔离；局部预览会漏到整窗 |
| `lighten(±10)` 派生 hover | anti-pattern | YoUI 已用鸿蒙 interactive 5%/10% `color-mix` |
| Trigger class + 每帧测盒 | anti-pattern | 对照已有 `popover-place`；不要再引入 ResizeObserver 循环 |
| duration enter≠exit | anti-pattern | Fade 必须同时长（Fluent / 鸿蒙已定） |
| 组件 token 里写死 11px padding | anti-pattern | 必须引用 spacing token |
| 中文按钮自动插空格 | anti-pattern | 与 Ant 相同，不要 |

## 架构设计经验

- **CSS 变量主题的热更新可以极薄**：改几个 `--*` 即可，不必重跑 cssinjs。前提是编译期已经把组件样式写成 `var(--token)`。
- **写到 `body` 就没有嵌套主题。** 要局部主题必须把变量挂在子树根（`data-theme` / class），不是 document。
- **浮层自研只值当「一个 Trigger」。** Arco 把定位收口，Select 不再自己算坐标。YoUI 已有 `popover-place`，缺的是 Tooltip/Popover 复用它，不是第二套 Trigger。
- **Less 全局 + 组件 token 文件** 和 YoUI「tokens/*.ts + 组件 CSS」同构。Arco 的失败点是组件 token 仍夹魔法数。

## 与当前工作

**能直接用**

- 运行时只改 CSS 变量。
- 浮层单原语；Button 形/色两轴；TextField 内外缀 + status。

**必须改写**

- 变量写在应用根或 `:root` / `[data-theme]`，不要 `document.body`。
- hover/pressed 继续鸿蒙 `color-mix`，不用 `lighten`。
- Trigger 行为用现有 `popover-place` + `YoPresence`，不搬 class 组件。

**明确不要用**

- 引进 `@arco-design/web-react` 或它的 Less。
- 抄 Arco 色板 / Inter 字体 / 2px 小圆角。
- 为每个组件再做一份 Less token 文件（YoUI 已有组件 CSS + 全局 token）。

## 阅读范围

`%TEMP%/YoAgentResearch/arco-design--arco-design`，HEAD `c2b050d`：

- `README.md`
- `components/style/theme/{default,global,css-variables}.less`
- `components/ConfigProvider/{index.tsx,interface.ts}`
- `components/Trigger/{index.tsx,interface.ts,getPopupStyle.ts}`
- `components/Dropdown/index.tsx`（`alignPoint={trigger==='contextMenu'}`）
- `components/Button/{interface.ts,style/token.less}`
- `components/Input/interface.tsx`

未读：Table/Form/DatePicker 全文、Vue 实现仓、Design Lab 云端。
