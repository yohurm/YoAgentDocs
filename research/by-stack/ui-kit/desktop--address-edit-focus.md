---
id: research.desktop-address-edit-focus
type: project-study
status: active
when: research
stack:
  capability: ui-kit
  languages: [TypeScript, C#]
  frameworks: [Chromium, WinUI3]
also_relevant: [client-runtime]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: files-community/Files
  url: https://github.com/files-community/Files
  head: 51bf632
  cloned_to: "%TEMP%/YoAgentResearch/files-community--Files"
studied_at: 2026-09-11
related: [research.files-community-Files, research.desktop-workbench-command-band]
---

# 地址栏展开：铬边界 / 全选 / 收回

## 入选理由

Yohu 文件路径栏「点空白进编辑」在 WebView2 里默认全选，且输入盒外的整栏仍被当成路径栏。需要对照资源管理器、Files Omnibar 与 Chromium 指针时序，而不是再在视图里补一次 `setSelectionRange`。

## 项目是什么

跨源对照：Windows 资源管理器地址栏（规范行为）、files-community/Files Omnibar（WinUI 源码）、Chromium「focus 后 mouseup 全选」缺陷。

## 架构

### 铬边界（Explorer / Files）

资源管理器：地址**铬**是一条有背景的栏。点铬内空白（面包屑右侧、仍在栏里）才进编辑；点栏外无事。Esc / 再点空白收回。

Files `Omnibar.xaml`：整条 Omnibar 是带边框的 TextBox 铬，不是「面包屑 hug + 后面一整条幽灵热区」。模式钮热区是固定宽 `OmnibarModeDefaultClickAreaWidth`（46），不 stretch 吃剩余。

Yohu 若输入盒 hug、槽却 `flex:1`，盒外空白在命中与视觉上都还是「路径栏」——和上述铬边界相反。

### 展开全选

Files `Omnibar.Events.cs` `GotFocus` 里主动 `_textBox.SelectAll()`。那是 **Files 的产品选择**（对标浏览器地址栏首次聚焦全选），不是 Web 默认。Yohu 用户不要预选。

Chromium / WebView2：在 **mousedown 期间**插入并 `focus()` 一个 `<input>`，随后的 **mouseup 落在该 input 上会改选区**（常见成全选或拖选）。Stack Overflow「Chrome select on focus」的稳妥做法：

1. 打开手势的 `pointerdown` 上 `preventDefault`（打断后续对新生 input 的默认选区）。
2. 输入层在手势结束前 `pointer-events: none`。
3. **`pointerup` 之后**再 focus，并按策略放光标。
4. 打开当次的 `mouseup` 若仍可能打到 input，再 `preventDefault`。

只在 `focus` 后立刻 `setSelectionRange` **挡不住** 随后的 mouseup。

Files 改文本时若来自建议选中，用 `Select(length, 0)` 把光标放到末尾（`Omnibar.cs` `ChangeTextBoxText`），与「不预选」一致。

### 收回与文字

铬宽度 = 当前可见盒子（浏览=面包屑簇，编辑=输入盒）。收回只裁这个盒子；盒子外的顶栏空白不是路径栏，不参与 clip。面包屑在编辑/收回期间退出文档流（不要 `visibility:hidden` 还占满槽），clip 播完再卸输入、再挂面包屑，避免字先没、框还在，或字在 hug 盒里裁、槽却仍是整栏。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 路径铬 = 可见盒子 | reuse-pattern | 盒外不是路径栏；点盒外取消编辑 |
| 打开手势与 focus 错开 | reuse-pattern | pointerup 后再 focus；挡 Chromium mouseup 全选 |
| Files GotFocus SelectAll | anti-pattern | 不要抄进 Yohu |
| 热区固定宽、不 stretch | adapt | 面包屑后只留一小段可点空白 |
| 收回只 clip 当前盒 | reuse-pattern | 文字与边框同一盒、同一条 clip |

## 架构设计经验

地址栏有两个几何：行（上级+槽，铺满顶栏）和铬（槽内可见控件）。命中、取消、clip 只认铬。指针手势与选区是 L3 策略，不是视图里再调一次 `select()`。

## 与当前工作

- **能直接用：** 铬 hug；打开门闩到 pointerup；取消看 field.contains；收回 clip 只打 field。
- **必须改写：** Files 的 SelectAll / 全宽 TextBox 铬。
- **不要用：** 槽 `flex:1` 当路径栏；`visibility:hidden` 面包屑继续占满槽；打开当帧 focus。

## 阅读范围

Files `Omnibar.xaml` / `Omnibar.cs` / `Omnibar.Events.cs`（HEAD `51bf632`）。Chromium 全选：公开讨论「Chrome select on focus / mouseup preventDefault」。资源管理器：微软「Navigate using the address bar」（点栏内空白进编辑）。
