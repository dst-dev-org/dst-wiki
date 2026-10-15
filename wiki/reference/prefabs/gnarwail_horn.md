---
title: "一角鲸的角（gnarwail_horn）"
created: "2026-11-06"
updated: "2026-11-06"
type: prefab
summary: "原版海洋施法物 gnarwail_horn：一角鲸掉落的可装备吹奏号角，右键点海面从目标两侧生成即时激活的 wave_med 攻击浪并顺带照料农田，耐久 10 次、耗尽即碎；不是武器（无 weapon 组件），也是 TECH.LOST 蓝图配方 trident 的原料（需 3 个）。"
aliases: ["gnarwail_horn", "一角鲸的角", "Gnarwail Horn", "海号角", "海号角武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/gnarwail_horn.lua", "prefabs/gnarwail.lua", "prefabs/wave.lua", "prefabs/pigking.lua", "ocean_util.lua", "messagebottletreasures.lua", "components/messagebottlemanager.lua", "components/spellcaster.lua", "components/equippable.lua", "components/finiteuses.lua", "components/farmplanttendable.lua", "components/combat.lua", "components/playeractionpicker.lua", "components/playercontroller.lua", "standardcomponents.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "mods.lua", "tuning.lua", "recipes.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 一角鲸的角（gnarwail_horn）

## 快速判断

- 职责与适用：`prefabs/gnarwail_horn.lua` 顶层 `fn` 注册的单注册可拾取施法物（物品＋装备）：equippable HANDS 槽持握展示、finiteuses 耐久 10、spellcaster 右键点海面施法（`canuseonpoint_water`）；实体字段 `spelltype = "MUSIC"`，动作文案走"演奏"。**不是武器**：无 weapon 组件、不提供伤害（combat 只认带 weapon 组件的 HANDS 物品），`nopunch` 标签只把攻击表现从拳击改成挥击；同文件无 quagmire 分支，`MakeHauntableLaunch` 可作祟弹射。[工厂](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L81-L153)、[注册](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L156)
- 优先使用：复刻"点水吹奏施法"按[吹奏施法链](#吹奏施法链)抄 spellcaster＋reticule＋SG 接线；改耐久／施法参数查[数值与消耗](#数值与消耗)；给玩家 SG 加吹奏类动作查 play_gnarwail_horn 条目；制作 trident 需要 3 个本物，见 [[reference/prefabs/trident]]；从零造同类物品查 [[cookbook/equipment/hand]]。
- 重要副作用：吹奏时在目标点两侧各尝试生成 1 道 wave_med 攻击浪（即时激活：推船、冲开地面物品、泼湿加 9 湿度，见 [[reference/prefabs/wave_med]]），**至少一道浪生成成功才扣 1 耐久**；无论是否成浪，只要持有者非空就以持有者为中心照料 20 范围内可照料农田；耐久耗尽给持有者推 `toolbroke` 后整体 `Remove`。[扣耐久](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L68-L69)、[照料](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L73-L75)
- 修改边界：标签、reticule、floater 挂公共端，组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L111-L114)）；吹奏动作状态同时存在于服务端 SGwilson 与客户端 SGwilson_client（`server_states` 接线），改动作须两端同步；本页不覆盖 wave_med 内部机制（归 [[reference/prefabs/wave_med]]）与一角鲸 AI／掉落细节（`prefabs/gnarwail.lua`，Wiki 待建）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `gnarwail_horn` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备＋施法物；无皮肤（skinprefabs.lua 全文无 gnarwail_horn，mods.lua 无 `__index` 回退之外的差异不涉及）。[注册](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L156) |

共同初始化（[gnarwail_horn.lua#L81-L153](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L81-L153)）：Transform／AnimState／Network → `MakeInventoryPhysics` → 公共端标签 `gnarwail_horn`／`nopunch`／`allow_action_on_impassable` → 实体字段 `spelltype = "MUSIC"` → reticule（目标点玩家前方 3.5、双摇杆参数、`validfn` 限海面且非阻挡）→ `SetBank("gnarwail_horn")`＋`SetBuild("gnarwail_horn")`＋`idle` → `MakeInventoryFloatable("med", 0, {1, 0.6, 1})` → `SetPristine`（非主模拟返回）→ 主模拟依次 floater `SetBankSwapOnFloat(true, -16, swap_gnarwailhorn_float)` → `MakeHauntableLaunch` → inspectable → inventoryitem → tradable → finiteuses → equippable → `inst.playsound` → spellcaster。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.GNARWAIL_HORN`＝一角鲸的角（Gnarwail Horn），掉落者 `STRINGS.NAMES.GNARWAIL`＝一角鲸；检视文案"不可思议！"；施法动作文案 `STRINGS.ACTIONS.CASTSPELL.MUSIC`＝演奏。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L343340-L343342)、[.po 一角鲸](../../../raw/current/scripts/languages/chinese_s.po#L343335-L343337)、[.po 检视](../../../raw/current/scripts/languages/chinese_s.po#L13324-L13327)、[.po 动作](../../../raw/current/scripts/languages/chinese_s.po#L475-L477)

### 数值与消耗

| 项 | 键／来源 | 实际值 |
|---|---|---|
| 耐久上限＝初始 | `TUNING.GNARWAIL_HORN.USES` | 10 次（[tuning.lua#L5120-L5122](../../../raw/current/scripts/tuning.lua#L5120-L5122)；图鉴 finiteuses=10 同值，[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L613)） |
| 每次吹奏消耗 | `create_waves` 内 `finiteuses:Use(1)`，条件 s1 or s2 成浪 | 1 耐久点/次，未设 `SetConsumption`（动作消耗路径不触达）([调用](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L68-L69)) |
| 农田照料半径 | `TUNING.GNARWAIL_HORN_FARM_PLANT_INTERACT_RANGE` | 20，以持有者为中心（[tuning.lua#L5124](../../../raw/current/scripts/tuning.lua#L5124)，注释"castspell action distance"与 `ACTIONS.CASTSPELL.distance`＝20 同源，[actions.lua#L455](../../../raw/current/scripts/actions.lua#L455)） |
| 攻击浪速度／prefab／idle | `SpawnAttackWave(wp, angle, nil, nil, 0.5, true)` 缺省参数 | 速度 6、`wave_med`、idle_time 0.5 秒（[调用](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L52)、[缺省](../../../raw/current/scripts/ocean_util.lua#L31-L33)） |
| 攻击浪落点 | 水花在目标点垂直方向 ±1.5；浪再沿施放方向前推 1.5（`WAVE_SPAWN_DISTANCE`），以最终点判海面 | [水花点](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L47-L66)、[前推](../../../raw/current/scripts/ocean_util.lua#L29-L55) |
| 船袭角生命（掉落源） | `TUNING.GNARWAIL.HORN_HEALTH` | 130（[tuning.lua#L5117](../../../raw/current/scripts/tuning.lua#L5117)、[挂载](../../../raw/current/scripts/prefabs/gnarwail.lua#L163)） |

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 击杀一角鲸（角完好时） | 战利品表 `gnarwail_loot_horn`＝4 鱼肉＋1 本物；角打断后只掉鱼肉。`SetHornBroken` 切换两套战利品，生成时默认设为完好。[掉落表](../../../raw/current/scripts/prefabs/gnarwail.lua#L29)、[切换](../../../raw/current/scripts/prefabs/gnarwail.lua#L387-L394)、[生成默认](../../../raw/current/scripts/prefabs/gnarwail.lua#L587-L590) |
| 打断船袭角 | 一角鲸船袭实体 `gnarwail_attack_horn`（health 130）死亡时 `DropLoot` 掉 1 本物。[实体](../../../raw/current/scripts/prefabs/gnarwail.lua#L137)、[战利品](../../../raw/current/scripts/prefabs/gnarwail.lua#L170) |
| 沉没宝箱 | 漂流瓶宝藏 `sunkenchest` 的 traveler 预设保底含 1 本物；宝藏经 messagebottlemanager 生成。[预设](../../../raw/current/scripts/messagebottletreasures.lua#L64-L76)、[生成入口](../../../raw/current/scripts/components/messagebottlemanager.lua#L210) |
| 配方原料（不产出） | trident 配方需 3 个本物（`TECH.LOST` 蓝图习得），制作不消耗合成途径之外的逻辑，见 [[reference/prefabs/trident]]。[配方](../../../raw/current/scripts/recipes.lua#L491) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：finiteuses | 上限＝当前＝10；`SetOnFinished(onusesfinished)`：给持有者推 `toolbroke` 后 `Remove`；施法侧直接 `Use(1)` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L134-L137)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L74) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`（`swap_gnarwailhorn`）＋ARM_carry 显隐 | 主模拟；槽默认 HANDS | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L141-L143)、[回调](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L28-L32) |
| 组件：spellcaster | `SetSpellFn(create_waves)`＋`canuseonpoint_water = true`（实体得 `castonpointwater` 标签）；组件级 `spelltype` 保持 nil → 无 `_spelluser` 施法者门槛 | 主模拟；`CanCast` 点位路径要求 `IsOceanAtPoint` 且非被阻挡地面点 | [[reference/components/spellcaster#关键接口]]；[接线](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L149-L151)、[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L173-L174)、[spelltype 默认](../../../raw/current/scripts/components/spellcaster.lua#L84) |
| 组件：reticule | `targetfn` 取玩家前方 3.5、`ease`／双摇杆参数、`validfn`＝`IsOceanAtPoint` 且非被阻挡 | 公共端；客户端瞄准环 | [[reference/components/reticule#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L96-L103)、[validfn](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L16-L17) |
| 组件：inspectable／inventoryitem／tradable | 检视、拾取（耐久回调读 `owner`、施法读 `GetGrandOwner`）与可交易；tradable 默认 `goldvalue` 0，未设 `tradefor`／`rocktribute` | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]、[[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L122-L130) |
| 组件：floater | `MakeInventoryFloatable("med", 0, {1, 0.6, 1})`；主模拟再 `SetBankSwapOnFloat(true, -16, {sym_build="gnarwail_horn", sym_name="swap_gnarwailhorn_float"})` | 公共端挂组件；浮水换符号主模拟设置 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L109)、[换装](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L116)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| API／动作：`ACTIONS.CASTSPELL` | 点海面动作入口：componentactions POINT 筛选按 `castonpointwater` 校验海面（排除驾驶／转舵中）；action fn 调 `CanCast`→`CastSpell`；`strfn` 读实体 `spelltype` 出"演奏"文案 | 客户端发起、服务端结算；动作距离 20 | [动作定义](../../../raw/current/scripts/actions.lua#L455)、[点位筛选](../../../raw/current/scripts/componentactions.lua#L2213-L2218)、[action fn](../../../raw/current/scripts/actions.lua#L3465-L3468)、[strfn](../../../raw/current/scripts/actions.lua#L3441-L3442) |
| API／数据：`allow_action_on_impassable` 标签 | playeractionpicker／playercontroller 将该标签并入 `alwayspassable`，允许对海面等不可通行点给点动作 | 客户端动作筛选 | [picker](../../../raw/current/scripts/components/playeractionpicker.lua#L463)、[controller](../../../raw/current/scripts/components/playercontroller.lua#L5405) |
| SG／状态：`play_gnarwail_horn` | 吹奏动作：tags `doing/busy/playing/canrotate`，播 `hornblow_pre`＋`hornblow`，17 帧播 `inst.playsound`（`hookline/creatures/gnarwail/horn`）并 `PerformBufferedAction`；27／30 帧按成败去 `busy` | 服务端 SGwilson 与客户端 SGwilson_client（`server_states` 同名绑定）；ActionHandler 按 `gnarwail_horn` 标签路由 | [ActionHandler](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1306)、[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9504-L9506)、[17 帧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9516-L9524)、[客户端状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L4436-L4439)、[客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L585-L588) |
| API／数据：`nopunch` 标签 | SGwilson attack 状态：装备物无 weapon 组件且有 `nopunch` 时攻击仍走"挥击"表现分支；combat `GetWeapon` 只认 weapon 组件，持本物攻击按徒手伤害结算 | 服务端表现＋伤害端；客户端同型分支镜像 | [attack 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8655-L8661)、[GetWeapon](../../../raw/current/scripts/components/combat.lua#L853-L863) |
| API／数据：`toolbroke` 事件 | 耐久耗尽回调推给持有者，SGwilson 监听后进 `toolbroke` 状态（工具碎裂演出） | 服务端事件 | [推事件](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L20-L25)、[监听](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2299-L2302) |
| 关联 Prefab：`wave_med`／`wave_splash` | 水花在角传入的 `wp` 点生成，浪在再前推 1.5 的最终点生成；wave_med `instant_rise` 态即激活（`waveactive=true`），碰撞推船（boatphysics ApplyForce）、冲开 `_inventoryitem` 物品、泼湿 +9 湿度，到陆地即消失 | 海面限定：`SpawnAttackWaves` 仅在 `IsPassableAtPoint` 失败（即海面）处生成 | [[reference/prefabs/wave_med]]；[海面判定](../../../raw/current/scripts/ocean_util.lua#L51)、[即时激活](../../../raw/current/scripts/ocean_util.lua#L65)、[推船](../../../raw/current/scripts/prefabs/wave.lua#L35-L40)、[推物](../../../raw/current/scripts/prefabs/wave.lua#L65-L66)、[泼湿](../../../raw/current/scripts/prefabs/wave.lua#L16-L18)、[到岸消失](../../../raw/current/scripts/prefabs/wave.lua#L47-L52)、[wave_splash 注册](../../../raw/current/scripts/prefabs/wave.lua#L195-L196) |
| 关联 Prefab：`gnarwail`（一角鲸）／`gnarwail_attack_horn` | 掉落来源（本体击杀／打断船袭角），见[获取途径](#获取途径) | 主模拟；一角鲸行为 Wiki 待建 | [掉落方](../../../raw/current/scripts/prefabs/gnarwail.lua#L137)、[L29](../../../raw/current/scripts/prefabs/gnarwail.lua#L29) |
| 关联 Prefab：`trident`／沉没宝箱 `sunkenchest` | trident 配方原料×3；traveler 沉没宝箱保底 1 个 | 制作链／宝箱链 | [[reference/prefabs/trident]]；[配方](../../../raw/current/scripts/recipes.lua#L491)、[宝箱预设](../../../raw/current/scripts/messagebottletreasures.lua#L64-L76) |
| API／数据：`tendable_farmplant` 标签＋`farmplanttendable:TendTo` | 吹奏时以持有者为中心 20 范围搜索并对可照料作物 `TendTo`；作物侧 `tendable` 且 `ontendtofn` 成功才结算并清 `tendable` | 主模拟；与浪是否生成无关 | [[reference/components/farmplanttendable#关键接口]]；[搜索](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L73-L75)、[TendTo](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31) |
| 资源：anim／音效 | gnarwail_horn.zip（bank＋build＋`swap_gnarwailhorn`／`swap_gnarwailhorn_float` 符号）、INV_IMAGE `gnarwail_horn`；吹奏音效 `hookline/creatures/gnarwail/horn` | 静态声明 | [声明](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L1-L10)、[音效](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L147) |

## 典型接入

### 吹奏施法链

- 前提与选择：理解原版接线再复刻或改参。完整链条：右键海面点 → componentactions POINT `spellcaster` 筛选（`castonpointwater` → `IsOceanAtPoint`，排除 `steeringboat`／`rotatingboat`）→ `ACTIONS.CASTSPELL`（动作距离 20）→ SGwilson ActionHandler 按 `gnarwail_horn` 标签进 `play_gnarwail_horn`（客户端先行预测，同名 server_states）→ 17 帧 `PerformBufferedAction` → action fn `CanCast`（spellcaster 同样海面校验）→ `CastSpell(nil, pos, doer)` → `create_waves` → 两次 `SpawnAttackWave`。[点位筛选](../../../raw/current/scripts/componentactions.lua#L2213-L2218)、[action fn](../../../raw/current/scripts/actions.lua#L3465-L3468)、[SG 路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1306)
- 生效与撤销：`create_waves` 先取 `GetGrandOwner()`，为空直接返回（动作与音效照演，但不生成浪、不扣耐久、不照料）；两浪的水花在垂直连线 ±1.5 处，浪本体再沿目标方向前推 1.5，以最终位置判海面；扣耐久与照料规则见[快速判断](#快速判断)。[Owner 检查](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L41-L45)
- 原版依据：实体字段 `spelltype="MUSIC"` 只供动作文案（actions.lua strfn），组件级 `spelltype` 未设，`CanCast` 不要求施法者标签；`inst.playsound` 由 SG 状态在 17 帧读取播放，是"prefab 数据 → SG 消费"的轻量接线样例。[spelltype 消费](../../../raw/current/scripts/actions.lua#L3441-L3442)、[playsound 消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9516-L9524)

### 调整耐久与施法参数

- 前提与选择：服务端 `modmain.lua`。耐久在 prefab 构造时读 `TUNING.GNARWAIL_HORN.USES`，postinit 时机在组件挂载后，用 `SetMaxUses`／`SetUses` 覆盖；农田照料半径每次施法运行时读 `TUNING`，modmain 环境直挂 `TUNING` 白名单（mods.lua env 无 `__index` 回退），`TheWorld` 等须 `GLOBAL.` 前缀（[TUNING 注入](../../../raw/current/scripts/mods.lua#L278)、[GLOBAL](../../../raw/current/scripts/mods.lua#L289)、[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)）。
- 生效与撤销：postinit 只影响新实例；`TUNING.GNARWAIL_HORN_FARM_PLANT_INTERACT_RANGE` 改动即时影响所有实例的照料搜索。停用模组恢复原值。

```lua
-- modmain.lua：耐久 30；农田照料半径 30
AddPrefabPostInit("gnarwail_horn", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.finiteuses:SetMaxUses(30)
    inst.components.finiteuses:SetUses(30)
end)

-- 照料半径每次施法运行时读取，改 TUNING 即时生效（含既有实例）
TUNING.GNARWAIL_HORN_FARM_PLANT_INTERACT_RANGE = 30
```

- 调整与验证：耐久单位"次"（每次成功成浪扣 1）；照料半径为世界单位。浪速 6 与 `wave_med` 是 `SpawnAttackWaves` 的缺省参数、不走 TUNING，要改须在 postinit 重新 `spellcaster:SetSpellFn` 包装 `create_waves` 或复刻该函数；动作状态在 SGwilson／SGwilson_client 双端，改吹奏表现须两端同步（见[陷阱](#常见陷阱与限制)）。

## 常见陷阱与限制

- **误区：它是武器／能加伤害**：无 weapon 组件，combat `GetWeapon` 只认带 weapon 组件的 HANDS 物品，持本物攻击按徒手结算；`nopunch` 标签只把攻击表现从空手拳击换成挥击分支。[GetWeapon](../../../raw/current/scripts/components/combat.lua#L853-L863)、[表现分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8655-L8661)
- **边界：扣耐久要求至少一道浪真的生成**：`SpawnAttackWaves` 在“目标点 ±1.5 再前推 1.5”的最终位置检查海面（`IsPassableAtPoint` 失败），两浪全部落空则不扣耐久；瞄准环 `validfn` 只校验目标点本身，边界点可能出现"环合法但两浪落空"。[最终点判定](../../../raw/current/scripts/ocean_util.lua#L43-L55)、[扣耐久条件](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L68-L69)
- **边界：只能点海面施法**：`canuseonpoint_water` 分支要求 `IsOceanAtPoint` 且非被阻挡地面点；驾驶／转舵中的船不给出动作。[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L173-L174)、[点位筛选](../../../raw/current/scripts/componentactions.lua#L2213-L2218)
- **边界：无主不生效**：物品不在持有链（如掉在地上被对准施放）时 `GetGrandOwner` 为空，动作与音效照演但无任何效果、不耗耐久。[早退](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L41-L45)
- **边界：农田照料以持有者为中心、与浪无关**：搜索半径 20 内带 `tendable_farmplant` 标签的实体直接 `TendTo`，浪打不打得到作物、目标点在哪都不影响；作物侧还需 `tendable` 且 `ontendtofn` 成功才结算。[搜索](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L73-L75)、[TendTo](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31)
- **边界：耐久耗尽即整个实体移除**：先给持有者推 `toolbroke` 再 `Remove`，无残骸掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L20-L25)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：吹奏动作是双端状态**：`play_gnarwail_horn` 在服务端 SGwilson 与客户端 SGwilson_client 各有一份（客户端经 `server_states` 绑定预测），模组替换或加状态须两端同名，否则预测失配。[客户端状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L4436-L4439)
- **边界：可作祟**：`MakeHauntableLaunch` 被作祟时按 `HAUNT_CHANCE_ALWAYS` 概率向作祟者反向 `Launch` 弹开（HAUNT_TINY）。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/gnarwail_horn.lua 全文（工厂、施法、装备、耐久、floater／hauntable、reticule）、spellcaster／equippable／finiteuses／farmplanttendable 关键方法、ocean_util `SpawnAttackWaves` 缺省值与海面条件、wave.lua wave_med 碰撞推船／推物／泼湿／到岸消失与 SGwave `instant_rise` 激活、SGwilson(_client) ActionHandler 与 `play_gnarwail_horn` 状态及 17 帧音效＋动作执行、`nopunch` 表现分支与 combat `GetWeapon` 徒手路径、`toolbroke` 监听、tuning 三键（USES／FARM 范围／HORN_HEALTH）、recipes L491、gnarwail.lua 掉落三途径（本体／船袭角／`SetHornBroken` 切换）、messagebottletreasures traveler 预设与 messagebottlemanager 生成入口、pigking `goldvalue` 消费、componentactions POINT 筛选与 actions.lua 动作入口、playeractionpicker／playercontroller `alwayspassable`、standardcomponents 两 helper、mods.lua 环境（TUNING 注入、`GLOBAL = _G`、全文件无 `__index`）、scrapbookdata 显示数据；全 scripts grep "gnarwail_horn" 复核其余命中（debugcommands、prefablist、scrapbook_prefabs、skins_defs_data 的图鉴 key）均非生成点。`attackdodger` 不在本体组件清单，全库消费者为伍迪鹅形态技能／鱼人王冠 buff（combat.lua Dodge 钩子），与本物无关。名称与文案以 chinese_s.po `msgctxt` 定向核验（NAMES／DESCRIBE／CASTSPELL.MUSIC）。
- 游戏运行：未运行。未覆盖：浪的实机观感（推船力度、泼湿范围、±1.5 落点）、17 帧音效实机对位、照料半径实测、"环合法但两浪落空"的实机边界。
- 结构检查：validate_page.py（0 error，122 行／24029 字节，standard 档）与 validate_source_links.py --strict（96 条源码引用、23 个源文件，0 error 0 warning）实际执行通过。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
