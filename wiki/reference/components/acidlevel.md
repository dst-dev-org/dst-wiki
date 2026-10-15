---
title: "acidlevel：酸雨侵蚀值与酸雨伤害"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在玩家（或石虾等）身上的酸雨计量：酸雨时周期累积侵蚀值并对装备、背包食物与本体造成伤害，普通雨则反向消退。"
aliases: [酸雨侵蚀, 酸雨计量, acidlevel, 酸雨伤害]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/acidlevel.lua", "prefabs/player_common.lua", "components/caveweather.lua", "prefabs/rocky.lua", "prefabs/bat_boss.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# acidlevel（酸雨侵蚀值与酸雨伤害）

## 快速判断

- 职责与适用：服务端计量组件。构造即 `WatchWorldState` 监听 `isacidraining`/`israining`：酸雨开始起周期任务（`ACIDRAIN_DAMAGE_TIME`=1.5s 一跳），普通雨起反向消退任务；`current` 0~`max`(100)，变化推 `acidleveldelta` 事件（[构造](../../../raw/current/scripts/components/acidlevel.lua#L101-L120)、[事件](../../../raw/current/scripts/components/acidlevel.lua#L275-L280)）。所有玩家无条件挂载（[player_common.lua:2743](../../../raw/current/scripts/prefabs/player_common.lua#L2743）。
- 优先使用：要免疫酸雨 → 玩家带 `acidrainimmune` 标签、装备 `rainimmunity` 组件或 `acidrainimmune` 装备（[tick 入口](../../../raw/current/scripts/components/acidlevel.lua#L147-L149)、[装备豁免](../../../raw/current/scripts/components/acidlevel.lua#L155-L157)）；要改伤害 → `SetOverrideAcidRainTickFn`。
- 重要副作用：酸雨跳内联动背包——`PerishInventoryTick` 让湿物品加速腐烂，防水装备被"熔蚀"（armor 扣血／fueled 扣燃料），命中本体同时触发 `player_classified.isacidsizzling` 嘶嘶表现（[装备伤害](../../../raw/current/scripts/components/acidlevel.lua#L39-L65)）。
- 修改边界：组件只管玩家本体与随身物品；石虾（rocky）复用本组件做"酸雨缩壳"状态机（[rocky 挂载](../../../raw/current/scripts/prefabs/rocky.lua#L556-L558)），洞穴世界由 caveweather 决定是否允许酸雨（[开关](../../../raw/current/scripts/components/caveweather.lua#L433-L436)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoDelta(delta)` | 数值 | `current` 钳制到 [0, max] 并推 `acidleveldelta`（含 old/newpercent）；存读走 `current`（[DoDelta](../../../raw/current/scripts/components/acidlevel.lua#L275-L280)、[存读](../../../raw/current/scripts/components/acidlevel.lua#L290-L301)）。 |
| `GetPercent()`／`SetPercent(p)` | 0~1 | SetPercent 换算成 DoDelta（[实现](../../../raw/current/scripts/components/acidlevel.lua#L282-L288)）。 |
| `SetIgnoreAcidRainTicks(bool)` | 布尔 | 切换时若酸雨任务在跑会立即调对应的开始/停止回调；石虾与 caveweather 用它做条件豁免（[实现](../../../raw/current/scripts/components/acidlevel.lua#L122-L140)）。 |
| `SetOverrideAcidRainTickFn(fn)` | `fn(inst, damage) → damage?` | 覆盖本体伤害值；返回 0 跳过默认伤害，返回负值可治疗（[设置](../../../raw/current/scripts/components/acidlevel.lua#L215-L221)、[调用](../../../raw/current/scripts/components/acidlevel.lua#L197-L200)）。 |
| `SetOnStart/StopIsAcidRainingFn` 等 4 个 | `fn(inst)` | 酸雨/普通雨开始与停止的回调（[回调注册](../../../raw/current/scripts/components/acidlevel.lua#L258-L272)）。 |
| `PerishInventoryTick(rate)` | 数值 | 对背包可湿物品按 `rate × ACIDRAIN_PERISHABLE_ROT_PERCENT × ACIDRAIN_DAMAGE_TIME` 腐烂；蝙蝠 Boss 吸附时也直接调用（[实现](../../../raw/current/scripts/components/acidlevel.lua#L142-L145)、[bat_boss 调用](../../../raw/current/scripts/prefabs/bat_boss.lua#L183-L184)）。 |

酸雨单跳流程（[DoAcidRainTick](../../../raw/current/scripts/components/acidlevel.lua#L147-L205)）：`rainimmunity` 组件或 `ignoreacidrainticks` 直接跳过 → 基础伤害 `ACIDRAIN_DAMAGE_TIME × DAMAGE_PER_SECOND`(2.0/s) → 率取 `moisture:_GetMoistureRateAssumingRain()` 或世界 `precipitationrate` → 防水装备按效果吸收并均摊反伤 → 背包腐烂 → `DoDelta` 累积 → 本体伤害乘率后经 override／直接 `health:DoDelta(-damage, false, "acidrain")`。

## 典型接入

### 给角色/生物条件性酸雨豁免

- 前提与选择：服务端实体已带 acidlevel（玩家默认有）；豁免用标签（静态）或 `SetIgnoreAcidRainTicks`（动态）。
- 生效与撤销：`SetIgnoreAcidRainTicks(true)` 只挡酸雨跳，普通雨消退照常；rocky 在缩壳状态机中用它并且回调里驱动自己的动画与百分比（[rocky 用法](../../../raw/current/scripts/prefabs/rocky.lua#L556-L558)）。

```lua
-- 服务端，实体已挂 acidlevel 后
local acid = inst:AddComponent("acidlevel")  -- 玩家已默认挂载，勿重复
acid:SetIgnoreAcidRainTicks(true)            -- 完全不吃酸雨 tick

-- 或注册回调，在酸雨开始/结束时做事
acid:SetOnStartIsAcidRainingFn(function(inst)
    inst:PushEvent("my_acidrain_start")
end)
```

- 调整与验证：调谐值见 ([tuning.lua:7745-7755](../../../raw/current/scripts/tuning.lua#L7745-L7755)（1.5s 一跳、2.0/s、燃料系数 2、腐烂 0.007）；`acidleveldelta` 携带 `oldpercent`/`newpercent`。

## 常见陷阱与限制

- **普通雨也在写这个值**：`DoRainTick` 以负率消退酸值；玩家在普通雨中浸泡越久酸值越低（[消退](../../../raw/current/scripts/components/acidlevel.lua#L207-L213)），不能把 `GetPercent()` 当"污染度"单向读。
- **防水装备会被熔蚀**：吸收伤害的 waterproofer 装备自身按 armor/fueled 路径承受等量伤害并触发嘶嘶状态（[熔蚀](../../../raw/current/scripts/components/acidlevel.lua#L160-L183)），不是无代价防护。
- **洞窟酸雨由世界开关**：caveweather 对每个玩家 `SetIgnoreAcidRainTicks(not canacidrain)`；地表世界默认 `ACIDRAIN_ENABLED` 由世界设置管理（[开关](../../../raw/current/scripts/components/caveweather.lua#L433-L436)、[调谐](../../../raw/current/scripts/tuning.lua#L7745)）。
- **读档只恢复数值不恢复任务**：`OnSave` 只存 `current`；雨态任务由构造后一帧的 `OnIsAcidRaining/OnIsRaining` 重建（[重建](../../../raw/current/scripts/components/acidlevel.lua#L116-L119)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/acidlevel.lua` 全部 308 行、player_common 挂载行、caveweather 开关、rocky 与 bat_boss 的复用调用点、tuning ACIDRAIN 块。未覆盖：`isacidsizzling` 客户端表现、rocky 缩壳状态机全貌。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
