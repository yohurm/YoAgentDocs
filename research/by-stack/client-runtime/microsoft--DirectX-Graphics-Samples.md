---
id: research.microsoft-DirectX-Graphics-Samples
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [HLSL, C++]
  frameworks: [D3D12, MiniEngine]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: microsoft/DirectX-Graphics-Samples
  url: https://github.com/microsoft/DirectX-Graphics-Samples
  head: 213dd4f
  cloned_to: "%TEMP%/YoAgentResearch/microsoft--DirectX-Graphics-Samples"
  note: sparse MiniEngine/Core/Shaders
studied_at: 2026-09-15
related:
  - research.desktop-mirror-downsample
---

# microsoft/DirectX-Graphics-Samples（MiniEngine mip）

## 入选理由

先前主题篇只引用网上的 `GenerateMipsCS.hlsli` 文件头。本轮 sparse 克隆核对原文。作者 James Stanard 写明：**缩超过 2× 时一次双线性不够**。这是 mip 路径自己的警告，也是「不要用 GenerateMips 当对准 dest 的缩小器」的源码锚点。MIT。

## 项目是什么

MiniEngine 预计算 mip 链：每级约 1/2，奇数边用 `NON_POWER_OF_TWO` 1/2/3 多拍。不是视频呈现架构，是 **反例 + 足迹定理**。

## 架构

`GenerateMipsCS.hlsli` 77–113：

```text
One bilinear sample is insufficient when scaling down by more than 2x.
```

| `NON_POWER_OF_TWO` | 含义 | 取样 |
|--------------------|------|------|
| 0 | 2 的幂 | 1 次双线性 |
| 1 | X 向 >2:1 | 2 次 |
| 2 | Y 向 >2:1 | 2 次 |
| 3 | 两轴 >2:1 | 4 次平均 |

随后把结果写入 OutMip1，再递归生成更低级。三线性采样会在两级 mip **之间**插值。Yohu 2.73× 若走这条链，lod≈1.45，混 1/2 与 1/4 级——比 dest 还软。这是已否决通路的源码证明。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| >2:1 必须多拍盖足迹 | reuse-pattern | 面积 3×3 / OBS 8 点 / MiniEngine 4 拍 |
| GenerateMips 当缩小器 | anti-pattern | 预计算 1/2 金字塔，不准 dest |
| 只在 POT 纹理上建 mip | lesson-only | 手机帧 1220×2712 不是 POT |

## 架构设计经验

1. **mip 是为「任意远距离采样」预计算的，不是为某一个 dest。**
2. **连 mip 作者都承认单拍双线性在 >2:1 会欠采样。** VP 一次双线性压到 clip 同一错误。

## 与当前工作

- 能直接用：>2:1 多拍定理。
- 不要用：`GenerateMips`、三线性、LOD bias 补锐。

## 阅读范围

`MiniEngine/Core/Shaders/GenerateMipsCS.hlsli`；`GenerateMipsLinearOddCS.hlsl`（`NON_POWER_OF_TWO 3`）。未读 MiniEngine 引擎其余部分。
