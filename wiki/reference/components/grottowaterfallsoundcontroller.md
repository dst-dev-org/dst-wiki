---
title: "grottowaterfallsoundcontroller"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界的水声调度组件：登记格罗托水池并维护 3 个移动音效发射器，把瀑布循环声定位到离本地玩家最近的水池并做音量淡入淡出；纯客户端听感，不联网。"
aliases: ["grottowaterfallsoundcontroller", "洞穴瀑布声控制器", "格罗托水声", "grotto waterfall sound"]
tags: [component, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/grottowaterfallsoundcontroller.lua", "prefabs/cave.lua", "prefabs/grotto_pool_big.lua", "prefabs/grotto_pool_small.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# grottowaterfallsoundcontroller

## 快速判断

- 职责与适用：挂在 cave 世界实体的听感组件（非专用服务器才挂载，([cave.lua:339-350](../../../raw/current/scripts/prefabs/cave.lua#L339-L350)）；每 5 帧把 `grotto/common/waterfall_LP` 循环声分配到距 `ThePlayer` 最近的最多 3 个已登记水池上，音量按池大小取 1.0/0.8。[常量](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L11-L23) [主循环](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L117-L155)
- 优先使用：让自定义水池发声 → 向世界推 `ms_registergrottopool`，`data = {pool = 实体, small = 布尔}`；不需要任何方法调用。[登记事件](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L189
- 重要副作用：初始化即生成 3 个 `grottopool_sfx` 特效实体作为发射器（`persists` 走 prefab 默认），随玩家移动被 `fader` 组件搬移位置并淡变音量；发射器自身不联网同步。[发射器创建](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L192-L197) [搬移](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L110-L115)
- 修改边界：无 OnSave/OnLoad，无公开方法；`ThePlayer` 无效时本轮直接跳过；专用服务器没有此组件，水池登记事件在专用服上无人消费。[空转保护](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L118-L123)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ms_registergrottopool`（监听） | `data = {pool, small}` | `pool` 为水池实体，`small` 决定音量档与半径折扣；重复登记被布尔表自然去重，池 `onremove` 自动注销。[TrackPool](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L167-L178) [注销](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L157-L165) |
| 候选筛选 | 私有闭包 | 池距玩家平方距离减去池半径平方后 `EMITTER_MAXDSQ = 1600`（即 40 距离）内才入围，按距离升序排序取前 `NUM_EMITTERS = 3`。[筛选](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L49-L69) |
| 发射器配对 | 私有闭包 | 已在播某池的发射器保持粘性配对，其余按序领走未认领的最近池；没有新池时 `KillAllSounds` 并清引用。[配对](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L73-L99) [回收](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L129-L154) |
| 声音事件名 | `grotto_waterfall` | `SetVolume`／`PlayingSound` 都用该名字控制同一循环，调整音量需沿用此名。[常量](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L22-L23) |

## 典型接入

### 让自定义洞穴水体接入瀑布声场

- 前提与选择：原版池在非专用服的每个实例（主机与客户端）各自登记——推送放在 prefab fn 的 `SetPristine` 之前、以 `not TheNet:IsDedicated()` 门控（([grotto_pool_big.lua:169-172](../../../raw/current/scripts/prefabs/grotto_pool_big.lua#L169-L172)）。
- 生效与撤销：登记即入围下一轮（≤0.17s 后）分配；实体移除时靠 `onremove` 自动注销，无需反注册调用。
- 原版依据：([grotto_pool_big.lua:44-46](../../../raw/current/scripts/prefabs/grotto_pool_big.lua#L44-L46)、([grotto_pool_small.lua:28-30](../../../raw/current/scripts/prefabs/grotto_pool_small.lua#L28-L30)、[oceanwhirlbigportal.lua:477](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L477)。

```lua
-- 水体 prefab fn 内，SetPristine 之前（两端执行，专用服跳过）
if not TheNet:IsDedicated() then
    inst:DoTaskInTime(0, function()
        TheWorld:PushEvent("ms_registergrottopool", { pool = inst, small = false })
    end)
end
```

- 调整与验证：`small = true` 用 0.8 音量且半径折扣更小（`TUNING.GROTTO_POOL_SMALL_RADIUS`）；专用服组件不存在，判断判空即可，无需重复注册。

## 常见陷阱与限制

- 只认 `ThePlayer`：本地玩家不存在或无效时整轮跳过（[L118-L123](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L118-L123)），观测该组件调试串需以真实客户端视角进入洞穴。
- 距离阈值按"平方距离−半径平方"算，超大自定义水池（半径 > 40）恒不入围，需调 `EMITTER_MAXDSQ` 级别才可复用（[L54-L62](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L54-L62)）。
- 发射器是独立 `grottopool_sfx` 实体，位置每 5 帧才更新一次，快移穿过水池会有可感知的滞后（[L197](../../../raw/current/scripts/components/grottowaterfallsoundcontroller.lua#L197）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 cave.lua 挂载条件（`IsDedicated` 分支）、两个水池与漩涡传送门的登记事件源。
- 游戏运行：未运行。
- 结构检查：未执行。
