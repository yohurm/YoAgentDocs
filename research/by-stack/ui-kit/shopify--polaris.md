---
id: research.shopify-polaris
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React]
also_relevant: [frontend]
utilization: [adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: shopify/polaris
  url: https://github.com/shopify/polaris
  head: 3f7954ae42fabf26d63cee68c23ceebfd7ef0972
  cloned_to: "%TEMP%/YoAgentResearch/shopify--polaris"
studied_at: 2026-09-10
related: [research.ant-design-ant-design, research.synthesis.ui-kit]
---

# shopify/polaris

## 入选理由

不是再一个中后台 React 全家桶，而是 **token 工程 + 强制只用 token 的 lint**。`polaris-tokens` 带 metadata（value + description），`stylelint-polaris` 禁 hex / 禁自造 custom property。用来补 YoUI 已有 `check-ui-tokens` 的缺口。React 包已归档，许可也限制「长得像 Shopify 后台」——只学纪律，不抄皮肤。

## 项目是什么

Shopify 设计体系 monorepo：`polaris-tokens` / `polaris-react` / `stylelint-polaris` / 文档站。根 README 写明 **Polaris React 已归档**（2025-10-01 起推 Web Components）。阅读 HEAD `3f7954a`。许可是 MIT 变体：只允许与 Shopify 互通的应用；独立应用必须「外观明显不同于 Shopify 产品」。

## 架构

```
polaris-tokens/src/themes/base/{color,space,motion,shadow,font,border,height,width,zIndex}
        │  MetaTokenProperties { value, description? }
        ▼
themes/{light,dark,light-mobile,light-high-contrast}  覆盖子集
        ▼
构建产物：JS tokens / metadata / css/styles.css（--p-*）

stylelint-polaris
  color-no-hex
  custom-property-allowed-list / disallowed-list
  declaration-property-value-disallowed-list（如 font-weight: 400）
  media-query-allowed-list
```

- **Token 按组，不按组件。** color 别名是 `bg-fill-brand-hover` 这种场景名，不是 `Button.defaultHoverBg`。深色是另一份 theme 覆盖，不是运行时算法。
- **Motion 是时长阶 + 曲线名 + keyframes 名。** `motion-duration-100`…`350` 与 YoUI 鸿蒙档位碰巧同阶；曲线是普通 ease-in/out，**没有** YoUI 的标准/减速/加速/弹簧规格名。
- **Metadata 一等公民。** `description` 写清「用在 Page/Frame 背景」。构建同时吐 JS 值与 CSS 变量。
- **Lint 比「禁止硬编码色」更狠。** 连 `font-weight: 400`、自造 `--p-foo`、非白名单 media 都不过。白名单来自 `getThemeVarNames(themeDefault)`，不是手写 regex。全局 `--p-*` 只许 emit 过的名字；组件私有变量用 `--pc-*`，业务禁止定义 `--p-` / `--pc-`。
- **React 组件（历史）：** AppProvider / ThemeProvider 灌 CSS 变量。Button 已是 `variant`（外形）× `tone`（critical/success），与 Ant/Arco 两轴同构。本轮不把归档包当依赖。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| token 带 description / 使用场景 | adapt | `emit-theme.ts` 或设计系统文档可补「何时用 Fg2 / Surface2」 |
| 主题是浅/深/高对比覆盖，不是算法 | reuse-pattern | 与 YoUI `Colors` / `DarkColors` 同构 |
| stylelint 禁 hex、禁自造 `--*` | adapt | 对照现有 token 纪律脚本，可加「只允许 `--yohu-*`」 |
| lint 白名单从 emit 生成 | adapt | `getThemeVarNames` ↔ `emit-theme.ts` 吐一份 token-manifest，regex 只作兜底 |
| `--p-*` 全局 / `--pc-*` 组件私有 | adapt | 模块禁止再发明 `--yohu-*`；组件内部临时变量用另一前缀，且必须引用全局 token |
| `--p-` 前缀隔离 | reuse-pattern | 已有 `--yohu-` |
| motion 用规格名而不是裸 ms | reuse-pattern | YoUI 已走 `MotionSpec`；Polaris 的 50ms 阶不要加 |
| 场景色名 `bg-fill-brand-hover` 爆炸 | lesson-only | YoUI 语义层保持短名（Accent / AccentHover），交互态在 `states.css` |
| 归档 React 包 + Web Components 迁移 | lesson-only | 不要跟迁 WC；YoUI 锁 Solid |
| 许可限制独立应用外观 | anti-pattern | **禁止**抄 Polaris 色、间距、组件皮肤、`--p-*` 名 |
| `motion-duration-5000` | anti-pattern | 不是控件 token |

## 架构设计经验

- **Token 工程的产品是「值 + 元数据 + lint」，不是组件个数。** Ant/Arco 赢在控件面；Polaris 赢在不让业务写 hex。
- **深色用覆盖表，高对比再覆盖一层。** 不必上 Ant 的 `generate(theme:'dark')`。YoUI 若要高对比，加第三板，不要在运行时调亮度。
- **许可会卡视觉。** 开源 UI 库不都是「MIT 随便抄」。先读 LICENSE，再谈 adapt。

## 与当前工作

**能直接用**

- token 分组 + 浅/深覆盖 + `--yohu-` 前缀（已有）。
- lint 思路：组件 CSS 只许 `var(--yohu-*)`。

**必须改写**

- 若加强 lint，规则写进现有 `scripts/check-ui-tokens.mjs`，不要依赖 `stylelint-polaris` 包。
- 时长继续鸿蒙 `MotionSpec`，不要引入 Polaris 的 50ms 阶和 bounce keyframes。

**明确不要用**

- `@shopify/polaris` / `@shopify/polaris-tokens` 进产品依赖。
- 抄 Shopify Admin 视觉或 Web Components。
- 把「场景色名」扩成上百个 `bg-fill-*`。

## 阅读范围

`%TEMP%/YoAgentResearch/shopify--polaris`，HEAD `3f7954a`：

- `README.md`、`LICENSE.md`
- `polaris-tokens/README.md`
- `polaris-tokens/src/themes/types.ts`
- `polaris-tokens/src/themes/base/{color,motion}.ts`
- `stylelint-polaris/README.md`
- `stylelint-polaris/index.js`（`getThemeVarNames`）
- `stylelint-polaris/plugins/custom-property-allowed-list/{index.js,README.md}`
- `stylelint-polaris/plugins/custom-property-disallowed-list/index.js`
- `stylelint-polaris/utils/index.js`（`color-no-hex` 等规则名）
- `polaris-react/src/components/Button/Button.tsx`（`variant` × `tone`）

未读：`polaris-react` 各组件实现（已归档）、Web Components 新仓、文档站全文。
