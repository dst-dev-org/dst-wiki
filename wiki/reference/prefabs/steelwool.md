---
title: 钢丝绵（steelwool）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 单文件注册的可燃杂物：既能当燃料（fuelvalue = MED_FUEL＝45 秒），又有 burnable 与 propagator，作祟只把它弹起而不点燃。来源是钢羊（spat）的保底 2 块＋50% 第三块与克劳斯袋；消费端是战争鞍具、刷子、小钢羊宠物蛋和兔人神龛的两件钢丝绵枕头。本页对照"燃料值／燃烧时间／枕头材质名"三件容易混的东西，并写清各自的科技与制作站门禁。
aliases: ["steelwool", "钢丝绵", "Steel Wool", "钢丝绒", "钢丝棉"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/steelwool.lua", "prefabs/spat.lua", "components/klaussackloot.lua", "components/hunter.lua", "prefabs/pillow_defs.lua", "prefabs/yotr_rabbitshrine.lua", "prefabs/critters.lua", "components/fuel.lua", "components/fueled.lua", "components/hauntable.lua", "components/lootdropper.lua", "components/builder.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "constants.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 钢丝绵（steelwool）

## 快速判断

- 职责与适用：单文件单注册的 `inventoryitem` 杂物——钢丝绵（`steelwool`）。它同时挂 `fuel`（燃料值 45 秒）与 `burnable`／`propagator`（燃烧 15 秒、会向周围传热），主要身份仍是配方材料（战争鞍具、刷子、宠物套件、活动枕头）。可借鉴：一件物品同时是"燃料"和"可燃物"时的两套接线，以及"材质名"不等于"物品"的命名复用（`pillow_defs` 的 `steelwool` 只是四材质之一）。[注册](../../../raw/current/scripts/prefabs/steelwool.lua#L46)
- 优先使用：查燃料值与燃烧行为 → [数值与参数](#数值与参数)；查从哪来、掉几块 → [获得链](#获得链)；查配方与科技门禁 → [用途与配方](#用途与配方)；想把自定义物品也做成这种"燃料＋可燃"杂物 → [典型接入](#典型接入)。
- 重要副作用：烧完不是消失而是**整堆变成等量灰烬**（`DefaultBurntFn` 用 `stackable.stacksize` 设灰烬堆）；作祟只弹起，不点燃（`MakeHauntableLaunchAndIgnite` 的点火段被 `--#HAUNTFIX` 注释）；两件枕头配方要在兔人神龛前制作，不是普通制作栏条目。
- 修改边界：不覆盖钢羊（spat）与克劳斯袋的生成、AI 与刷新（两者均 Wiki 待建）；不覆盖枕头的伤害／击退与神龛小游戏本体（归 [[reference/prefabs/role-tools]]）；不覆盖战争鞍具与刷子的装备与耐久机制（刷子归 [[reference/prefabs/living-tools]]）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `steelwool` | 单文件单注册 `Prefab("steelwool", fn, assets)`。公共端：`CreateEntity` → Transform／AnimState／Network（**没有** SoundEmitter）→ `MakeInventoryPhysics` → bank／build `steel_wool`／`PlayAnimation("idle")` → `pickupsound = "cloth"` → `MakeInventoryFloatable(inst, "med", nil, 0.72)` → `SetPristine`，非主模拟返回。 | 主模拟 6 段：`inspectable`、`inventoryitem`、`stackable`（`STACK_SIZE_SMALLITEM`＝40）、`fuel`（`MED_FUEL`）、`MakeSmallBurnable(inst, MED_BURNTIME)`＋`MakeSmallPropagator(inst)`、`MakeHauntableLaunchAndIgnite`。[fn](../../../raw/current/scripts/prefabs/steelwool.lua#L6-L44)、[组件段](../../../raw/current/scripts/prefabs/steelwool.lua#L29-L41) |

`MakeSmallBurnable` 会顺手把 `burnable` 的点燃／熄灭／烧尽回调设成默认三件套（`DefaultBurnFn`／`DefaultExtinguishFn`／`DefaultBurntFn`），本文件没有覆盖其中任何一个。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)

### 名称与实显

名称走注册名直出：`entityscript` 用 `STRINGS.NAMES[string.upper(prefab)]` 填 `inst.name`。[取名路径](../../../raw/current/scripts/entityscript.lua#L843)

同版简中 `NAMES.STEELWOOL`＝**钢丝绵**（英文 `Steel Wool`）。[po](../../../raw/current/scripts/languages/chinese_s.po#L350044-L350047)

- 注：既有页 [[reference/prefabs/living-tools]] 的刷子配方行把它写作"钢丝绒"，与本版实显不一致；本页按 `.po` 用"钢丝绵"。俗称"钢丝棉／钢丝绒"只用于检索。

## 数值与参数

| 项 | 实际值 | 证据 |
|---|---|---|
| 堆叠上限 | 40（`stackable.maxsize = STACK_SIZE_SMALLITEM`，本文件显式写入） | [L32-L33](../../../raw/current/scripts/prefabs/steelwool.lua#L32-L33)、[tuning](../../../raw/current/scripts/tuning.lua#L82-L83) |
| 浮水 | `med` 档、缩放 0.72 | [L21](../../../raw/current/scripts/prefabs/steelwool.lua#L21) |
| 拾取音 | `cloth` | [L19](../../../raw/current/scripts/prefabs/steelwool.lua#L19) |
| 燃料值 | `MED_FUEL`＝`seg_time × 1.5`＝45（秒；`seg_time`＝30） | [L35-L36](../../../raw/current/scripts/prefabs/steelwool.lua#L35-L36)、[seg_time](../../../raw/current/scripts/tuning.lua#L19)、[MED_FUEL](../../../raw/current/scripts/tuning.lua#L1675) |
| 燃烧时间 | `MED_BURNTIME`＝`seg_time × 0.5`＝15（秒） | [L38](../../../raw/current/scripts/prefabs/steelwool.lua#L38)、[MED_BURNTIME](../../../raw/current/scripts/tuning.lua#L1682) |
| 传播参数 | `flashpoint` 5–10、`decayrate` 0.5、`propagaterange` 3–5、`heatoutput` 3–5、`damagerange` 2、`damages = true` | [helper](../../../raw/current/scripts/standardcomponents.lua#L200-L213) |
| 作祟 | 只抛起（`Launch`、`LAUNCH_SPEED_SMALL`），冷却 `HAUNT_COOLDOWN_SMALL` | [L41](../../../raw/current/scripts/prefabs/steelwool.lua#L41)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：fuel` | `fuelvalue = MED_FUEL`，其余取默认（`fueltype = FUELTYPE.BURNABLE`，自动挂 `BURNABLE_fuel` 标签） | 主模拟；只有 `fueltype` 匹配的 `fueled` 目标才收它 | [[reference/components/fuel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/steelwool.lua#L35-L36)、[默认值](../../../raw/current/scripts/components/fuel.lua#L10-L19) |
| `组件：fueled`（消费侧） | 投进篝火／火坑时按 `fuelvalue × bonusmult × wetmult × masterymult` 加燃料，然后 `item:Remove()` | 服务端；`wetmult` 取 `WET_FUEL_PENALTY`＝0.75，`fuelmaster` 另给加成 | [[reference/components/fueled#关键接口]]；[结算](../../../raw/current/scripts/components/fueled.lua#L197-L230)、[受潮折减](../../../raw/current/scripts/tuning.lua#L1671) |
| `组件：burnable` | `MakeSmallBurnable(inst, MED_BURNTIME)`：FX 等级 2、火焰特效 `fire`、烧尽走 `DefaultBurntFn` | 服务端；`DefaultBurntFn` 在非海面生成等量灰烬后移除本体 | [[reference/components/burnable#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44) |
| `组件：propagator` | `MakeSmallPropagator`：燃烧时向附近可燃物传热，并对 2 距离内同时带 `propagator` 与 `health` 的目标造成火焰伤害 | 服务端；`damages` 只作用于"有 health 且可传播"的目标，非结构 | Wiki 待建；[参数](../../../raw/current/scripts/standardcomponents.lua#L200-L213)、[伤害分支](../../../raw/current/scripts/components/propagator.lua#L186-L235) |
| `组件：hauntable` | `MakeHauntableLaunchAndIgnite`：本版只剩抛起；点火段整段注释，且成功后落到函数末尾的 `return false` | 主模拟；`DoHaunt` 对 false 走"失败分支"，仍设冷却并起 shader 效果，只是不推 `haunted` 事件 | [[reference/components/hauntable#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[失败分支](../../../raw/current/scripts/components/hauntable.lua#L106-L112) |
| `API／数据：LootDropper 机会表` | spat 用 `SetChanceLootTable('spat')`；`1.0` 条目必进、`< 1.0` 条目逐行独立掷骰，本页不涉及兜底条目 | 服务端 `GenerateLoot`；每行按击杀者幸运修正 | [[reference/components/lootdropper#关键接口]]；[掷骰](../../../raw/current/scripts/components/lootdropper.lua#L213-L227) |
| `关联 Prefab：spat`（钢羊） | 主要来源：共享表里 2 条 1.0＋1 条 0.5 的钢丝绵；钢羊是怪物足迹追猎的备选野兽 | 服务端；钢羊本体 Wiki 待建 | Wiki 待建；[表](../../../raw/current/scripts/prefabs/spat.lua#L35-L46)、[挂表](../../../raw/current/scripts/prefabs/spat.lua#L250-L251)、[追猎选择](../../../raw/current/scripts/components/hunter.lua#L368-L391) |
| `关联 Prefab：klaus_sack` | 克劳斯袋第四组掉落（`giant_loot3`）含钢丝绵，每袋从 9 项里取 2 个不同条目 | 服务端；开袋时按 `RollKlausLoot` 预先算好 | Wiki 待建；[表](../../../raw/current/scripts/components/klaussackloot.lua#L23-L34)、[抽取](../../../raw/current/scripts/components/klaussackloot.lua#L115-L124) |
| `API／数据：pillow_defs.steelwool` | 枕头材质数据（`strengthmult 2.2`／`laglength 0.20`／`defense_amount 0.7`／prize 1.0）；`MakeHandPillow`／`MakeBodyPillow` 按材质名拼 prefab | 模块加载期注册；具体数值与装备行为归 role-tools 页 | [[reference/prefabs/role-tools]]；[材质项](../../../raw/current/scripts/prefabs/pillow_defs.lua#L33-L42) |
| `缺失组件（否定断言）` | 无 `tradable`／`edible`／`cookable`／`bait`／`perishable`／`repairer` | 由 `prefabs/steelwool.lua` 全文（46 行）确认 | [全文](../../../raw/current/scripts/prefabs/steelwool.lua#L6-L44) |

## 获得链

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 钢羊（`spat`，怪物足迹追猎的备选野兽） | 2–3 块（两行 1.00 保底，第三行 0.50） | 共享表另有 4 条 1.00 肉与脓鼻涕 1.00＋0.50；每行独立掷骰，`< 1.0` 行按击杀者幸运修正；**钢羊普通燃烧中死亡时这 2–3 块钢丝绵被换成 ash**（受控燃烧则保留原名、落地尝试闷烧；同表 `meat` 走 `cooked`..v 分支变 `cookedmeat`），所以只有普通火要先灭 | [表](../../../raw/current/scripts/prefabs/spat.lua#L35-L46)、[挂表](../../../raw/current/scripts/prefabs/spat.lua#L250-L251)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L349739-L349742)、[可燃](../../../raw/current/scripts/prefabs/spat.lua#L264) |
| 克劳斯袋（`klaus_sack`） | 0–1 块 | 第四组掉落从 `giant_loot3`（9 项）取 2 个互不相同的条目，钢丝绵是其中之一 | [表](../../../raw/current/scripts/components/klaussackloot.lua#L23-L34)、[抽取](../../../raw/current/scripts/components/klaussackloot.lua#L115-L124) |

数量说明：`spat` 的两条 1.00 不参与概率判定、必进包，第三条 0.50 独立掷骰，所以每只 spat 的钢丝绵是 2 块或 3 块，不是"平均 2.5 块保底"。克劳斯袋的两次取值必然不同（第二次从 `math.random(1, #giant_loot3 - 1)` 取号，与第一次相同时改用末项），因此一袋最多出现 1 块钢丝绵，不会是 2 块。这 2–3 块也只在普通燃烧外成立：钢羊带 `burnable`，死亡时仍在普通燃烧，`DropLoot` 会把没有熟制变体的条目换成 `ash`（肉变 `cookedmeat`）；受控燃烧走保留分支，钢丝绵落地尝试闷烧。所以普通火要先灭，受控火可保留。

## 用途与配方

| 消费端 | 消耗 | 科技／门禁 | 证据 |
|---|---|---|---|
| 战争鞍具 `saddle_war` | 4（另需兔子 4、原木 10） | `TECH.SCIENCE_TWO`（炼金引擎） | [L683](../../../raw/current/scripts/recipes.lua#L683)、[SADDLE_WAR](../../../raw/current/scripts/languages/chinese_s.po#L348899-L348902) |
| 刷子 `brush` | 1（另需海象牙 1、金块 2） | `TECH.SCIENCE_TWO` | [L685](../../../raw/current/scripts/recipes.lua#L685)、[BRUSH](../../../raw/current/scripts/languages/chinese_s.po#L339964-L339967) |
| 小钢羊 `critter_lamb_builder`（宠物蛋） | 1（另需鳄梨酱 1） | `TECH.ORPHANAGE_ONE`＋`nounlock`＋`actionstr="ORPHANAGE"`；已有宠物时 `DoBuild` 直接拒绝 | [L822](../../../raw/current/scripts/recipes.lua#L822)、[宠物门禁](../../../raw/current/scripts/components/builder.lua#L664-L669)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L341889-L341892) |
| 钢丝绵枕头 `handpillow_steelwool` | 2（另需幸运金块 5、丝绢 2） | `TECH.RABBITOFFERING_THREE`＋`nounlock`＋`actionstr="PERDOFFERING"`：需兔人神龛 | [L1138](../../../raw/current/scripts/recipes.lua#L1138) |
| 钢丝绵枕头盔甲 `bodypillow_steelwool` | 2（另需幸运金块 6、丝绢 2） | 同上 | [L1143](../../../raw/current/scripts/recipes.lua#L1143) |

**三条门禁的分工**：

1. `TECH.SCIENCE_TWO` 就是炼金引擎的科技树（`SCIENCE = 2`），战争鞍具与刷子都是"造出来就一直用"的常规装备，没有额外解锁。[ALCHEMYMACHINE](../../../raw/current/scripts/tuning.lua#L1260-L1270)、[SCIENCE_TWO](../../../raw/current/scripts/constants.lua#L1245-L1246)
2. 小钢羊（宠物蛋）属"孤儿院"页：`ORPHANAGE` 只由岩石巢穴（`CRITTERLAB`）给到 1 级，`nounlock` 表示不需要蓝图解锁、但要在巢穴前制作；`builder` 还会在有宠物／宠物栏已满时拒绝。[巢穴树](../../../raw/current/scripts/tuning.lua#L1316-L1318)、[ORPHANAGE_ONE](../../../raw/current/scripts/constants.lua#L1271)、[配方](../../../raw/current/scripts/recipes.lua#L819-L823)
3. 两件枕头要 `RABBITOFFERING = 3`，本版只有兔人神龛的 `RABBITSHRINE` 树给得到这一级，所以实际只能在神龛前制作；`nounlock` 只表示不写解锁记录、也不进蓝图。配方里的 `guacamole` 与 `critters.lua` 给羊的 `favoritefood` 是同一个东西。[RABBITSHRINE](../../../raw/current/scripts/tuning.lua#L1349-L1352)、[RABBITOFFERING_THREE](../../../raw/current/scripts/constants.lua#L1279)、[羊的偏爱食物](../../../raw/current/scripts/prefabs/critters.lua#L370-L371)

**与神龛的关系**：`yotr_rabbitshrine`（简中实显「兔人神龛」）只是把四材质的枕头 prefab 列进自己的 `prefabs` 预载，并按材质名随机配"身体枕＋手持枕"两件套当作小游戏道具；它不消耗钢丝绵物品，也不生成钢丝绵。[预载](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L34-L37)、[套装搭配](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L79-L94)

## 典型接入

### 把自定义物品做成"燃料＋可燃"杂物

- 前提与选择：服务端 `modmain.lua`；用 `standardcomponents` 的两个 helper 而不是手写回调，能直接继承"烧尽变等量灰烬"的默认行为。两个 helper 分别挂 `burnable` 与 `propagator`：`AddComponent` 命中已有组件会取回旧表，helper 随即覆盖闪燃点、传播范围、热输出与伤害开关，所以两者要分开守卫，下面按组件是否存在分别补齐。
- 生效与撤销：追加发生在 prefab 构造之后；`fuel` 的 `fueltype` 默认 `BURNABLE`，只有同类 `fueled` 目标会收；移除模组即恢复。物品需已有 `AnimState`（火焰特效按 `fxdata` 追随实体，可选跟随符号）。

```lua
-- modmain.lua：让自定义杂物也能当燃料烧、并被火引燃
AddPrefabPostInit("my_scrap", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.fuel == nil then
        inst:AddComponent("fuel")
        inst.components.fuel.fuelvalue = GLOBAL.TUNING.MED_FUEL
    end
    if inst.components.burnable == nil then
        GLOBAL.MakeSmallBurnable(inst, GLOBAL.TUNING.MED_BURNTIME)
    end
    if inst.components.propagator == nil then
        GLOBAL.MakeSmallPropagator(inst)
    end
end)
```

- 调整与验证：`fuelvalue` 的单位是 `fueled` 的秒数（`MED_FUEL`＝45），`MED_BURNTIME` 是烧尽所需秒数（15）；想"烧完不掉灰"要自己覆盖 `burnable:SetOnBurntFn`。已有 `propagator` 或 `burnable` 的目标靠上面的分守卫避免覆盖；目标本来只可燃、不传播时不要补 `propagator`，否则会改变它的传热行为；自定义物品堆叠超过 40 时，烧尽只留 40 个灰烬（`min(灰烬 maxsize, stacksize)`），海上不产灰。本例为源码推导，未运行游戏。

## 常见陷阱与限制

- **误区："可燃"等于"会消失"。** 钢丝绵烧尽时 `DefaultBurntFn` 在非海面生成灰烬，堆叠数取 `min(灰烬上限 40, 本体 stacksize)`，海上则完全不产灰；钢丝绵两值都是 40，所以整堆 40 块正好 40 个灰烬，不是 1 个。[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[灰烬上限](../../../raw/current/scripts/prefabs/ash.lua#L69-L70)
- **误区：`MakeHauntableLaunchAndIgnite` 会点燃。** 本版点火段被 `--#HAUNTFIX` 整段注释，只剩 `Launch` 与 `is_landed` 复位；而且 `return true` 在被注释的块里，函数成功抛起后仍返回 `false`，`DoHaunt` 走"失败分支"（照样设冷却、起特效，但不推 `haunted` 事件）。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[分支](../../../raw/current/scripts/components/hauntable.lua#L88-L114)
- **边界：燃料值不是燃烧时间。** `fuelvalue` 决定"投进火里加多少秒油"，`MED_BURNTIME` 决定"被点燃后自身烧多久"，两者取自不同 tuning、互不影响；受潮时燃料值还要乘 `WET_FUEL_PENALTY`＝0.75。[结算](../../../raw/current/scripts/components/fueled.lua#L197-L230)、[受潮](../../../raw/current/scripts/tuning.lua#L1671)
- **边界：`propagator` 的伤害面很窄。** `damages` 只在目标同时带 `propagator` 与 `health` 时结算火焰伤害，所以燃烧的钢丝绵能烧到其它可燃物／可燃生物，却不会烧到没有 `propagator` 的石头或建筑。[伤害条件](../../../raw/current/scripts/components/propagator.lua#L219-L233)
- **误区：钢丝绵是"羊毛"，能用剪刀剃。** 它没有 `shearable`／`beard`／`brushable` 任何一条接线，也不是皮弗娄牛的产出；牛皮弗娄牛剃毛掉的是 `beefalowool`，两者只在枕头配方里作为不同材质并列。牛与刷子分见 [[reference/prefabs/beefalo]]、[[reference/prefabs/living-tools]]。[全文](../../../raw/current/scripts/prefabs/steelwool.lua#L6-L44)
- **边界：两件枕头只能在兔人神龛前做。** 它们要 `TECH.RABBITOFFERING_THREE`，这一级只由 `RABBITSHRINE` 科技树给到，站在别处做不出来；`nounlock` 只是不写解锁记录、也不会被蓝图抽到，不要按"先学配方"的思路找前置。[神龛树](../../../raw/current/scripts/tuning.lua#L1349-L1352)、[蓝图排除](../../../raw/current/scripts/prefabs/blueprint.lua#L44-L48)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：`prefabs/steelwool.lua` 全文与名称键；`spat` 的共享掉落表、`lootdropper` 挂表与追猎来源（`hunter.ALTERNATE_BEASTS`）；`klaussackloot` 的 `giant_loot3` 与两次不同取值；`fuel`／`fueled` 的燃料结算与受潮折减；`MakeSmallBurnable`／`MakeSmallPropagator`／`DefaultBurntFn`（含海面与灰烬上限）；`AddComponent` 的重复挂载与两种守卫组合；`MakeHauntableLaunchAndIgnite` 与 `Hauntable:DoHaunt` 的分支；钢羊的可燃性与 `DropLoot` 的烧毁改写（`spat.lua` 的可燃接线、`lootdropper.lua#L415-L445`）；`recipes.lua` 的 5 条消费行与 `builder`／`craftingstation` 的 `nounlock` 路径；`pillow_defs`、`yotr_rabbitshrine`、`critters` 的材质与宠物关系。未覆盖：spat 与克劳斯袋的生成／AI 本体；枕头的实际伤害、击退与神龛小游戏流程；刷子／战争鞍具的装备与耐久；运行表现。
- 游戏运行：未运行；数量、燃料值与燃烧时间均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 见交付回报；新增的 `modmain` 片段做了 Lua 语法检查（见交付回报），未运行游戏。

相关：[[reference/prefabs/role-tools]]（钢丝绵枕头与枕头盔甲）、[[reference/prefabs/living-tools]]（刷子）、[[reference/prefabs/beefalo]]（皮弗娄牛与刷毛）、[[reference/prefabs/attackfx_handpillow_petals]]（枕头攻击碎屑特效）。

返回 [[reference/prefabs/index|原版 Prefab 索引]]
