---
id: research.harmony-fluent-inline-end-clip
type: topic-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, CSS, C#]
  frameworks: [fluentui, WinUI3, SolidJS]
also_relevant: [frontend]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repos:
    - microsoft/fluentui
    - microsoft/WinUI-Gallery
  url: https://github.com/microsoft/fluentui
  head: f51ba35
  cloned_to:
    - "%TEMP%/YoAgentResearch/microsoft--fluentui"
    - "%TEMP%/YoAgentResearch/microsoft--WinUI-Gallery"
studied_at: 2026-09-09
related:
  - research.microsoft-fluentui
  - research.microsoft-WinUI-Gallery
  - research.harmony-motion-system
  - research.harmony-apple-motion-spec-unification
---

# 贴右横向开合：Fluent Collapse × WinUI SplitView × CSS 插值

## 入选理由

Yohu 终端发送栏要「往右收缩、往左展开」。v1.92 用 `grid-template-columns: 0fr auto` ↔ `minmax(0, 1fr) 0fr`，WebView2 里**没有动画**，收起后还剩一条发丝空带。需要对照：谁把横向折叠做成可插值的尺寸裁切，以及我们自己已经跑通的 `rail` / `YoSwap`。

## 项目是什么

- **Fluent Collapse**（`react-motion-components-preview`）：横向 = 测 `scrollWidth`，关键帧走 **`maxWidth` + `overflowX: hidden`**，可选 opacity 第二轨。
- **WinUI SplitView**（Gallery）：同一根轴上 `CompactPaneLength` → `OpenPaneLength`；`PanePlacement=Right` 时从右缘长出。闭合不是卸 DOM。
- **Yohu 已有**：侧栏 `0` ↔ `var(--yohu-layout-shell-nav)`（同函数列表）；预览栏 `calc(...)` ↔ `0`；`YoSwap` 锁固有宽 + `justify-content: flex-end` + `width` 过渡。

## 架构

```
尺寸 atom（maxWidth / maxHeight，overflow 裁切）
  ∥ 空白 atom（padding/margin 收到 0，避免 0 尺寸仍占盒模型）
  ∥ 可选 fade atom
宿主保持挂载；闭合尺寸可以是 0 也可以是 compact 长度
```

Fluent `sizeEnterAtom`（clone `f51ba35`）：

1. 起点：`maxWidth = outSize`（默认 `0px`，测试里可用 `5px` 当 compact）+ `overflowX: hidden`
2. `offset: 0.9999`：仍 hidden，尺寸到测得的 `scrollWidth` px
3. 末帧：`maxWidth/overflow` 设回 `unset`，避免动画结束后把内容锁死

WinUI Gallery `SplitViewPage.xaml`：`CompactPaneLength` 绑定滑条，`DisplayMode` 含 CompactOverlay；右侧放置时 pane 仍是**同一控件变宽**，不是两列 XOR。

CSS Grid 能插值的前提（与 `rail` 相同）：**轨道数量不变，且每一对值的类型能插**。`0fr`↔`1fr` 可以；`auto`↔`0fr`、`0fr`↔`minmax(0,1fr)` 不行，浏览器当离散跳变。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 横向折叠 = 尺寸 + overflow 裁切 | reuse-pattern | 与 `YoSwap` / 预览槽同一模型 |
| 闭合尺寸用 compact 长度不是卸节点 | reuse-pattern | 把手 = SplitView compact / Fluent `outSize` |
| 内容锁在打开宽，从 inline-end 裁 | reuse-pattern | 动画系统 §9.2；禁止 `min-width:0` 把输入栏压扁 |
| 尺寸轨与透明度轨分开 | adapt | 宽度 `spatialPanel`；内容 `effectsEnter/Exit` |
| 两列 `0fr auto` ↔ `minmax 1fr 0fr` | anti-pattern | 函数列表不同，WebView2 不播 |
| 宿主 `container-type` 套在会变窄的裁切盒上 | anti-pattern | `100cqw` 变成把手宽，打开瞬间跳 |
| WAAPI / 每帧测 `scrollWidth` | anti-pattern | 产品约束 CSS-first；打开宽用祖先 `cqi` |

## 架构设计经验

- **空间运动走可插值长度**：`width` %↔px、`grid-template-columns` 里同构的 `0`↔`var(--sidebar)`。不要靠 `fr` 和 `minmax`/`auto` 混搭。
- **边缘是同一块铬**：闭合只露出贴 inline-end 的 compact 面（上+起边 hairline、起-起角 radius）；不要让 stretched 的 100% 宿主空着一条顶边框。
- 鸿蒙效率型：物体一直在布局里，用**标准曲线**持续运动（`spatialPanel`），不是减速入场的对话框。
- 发丝空带来自「列还在、只把内容 opacity:0」或「auto 列收不掉」。把手应 **absolute 叠在裁切盒上**，不要占第二列。

## 与当前工作

- **能直接用：** `width` 从 `--yohu-control-height` 到 `100%`；祖先 `container-type: inline-size`；第一子锁 `100cqi` 贴 end；overflow 裁切；双轨淡入。
- **必须改写：** 不引入 react-motion；不测 `scrollWidth`（打开宽 = 舞台 inline-size）。
- **不要用：** 纵向 `panel` XOR；模块 `@keyframes`；把 `container-type` 写在会变窄的 dock 上。

## 阅读范围

- `packages/react-components/react-motion-components-preview/library/src/components/Collapse/{collapse-atoms,collapse-types,Collapse}.ts`
- WinUI-Gallery `Samples/SplitView/SplitViewPage.xaml`（已有 clone）
- Yohu `shell.css` rail、`files.css` 预览列、`tokens/motion.css` `yohu-swap`
- 未读 Fluent 全仓；未把 WAAPI 关键帧抄进 YoUI
