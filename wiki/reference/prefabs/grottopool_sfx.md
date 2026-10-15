---
title: "洞穴水池瀑布声发射器（grottopool_sfx）"
created: "2026-11-02"
updated: "2026-11-02"
type: prefab
summary: "洞穴水声控制器使用的非联网声音实体：构造、移动淡变及清理边界。"
aliases: ["grottopool_sfx", "洞穴水池声音", "格罗托瀑布声发射器"]
tags: [prefab, fx]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/grottopool_sfx.lua", "components/grottowaterfallsoundcontroller.lua", "components/fader.lua", "prefabs/cave.lua", "prefabs/grotto_pool_big.lua", "prefabs/grotto_pool_small.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 洞穴水池瀑布声发射器（grottopool_sfx）

## 快速判断

- 单一声音载体，没有动画和网络实体；工厂只装配实体与 `fader`，不会自行播放声音。控制器生成并复用它来播放水池瀑布循环声；筛选与分配流程见 [[reference/components/grottowaterfallsoundcontroller#关键接口]]。[工厂](../../../raw/current/scripts/prefabs/grottopool_sfx.lua#L1-L20)、[播放](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L137-L142)
- 本页只覆盖一个简单工厂与其直接调用链，采用 simple 档；中文为按用途核验的效果描述，非官方译名。简中 `languages/chinese_s.po` 定向检索注册名的 `STRINGS.NAMES` 键未命中。

## 注册与构造

| 注册名 | 工厂与构造 | 差异与证据 |
|---|---|---|
| `grottopool_sfx` | `Prefab` 返回 `grottopool_sfx` 工厂；创建 Transform、SoundEmitter 和 `fader` | 带 `CLASSIFIED`／`FX`，注释为非联网实体；显式 `persists = false`。`SetCanSleep` 是注释，不能当作已禁休眠；无动画、网络组件及工厂资源表。[工厂与注册](../../../raw/current/scripts/prefabs/grottopool_sfx.lua#L1-L20) |

## 关联机制导航

| 类型／对象 | 实际关系与边界 | 证据 |
|---|---|---|
| 水声控制器 | 洞穴世界在非专用服分支添加 `grottowaterfallsoundcontroller`；其初始化用零延时任务创建 3 个发射器，之后每 5 帧处理一次。客户端与非专用主机各有本地实体，不由该 prefab 网络同步。 | [安装](../../../raw/current/scripts/prefabs/cave.lua#L339-L350)、[生成与周期](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L189-L197) |
| 播声与切换 | 控制器将发射器分配给附近已登记的水池，在池位播放 `grotto/common/waterfall_LP`，事件名 `grotto_waterfall`；未分配新池时 `KillAllSounds` 并清配对。播放、池选择与音量属于控制器，不是工厂自动行为。 | [常量](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L15-L23)、[处理](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L117-L154)、[[reference/components/grottowaterfallsoundcontroller#关键接口]] |
| `fader` | 切换发射器且声音正在播放、旧池记录仍有效时，控制器以 0.2 秒降至 0.05；完成时若新池仍有效，移动到新池并以 0.2 秒淡入目标音量。声音未播放或旧池无效时直接移位并播放。`fader` 按传入的 setter 与结束回调更新，不自行决定声音。 | [淡变回调](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L105-L115)、[切换](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L137-L151)、[组件](../../../raw/current/scripts/components/fader.lua#L8-L24) |
| 池依赖声明 | 大、小水池的 `prefabs` 表都声明该名，但实际生成调用位于控制器；声明不能视作每个池自行创建发射器。 | [大池](../../../raw/current/scripts/prefabs/grotto_pool_big.lua#L8-L15)、[小池](../../../raw/current/scripts/prefabs/grotto_pool_small.lua#L8-L13)、[生成](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L192-L195) |

## 常见陷阱与限制

- 停止声音不等于移除实体：控制器未分配池时只杀声音并清记录；在本快照控制器文件中未找到逐个移除这三个发射器的清理回调，世界卸载或外部移除时的引擎表现未核验。[未分配分支](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L129-L137)
- 该实体没有自动持续播放的配置；直接生成只能得到空的 SoundEmitter 与 `fader`，水池登记和声音播放需按控制器接线。[工厂](../../../raw/current/scripts/prefabs/grottopool_sfx.lua#L1-L20)、[播放](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L137-L142)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394。核验工厂、控制器生成与播声分支、`fader` 调用和洞穴端分界；在 `raw/current/scripts` 全树检索完整注册名及 `grottopool` 前缀，区分真正生成、资源依赖与 prefab 清单。未核验引擎卸载时资源回收、其他模组动态生成及声音素材内容。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
