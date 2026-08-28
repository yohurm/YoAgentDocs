---
id: research.barry-ran-QtScrcpy
type: project-study
status: active
when: research
stack:
  capability: client-runtime
  languages: [C++]
  frameworks: [Qt, FFmpeg, OpenGL]
also_relevant: [windows-desktop]
utilization: [lesson-only, anti-pattern]
source:
  platform: github
  repo: barry-ran/QtScrcpy
  url: https://github.com/barry-ran/QtScrcpy
  cloned_to: "%TEMP%/YoAgentResearch/barry-ran--QtScrcpy"
studied_at: 2026-08-25
related:
  - research.synthesis.client-runtime
  - research.Genymobile-scrcpy
---

# barry-ran/QtScrcpy

## 入选理由

把 scrcpy **嵌进桌面工作台**做得最完整的开源 GUI：设备列表、一键启停、侧栏导航键、多设备、置顶/全屏/只录不显示。~31k star，`dev` HEAD 2026-08-20。Yohu 要的不是 Qt/FFmpeg，而是「工作台里投屏模块有哪些按钮、哪些是产测需要的」。先前因与 scrcpy 拖入同构而落选；本轮主题换成投屏产品清单，重新入选。

## 项目是什么

用 Qt 重写 scrcpy **客户端**（解码仍 FFmpeg，渲染 OpenGL），server 仍是官方 jar 换文件。另有商业「极限投屏」做 500 台批量，开源版覆盖单机调试/游戏键位。

## 架构

```
Dialog（设备表 + 启动参数）
    → IDeviceManage 启停多路
        → AdbProcess（adb 命令）
        → 推 scrcpy-server + 隧道 + app_process
        → VideoForm（独立 Qt 窗口，不是嵌入主 Dialog）
            ToolForm 悬浮条：全屏 / 通知栏 / 旋转 / 触摸开关 /
            亮屏熄屏 / Power / 音量 / 多任务 / 菜单 / Home / 返回 / 截图
        → QYUVOpenGLWidget
```

`dialog.ui` 启动项：`maxSize` 640–1920/原始、码率、后台录制、窗口置顶、stay awake、Reverse 开关（多设备 forward 失败时关掉 reverse）。`encoderpreset.cpp` 用 VBR 档位省电，不改 server。

画面在 **独立 VideoForm 窗口**，主窗口只做设备与参数。这和 Yohu「单一内容区模块页」不同：他们是「控制台 + 每设备一个原生窗」。

## 利用价值

| 点 | 方式 | 说明 |
|----|------|------|
| 工具条能力清单（Home/Back/Power/音量/截图/熄屏保持镜像） | lesson-only | 映射到 `@yohu/ui` 按钮，不要抄 Qt 窗 |
| 启动前参数：max-size、码率、Reverse、stay awake | lesson-only | 设置项或页眉下拉；默认值走产测保守档 |
| 多设备「停止所有服务」 | lesson-only | Yohu v1 建议单设备；多路是远期 |
| 独立 VideoForm + FFmpeg/OpenGL | anti-pattern | 体积；且画面离开工作台 |
| 游戏键位映射 / 群控 | anti-pattern | 超出产测工作台；有文件/终端模块可发命令 |
| sndcpy 音频（README 已不推荐） | anti-pattern | 音频放远期；不要第二套 sidecar |

## 架构设计经验

- **产品清单 ≠ 渲染栈。** QtScrcpy 证明「工作台需要导航键和截图」，同时证明「换 UI 框架就要自己写客户端」——Yohu 已经有 WebView，不要再引入 Qt。
- **主窗管会话，画面窗管像素。** 他们拆成 Dialog / VideoForm。Yohu 应对齐为：壳设备栏 + 模块 store 管会话，面板里的 canvas 管像素；不要为每台设备再开 Tauri 窗口（需求已写「撕出独立 OS 窗口」为远期）。
- **Reverse 是可关的兼容开关。** 多设备或某些 ROM 上 reverse 失败；设置里留「强制 forward」比静默重试更可解释。
- **截图是客户端动作。** `device->screenshot()` 在 ToolForm，不经过再开一路 `screencap` 也行（抓当前帧）。WebCodecs 路径等价于 canvas 导出。

## 与当前工作

- 能直接用：功能分层时对照 ToolForm 按钮；启动参数集合；熄屏保持镜像作为 P1。
- 必须改写：所有控件进 `YoChrome` / 面板工具条；会话状态进 core 槽位；无独立 SDL/Qt 窗。
- 不要用：Qt、FFmpeg 渲染、群控、键位脚本、商业极限投屏的规模指标当本期验收。

## 阅读范围

`README_zh.md` 功能/界面解释、`QtScrcpy/ui/toolform.ui`+`.cpp`、`dialog.ui`/`dialog.cpp` 启动参数、`encoderpreset.cpp`、`videoform.cpp` 全屏相关注释。未读 `QtScrcpyCore` 里 FFmpeg 解码实现。
