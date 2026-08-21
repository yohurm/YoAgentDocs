---
id: rules.type.frontend
type: rule
status: draft
severity: should
scope: type
when: new-work
when_to_use: 新建或约束 Web 前端（页面应用）时
related: [rule.modification.frontend, rules.type.ui-kit]
---

# 前端类型包

页面、路由、数据获取。可复用控件走 [ui-kit](../ui-kit/)，不要和页面逻辑混在一篇规则里。

## 默认边界

- 跟随仓库已有框架与目录；不平行引入第二套 UI 库或状态库。
- 样式跟现有 token；没有设计系统时模仿邻近页面。

## UI 与实现分层

```
页面组件 → 仓库已有的 store / service → API 客户端 → 后端
```

- 页面不直访后端细节，禁止在组件里散落请求与持久化。
- 层名跟该前端仓库走。只有仓库自己声明了 MVVM 时才用 ViewModel 这个词；默认叫 store / service。
- 可复用控件走 [ui-kit](../ui-kit/)，不要把页面 store 塞进组件库。

对照 [stack-layering.md](../../common/stack-layering.md)。

## 质量门禁

- 主用户路径可键盘或指针走通；加载、空态、错误态与周围页面一致。
- 能打开开发服务器或已构建页面时，对改动路径做一次实际浏览（浏览器）。纯文档仓库除外。
- 不把密钥和仅服务端才该有的逻辑放进前端包。

## 不要默认引入

- 新的 CSS 方案、新的路由库、与现有构建链冲突的打包工具。
