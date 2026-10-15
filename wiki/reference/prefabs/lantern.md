---
title: "提灯（lantern）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版手持照明装备 lantern：HANDS 槽 machine 开关＋CAVE 燃料计时，装备自动点亮、入袋自动熄灭、落地按剩余燃料重新点亮，亮度与半径随剩余燃料渐变；无 weapon 组件按空手伤害；耗尽不销毁可反复充能。"
aliases: ["lantern", "提灯", "灯笼", "mininglantern", "开关提灯", "提灯照明"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/mininglantern.lua", "components/equippable.lua", "components/fueled.lua", "components/fuel.lua", "components/machine.lua", "components/inventoryitem.lua", "components/inventory.lua", "components/inspectable.lua", "components/combat.lua", "componentactions.lua", "actions.lua", "prefabs/lightbulb.lua", "prefabs/fireflies.lua", "prefabs/slurtleslime.lua", "prefabs/charlie_boss_trial.lua", "stategraphs/SGwilson.lua", "prefabs/player_common.lua", "prefabskin.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "constants.lua", "class.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 提灯（lantern）

## 快速判断

- 职责与适用：`prefabs/mininglantern.lua` 注册两个 Prefab——本体 `lantern`（可拾取手持装备）与照明实体 `lanternlight`，注册名与文件名不同。本体是 machine 开关灯＋fueled（CAVE 燃料）驱动的可复用照明装备：无 weapon 组件、无点燃/点火能力、耗尽不销毁。[注册](../../../raw/current/scripts/prefabs/mininglantern.lua#L302-L303)；同文件无 quagmire 分支（全文核验）
- 优先使用：复刻“可充燃料照明＋开关”按[关联机制导航](#关联机制导航)组装并查 [[cookbook/equipment/hand]]；改燃料／光照数值查[数值表](#注册与构造)；改开关行为读[开关灯流程](#开关灯与状态流转machine)；皮肤特效分工见[皮肤挂接协议](#皮肤挂接协议与-fx-页分工)；对照同批手持照明 [[reference/prefabs/torch]]／[[reference/prefabs/lighter]]。
- 重要副作用：有燃料时装备即自动点亮、入袋自动熄灭；落地固定先 `turnoff` 再 `turnon`——光挂到提灯本体，余量非空就重新点亮（关着丢下也会亮），余量为空则保持熄灭。光强／半径随燃料余量渐变；耗尽自动熄灭，只有装备且有持有者时才广播 `torchranout`（当前无原版监听者）。右键可手动开关（仅装备态或地面）。[turnon](../../../raw/current/scripts/prefabs/mininglantern.lua#L64-L89)、[turnoff](../../../raw/current/scripts/prefabs/mininglantern.lua#L91-L110)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/mininglantern.lua#L251-L257)）；光照参数硬编码在 `fuelupdate`（非 TUNING 键）；`lanternlight` 仅主模拟生成但为联网实体；皮肤 FX 名单与特效内部机制归各 `lantern_*_fx` 页，本页只记挂接协议。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lantern` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤仅换 build／图标，不换工厂（见[皮肤挂接协议](#皮肤挂接协议与-fx-页分工)）[注册](../../../raw/current/scripts/prefabs/mininglantern.lua#L302) |
| `lanternlight` | `lanternlightfn()` 纯 FX 实体，随文件同批注册 | 主模拟生成、联网、`persists=false`；[构造](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)、[注册](../../../raw/current/scripts/prefabs/mininglantern.lua#L303) |

`lantern` 共同初始化（[mininglantern.lua#L232-L299](../../../raw/current/scripts/prefabs/mininglantern.lua#L232-L299)）：实体组件＋`MakeInventoryPhysics` → bank `lantern`＋build `lantern`＋`idle_off` → 公共端标签 `light` → `MakeInventoryFloatable("med", 0.2, 0.65)`（中等浮态、纵向偏移 0.2、缩放 0.65）→ `SetPristine`（非主模拟返回）→ 主模拟依次 inspectable → inventoryitem（OnDropped／OnPutInInventory 回调）→ equippable → fueled → machine（`cooldowntime=0`）→ `MakeHauntableLaunch` → equippable 三回调 → `OnRemoveEntity` → `_onownerequip` 兜底监听。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790)、[作祟弹飞](../../../raw/current/scripts/standardcomponents.lua#L1053)

名称经简中 .po `msgctxt` 核验：`STRINGS.NAMES.LANTERN`＝提灯（Lantern）；通用检视语＝“更文明的灯具。”；`lanternlight` 无 NAMES 翻译条目（LANTERN 前缀仅一条名称键）。[名称](../../../raw/current/scripts/languages/chinese_s.po#L344450-L344452)、[检视语](../../../raw/current/scripts/languages/chinese_s.po#L14430-L14432)

数值（tuning 键＋实际值；`seg_time`＝30 秒、`night_time`＝60 秒、`dusk_time`＝120 秒，[tuning.lua#L19-L29](../../../raw/current/scripts/tuning.lua#L19-L29)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 燃料总量 | `TUNING.LANTERN_LIGHTTIME`＝`(night_time+dusk_time)*2.6` | 468 燃料单位＝rate 1 下 468 秒（[tuning.lua#L1727](../../../raw/current/scripts/tuning.lua#L1727)） |
| 点亮首扣时长 | `TUNING.TURNON_FUELED_CONSUMPTION`／`TURNON_FULL_FUELED_CONSUMPTION` | 0.5 秒量／满燃料 1/60 秒量（[tuning.lua#L1668-L1669](../../../raw/current/scripts/tuning.lua#L1668-L1669)） |
| 光照半径／光强 | `fuelupdate` 硬编码 `Lerp(3, 5, fuelpercent)`／`Lerp(.4, .6, fuelpercent)` | 满料半径 5／强度 0.6，空料 3／0.4，衰减恒 0.9（[fuelupdate](../../../raw/current/scripts/prefabs/mininglantern.lua#L34-L41)） |
| 光色 | `lanternlightfn` 硬编码 | (180,195,150)/255（[构造](../../../raw/current/scripts/prefabs/mininglantern.lua#L216)） |
| 开关冷却 | `machine.cooldowntime`（本 prefab 显式置 0） | 0，`StartCooldown` 不生效（[挂载](../../../raw/current/scripts/prefabs/mininglantern.lua#L267-L270)、[machine 默认 3](../../../raw/current/scripts/components/machine.lua#L22)） |
| 空手攻击伤害 | `TUNING.UNARMED_DAMAGE`（无 weapon 组件回退） | 10 生命值（[tuning.lua#L346](../../../raw/current/scripts/tuning.lua#L346)、[player_common 接线](../../../raw/current/scripts/prefabs/player_common.lua#L2687)） |
| 湿燃料折扣 | `TUNING.WET_FUEL_PENALTY` | 0.75（[tuning.lua#L1671](../../../raw/current/scripts/tuning.lua#L1671)） |

获取途径与燃料来源：

| 途径 | 证据 |
|---|---|
| 配方：twigs 3＋rope 2＋lightbulb 2，`TECH.SCIENCE_TWO`＝`{SCIENCE=2}`（炼金引擎档） | [配方](../../../raw/current/scripts/recipes.lua#L433)、[科技](../../../raw/current/scripts/constants.lua#L1246)；配方不含萤火虫（萤火虫用于 pumpkin_lantern／minerhat，[对照](../../../raw/current/scripts/recipes.lua#L430-L432)） |
| 充能：荧光果 `fuel` 组件 `fuelvalue=MED_LARGE_FUEL`（90）、萤火虫 `LARGE_FUEL`（180）、蛞蝓黏浆 `MED_FUEL`（45），fueltype 均 `CAVE` | [lightbulb](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L58)、[fireflies](../../../raw/current/scripts/prefabs/fireflies.lua#L233-L235)、[slurtleslime](../../../raw/current/scripts/prefabs/slurtleslime.lua#L63-L65)、[tuning](../../../raw/current/scripts/tuning.lua#L1675-L1677) |
| 消费端：影之手按 `light`＋`turnedon` 标签寻点亮提灯为目标 | [charlie_boss_trial](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L318-L321) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：machine | `turnonfn/turnofffn` 接开关灯、`cooldowntime=0`；`ison` 进存档，`OnLoad` 按值重放 TurnOn/TurnOff | 主模拟；构造时 `enabled=true` 即加 `enabled` 标签（watcher 在 ctor 赋值时触发，[class.lua](../../../raw/current/scripts/class.lua#L181-L193)） | [[reference/components/machine#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/mininglantern.lua#L267-L270)、[存档](../../../raw/current/scripts/components/machine.lua#L50-L61) |
| 组件：fueled | `fueltype=FUELTYPE.CAVE`＋`accepting=true`；`InitializeFuelLevel(LANTERN_LIGHTTIME)`；耗尽 `nofuel`、更新 `fuelupdate`、充能 `ontakefuel`、首扣 `SetFirstPeriod` | 主模拟；`currentfuel<=0` 加 `fueldepleted` 标签挡 TURNON | [[reference/components/fueled#关键接口]]；[配置](../../../raw/current/scripts/prefabs/mininglantern.lua#L265-L278)、[标签](../../../raw/current/scripts/components/fueled.lua#L74-L81) |
| 组件：equippable | 全套回调 OnEquip／OnUnequip／OnEquipToModel；槽默认 HANDS | 主模拟；装备态重物入 BODY 时换手逻辑见[开关灯流程](#开关灯与状态流转machine) | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mininglantern.lua#L284-L286) |
| 组件：inventoryitem | `SetOnDroppedFn(ondropped)`、`SetOnPutInInventoryFn(turnoff)`；`ChangeImageName` 切换 `lantern_lit`／基础图标 | 主模拟回调；`imagename` 经 `imagechange` 事件同步客户端 | [[reference/components/inventoryitem#关键接口]]；[回调](../../../raw/current/scripts/prefabs/mininglantern.lua#L259-L261)、[ChangeImageName](../../../raw/current/scripts/components/inventoryitem.lua#L470-L473) |
| 组件：inspectable | 裸挂载无自定义检视状态，走通用检视语 | 主模拟 | [[reference/components/inspectable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/mininglantern.lua#L257)、[构造](../../../raw/current/scripts/components/inspectable.lua#L1-L8) |
| 组件：floater／hauntable | 公共端 `MakeInventoryFloatable("med", 0.2, 0.65)`；主模拟 `MakeHauntableLaunch` 作祟弹飞 | floater 公共端、hauntable 主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/mininglantern.lua#L249)、[调用](../../../raw/current/scripts/prefabs/mininglantern.lua#L282) |
| 关联 Prefab：`lanternlight` | 点亮时 `SpawnPrefab` 并 `SetParent(owner 或本体)`；`fuelupdate` 每次燃料更新刷 Light 参数；宿主熄灭／移除时 `Remove`（`onremove` 监听清 `_light`） | 主模拟生成、联网实体客户端可见；`persists=false` 不入存档 | 详见[下节](#lanternlight照明实体同页简述)；[生成](../../../raw/current/scripts/prefabs/mininglantern.lua#L70-L77)、[清理](../../../raw/current/scripts/prefabs/mininglantern.lua#L43-L45) |
| 关联 Prefab：`lantern_*_fx` 皮肤特效 | 本体只推 `lantern_on/off` 事件；生成、跟随、Kill 全在皮肤侧协议与各 FX 页 | 仅主模拟；无皮肤不接 FX | 见[皮肤挂接协议](#皮肤挂接协议与-fx-页分工)；各 FX 页：[[reference/prefabs/lantern_flower_fx_held]]（同族主页）等 |
| API／事件：`lantern_on`／`lantern_off` | turnon／turnoff 末尾 `PushEvent`；皮肤 FX 与模组监听入口 | 主模拟本地 `PushEvent`；远端客户端看到的是联网光源与 FX，不能直接监听该事件 | [推](../../../raw/current/scripts/prefabs/mininglantern.lua#L87)、[推](../../../raw/current/scripts/prefabs/mininglantern.lua#L109)、[皮肤监听](../../../raw/current/scripts/prefabskin.lua#L3489-L3492) |
| API／事件：`machineturnedon`／`machineturnedoff`／`takefuel` | TURNON/TURNOFF 动作经 `Machine:TurnOn/Off` 推送；充能经 `TakeFuelItem` 推送 `takefuel` | 主模拟；本体 `turnon/turnoff` 内不重复推 | [machine](../../../raw/current/scripts/components/machine.lua#L86-L93)、[TakeFuelItem](../../../raw/current/scripts/components/fueled.lua#L223-L226) |
| API／事件：`torchranout` | 装备态耗尽先 `turnoff` 再向持有者推送，data=`{prefab, equipslot}`（沿火把命名） | 当前全 scripts 无监听者（`ListenForEvent("torchranout")` 检索为空，检索范围 raw/current/scripts） | [nofuel](../../../raw/current/scripts/prefabs/mininglantern.lua#L172-L184) |
| API／数据：`light`／`turnedon` 标签 | `light` 公共端常驻：攻击动画分支（无 weapon 时仍播标准 atk）＋影之手寻的目标；`turnedon` 由 machine `ison` watcher 维护 | 两端可读；`turnedon` 仅点亮时存在 | [SGwilson 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8653-L8662)、[影之手](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L318-L321)、[machine 标签](../../../raw/current/scripts/components/machine.lua#L1-L3) |
| API／数据：CAVE 燃料配对 | 燃料物 `fuel` 组件加 `CAVE_fuel` 标签、本体 `fueled.accepting` 加 `CAVE_fueled`，动作收集按标签配对给 ADDFUEL/ADDWETFUEL | 服务端 `TakeFuelItem` 结算（湿燃料×0.75）；`CanAcceptFuelItem` 要求 fueltype 匹配 | [[reference/components/fuel#关键接口]]；[收集](../../../raw/current/scripts/componentactions.lua#L1301-L1320)、[配对](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[标签](../../../raw/current/scripts/components/fuel.lua#L1-L6) |
| API／数据：无 weapon 组件 | 装备态攻击经 `Combat:GetWeapon()` 取不到 weapon → `CalcDamage` 回退 `defaultdamage`＝10 | 主模拟；不磨损、无耐久、无点燃掷点 | [[reference/components/combat#关键接口]]；[GetWeapon](../../../raw/current/scripts/components/combat.lua#L853-L865)、[回退](../../../raw/current/scripts/components/combat.lua#L901-L902) |
| 资源：anim／图标／音效 | `lantern.zip`（bank/build）／`swap_lantern.zip`（持握符号＋`lantern_overlay`）／`wilson.fsb`／`INV_IMAGE lantern_lit` | 静态声明；皮肤 build 经 `GetSkinBuild` 覆盖符号 | [声明](../../../raw/current/scripts/prefabs/mininglantern.lua#L1-L7)、[onequip 覆盖](../../../raw/current/scripts/prefabs/mininglantern.lua#L126-L146) |

### lanternlight（照明实体，同页简述）

`lanternlight` 是 Transform＋Light＋SoundEmitter＋Network 的纯 FX 实体：`FX` 标签、`persists=false`、无 SG／Brain（[构造](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)）。光色 (180,195,150)/255 在 pristine 段两端一致；半径／强度／衰减由宿主 `fuelupdate` 设置，时机为点亮生成瞬间与每次 `DoUpdate`（消耗周期 1 秒），故亮度随燃烧每秒渐变（[fueled 周期](../../../raw/current/scripts/components/fueled.lua#L269-L278)）。实体唤醒播 `dontstarve/wilson/lantern_LP` 循环音、休眠停止（[音效](../../../raw/current/scripts/prefabs/mininglantern.lua#L194-L202)）；点亮音 `lantern_on` 播在光实体上、熄灭音 `lantern_off` 延迟一帧播在持有者上，快速开关会互相取消（[声音任务](../../../raw/current/scripts/prefabs/mininglantern.lua#L14-L32)）。Light 数值在主模拟设置，引擎自动同步客户端。

### 开关灯与状态流转（machine）

玩家侧收集（[componentactions.lua#L526-L543](../../../raw/current/scripts/componentactions.lua#L526-L543)）：右键且无 `cooldown`／`fueldepleted`／`alwayson`／`emergency` 标签、有 `enabled` 标签时，按 `turnedon` 有无插入 TURNOFF 或 TURNON；被持有时必须已装备，落地或装备态可用。执行（[actions.lua#L3285-L3304](../../../raw/current/scripts/actions.lua#L3285-L3304)）：`TURNON.fn→machine:TurnOn→turnonfn`——fuel 非空才 `StartConsuming`、生成/复用 `lanternlight` 挂到持有者或本体、`idle_on`、显示 `LANTERN_OVERLAY`、图标换 `_lit`、`machine.ison=true`；`TURNOFF.fn` 反之并推 `machineturnedoff`。动作定义 `priority=2`、`invalid_hold_action=true`（[注册](../../../raw/current/scripts/actions.lua#L443-L444)）。

自动状态流转（全部主模拟）：

1. 装备（[onequip](../../../raw/current/scripts/prefabs/mininglantern.lua#L126-L146)）：fuel 非空即点亮，空灯隐藏 overlay 不点亮。
2. 入袋（[OnPutInInventory→turnoff](../../../raw/current/scripts/components/inventoryitem.lua#L348-L358)）：卸下回背包、捡起地面亮灯都经此熄灭；换装路径经 `Inventory:Equip` 给回旧物品同样触发（[inventory.lua](../../../raw/current/scripts/components/inventory.lua#L1349-L1365)）。
3. 丢弃（[ondropped](../../../raw/current/scripts/prefabs/mininglantern.lua#L121-L124)）：无条件先 `turnoff` 再 `turnon`——光改挂提灯本体；余量非空就重新点亮并继续耗燃料，余量为空保持熄灭。
4. 兜底（[_onownerequip](../../../raw/current/scripts/prefabs/mininglantern.lua#L290-L297)）：本体已点亮但未装备时，监听持有者 `equip` 事件（data `{item, eslot}`，[推送](../../../raw/current/scripts/components/inventory.lua#L1377)）：装上其他 HANDS 物品或 BODY 重物即强制熄灭。
5. 耗尽（[nofuel](../../../raw/current/scripts/prefabs/mininglantern.lua#L172-L184)）：熄灭、空灯保留可再充；装备态另推 `torchranout`。
6. 充能（[ontakefuel](../../../raw/current/scripts/prefabs/mininglantern.lua#L186-L190)）：装备态喂燃料后立即重新点亮；动作文案“添加燃料”（[po](../../../raw/current/scripts/languages/chinese_s.po#L155-L157)）。
7. 存档：machine `ison` 与 fueled 余量分开保存；`SetPersistData` 按 `pairs(data)` 遍历组件 `OnLoad`，没有跨组件顺序保证——machine 的 `TurnOn` 只按当时余量判断，不能假定 fueled 一定先恢复（[读档循环](../../../raw/current/scripts/entityscript.lua#L1979-L2013)、[machine OnLoad](../../../raw/current/scripts/components/machine.lua#L56-L61)、[fueled OnLoad](../../../raw/current/scripts/components/fueled.lua#L139-L143)）。

### 皮肤挂接协议（与 FX 页分工）

本体侧：`onequip` 按 `GetSkinBuild` 用 `OverrideItemSkinSymbol` 覆盖 `swap_object`／`lantern_overlay` 并推 `equipskinneditem`／`unequipskinneditem`；图标按 `GetSkinName().."_lit"` 切换（[onequip](../../../raw/current/scripts/prefabs/mininglantern.lua#L126-L146)、[turnon](../../../raw/current/scripts/prefabs/mininglantern.lua#L86)）。皮肤侧（[prefabskin.lua#L3475-L3499](../../../raw/current/scripts/prefabskin.lua#L3475-L3499)）：`lantern_init_fn` 设 skin build，读 `SKIN_FX_PREFAB[build_name]` 得 held／ground 两个 FX 注册名并监听 `lantern_on`／`lantern_off`／`unequipped`／`onremove`；点亮且被装备时 held FX 挂持有者跟随 `swap_object`，点亮且落地时 ground FX 挂提灯本体，熄灭走 `lantern_off`（KillFX 或 Remove，[L3411-L3473](../../../raw/current/scripts/prefabskin.lua#L3411-L3473)）。特效内部机制归各 FX 页（flower 为同族主页，gothic／insect／winter 同构，crystal／tesla 独立工厂），本页不复制；照明始终由 `lanternlight` 承担。

## 典型接入

### 提高提灯照明半径（在原版渐变上叠加）

- 前提与选择：服务端 `modmain.lua`。半径／光强是 `fuelupdate` 内的硬编码 Lerp，不经 TUNING；`fuelupdate` 本体是 local 函数，但已通过 `fueled:SetUpdateFn` 存为 `fueled.updatefn`，postinit 包一层即可在原版每秒更新后再叠加。modmain 环境只直挂 `TUNING` 等白名单、无 `__index=_G` 回退，`TheWorld` 必须 `GLOBAL.` 前缀（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L292)）。
- 生效与撤销：影响之后生成的所有 lantern 实例（存档重建也重跑 postinit）；每次 `DoUpdate`（1 秒）重设半径，停用模组恢复原值。`turnon` 生成 light 瞬间直接调 local `fuelupdate`（不走 updatefn），加成从下一次更新起生效。

```lua
-- modmain.lua：提灯照明半径整体 +1（上限 6）
AddPrefabPostInit("lantern", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local fueled = inst.components.fueled
    if fueled == nil or fueled.updatefn == nil then
        return
    end
    local oldupdate = fueled.updatefn
    fueled.updatefn = function(inst_)
        oldupdate(inst_)
        local light = inst_._light
        if light ~= nil then
            light.Light:SetRadius(math.min(6, light.Light:GetRadius() + 1))
        end
    end
end)
```

- 调整与验证：原版半径范围 3~5 随燃料渐变，本例满料 6、空料 4（[fuelupdate](../../../raw/current/scripts/prefabs/mininglantern.lua#L34-L41)）；只改燃料总量可在 modmain 顶层赋值 `TUNING.LANTERN_LIGHTTIME`（实例构造时读取，[InitializeFuelLevel](../../../raw/current/scripts/components/fueled.lua#L289-L303)）。`_light` 仅主模拟存在，客户端实例不进本分支。

## 常见陷阱与限制

- **误区：提灯耗尽即销毁**：`nofuel` 只 `turnoff`，空灯保留在背包／地面，重新喂 CAVE 燃料即可再用（对比 torch／lighter 耗尽 `Remove`）。只有 hauntable 作祟等外部移除才丢失。[nofuel](../../../raw/current/scripts/prefabs/mininglantern.lua#L172-L184)
- **边界：丢在地上的点亮提灯继续耗燃料**：`ondropped` 重挂光到本体并保持 `StartConsuming`，直到耗尽或被捡起；燃料照常按 468 秒扣减。[ondropped](../../../raw/current/scripts/prefabs/mininglantern.lua#L121-L124)
- **边界：背包里的提灯不能右键开关**：machine 收集分支对“被持有且未装备”直接返回；开关只在装备态或落地可用，入袋自动熄灭是设计行为。[收集分支](../../../raw/current/scripts/componentactions.lua#L526-L543)
- **边界：手持提灯攻击是空手 10 伤害**：无 weapon 组件，`Combat:GetWeapon()` 返回 nil 走 `defaultdamage`；`light` 标签只决定攻击动画分支，不产生伤害或点燃，也不磨损本体。[GetWeapon](../../../raw/current/scripts/components/combat.lua#L853-L865)
- **边界：`torchranout` 当前无原版消费者**：事件名沿用火把，data 只有 `prefab`／`equipslot`；模组要播报／自动换灯需自行监听。检索范围：raw/current/scripts 全部 .lua。[nofuel](../../../raw/current/scripts/prefabs/mininglantern.lua#L172-L184)
- **边界：改光参数必须覆盖 `fuelupdate` 生效路径**：每秒 `DoUpdate` 都会用 Lerp(3,5) 覆盖半径，直接 postinit 改一次 `_light.Light` 会被下一次更新冲掉；见[典型接入](#典型接入)。空灯充能后亮度按新百分比渐变（DoDelta 只推事件，靠下一 tick 更新）。[DoUpdate](../../../raw/current/scripts/components/fueled.lua#L325-L337)
- **边界：`fueldepleted` 与 `turnon` 双重拦截**：燃料为 0 时 TURNON 被动作收集挡住，`turnon` 内还有 `IsEmpty` 检查兜底；machine `OnLoad` 重放 TurnOn 也过同一检查，空灯存档恢复不会假亮。[turnon](../../../raw/current/scripts/prefabs/mininglantern.lua#L64-L65)、[machine OnLoad](../../../raw/current/scripts/components/machine.lua#L56-L61)

## 验证记录

- 源码审阅：2026-11-07（第二轮修订），Steam buildid 26335394。复核 ondropped 的先关后开、nofuel 的装备／持有者门禁、PushEvent 本地语义与 SetPersistData 的组件遍历顺序。早前核验 mininglantern 全文（两注册、工厂分界、turnon/turnoff/nofuel/ondropped/onequip 系列、lanternlight 构造）、equippable／fueled／fuel／machine／inventoryitem／inventory／inspectable／combat、componentactions、actions、tuning 键、配方与科技、FUELTYPE、皮肤协议、charlie_boss_trial、SGwilson、standardcomponents、po、mods、class watcher；torchranout 全域检索无监听者；同文件无 quagmire 分支。
- 游戏运行：未运行。未覆盖：468 秒燃尽与亮度渐变观感为源码推导而非运行测量；存档开关灯恢复、丢地燃尽实机表现；皮肤 FX 实装外观归各 FX 页范围。
- 结构检查：validate_page.py 与 strict 均通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]；燃料机制详见 [[reference/components/fueled]]；同族照明对照 [[reference/prefabs/torch]]／[[reference/prefabs/lighter]]／[[reference/prefabs/redlantern|红灯笼]]。
