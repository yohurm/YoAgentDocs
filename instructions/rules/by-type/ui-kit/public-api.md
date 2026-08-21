---
id: rules.type.ui-kit.public-api
type: rule
status: active
severity: must
scope: type
when: always
when_to_use: 增加或修改 Yo 组件对外接口时
related: [rules.type.ui-kit.file-srp, rules.type.ui-kit.lifecycle]
---

# 对外 API

## 必须

- 宿主只依赖 `YoXxx`（及必要的公开类型/枚举）。实现类型不出现在 `api/` 或包入口。
- **L5 门面要薄。** `api/` 是接口（或同等薄转发）：枚举、公开数据、`create()` / 静态转发、`destroy` 与浮层呈现回调。禁止在 `api/` 写几何拟合、Path / Outline / Drawable、手势、测量算法或资源释放顺序。
- `api/` 只允许依赖同能力的装配类（`RippleImpl`、`CornerImpl`、`TabsImpl`）。**禁止** `import …internal`。
- `api/`（或 `index.ts` 的公开面）禁止补丁类、兼容别名、实验目录。过渡代码放 internal，做完即删。
- 一种组件一条主入口。多种预设 = 参数/建造配置，不是平行的 `YoXxxWarning`、`YoXxxList` 实现树。
- 对外效果（光感、模糊、Loading）也是 Yo 组件或 Yo 能力，不标注「实验室 / 某某项目样式」。

## 应当

- 公开方法表达意图（`barStyle`、`overlap`），不把内部层名漏出去（除非那就是稳定契约）。
- 新增导出时同步登记该仓库的组件清单或设计文档；本知识库不维护具体色值表。

## 反例

- `api/feedback/dialog` 下的一次性适配类。
- `api/common/YoCorner` 写成带 Rosen 拟合、`writePath`、并 `import widget.common.corner.internal.*` 的工具类。正确：`YoCorner` 接口 + `CornerImpl` + `internal/`。
- 把 `ImmersiveTier` 等内部枚举散落成五个顶层 API 文件，而宿主只需 `YoImmersiveLight`。
- 模块里 `class="自写卡片"` 绕过 `YoPanel`。
