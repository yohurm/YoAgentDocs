---
id: research.ant-design-ant-design
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript]
  frameworks: [React, CSS-in-JS]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern, lesson-only]
source:
  platform: github
  repo: ant-design/ant-design
  url: https://github.com/ant-design/ant-design
  head: 636164932bb51898fb61be1b8a4b46077ae1c7d9
  cloned_to: "%TEMP%/YoAgentResearch/ant-design--ant-design"
studied_at: 2026-09-10
related: [research.synthesis.ui-kit, research.microsoft-fluentui, research.radix-ui-primitives]
---

# ant-design/ant-design

## 入选理由

Yohu 要把 YoUI 从「够用的桌面控件」升成完整设计体系。Ant Design 是用户指定的主参考：有官方设计价值观、Seed→Map→Alias 派生、ConfigProvider 子树主题、统一浮层入口、以及 Button/Input/Select 的复合 API。本仓是 React + CSS-in-JS，**只抄分层与契约，不抄运行时**。已有 Fluent / Radix / Spectrum 切片只覆盖动效与表头，没有覆盖企业级 token 与复合控件。

## 项目是什么

蚂蚁集团体验技术部的企业级设计语言 + React 组件库（`antd`，阅读时 HEAD 为 2026-09-10 `6361649`，版本线 v6.6.x）。规范在 [ant.design](https://ant.design/docs/spec/introduce-cn)：价值观是自然 / 确定性 / 意义感 / 生长性。实现是 `components/` 单仓，主题在 `components/theme/`，样式走 `@ant-design/cssinjs`，浮层委托 `@rc-component/trigger` / `tooltip` / `select`。

## 架构

```
官方规范（价值观 / 色板 / 模式）
        │
SeedToken（设计师意图：colorPrimary、fontSize、borderRadius、controlHeight、sizeUnit/sizeStep、motionUnit）
        │  algorithm（default / dark / compact 可组合）
        ▼
MapToken（10 阶色板、字号梯度、尺寸梯度、controlHeightSM/LG、motionDuration*、radius 阶）
        │  formatToken（alias.ts）
        ▼
AliasToken（开发者名：colorTextPlaceholder、controlItemBgHover、colorSplit…）
        │
        ├── ComponentToken（每组件 prepareComponentToken，如 Button.defaultHoverBg）
        ├── ConfigProvider.theme（token / algorithm / components / cssVar / hashed / zeroRuntime）
        └── useToken → cssinjs 注入；v5.12+ 可 cssVar；v6 可 zeroRuntime 预生成 CSS
```

- **Seed 注释写死「DO NOT MODIFY… CONTACT DESIGNER」**（`interface/seeds.ts`）。`colorTextBase` / `colorBgBase` 是派生底，禁止业务直接用。
- **default algorithm**（`themes/default/index.ts`）：`@ant-design/colors.generate` 出 10 阶 → `genColorMapToken` / `genFontMapToken` / `genSizeMapToken` / `genControlHeight` / `genCommonMapToken`。
- **尺寸公式**：`size = sizeUnit * sizeStep`（默认 4×4=16）；`controlHeightSM/XS/LG = controlHeight × 0.75/0.5/1.25`。
- **紧凑算法**：以 `fontSizeSM` 为新字号底，`controlHeight - 4`，再跑 compact size map。
- **暗色算法**：同 seed，色板 `generate(..., { theme: 'dark' })`，只换颜色映射，尺寸/圆角沿用 default map。
- **动效**：`motionDurationFast/Mid/Slow = (motionBase + motionUnit × 1/2/3)s`，默认 0.1 / 0.2 / 0.3s。`seeds.ts` 自带 TODO：缺人收敛 Motion Token。`motion: false` 时 alias 把三段时长写成 `0s`。
- **ConfigProvider**：全局 `getPopupContainer`、`componentSize`、`componentDisabled`、`locale`、`prefixCls`，以及按组件名的配置袋（`button` / `select` / `modal`…）。嵌套 `theme.inherit` 默认 true。
- **静态方法陷阱**：`Modal.xxx` / `message.xxx` 自己 `ReactDOM.render`，拿不到外层 context。官方补丁是 `App` 挂 `useMessage` / `useModal` / `useNotification` 的 `contextHolder`。
- **浮层**：Tooltip/Select 都走 `@rc-component/*`。`_util/placements.ts` 用 `adjustX/Y` + `shiftX/Y` 做溢出。`Tooltip.UniqueProvider` 把多个 tooltip 收成**一个**共享 popup（`@rc-component/trigger` 的 UniqueProvider），避免 N 个 Portal。
- **Button**：`type` 是旧快捷键，真实轴是 `color × variant`（`ButtonTypeMap`：`primary → [primary, solid]`，`default → [default, outlined]`）。另有 ComponentToken 几十个（默认钮各态色、阴影、iconGap）。
- **Input**：`prefix`/`suffix` 在字段内；`addonBefore`/`addonAfter` 在字段外（group）；`status=error|warning` 一等公民；`variant` 含 outlined / filled / borderless。
- **Select**：皮肤层，逻辑在 `@rc-component/select`。语义槽拆 `root/prefix/suffix/input/popup.listItem`。吃 Form `status` 与 Compact 上下文。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Seed → Map → Alias 三层派生 | reuse-pattern | 与 YoUI 已有 Primitive→Semantic→Component **同构**。业务只许碰语义/组件层 |
| Seed 只给设计师、禁止业务读 Base | reuse-pattern | 对齐 `seeds.ts` 对 `colorTextBase`/`colorBgBase` 的禁令 |
| 算法可组合（dark + compact） | adapt | YoUI 已有 theme × density 两轴；保持两轴正交，不要把密度写进色板 |
| `getDesignToken(config)` 纯函数导出全量 alias | adapt | 对照 `emit-theme.ts`：派生必须可单测、可写磁盘，不要运行时算 |
| ConfigProvider 子树主题 + 组件级 override | adapt | YoUI 今天只有全局 `setTheme`/`setDensity`。设置页预览、局部强调可以要一棵 Solid context，但默认仍全局 CSS 变量 |
| `getPopupContainer` 单入口 | adapt | Select / 右键 / 未来 Tooltip 共用一个挂载点（对照已有唯一 `YoContextMenuHost`） |
| Tooltip UniqueProvider（共享一个 popup） | adapt | 工具栏密集图标提示不要每钮一个 Portal |
| Button `color × variant` 两轴 | adapt | 现在 `YoButtonVariant` 把色和形混在一起（primary/secondary/ghost/danger） |
| Input `prefix/suffix` 内、`addon*` 外、`status` | adapt | `YoTextField` 已有 clearable，缺前缀图标与 error/warning |
| `App` / hooks 替代静态 `message.xxx` | reuse-pattern | 已有 `createToaster`；禁止再加脱离树的命令式 Modal |
| `@ant-design/colors` 从主色生成 10 阶 | anti-pattern | YoUI Primitive 锁鸿蒙官方表，禁止用品牌色算法另造色板 |
| CSS-in-JS + hash class + 运行时注入 | anti-pattern | YoUI 是 TS 常量 → `theme.css`；v6 `zeroRuntime` 只说明他们也在往静态退 |
| Wave 点击波纹、中文自动插空格 | anti-pattern | 桌面工作台不要网页营销反馈 |
| `type` 与 `color/variant` 双 API 并存 | anti-pattern | 不要再留一套兼容快捷键 |
| Form 受控校验引擎 / Table 全家桶 | lesson-only | 设置页继续 `YoFormRow`；清单继续 `YoCol*` + `YoVirtualList` |
| 动效 Fast/Mid/Slow = 100/200/300ms | anti-pattern | YoUI `MotionSpec` 已锁鸿蒙；Ant 自己都承认 Motion Token 未收敛 |
| 设计价值观四条 | lesson-only | 评估语言可参考「克制 / 模块化 / 即时反馈」；产品原则仍以 UI 设计系统 8 条 + 鸿蒙为准 |

## 架构设计经验

- **派生关系比变量个数重要。** 改 `borderRadius` 必须带动 XS/SM/LG；改 `controlHeight` 必须带动 SM/LG。手写两套深浅板可以，但尺寸/圆角/控件高必须是公式，不能各组件私自写 28/32/40。
- **皮肤与行为拆仓。** antd 的 Select/Tooltip/Modal 大多是皮肤 + token；行为在 `@rc-component/*`。YoUI 已走这条：`select-model` / `col-model` / `context-menu` 与 `Yo*` 视图分开。继续保持，不要把定位算法写进 Button.css。
- **命令式 API 必须能挂回树。** `App` 的存在证明：脱离 React 树的 `Modal.confirm` 会丢主题、尺寸、locale。YoUI 的 Host（Toaster / ContextMenuHost）是正确形态。
- **组件 token 是第三层，不是第二套全局色。** Button 的 `defaultHoverBg` 从 alias 算出来，允许 ConfigProvider `components.Button` 覆盖，但不允许页面写 hex。
- **浮层 z 与挂载点是全局政策。** `zIndexPopupBase=1000` + `getPopupContainer`。YoUI 若再加 Tooltip/Popover，必须进同一政策，不能各写 `z-index: 9999`。

## 与当前工作

**能直接用**

- 三层 token 纪律（业务禁碰 Primitive / Seed）。
- 尺寸与控件高用公式派生；密度只改 seed（或已有 density 轴），不改色板。
- 浮层单 Host + 单 `getPopupContainer`。
- Button 色/形两轴；TextField 内缀 + status。
- Toast / Dialog 继续走树内 Host，不写静态 `xxx.info()`。

**必须改写**

- 算法输出写进现有 `tokens/*.ts` + `emit-theme.ts`，不要 `@ant-design/cssinjs`。
- 色板数字继续鸿蒙官方表，不用 `generate(#1677ff)`。
- 时长/曲线继续 `MotionSpec`（100/150/160/200/300/350 + 标准/减速/加速/弹簧），不抄 Ant `motionEaseOutCirc`。
- ConfigProvider 若做，用 Solid context + `data-theme` / CSS 变量，不要 hash class。
- Tooltip 用已有 `popover-place` + `YoPresence`，不要 `@rc-component/trigger`。

**明确不要用**

- 把 `antd` 或 `@ant-design/cssinjs` 引进 `@yohu/ui`。
- 抄 `#1677ff` / 6px 默认圆角 / 32px 控件高当产品默认（YoUI 已是鸿蒙 PC + comfortable）。
- 做 YoTable / YoForm 引擎 / DatePicker / Upload。
- Wave、中文插空格、`autoInsertSpace`。
- 为「局部主题」在运行时重算整份色板。

## 阅读范围

官方：`https://ant.design/docs/spec/introduce-cn`、`values-cn`、`colors-cn`、`https://ant.design/docs/react/customize-theme-cn`。

源码（`%TEMP%/YoAgentResearch/ant-design--ant-design`，HEAD `6361649`）：

- `README-zh_CN.md`
- `components/theme/themes/seed.ts`
- `components/theme/interface/seeds.ts`
- `components/theme/themes/default/index.ts`
- `components/theme/themes/dark/index.ts`
- `components/theme/themes/compact/index.ts`
- `components/theme/themes/shared/{genSizeMapToken,genControlHeight,genRadius,genCommonMapToken}.ts`
- `components/theme/util/alias.ts`
- `components/theme/{useToken,context,getDesignToken}.ts`
- `components/config-provider/index.tsx`（`ConfigProviderProps`）
- `components/button/{Button.tsx,style/token.ts}`
- `components/input/{utils.ts,style/variants.ts,demo/status.tsx,demo/presuffix.tsx}`
- `components/select/index.tsx`
- `components/tooltip/{index.tsx,UniqueProvider/index.tsx}`
- `components/_util/placements.ts`
- `components/app/App.tsx`

未读：Table / Form / DatePicker / Upload 实现全文、`@ant-design/cssinjs` 独立仓、图标仓、官网 dumi 站。
