---
title: "grue"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家黑暗惩罚（查理）组件：完全黑暗中按 5-11 秒节奏先警告后攻击，按免疫来源表豁免，伤害走 combat 的 darkness 刺激并扣理智，支持非致命钳制与抗性次数。"
aliases: ["grue", "查理", "黑暗惩罚", "Charlie 组件"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/grue.lua", "prefabs/player_common.lua", "tuning.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# grue

## 快速判断

- 职责与适用：挂在每个玩家实体（服务端）的组件（([player_common.lua:2812-2813](../../../raw/current/scripts/prefabs/player_common.lua#L2812-L2813)）；完全黑暗（不在光下、非无敌、未死、`CanEntitySeeInDark` 为假）时启动计时循环：先在 40%-80% 进度播警告声并延迟 1 秒推 `heargrue`，到点后按等级结算攻击或抵抗。[初始化监听](../../../raw/current/scripts/components/grue.lua#L44-L67) [计时循环](../../../raw/current/scripts/components/grue.lua#L190-L223)
- 优先使用：加免疫情景用 `AddImmunity(source)`／`RemoveImmunity(source)`（来源字符串可自定义，如睡眠、区域）；调攻击前耐受次数用 `SetResistance(n)`。[免疫表](../../../raw/current/scripts/components/grue.lua#L143-L159) [抗性](../../../raw/current/scripts/components/grue.lua#L139-L141)
- 重要副作用：攻击经 `combat:GetAttacked(nil, damage, nil, "darkness")`（darkness 为 stimuli），同时 `sanity:DoDelta(-SANITY_MEDLARGE)`；等级超过 `resistance` 才扣血，之前只推 `resistedgrue`。[Attack](../../../raw/current/scripts/components/grue.lua#L170-L188) [结算](../../../raw/current/scripts/components/grue.lua#L206-L222)
- 修改边界：死亡 `Stop()`，复活（`ms_respawnedfromghost`）且无免疫时自动 `Start()`；组件移除时清空免疫表防止移除过程中重启更新。[死亡与复活](../../../raw/current/scripts/components/grue.lua#L33-L42) [移除保护](../../../raw/current/scripts/components/grue.lua#L86-L92)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | `warndelay = 1`、`nonlethal = TUNING.NONLETHAL_DARKNESS`（默认 false）、`nonlethal_pct = 0.2`、空免疫表；延迟 0 帧任务做初始监听与免疫判定（在光下／夜视／无敌）。([grue.lua:69-84](../../../raw/current/scripts/components/grue.lua#L69-L84) |
| `AddImmunity(source)`／`RemoveImmunity(source)` | source 键（nil 时用 self） | 以表键登记免疫来源，任一免疫存在即 `Stop()`，全部移除后 `Start()`；事件桥接自动维护 `light`（enterdark/enterlight）、`nightvision`、`invincible` 三类。([grue.lua:143-159](../../../raw/current/scripts/components/grue.lua#L143-L159) [事件桥](../../../raw/current/scripts/components/grue.lua#L1-L31) |
| `grueimmunearea`（监听） | `data = {isimmune, name}` | 区域免疫：按 `data.name` 增删（原版推送点为 virtualroomset 虚拟房间进出）。([grue.lua:25-31](../../../raw/current/scripts/components/grue.lua#L25-L31) [virtualroomset.lua:793](../../../raw/current/scripts/components/virtualroomset.lua#L793 |
| `CheckForStart()` | 返回 boolean | 非无敌、不在光下、未死亡、`CanEntitySeeInDark` 为假才可启动；保留为公开方法以兼容旧模组。([grue.lua:111-116](../../../raw/current/scripts/components/grue.lua#L111-L116) [定义](../../../raw/current/scripts/simutil.lua#L513-L514) |
| `Start()`／`Stop()` | 无参 | 启动时 `level = 0`、首击 5-10 秒、警告点为首击时长的 40%-80%；停止仅清 level 与更新。([grue.lua:118-137](../../../raw/current/scripts/components/grue.lua#L118-L137) |
| `SetResistance(resistance)` | number | 被攻击结算前可承受的等级数；`level > resistance` 才实际掉血。([grue.lua:139-141](../../../raw/current/scripts/components/grue.lua#L139-L141) |
| `SetSounds(warn, attack)` | 两个声音路径 | 默认 nil；玩家挂载时设为 `dontstarve/charlie/warn`／`.../attack`。([grue.lua:127-130](../../../raw/current/scripts/components/grue.lua#L127-L130) [player_common.lua:2813](../../../raw/current/scripts/prefabs/player_common.lua#L2813 |
| `OnUpdate(dt)` | 组件更新 | 首击后每次重置为 5-11 秒（`5 + random*6`）；警告音只在各周期中点附近播一次。([grue.lua:190-223](../../../raw/current/scripts/components/grue.lua#L190-L223) |

## 典型接入

### 给玩家挂场景化黑暗免疫

- 前提与选择：宿主须有 combat／health／sanity（攻击结算依赖）；免疫以字符串来源登记，配对增删即可，组件自动启停更新循环。
- 生效与撤销：`AddImmunity` 立即停止黑暗计时（已排的 `heargrue` 延迟事件仍可能触发一次）；全部来源移除后从零重新计时。
- 原版依据：睡眠状态在 SG 进入/离开时挂 `sleeping` 免疫（([SGwilson.lua:301-302](../../../raw/current/scripts/stategraphs/SGwilson.lua#L301-L302)、[321-322](../../../raw/current/scripts/stategraphs/SGwilson.lua#L321-L322)）。

```lua
-- 服务端，任意需要临时豁免黑暗的时机：
local grue = player.components.grue
if grue ~= nil then
    grue:AddImmunity("mymod_darkness_shield")
end
-- 撤销（全部来源清空后自动恢复计时）：
-- grue:RemoveImmunity("mymod_darkness_shield")
-- 提升耐受次数（挨 1 次才真正掉血）：
-- grue:SetResistance(1)
```

- 调整与验证：`SetResistance` 不持久化、存档不恢复，需自行在载入后重设；`heargrue`／`attackedbygrue`／`resistedgrue` 三个事件可用于自定义表现。

## 常见陷阱与限制

- 伤害固定走 combat：`GetAttacked(nil, damage, nil, "darkness")`（([grue.lua:187](../../../raw/current/scripts/components/grue.lua#L187）意味着护甲会正常吸收、`damagetyperesist` 等按 stimuli 处理；想改伤害量应调 `TUNING.GRUEDAMAGE`（[tuning.lua:762](../../../raw/current/scripts/tuning.lua#L762）而非覆写 Attack。
- 非致命模式默认关闭：`NONLETHAL_DARKNESS = false`（([tuning.lua:6850-6851](../../../raw/current/scripts/tuning.lua#L6850-L6851)）；开启后伤害被钳制到不低于 20% 最大生命的水平（([grue.lua:173-184](../../../raw/current/scripts/components/grue.lua#L173-L184)）。
- `SetSleeping` 已废弃但仍可用（([grue.lua:162-168](../../../raw/current/scripts/components/grue.lua#L162-L168)），新代码应直接用 `AddImmunity("sleeping")`，避免与 SG 桥接的增删互相踩。
- 初始免疫判定在延迟 0 帧任务里（([grue.lua:83](../../../raw/current/scripts/components/grue.lua#L83）：挂载同一帧内的光照状态尚不可靠，若在 PostInit 立即 `Start()` 可能与桥接状态冲突。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 player_common 挂载与默认音效、SGwilson 睡眠免疫桥、simutil 的 CanEntitySeeInDark、virtualroomset 区域免疫推送与相关调谐键。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/combat]]、[[reference/components/sanity]]、[[reference/components/health]]
