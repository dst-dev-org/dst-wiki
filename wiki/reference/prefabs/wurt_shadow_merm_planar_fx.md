---
title: "沃特位面增益与投泥带外围特效族（wurt_shadow_merm_planar_fx 等 5 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "沃特技能外围的 5 个表现注册：暗影鱼人位面 buff 的持续跟随层与月影鱼人的一次性出入场动画（同一 debuff 实体按目标阵营二选一生成）、投泥带落点改泥的联网物理弹、投泥带充能期的手持符号跟随层（暗影/月影同工厂两变体）。"
aliases: ["wurt_shadow_merm_planar_fx", "wurt_lunar_merm_planar_fx", "wurt_terraform_projectile", "wurt_swampitem_shadow_chargedfx", "wurt_swampitem_lunar_chargedfx", "wurt_merm_planar", "沃特位面增益特效", "鱼人位面buff特效", "暗影鱼人跟随层", "月影鱼人出场动画", "投泥带充能特效", "沼泽弹投射物", "绝望投泥带", "光芒投泥带"]
tags: [prefab, fx, item, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wurt.lua", "prefabs/wurt_terraform_item.lua", "prefabs/wurt_swamp_terraformer.lua", "prefabs/horrorfuel.lua", "prefabs/purebrilliance.lua", "prefabs/skilltree_wurt.lua", "prefabs/merm.lua", "prefabs/merm_fx.lua", "stategraphs/SGwilson.lua", "components/complexprojectile.lua", "components/spellcaster.lua", "recipes.lua", "tuning.lua", "fx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃特位面增益与投泥带外围特效族（wurt_shadow_merm_planar_fx 等 5 注册）

## 快速判断

- 职责与适用：5 个 FX／弹体注册分三链——①`wurt_shadow_merm_planar_fx`（暗影鱼人位面增益的持续跟随层，含循环音效）、②`wurt_lunar_merm_planar_fx`（月影鱼人位面增益的一次性 `pre`／`pst` 动画，非持续层）、③`wurt_terraform_projectile`（投泥带落点改泥的联网物理弹）、④⑤`wurt_swampitem_shadow_chargedfx`／`wurt_swampitem_lunar_chargedfx`（投泥带充满电时挂在持有者 `swap_object` 符号上的充能层）。三条链都由沃特（官方名，见下）的暗影／月影阵营技能门禁触发。[FX 注册](../../../raw/current/scripts/prefabs/wurt.lua#L877-L880)、[弹体与充能注册](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L463-L468)
- 优先使用：查"位面 buff 的鱼人附层/眼睛发光"用 `wurt_merm_planar` debuff 链；查"投泥带扔出去怎么落、落地发生什么"用 projectile 链；查"投泥带什么时候手上冒光、冒什么光"用 chargedfx 生命周期。
- 重要副作用：两个 FX 本体都不结算伤害——位面加伤（基础 20）写在 debuff 实体对 `planardamage` 的赋值上，改泥写在落地后生成的无动画代理 `wurt_swamp_terraformer` 上；弹体自身也 `persists=false` 且无武器组件，属联网表现／物理壳。充能层无自删；debuff 实体由 `expire` 计时到点或目标死亡触发 `debuff:Stop`，随后 `OnDetached` 自删；月影入场／退场 FX 是各自一次性实例，工厂或生成处登记 `animover` 自删。[加伤赋值](../../../raw/current/scripts/prefabs/wurt.lua#L721-L724)、[代理改泥](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L107-L155)、[月影自删](../../../raw/current/scripts/prefabs/wurt.lua#L870)
- 修改边界：只覆盖 5 注册的工厂、生成点、父子／跟随与清理；`wurt_swampitem_shadow`／`wurt_swampitem_lunar` 两件本体、`wurt_terraform_cast_debuff` 施法冷却 debuff、`wurt_swamp_terraformer` 改泥代理与鱼人本体玩法只作链条辨析不展开，改泥与回滚的完整细节归代理本体。
- 名称：`WURT`＝"沃特"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352519-L352522)）、宿主物品 `WURT_SWAMPITEM_SHADOW`＝"绝望投泥带"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352529-L352532)）、`WURT_SWAMPITEM_LUNAR`＝"光芒投泥带"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352524-L352527)）、`HORRORFUEL`＝"纯粹恐惧"（[po](../../../raw/current/scripts/languages/chinese_s.po#L343964-L343967)）、`PUREBRILLIANCE`＝"纯粹辉煌"（[po](../../../raw/current/scripts/languages/chinese_s.po#L347384-L347387)）；5 个覆盖注册名与 `wurt_merm_planar` 在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 显示名键，标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wurt_shadow_merm_planar_fx` | `shadowbufffn_fx`：Transform／AnimState／Network／Follower／SoundEmitter，标签 `FX` → bank=build=`merm_shadow_fx`、循环 `buff_idle`、`horror_fx` 符号提亮＋泛光 → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 持续跟随层；工厂动画会被生成方 `buff_pre`→`buff_idle` 重播覆盖；音效由生成方起循环。[工厂](../../../raw/current/scripts/prefabs/wurt.lua#L816-L844)、[注册](../../../raw/current/scripts/prefabs/wurt.lua#L879) |
| `wurt_lunar_merm_planar_fx` | `lunarbufffn_fx`：Transform／AnimState／Network／Follower（无 SoundEmitter），标签 `FX` → bank=`lunar_fx`、build=`merm_lunar_fx`、播一次 `pre`、`lunar_fx` 符号提亮＋泛光 → `SetPristine` → 客户端早退 → 服务端登记 `animover` 自删，`persists=false`。 | 一次性入场动画：工厂播 `pre` 播完即自删；退场是另生成一个播 `pst` 的实例。[工厂](../../../raw/current/scripts/prefabs/wurt.lua#L846-L875)、[注册](../../../raw/current/scripts/prefabs/wurt.lua#L880) |
| `wurt_terraform_projectile` | `terraform_projectile`：Transform／AnimState／SoundEmitter／Network／Physics（质量 1、碰撞只对地面、睡眠不移除）→ bank=build=`wurt_swampbomb`、循环 `blob_loop`、`NOCLICK`＋优化标签 `projectile`／`complexprojectile` → 非专服加本地地面阴影（0.8×0.5）→ `SetPristine` → 客户端早退 → 服务端 `persists=false`、complexprojectile 重力 -20／水平速度 0.1／发射偏移 (0,20,0)／OnHit，0 帧任务播撞击音。 | 联网物理弹壳：无伤害组件；从目标点上空 20 落下而非玩家抛出弧线（生成即在落点，再 `Launch` 抬升）。[工厂](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L338-L386)、[Launch 语义](../../../raw/current/scripts/components/complexprojectile.lua#L134-L159)、[注册](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L467) |
| `wurt_swampitem_shadow_chargedfx` | `charged_fx_common("shadow")`：Transform／AnimState／Network，标签 `FX` → bank=build=`wurt_swampitem_charged_fx`、循环 `shadow`、FinalOffset -1、alpha 0.5、点过滤 → `SetPristine` → 客户端早退 → 服务端随机起始帧，`persists=false`。 | 充能层暗影变体；无 Follower 组件——由生成方运行时补挂。[工厂](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L422-L448)、[变体](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L450-L452)、[注册](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L464) |
| `wurt_swampitem_lunar_chargedfx` | 同工厂同 bank/build，仅循环动画换 `lunar`，另加泛光与 LightOverride 0.1。 | 与暗影变体同工厂同 build，差异是动画名与渲染微调（台账"仅 build 不同"经核为"仅动画与泛光不同"）。[变体](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L454-L461)、[注册](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L466) |

同文件未被本页收为覆盖注册的还有：`wurt_swampitem_shadow`／`wurt_swampitem_lunar`（两件投泥带本体，[工厂](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L167-L236)、[暗影](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L239-L259)、[月影](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L293-L314)）与 `wurt_terraform_cast_debuff`（施法冷却 debuff，[工厂](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L394-L419)）；`wurt.lua` 的 `wurt_merm_planar`（debuff 实体）见下节。台账批 9 所记 3 注册经回源复核实为 **5 注册**（chargedfx 有暗影／月影两个变体）。

## 关联机制导航

### wurt_merm_planar 位面增益链：谁生成、挂给谁

| 生成链 | 触发与条件 | 证据 |
|---|---|---|
| 光芒投泥带落点 | 月影弹 `OnHit_Lunar`：以落点为中心按 `WURT_TERRAFORMING_TILERANGE`(3) 换算对角距离找 `_combat`＋`_health`＋`merm` 实体，只有**拥有者的跟随者**才结算——先 `DoLunarMutation`（若存在）再 `AddDebuff("wurt_merm_planar", ...)`。 | [OnHit_Lunar](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L264-L291)、[额外命中挂接](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L307) |
| 纯粹恐惧施法 | `Wurt_MermSpellFn`：对施法者每个存活 `shadowminion` 跟随者 `AddDebuff`，随后物品自毁（stackable 取一层移除）。 | [horrorfuel.lua L134-L147](../../../raw/current/scripts/prefabs/horrorfuel.lua#L134-L147) |
| 纯粹辉煌施法 | 同函数结构，目标换 `lunarminion` 跟随者。 | [purebrilliance.lua L36-L49](../../../raw/current/scripts/prefabs/purebrilliance.lua#L36-L49) |

施法资格走 spellcaster 的 spelltype 门禁：`spellcaster` 要求施法者有 `<spelltype>_spelluser` 标签（[spellcaster.lua L155](../../../raw/current/scripts/components/spellcaster.lua#L155)），该标签由沃特技能树暗影／月影阵营 1 级激活／取消时增删（[暗影](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L377)、[月影](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L424)）；两件投泥带的配方也分别锁 `wurt_shadow_allegiance_2`／`wurt_lunar_allegiance_2`（[recipes.lua L125-L126](../../../raw/current/scripts/recipes.lua#L125-L126)）。两物品的施法在 SGwilson 的 `veryquickcastspell` 状态播 `castsound` 并在 9 帧执行动作（[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16327-L16361)、[路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1311)）。

### 附层生成与拆解（debuff 实体的两个分支）

debuff 实体 `wurt_merm_planar`（master-only、`CLASSIFIED`、`persists=false`、`keepondespawn=true`，[工厂](../../../raw/current/scripts/prefabs/wurt.lua#L788-L814)）在 `OnAttached` 里父挂目标、起 `expire` 计时（`WURT_PLANAR_TIME_BUFF_LASTS`＝8×游戏段＝240 秒）、把目标 `planardamage` 基础伤害设 20 并 `UpdateDamageAndHealth`（[OnAttached](../../../raw/current/scripts/prefabs/wurt.lua#L712-L747)、[调值](../../../raw/current/scripts/tuning.lua#L8331-L8333)），然后按目标阵营二选一：

- **暗影分支**：生成 `wurt_shadow_merm_planar_fx`，播 `buff_pre`→`buff_idle`，FX 自身 SoundEmitter 起循环 `meta4/shadow_merm/buff_idle`，父挂目标，引用存 `inst.bufffx`（[L726-L734](../../../raw/current/scripts/prefabs/wurt.lua#L726-L734)）。拆解时 FX 播 `buff_pst`＋退场音、`animover` 自删（[OnDetached](../../../raw/current/scripts/prefabs/wurt.lua#L749-L779)）。
- **月影分支**：生成 `wurt_lunar_merm_planar_fx` 播一次 `pre` 即自删（工厂已登记）；持久表现改写在目标本体上——`updateeyebuild()` 换眼睛 build、目标泛光、目标播 `meta4/lunar_merm/buff`（[L736-L745](../../../raw/current/scripts/prefabs/wurt.lua#L736-L745)）。拆解时目标还原，再**另生成**一个播 `pst` 的实例退场（[L767-L776](../../../raw/current/scripts/prefabs/wurt.lua#L767-L776)）。

鱼人端消费：`updateeyebuild` 按 `HasDebuff("wurt_merm_planar")` 增删眼睛 override build（[merm.lua L943-L960](../../../raw/current/scripts/prefabs/merm.lua#L943-L960)）；净变量 `planarbuffed` 置位后客户端给鱼人双眼挂火焰跟随体（[planarbuffed_changed](../../../raw/current/scripts/prefabs/merm.lua#L1522-L1528)、[净变量](../../../raw/current/scripts/prefabs/merm.lua#L1561)）；月影鱼人荆棘 FX 也会读该 debuff 补泛光（[merm_fx.lua L72-L74](../../../raw/current/scripts/prefabs/merm_fx.lua#L72-L74)）。

### 投泥带施法与落地链

`CastTerraformingSpell` 在目标点生成 `wurt_terraform_projectile`，把物品的 `_terraform_tile_type`／`_extra_onhit_fn`／`_landed_sound` 三个字段复制到弹上，再 `complexprojectile:Launch`（[L46-L77](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L46-L77)）；随后物品进 rechargeable 放电、给持有者挂 `wurt_terraform_cast_debuff`（冷却＝一天，`CanCastTerraformingSpell` 拒绝续投）、扣理智并把背包里其余投泥带一齐放电（[L57-L76](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L57-L76)、[冷却调值](../../../raw/current/scripts/tuning.lua#L8280-L8285)）。弹体命中 `OnHitTerraformer`：在落点生成**无动画代理** `wurt_swamp_terraformer`，`SetType(tile_type or "SHADOW")` 后 `DoTerraform()`，落地音播在代理身上，然后弹体自删（[OnHitTerraformer](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L317-L332)）。代理在以落点为中心半径 3 格（偏移 -3..+3，7×7 格）的范围内把可改地块换成 `SHADOW_MARSH`／`LUNAR_MARSH`（跳过阻挡／海洋／临时地块），逐块延迟铺 `wurt_swamp_terraform_fx`，`undo_terraforming` 计时（120 秒）到点回滚原地块（undertile 记录）后自删（[DoTerraform](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L107-L155)、[回滚](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L157-L198)、[注册](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L268)）。月影弹额外跑 `OnHit_Lunar`（鱼人增益＋`groundpoundring_fx` 冲击环，[L264-L291](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L264-L291)）；改泥期间逐块另播 `wurt_terraformer_fx_shadow`／`_lunar`（注册于 [fx.lua L3492/L3502](../../../raw/current/scripts/fx.lua#L3492-L3502)，由代理按类型查表生成，[L7-L11](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L7-L11)）。

### 充能特效生命周期

充能层只在投泥带**充满电且已装备**时存在：`OnCharged`（rechargeable 充满）或 `onequip` 后延迟 1 帧的 `OnEquip_CheckForChargedFX`（持有者无冷却 debuff 才加）调 `add_charged_fx`——按物品 `_fx_type` 选变体，运行时补 `Follower` 并 `FollowSymbol(owner, "swap_object", 0, -100, 0)`、父挂持有者（[add_charged_fx](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L88-L102)、[OnCharged](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L110-L117)、[装备检查](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L120-L138)）。`OnDischarged`（施法放电）与 `unequip` 都显式移除（[OnDischarged](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L104-L108)、[unequip](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L139-L145)）；捡起新投泥带时按冷却剩余比例压低电量防止换装重置（[on_item_putininventory](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L148-L164)）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `wurt_shadow_merm_planar_fx` | 无自删；debuff `OnDetached` 时播 `buff_pst` 后 `animover` 自删（[L756-L765](../../../raw/current/scripts/prefabs/wurt.lua#L756-L765)） | 有 `IsValid` 守卫；宿主目标移除时是否随引擎父挂回收未核验。 |
| `wurt_lunar_merm_planar_fx` | 工厂登记 `animover` 自删；`pre`/`pst` 播完即走（[L870](../../../raw/current/scripts/prefabs/wurt.lua#L870)、[L772-L775](../../../raw/current/scripts/prefabs/wurt.lua#L772-L775)） | buff 持续期没有月影 FX 实体在场。 |
| `wurt_terraform_projectile` | 命中 `OnHitTerraformer` 末尾自 `Remove`（[L331](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L317-L332)） | `persists=false` 不存档；若未命中（如被打断）依赖 complexprojectile 物理落地，未见其他移除点。 |
| `wurt_swampitem_*_chargedfx` | 无自删；`OnDischarged`／`unequip` 显式移除，引用单槽存物品 `_charged_vfx`（[L88-L108](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L88-L108)、[L139-L145](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L139-L145)） | 持有者移除时是否随引擎父挂回收未核验；物品在装备态被直接移除（不经历 unequip）的路径也未核验。 |
| 链上代理 | `wurt_swamp_terraformer` 由 `remove` 计时自删（[OnTimerDone](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L200-L208)） | 代理未置 `persists=false`，带 `OnSave`/`OnLoad`（保存已改地块与待办计数，读档续铺或续回滚，[存读](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L211-L230)）。 |

## 典型接入

### 给"阵营 buff"做随生灭的跟随表现层

- 前提与选择：debuff 实体多为 master-only 无网络实体，客户端表现需另挂联网 FX；暗影模式是"FX 父挂目标＋buff 持引用，detach 时播 `pst` 再删"，月影模式是"入场/退场各一次性 FX＋持久表现写回宿主本体"。暗影持续层不要在 FX 工厂里自删；月影的入场／退场是一次性实例，工厂或生成处登记 `animover` 自删才是正确路径。debuff 自身的计时期满会 `Stop` 并删 debuff，不等于 FX 自删。
- 生效与撤销：生成点在 debuff 的 `OnAttached`，撤销在 `OnDetached`；引用挂在 debuff 实体的 `bufffx` 单槽上。[OnAttached](../../../raw/current/scripts/prefabs/wurt.lua#L726-L734)、[OnDetached](../../../raw/current/scripts/prefabs/wurt.lua#L749-L779)

```lua
-- 服务端：debuff 附身时挂持续跟随层，退场动画播完再删
local fx = SpawnPrefab("wurt_shadow_merm_planar_fx")
fx.entity:SetParent(target.entity)
inst.bufffx = fx
-- detach 时：
if inst.bufffx and inst.bufffx:IsValid() then
    inst.bufffx.AnimState:PlayAnimation("buff_pst")
    inst.bufffx:ListenForEvent("animover", function(f) f:Remove() end)
end
```

- 调整与验证：替换点是注册名、动画名与音效键；月影一次性实例必须在工厂或生成处登记 `animover` 自删，否则残留在目标身上。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：位面加伤不在 FX 上。** 伤害写在 debuff 的 `OnAttached` 里对 `target.components.planardamage` 的赋值（20，[L721-L724](../../../raw/current/scripts/prefabs/wurt.lua#L721-L724)）；两个 planar FX 都只是表现层，删 FX 不掉 buff，拆 buff 才拆 FX。
- **边界：月影 buff 的"持续层"在鱼人本体上。** `wurt_lunar_merm_planar_fx` 只在附身／退场瞬间各出现一次（`pre`／`pst`）；持续的眼睛 build、泛光与双眼火焰分别由 `updateeyebuild`、目标泛光与 `planarbuffed` 净变量承担（[updateeyebuild](../../../raw/current/scripts/prefabs/merm.lua#L943-L960)、[planarbuffed_changed](../../../raw/current/scripts/prefabs/merm.lua#L1522-L1528)）。别按暗影的"持续跟随层"模式去理解月影。
- **边界：buff 的"续期"路径是死代码。** `OnExtendedBuff` 引用的 `OnKillBuff` 在 `wurt.lua` 内无定义、`inst.bufftask` 也从未赋值（全域 grep 仅此一处引用），该函数实际是空操作，增益时长只能靠 `expire` 计时到点结束（[L781-L786](../../../raw/current/scripts/prefabs/wurt.lua#L781-L786)）。
- **边界：充能层跟随的是 `swap_object` 符号。** 偏移 (0, -100) 按该符号设计；nodangle／骑乘等改变持握符号的表现会跟着换位置，`Follower` 是生成方运行时补挂的（[add_charged_fx](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L88-L102)）。
- **误区：投泥带是"扔出去的抛物线弹"。** 弹体生成在目标点并从其上空 20 落下（Launch 偏移沿朝向加 y=20，[Launch](../../../raw/current/scripts/components/complexprojectile.lua#L134-L159)），不是从玩家手上的抛掷弧线；真正的抛掷物链是鱼食袋 `chum` 那套。
- **边界：改泥的回滚不可被弹体二次打断。** 落地即弹体自删，之后一切归代理 `wurt_swamp_terraformer`；同一地点再扔一发会生成第二个代理，两个代理各自记录自己改过的地块（[DoTerraform](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L107-L155)），重叠地块回滚次序未核验。
- **边界：月影弹的鱼人增益只认"拥有者的跟随者"。** AOE 检索范围是对角线长度，但结算前先用 `owner.components.leader:IsFollower` 过滤（[L284-L289](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L284-L289)）；对伤害结算的代码已被注释掉，弹幕本身无伤害。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wurt.lua`（FX 两工厂、bufffn 全函数族、注册块）、`prefabs/wurt_terraform_item.lua` 全文、`prefabs/wurt_swamp_terraformer.lua` 全文；按需读取 `prefabs/horrorfuel.lua`／`purebrilliance.lua` 的 Wurt_MermSpellFn 与 spellcaster 段、`prefabs/skilltree_wurt.lua` 两个 allegiance 节点、`prefabs/merm.lua` 的 updateeyebuild／planarbuffed 段、`prefabs/merm_fx.lua` SetFXOwner、`stategraphs/SGwilson.lua` veryquickcastspell 状态与路由、`components/complexprojectile.lua` Launch。对 5 个注册名以完整名在 `raw/current/scripts` 全域反查：`wurt_shadow_merm_planar_fx` 命中注册（wurt.lua:879）、依赖（:28）、生成（:727）——生成点唯一；`wurt_lunar_merm_planar_fx` 命中注册（:880）、依赖（:29）、生成（:737、:772）——两处都在 debuff 链；`wurt_terraform_projectile` 命中注册（wurt_terraform_item.lua:467）、依赖（:14）、生成（:49）——生成点唯一；`wurt_swampitem_shadow_chargedfx`／`wurt_swampitem_lunar_chargedfx` 各命中注册（:464／:466）、依赖（:19／:20）、生成（:90 按物品字段动态选名）——字符串常量唯一，运行时二选一。`wurt_merm_planar` 的 AddDebuff 生成点共 3 处（wurt_terraform_item.lua:288、horrorfuel.lua:138、purebrilliance.lua:40）。`OnKillBuff` 全域仅 wurt.lua:784 一处引用（未定义）。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`WURT`＝"沃特"、`WURT_SWAMPITEM_SHADOW`＝"绝望投泥带"、`WURT_SWAMPITEM_LUNAR`＝"光芒投泥带"、`HORRORFUEL`＝"纯粹恐惧"、`PUREBRILLIANCE`＝"纯粹辉煌"（锚点见"快速判断"）；5 个覆盖注册名与 `wurt_merm_planar` 均无 NAMES 键，标题使用效果描述。
- 分档说明：`standard`——5 注册三链，各链独立小节；未选 complex，因无多变体状态机，链条均为单层生成／清理。
- 未覆盖：动画包帧数与视觉、投泥带本体在 SGwilson 内的完整施法动画细节、`wurt_swamp_terraformer` 的重叠地块回滚次序与读档续跑运行表现、充能层在持有者移除时的引擎级联、月影 FX 跨端可见性、技能树完整结构；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- 返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]（角色技能/沃特）
