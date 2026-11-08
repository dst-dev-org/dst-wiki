---
title: 伞族（umbrella、grass_umbrella、voidcloth_umbrella、winona_telebrella）
created: "2026-11-07"
updated: "2026-11-09"
type: prefab
summary: 四把 HANDS 槽手持伞的工厂、组件组合与消费端：雨伞／花伞的防水保温与 fueled／perishable 两种耐久语义，暗影伞的地面雨罩机（raindome、酸雨免疫、破损修复、跟随 FX），传送伞的温娜技能门禁传送与电池充能；含配方、官方名、皮肤与获取途径。
aliases: ["umbrella", "grass_umbrella", "voidcloth_umbrella", "winona_telebrella", "雨伞", "花伞", "暗影伞", "传送伞", "Pretty Parasol", "Portasol", "Umbralla", "猪皮伞", "草伞", "温娜传送伞", "雨伞耐久"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/umbrella.lua", "prefabs/voidcloth_umbrella.lua", "prefabs/winona_telebrella.lua", "prefabs/forge_repair_kits.lua", "prefabs/player_common.lua", "prefabs/hermitcrab.lua", "prefabs/skilltree_winona.lua", "prefabs/skinprefabs.lua", "prefabs/charlie_stage.lua", "prefabs/winona_teleport_pad.lua", "components/equippable.lua", "components/fueled.lua", "components/waterproofer.lua", "components/insulator.lua", "components/raindome.lua", "components/remoteteleporter.lua", "components/machine.lua", "components/forgerepair.lua", "components/inventory.lua", "components/inventoryitem.lua", "components/acidlevel.lua", "components/lunarhailmanager.lua", "components/weather.lua", "components/caveweather.lua", "components/sanity.lua", "componentactions.lua", "actions.lua", "recipes.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po", "util/sourcemodifierlist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 伞族（umbrella、grass_umbrella、voidcloth_umbrella、winona_telebrella）

## 快速判断

- 职责与适用：`prefabs/umbrella.lua` 一文件注册两把基础伞（`umbrella` 雨伞、`grass_umbrella` 花伞），`voidcloth_umbrella.lua` 注册暗影伞＋跟随 FX，`winona_telebrella.lua` 注册传送伞＋跟随 FX。四把都是 HANDS 槽装备（[equippable.lua#L29](../../../raw/current/scripts/components/equippable.lua#L29) 默认槽，未改写），共用 `umbrella` 标签。
- 优先使用：查"某把伞的组件／耐久／配方"用[注册与构造](#注册与构造)与[耐久链](#耐久链)；查"防水保温怎么生效"用[消费端](#防水与保温消费端)；暗影伞开罩／破损用[雨罩机](#暗影伞雨罩机)，传送伞用[传送与充能](#传送伞传送与充能)。
- 重要副作用：基础伞耗尽即移除并推 `umbrellaranout`，玩家自动换装库存中另一把伞（[player_common.lua#L625-L636](../../../raw/current/scripts/prefabs/player_common.lua#L625-L636)）；暗影伞开罩后不可拾起、罩内掉理智、酸雨中装备反而回燃料；传送伞传送一次最多扣 1 天燃料，找不到目的地也照扣。
- 修改边界：覆盖四注册＋两 FX 的工厂、组件、配方、官方名、皮肤与已核验消费端标签；不覆盖传送 pad／电池内部机制、moisture 组件全文与动画资源。数值为快照源码推导值，未运行测量。

## 注册与构造

| 注册名（官方名） | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `umbrella`（雨伞） | `common_fn("umbrella")` → 主模拟 `pigskin()`：fueled USAGE＋waterproofer HUGE＋装备回调 | item＋equipment；耗尽即移除，无破损形态。工厂 [umbrella.lua#L168-L192](../../../raw/current/scripts/prefabs/umbrella.lua#L168-L192)、注册 [L194-L195](../../../raw/current/scripts/prefabs/umbrella.lua#L194-L195) |
| `grass_umbrella`（花伞） | `common_fn("parasol")` 同一骨架 → `grass()`：perishable＋waterproofer MED＋可燃 | item＋equipment；bank/build 用 parasol，装备换 `swap_parasol`（[umbrella.lua#L44-L68](../../../raw/current/scripts/prefabs/umbrella.lua#L44-L68)）。工厂 [umbrella.lua#L132-L166](../../../raw/current/scripts/prefabs/umbrella.lua#L132-L166) |
| `voidcloth_umbrella`（暗影伞） | `UmbrellaFn()` 独立工厂：pristine 标签＋raindome/floater → 主模拟全套组件 | item＋equipment；双形态：完好 ↔ 破损（`isbroken` net_bool 同步，卸装备不可用）。工厂 [voidcloth_umbrella.lua#L335-L435](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L335-L435) |
| `voidcloth_umbrella_fx`（跟随 FX） | `FollowSymbolFxFn()`：网络化实体，`swap_loop1` 跟随持有者 `swap_object` | fx；装备挂父级、卸下移除；highlightchild＋colouraddersync 接受高亮染色（[voidcloth_umbrella.lua#L437-L523](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L437-L523)）；工厂与清理归 [[reference/prefabs/voidcloth_boomerang_fx#持握层联网骨架本地帧层的分工]] |
| `winona_telebrella`（传送伞） | `fn()` 独立工厂：pristine 标签＋remoteteleporter/circuitnode/powerload | item＋equipment；equippable 动态挂——燃料 section>0 才有、为 0 自动卸（[winona_telebrella.lua#L105-L125](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L105-L125)）。工厂 [winona_telebrella.lua#L378-L472](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L378-L472) |
| `winona_telebrella_swap_fx`（跟随 FX） | `fxfn()`：网络化实体，跟随 `swap_object`，LED 符号 | fx；装备生成两份（`swap1`/`swap20`），技能点亮 LED（[winona_telebrella.lua#L20-L50](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L20-L50)、[L490-L523](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L490-L523)）；两层换装语义归 [[reference/prefabs/winona_battery_high_shatterfx#winonatelebrellaswapfx-的两层换装语义]] |

`common_fn` 共同骨架（[umbrella.lua#L88-L130](../../../raw/current/scripts/prefabs/umbrella.lua#L88-L130)）：inventory physics＋pristine 标签 `nopunch`／`umbrella`／`waterproofer`＋floatable；主模拟挂 tradable／waterproofer／inspectable／inventoryitem／equippable／**insulator `SetSummer()`**＋hauntable。

## 关联机制导航

### 组件分档

| 组件 | umbrella | grass_umbrella | voidcloth_umbrella | winona_telebrella | 证据 |
|---|---|---|---|---|---|
| waterproofer | HUGE 0.9 | MED 0.5 | ABSOLUTE 1.0 | SMALLMED 0.35 | [umbrella.lua#L145](../../../raw/current/scripts/prefabs/umbrella.lua#L145)、[L180](../../../raw/current/scripts/prefabs/umbrella.lua#L180)、[voidcloth_umbrella.lua#L397-L398](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L397-L398)、[winona_telebrella.lua#L419-L420](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L419-L420) |
| insulator（夏季） | MED 120 秒 | MED 120 秒 | LARGE 240 秒 | 无 | [umbrella.lua#L147](../../../raw/current/scripts/prefabs/umbrella.lua#L147)、[L182](../../../raw/current/scripts/prefabs/umbrella.lua#L182)、[voidcloth_umbrella.lua#L400-L402](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L400-L402) |
| fueled | USAGE 6 天，装备期耗 | 无 | MAGIC 15 天，装备期耗；开罩 ×1.5 | MAGIC 15 天，装备期耗；电池可充 | [tuning.lua#L7804](../../../raw/current/scripts/tuning.lua#L7805)、[L3704-L3707](../../../raw/current/scripts/tuning.lua#L3704-L3707) |
| perishable | 无 | 2 天全程腐坏 | 无 | 无 | [umbrella.lua#L139-L143](../../../raw/current/scripts/prefabs/umbrella.lua#L139-L143)、[tuning.lua#L1735](../../../raw/current/scripts/tuning.lua#L1735) |
| dapperness | 无 | +SMALL ≈16／天 | −MED（magic）≈−26.7／天 | 无 | [umbrella.lua#L152](../../../raw/current/scripts/prefabs/umbrella.lua#L152)、[voidcloth_umbrella.lua#L73-L74](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L73-L74) |
| 独有 | — | fuel＋burnable | machine＋raindome＋shadowlevel 3＋sanityaura（开罩） | remoteteleporter＋circuitnode＋powerload | 各工厂专属段 |

### 防水与保温消费端

- 防水：装备栏各 equipslot 的 waterproofer effectiveness 求和（[inventory.lua#L2715-L2731](../../../raw/current/scripts/components/inventory.lua#L2715-L2731)）供 moisture 折算淋雨增速；pristine `waterproofer` 标签只做客户端过滤优化。暗影伞的 raindome 更进一步：罩内地面物品连潮湿都不继承（[inventoryitem.lua#L150-L156](../../../raw/current/scripts/components/inventoryitem.lua#L150-L156)）。
- 保温：insulator type SUMMER，temperature 组件计入夏季隔热（[temperature.lua#L260-L266](../../../raw/current/scripts/components/temperature.lua#L260-L266)）即减缓过热、冬季无效；MED=120 秒／LARGE=240 秒（seg_time=30，[tuning.lua#L2261-L2263](../../../raw/current/scripts/tuning.lua#L2261-L2263)）。
- `umbrella` 标签：装备时雨声切伞声（[weather.lua#L991](../../../raw/current/scripts/components/weather.lua#L991)、[caveweather.lua#L693](../../../raw/current/scripts/components/caveweather.lua#L693)）；传送伞因 `metal` 标签用专属雨声（[weather.lua#L303-L312](../../../raw/current/scripts/components/weather.lua#L303-L312)）。隐居蟹雨天只接受一把 `umbrella` 标签赠礼并自动装备（[hermitcrab.lua#L190-L196](../../../raw/current/scripts/prefabs/hermitcrab.lua#L190-L196)、[hermitcrabbrain.lua#L777-L778](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L777-L778)）。
- 换算基准：`total_day_time`=480 秒、`day_time`=300 秒，别混用（[tuning.lua#L2214-L2215](../../../raw/current/scripts/tuning.lua#L2214-L2215)）。

### 耐久链

- **雨伞**：fueled USAGE 打 `needssewing` → 缝纫包唯一补给（[fueled.lua#L5-L18](../../../raw/current/scripts/components/fueled.lua#L5-L18)）；装备 `StartConsuming`、卸下／入袋停（[umbrella.lua#L22](../../../raw/current/scripts/prefabs/umbrella.lua#L22)、[L35](../../../raw/current/scripts/prefabs/umbrella.lua#L35)、[L38-L42](../../../raw/current/scripts/prefabs/umbrella.lua#L38-L42)）。耗尽 `onperish`：移除自身并推 `umbrellaranout`，玩家监听后自动装备库存中另一把 `umbrella` 标签同槽物品（[umbrella.lua#L70-L86](../../../raw/current/scripts/prefabs/umbrella.lua#L70-L86)、[player_common.lua#L667](../../../raw/current/scripts/prefabs/player_common.lua#L667)）。
- **花伞**：perishable 2 天＝960 秒（2×total_day_time，[tuning.lua#L1735](../../../raw/current/scripts/tuning.lua#L1735)），**背包箱内照常计时**，带 `show_spoilage`；另有 fuel（LARGE_FUEL）＋SmallBurnable/Propagator 可当柴烧（[umbrella.lua#L159-L163](../../../raw/current/scripts/prefabs/umbrella.lua#L159-L163)）；耗尽同走 `onperish`。
- **暗影伞**：FUELTYPE.MAGIC 无对应燃料物品（[constants.lua#L1999](../../../raw/current/scripts/constants.lua#L1999) 注释），不吃任何补给；唯一恢复是虚空修补套件 `voidcloth_kit`（forgerepair 对 fueled 直接回满，[forgerepair.lua#L40-L45](../../../raw/current/scripts/components/forgerepair.lua#L40-L45)、[forge_repair_kits.lua#L57](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L57)）。破损修复链见[雨罩机](#暗影伞雨罩机)。
- **传送伞**：同 MAGIC 不可喂燃料；掉地自动连 `engineeringbattery` 电路，通电电池周期 `AddBatteryPower` 推供电窗口，充能期 fueled.rate＝−15——负 rate 经 `DoDelta(-dt*rate*mod)` 变回充（[fueled.lua#L327](../../../raw/current/scripts/components/fueled.lua#L327)），满电自停；快充技能使速率 ×2（[winona_telebrella.lua#L158-L188](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L158-L188)、[tuning.lua#L7268](../../../raw/current/scripts/tuning.lua#L7269)）。供电窗口细节归发电机电池页（Wiki 待建）。

### 暗影伞雨罩机

- 开关：machine `SetGroundOnlyMachine(true)` → 右键 TURNON/TURNOFF 只在**地面**可用，持有或漂浮时动作被排除（[componentactions.lua#L528-L545](../../../raw/current/scripts/componentactions.lua#L528-L545)、[machine.lua#L46-L48](../../../raw/current/scripts/components/machine.lua#L46-L48)），冷却 0.5 秒。
- 开罩（[voidcloth_umbrella.lua#L240-L271](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L240-L271)）：`canbepickedup=false`、fueled.rate＝1.5（罩内 15÷1.5≈10 天）、raindome Enable 半径 16（[tuning.lua#L7806](../../../raw/current/scripts/tuning.lua#L7807)）、临时挂 sanityaura −SANITYAURA_SMALL（约 −200／天，范围 RADIUS²）＋barrier 动画＋本地 WaveFX/DomeFX（经 `net_event` triggerfx 客户端同步）＋循环音效。入袋／上浮自动关罩（[L312-L320](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L312-L320)）。
- raindome 消费端：圈内实体挂 rainimmunity；全局 `IsUnderRainDomeAtXZ` 把罩内点排除在月雹落点（[lunarhail.lua#L169](../../../raw/current/scripts/prefabs/lunarhail.lua#L169)）与洞穴酸雨（[caveacidrain.lua#L226](../../../raw/current/scripts/prefabs/caveacidrain.lua#L226)）之外（[raindome.lua#L29-L59](../../../raw/current/scripts/components/raindome.lua#L29-L59)）。
- 酸雨免疫：pristine `acidrainimmune` → 装备者酸雨伤害清零（[acidlevel.lua#L156-L157](../../../raw/current/scripts/components/acidlevel.lua#L156-L157)）、`IsAcidSizzling` 恒假（[entityscript.lua#L811-L815](../../../raw/current/scripts/entityscript.lua#L811-L815)）；酸雨期装备回调给 fueled.rate_modifiers 设 −1 修饰（[voidcloth_umbrella.lua#L11-L17](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L11-L17)），见[陷阱](#常见陷阱与限制)。
- 破损与修复：燃料耗尽 `OnPerish` → 关罩、播 `broken`、移除 equippable（装备中先卸入背包）、`SetIsBroken(true)`、加 `broken` 标签、检查名覆盖 BROKEN_FORGEDITEM（[voidcloth_umbrella.lua#L108-L142](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L108-L142)）；`OnLoad` 空燃料复现破损（[L329-L333](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L329-L333)）。`voidcloth_kit` 修复重挂 equippable 并清破损态（[L144-L152](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L144-L152)）；MakeForgeRepairable 挂 repairable 材料链（[standardcomponents.lua#L1851](../../../raw/current/scripts/standardcomponents.lua#L1851)）。
- `lunarhailprotection`：装备即免月雹伤害（[lunarhailmanager.lua#L150](../../../raw/current/scripts/components/lunarhailmanager.lua#L150)）；`shadow_item` 进暗影阵营判定，Wanda 持有时负 dapperness 按年龄态减免（×1／0.33／0，[wanda.lua#L201-L213](../../../raw/current/scripts/prefabs/wanda.lua#L201-L213)、[tuning.lua#L5024-L5026](../../../raw/current/scripts/tuning.lua#L5024-L5026)）。

### 传送伞传送与充能

- 动作：右键 REMOTE_TELEPORT（[actions.lua#L660](../../../raw/current/scripts/actions.lua#L660)）；因 pristine `engineering` 标签，动作只有 `handyperson`（温娜）可见（[componentactions.lua#L2865-L2872](../../../raw/current/scripts/componentactions.lua#L2865-L2872)、[L2451-L2458](../../../raw/current/scripts/componentactions.lua#L2451-L2458)）。
- 技能门禁：`CanActivate` 要求技能 `winona_wagstaff_2` 已激活，否则 "NOSKILL"（[winona_telebrella.lua#L274-L279](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L274-L279)）；该技能锁＝`winona_wagstaff_1` 已激活且未点 `winona_charlie_2`（[skilltree_winona.lua#L471-L490](../../../raw/current/scripts/prefabs/skilltree_winona.lua#L471-L490)）。
- 目的地：温娜传送 pad 须 `IsPowered()`（[winona_telebrella.lua#L281-L283](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L281-L283)）且路径允许。选台按传送者与 pad 的距离：先取超过 `角色物理半径 + TELEPAD_DETECTION_RADIUS(2)` 的合格台中最近的，外侧没有才取阈值内最远的——变量名 outofcamera／incamera 只是命名，不是镜头判定（[remoteteleporter](../../../raw/current/scripts/components/remoteteleporter.lua#L90-L129)、[阈值](../../../raw/current/scripts/tuning.lua#L7240-L7242)）。传送者与半径 1.84 内可拾取地面物品一起位移；找到目的地也最多扣 1 天（min(1 天, 余量−0.00001)）（[winona_telebrella](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L289-L295)）。
- 状态：燃料 section 归零自动卸装备，再充回 section>0 自动重挂（[winona_telebrella.lua#L105-L125](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L105-L125)）；检查状态 CHARGING／CHARGED／OFF／MISSINGSKILL（仅温娜本人可见，[L307-L321](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L307-L321)）。

## 典型接入

### 调整原版伞的数值

- 前提与选择：modmain 里 `AddPrefabPostInit` 是不动原版文件的入口；只调数值无需处理客户端，组件配置在主模拟侧已同步；mod 环境由 `mods.lua` `CreateEnvironment` 白名单提供、无 `__index=_G`（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）。
- 生效与撤销：postinit 在 prefab 构造完成后执行，随实体存档；卸载 mod 后字段随实体重建消失。原版依据见[组件分档](#组件分档)。

```lua
-- modmain.lua：花伞防水提到猪皮伞档、不再全程腐坏
AddPrefabPostInit("grass_umbrella", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.waterproofer:SetEffectiveness(TUNING.WATERPROOFNESS_HUGE)
    if inst.components.perishable ~= nil then
        inst:RemoveComponent("perishable")
        inst:RemoveTag("show_spoilage")
    end
end)
```

- 调整与验证：`TUNING` 在 mod 环境白名单内可直用；`TheWorld` 等游戏全局必须 `GLOBAL.` 前缀。数值单位见[消费端](#防水与保温消费端)；示例仅语法自检，未在游戏内运行。

## 配方与获取

| 注册名 | 配方 | 科技／门禁 |
|---|---|---|
| `grass_umbrella` | 4 twigs＋3 cutgrass＋6 petals | TECH.NONE（[recipes.lua#L524](../../../raw/current/scripts/recipes.lua#L524)） |
| `umbrella` | 6 twigs＋1 pigskin＋2 silk | TECH.SCIENCE_ONE（[recipes.lua#L525](../../../raw/current/scripts/recipes.lua#L525)） |
| `winona_telebrella` | 4 sewing_tape＋4 transistor＋4 twigs＋1 winona_machineparts_2 | TECH.LOST＋builder_skill winona_wagstaff_2（[recipes.lua#L320](../../../raw/current/scripts/recipes.lua#L320)） |
| `voidcloth_umbrella` | 5 horrorfuel＋1 voidcloth | TECH.SHADOWFORGING_TWO，nounlock，影坊（[recipes.lua#L868](../../../raw/current/scripts/recipes.lua#L868)） |

另有获取：查理舞台奖励池 weather 池含 `umbrella`、tailor 池含 `grass_umbrella`（[charlie_stage.lua#L160-L161](../../../raw/current/scripts/prefabs/charlie_stage.lua#L160-L161)）。

### 皮肤

- `umbrella`：umbrella_circus 马戏团伞顶、umbrella_heart 情人雨伞、umbrella_parasol 不祥蕾丝阳伞、umbrella_spider 蜘蛛伞（[skinprefabs.lua#L13362-L13400](../../../raw/current/scripts/prefabs/skinprefabs.lua#L13362-L13400)）；`grass_umbrella`：parasol_polkadot、parasol_spring（[skinprefabs.lua#L8985-L9003](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8985-L9003)）。
- `voidcloth_umbrella`／`winona_telebrella`：按 base_prefab 反查无皮肤；装备回调仍预留 `GetSkinBuild` 分支（[voidcloth_umbrella.lua#L20-L26](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L20-L26)），未来加皮肤无需改工厂。

## 常见陷阱与限制

- **误区：伞能防雷。** 四把伞都没设 `equippable.insulated`（默认 false，[equippable.lua#L39](../../../raw/current/scripts/components/equippable.lua#L39)），闪电照常导电；防雷语义（rainhat／eyebrellahat 的 insulated 链）见 [[reference/prefabs/hats-environment|环境与功能帽族]]。传送伞 `metal` 标签只影响雨声。
- **边界：两把基础伞的"耐久"语义不同。** 雨伞 fueled 只在**装备期**消耗（收起不掉），花伞 perishable **全程**腐坏（背包里也烂）；"放着就不用修"只对雨伞成立。
- **边界：MAGIC 燃料不吃物品。** 暗影伞／传送伞无法用 nightmarefuel 等充能（nightmarefuel 的 fuel.fueltype 是 NIGHTMARE，[nightmarefuel.lua#L82](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L82)）；暗影伞只能 voidcloth_kit 修，传送伞只能电池充。
- **边界：酸雨中装备暗影伞燃料回升。** 酸雨修饰值是 −1 而非 0：`DoDelta(-dt*rate*mod)`（[fueled.lua#L327](../../../raw/current/scripts/components/fueled.lua#L327)）在 mod=−1 时变号为正，装备中燃料以正常速率**回充**（封顶满）；只想免疫消耗应设 0。源码推导行为，未运行验证。
- **边界：暗影伞开罩是持续开销。** 罩内 fueled ×1.5（15→10 天）＋−200／天理智＋不可拾起；雨罩只把罩内点排除在天候（雨湿／月雹／酸雨落点）之外，不挡已落地的伤害实体。
- **边界：传送伞两层门禁。** 非 Winona 看不到动作（`engineering` 标签过滤），温娜不点 `winona_wagstaff_2` 在服务端被 "NOSKILL" 拒绝；配方也绑同技能。
- **边界：传送目的地必须通电，失败也扣费。** pad 需 `IsPowered()`；无合格目标时 `Teleport` 仍以 `success=false` 调 `onteleportedfn`，传送伞回调不查 `success`，照常扣最多 1 天（余额不足留约 0.00001）。动作返回 `NODEST` 后仍走 `remote_teleport_in` 退出，`OnStopTeleport` 清尾数；只有 `CanActivate` 在 `Teleport` 前拒绝才不扣费。[动作](../../../raw/current/scripts/actions.lua#L6032-L6043)、[无目标](../../../raw/current/scripts/components/remoteteleporter.lua#L123-L137)、[扣费](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L285-L303)、[退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24955-L24970)
- **边界：官方名与注册名不能互推。** 花伞＝grass_umbrella（Pretty Parasol）、传送伞＝winona_telebrella（Portasol）、暗影伞英文官方名拼作 "Umbralla"；名称键核验见[验证记录](#验证记录)。

## 验证记录

- 源码审阅：2026-11-07（第二轮修订），steam-buildid:26335394。复核 remoteteleporter 的阈值选台、附近物品过滤与失败回调，以及 SGwilson 的 actionfailed／onexit 扣费收尾。早前核验三个本体文件全文、五个核心组件（raindome／remoteteleporter／forgerepair／waterproofer／insulator）及正文引用的消费端／数据文件（锚点见正文）。未覆盖：pad／电池内部、moisture 完整公式、acidlevel 数值、资源内容与运行表现。
- 名称核验：`languages/chinese_s.po` 按 `STRINGS.NAMES.<KEY>` 检索——UMBRELLA 雨伞 [L351141](../../../raw/current/scripts/languages/chinese_s.po#L351141)、GRASS_UMBRELLA 花伞（Pretty Parasol）[L343406](../../../raw/current/scripts/languages/chinese_s.po#L343406)、VOIDCLOTH_UMBRELLA 暗影伞（Umbralla）[L351366](../../../raw/current/scripts/languages/chinese_s.po#L351366)、WINONA_TELEBRELLA 传送伞（Portasol）[L352106](../../../raw/current/scripts/languages/chinese_s.po#L352106)、VOIDCLOTH_KIT 虚空修补套件 [L351356](../../../raw/current/scripts/languages/chinese_s.po#L351356)；皮肤名 umbrella_circus／umbrella_heart／umbrella_parasol／umbrella_spider 同文件检索命中。
- 穷举复核：`grep base_prefab` 确认 umbrella 4 皮肤、grass_umbrella 2 皮肤、另两注册 0 皮肤；`grep OnDepleted` 确认 winona_telebrella.lua 的 OnDepleted（L90-L103）无调用点，耗尽回收实际走 fueled section 回调。
- 结构检查：validate_page.py 与 strict 均通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats-environment|环境与功能帽族]]（eyebrellahat 眼球伞带 `umbrella` 标签与雷电绝缘）
- [[reference/components/waterproofer|waterproofer 组件]]、[[reference/components/insulator|insulator 组件]]、[[reference/components/equippable|equippable 组件]]（防水／保温／装备槽）
- [[reference/components/fueled|fueled 组件]]、[[reference/components/perishable|perishable 组件]]（两种耐久语义）
- [[reference/components/raindome|raindome 组件]]、[[reference/components/machine|machine 组件]]（暗影伞雨罩机）
- [[reference/components/remoteteleporter|remoteteleporter 组件]]、[[reference/components/circuitnode|circuitnode 组件]]、[[reference/components/powerload|powerload 组件]]（传送伞传送与充能）
- 返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|原版 Prefab 索引]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 4 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
