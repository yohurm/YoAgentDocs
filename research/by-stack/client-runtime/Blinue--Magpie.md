---
id: research.Blinue-Magpie
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++, HLSL]
  frameworks: [D3D11, WinUI]
also_relevant: [windows-desktop]
utilization: [reuse-pattern, anti-pattern]
source:
  platform: github
  repo: Blinue/Magpie
  url: https://github.com/Blinue/Magpie
  head: 8718e50
  cloned_to: "%TEMP%/YoAgentResearch/Blinue--Magpie"
studied_at: 2026-09-15
related:
  - research.desktop-mirror-downsample
  - research.haasn-libplacebo
---

# Blinue/Magpie

## 入选理由

Windows 上把 **窗口 Fit** 和 **具名效果链** 拆开的 D3D11 放大工具。效果是 MagpieFX（计算着色器），输出尺寸由 `ScalingType` 算，与效果名无关。Yohu 主路径是缩小，不搬 Anime4K/FSR；要的是「几何枚举 ≠ 核文件」。GPL-3，禁止 vendoring。

## 项目是什么

把游戏窗抓到交换链，按效果链放大（也可窗口模式）。捕获、效果、叠加层、光标插值是不同模块。

## 架构

```text
Capture（GraphicsCapture / DDA / …）
  → EffectDrawer 链（每个 EffectOption.name + ScalingType）
      _CalcOutputSize：Normal | Fit | Absolute | Fill
  → 若输出仍大于交换链：_AppendBicubic（再挂一个 Bicubic 效果）
  → Present / Overlay
```

`ScalingType`（`ScalingOptions.h` 58–63）：Normal=倍数；Fit=相对屏幕最大等比；Absolute=像素；Fill=铺满。`EffectDrawer::_CalcOutputSize`（`EffectDrawer.cpp` 300–350）只算 `SIZE`，不选核。效果名在 `EffectOption.name`，由 `EffectCompiler` 编译 MagpieFX。

缩小是 **另挂一个 Bicubic 效果**，不是改 Fit（`Renderer.cpp` 591–606、653–664）：全屏时若效果输出大于交换链就 `_AppendBicubic`，`scalingType = Fit`。窗口模式则可能 Fill + 追加 Bicubic。几何政策与「用哪个 .hlsl」始终是两个字段。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| ScalingType 与 Effect.name 分字段 | reuse-pattern | Yohu：`present_dest` vs `scale` 模块 |
| 缩小是链上多一节，不是改窗口算法 | reuse-pattern | Convert 之后再 Scale |
| Anime4K / FSR / CRT | anti-pattern | 游戏放大；嵌入槽 0.37× 不需要 |
| MagpieFX 编译器 + 缓存 | anti-pattern | GPL-3；Yohu 一个面积核即可 |
| 窗口模式把 Fit×1 当 Fill | anti-pattern | 注释承认会吃掉黑边；Yohu 占用必须锁比例 |

## 架构设计经验

1. **Fit 枚举可以没有核名。** 换 Bicubic 参数不改 `ScalingType::Fit`。
2. **效果链是有序图。** 捕获尺寸 → 效果输出尺寸 → 必要时再降采样效果 → 交换链。Yohu 更短：crop RGB → 一节 Scale → dest。
3. **追加缩小效果的条件是「输出大于呈现表面」。** 与 dest 政策分离。

## 与当前工作

- 能直接用：几何类型与核文件分开；缩小是独立 pass。
- 必须改写：Yohu 核是面积平均，不是 Bicubic B=0 C=0.5；dest 永远 contain，不 Fill。
- 不要用：MagpieFX、FSR、把占用改成铺满 avail。

## 阅读范围

`src/Magpie.Core/include/ScalingOptions.h` 58–74；`EffectDrawer.cpp` 300–350；`Renderer.cpp` 580–664；`LICENSE` GPL-3。未读完所有内置效果 hlsl。
