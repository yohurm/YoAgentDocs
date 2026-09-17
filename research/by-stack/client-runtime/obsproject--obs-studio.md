---
id: research.obsproject-obs-studio
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C, HLSL]
  frameworks: [libobs, D3D11]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, adapt, anti-pattern]
source:
  platform: github
  repo: obsproject/obs-studio
  url: https://github.com/obsproject/obs-studio
  head: caaa022
  cloned_to: "%TEMP%/YoAgentResearch/obsproject--obs-studio"
studied_at: 2026-09-15
related:
  - research.desktop-mirror-downsample
  - research.haasn-libplacebo
---

# obsproject/obs-studio

## 入选理由

把 **Scale 做成具名滤镜**，不写进编码、也不写进场景合成器。`dest < src/2` 时（除非 POINT）强制换 8 点 `BILINEAR_LOWRES`。Yohu 真机 0.37× 正好掉进这档。面积核存在，但 **>2× 缩小时会被低分双线性盖掉**——这是反例。

## 项目是什么

直播/录制工作台。源 → 滤镜链 → 场景合成 → 画布 → 输出缩放。三处都会缩，但互不替代。

## 架构

```text
源 ── filters[0]（可含 scale_filter）──► 场景 item 变换
画布 ── obs-video 输出缩放 ──► 推流/录像
```

`scale_filter` 注册为 `OBS_SOURCE_TYPE_FILTER`（`plugins/obs-filters/scale-filter.c` 576–591）。核是枚举：Point / Bilinear / Bicubic / Lanczos / Area。关键分支（215–218）：

```text
lower_than_2x = cx_out < cx/2 || cy_out < cy/2
if (lower_than_2x && sampling != POINT)
    type = OBS_EFFECT_BILINEAR_LOWRES
```

效果文件 `libobs/data/bilinear_lowres_scale.effect` 36–55：用 `ddx/ddy` 足迹上 8 个偏移（1/16…7/16）平均，注释 *Simulate Direct3D 8-sample pattern*。`area.effect` 37–86 才是按相交面积加权的盒滤。场景变换（`obs-scene.c` 766–774）同样 `<0.5` 先低分双线性，Area 只在未触发时用。

滤镜中间 RT：`gs_texrender_create` **不** 建 mip。缩小质量来自 shader，不是 `GenerateMips`。

输出缩放（`obs-video.c` 227–228）用 **两轴都** `< 半画布` 才低分，谓词与滤镜的 OR 不同——同一产品三处阈值都不统一，说明「核策略」不该和「dest 政策」焊死。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| Scale 是具名步骤，不是 Present | reuse-pattern | Yohu 核不进 `gpu.rs` 神文件 |
| >2:1 必须覆盖足迹 | reuse-pattern | MiniEngine / OBS 8 点 / Yohu 3×3 同意图 |
| 不建 mip 链 | reuse-pattern | 与 ADR-v6-027 一致 |
| `dest<src/2` 静默丢掉 Area | anti-pattern | 0.37× 正该用面积核；OBS 换成 8 点双线性 |
| 滤镜分辨率与核绑在同一插件 UI | anti-pattern | dest 政策（contain）与核要分模块 |
| GPL-2 整段搬 effect | anti-pattern | 只借结构，不拷着色器 |

## 架构设计经验

1. **编码分辨率、场景变换、输出缩放是三件事。** Yohu 编码已是 session；Fit 是 contain；Scale 是第三步。
2. **>2:1 的默认换核是性能启发式，不是质量最优。** 嵌入槽不要抄这条 override。
3. **POINT 例外说明「最近邻」是显式选择。** Yohu 只在 1:1 / 整数放大用 nearest。

## 与当前工作

- 能直接用：核独立；无 mip；>2:1 盖足迹。
- 必须改写：Yohu 已选面积核，禁止再自动切低分双线性。
- 不要用：把 scale-filter 当占用卡片；搬 GPL 效果文件。

## 阅读范围

`plugins/obs-filters/scale-filter.c` 143–264、397–443、576–591；`libobs/data/bilinear_lowres_scale.effect`；`libobs/data/area.effect` 37–86；`libobs/obs-scene.c` 743–791；`libobs/obs-video.c` 219–243。
