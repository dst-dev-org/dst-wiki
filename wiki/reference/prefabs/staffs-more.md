---
title: "法杖补充族（staff_lunarplant／staff_tornado／king_cane／yoth_knightstick／moonstorm_static 等 12 注册）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "原版法杖补充族四文件加月暴静电系统：亮茄魔杖（裂隙远程位面武器与破损修补循环）、天气风向标（龙卷风召唤实体）、国王魔杖（远古大门 Charlie 遗物、嵌宝石充能法杖）、滑稽马（骑士之年疾驰杖）与静电球族（捕捉-实验-产物链）；覆盖注册、官方名、配方、装备归属、机制与消费端。"
aliases: ["staff_lunarplant", "亮茄魔杖", "亮茄法杖", "Brightshade Staff", "staff_tornado", "天气风向标", "Weather Pain", "龙卷风魔杖", "tornado", "龙卷风", "king_cane", "国王魔杖", "King's Staff", "王杖", "yoth_knightstick", "滑稽马", "Shtick Horse", "骑士杖", "moonstorm_static", "moonstorm_static_nowag", "能量静电", "Energetic Static", "moonstorm_static_item", "约束静电", "Restrained Static", "moonstorm_static_catcher", "静电约束仪", "Static Restrainer", "moonstorm_static_roamer", "未约束的静电", "Unrestrained Static"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/staff_lunarplant.lua", "prefabs/staff_tornado.lua", "prefabs/king_cane.lua", "prefabs/yoth_knightstick.lua", "prefabs/moonstorm_static.lua", "prefabs/staff_common.lua", "prefabs/brilliance_projectile_fx.lua", "prefabs/atrium_gate.lua", "prefabs/moonbase.lua", "prefabs/fans.lua", "prefabs/forge_repair_kits.lua", "prefabs/spear_wathgrithr.lua", "prefabs/hats.lua", "prefabs/player_common_extensions.lua", "brains/wagstaff_npcbrain.lua", "brains/tornadobrain.lua", "brains/moonstormstaticbrain.lua", "stategraphs/SGtornado.lua", "stategraphs/SGmoonstormstatic.lua", "stategraphs/SGwilson.lua", "components/moonstormmanager.lua", "components/moonstormstaticcatcher.lua", "components/moonstormstaticcapturable.lua", "components/corruption.lua", "components/rechargeable.lua", "components/setbonus.lua", "components/forgerepairable.lua", "components/spellcaster.lua", "components/equippable.lua", "standardcomponents.lua", "componentactions.lua", "actions.lua", "recipes.lua", "recipes_filter.lua", "tuning.lua", "constants.lua", "prefabskins.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 法杖补充族（staff_lunarplant／staff_tornado／king_cane／yoth_knightstick／moonstorm_static 等 12 注册）

## 快速判断

- 职责与适用：五文件共 12 个注册，分四组：①[亮茄魔杖](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L159-L233)（裂隙月球锻造远程武器，全伤害走 planardamage，配独立弹体 [[reference/prefabs/brilliance_projectile_fx]]）；②[天气风向标](../../../raw/current/scripts/prefabs/staff_tornado.lua#L55-L116)与 [tornado](../../../raw/current/scripts/prefabs/staff_tornado.lua#L132-L173) 龙卷风实体、[滑稽马](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L44-L111)（骑士之年活动装备）；③[国王魔杖](../../../raw/current/scripts/prefabs/king_cane.lua#L257-L370)（远古大门 Charlie 战败仪式掉落，嵌宝石换效果，共用 [[reference/prefabs/staffs|基础法杖族]] 的 staff_common 工厂）；④[月暴静电族](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L546-L550) 五注册（世界静电球三种＋库存产物＋手持捕捉器）。选 `complex` 是因为施法消耗模型（finiteuses／rechargeable／corruption 三套并存）、静电实验协作链与"哪条注册是装备主体"的归属判断必须跨文件整体对照。
- 优先使用：查注册、配方与官方名看[注册与构造](#注册与构造)；改亮茄伤害与弹跳看[亮茄行](#注册与构造)与 [[reference/prefabs/brilliance_projectile_fx]]；改换宝石与分色效果看 [[reference/prefabs/staffs#复用与换装kingcane]]，改腐化看 [[reference/components/corruption]]；改龙卷风伤害与工作看[天气回路行](#注册与构造)；造"静电球式"升级材料看[典型接入](#典型接入)。
- 重要副作用：亮茄命中唤醒沉睡者并推 `attacked`（damage 0）[OnAttack](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L68-L85)；tornado 每 5 帧对范围内目标 7 点风伤并做 2 点工作 [destroystuff](../../../raw/current/scripts/stategraphs/SGtornado.lua#L60-L98)；king_cane 每次施法喂腐化、满腐化转 `shadow_item`＋shadowdominance [OnCorruptedFn](../../../raw/current/scripts/prefabs/king_cane.lua#L189-L199)；滑稽马仅移动时耗燃料、疾驰滑倒额外扣损耗 [locomote 回调](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L93-L101)；静电球离开月暴风暴区开始衰变、风暴结束被 Kill [stormstopped](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L49-L56)。
- 修改边界：全部效果逻辑在主模拟（`TheWorld.ismastersim` 分支后），客户端只有 reticule／net 变量监听；tornado 的 SG／Brain 只记到伤害与工作入口，不展开行为树内部；月暴实验流程与风暴本体归 [[reference/components/moonstormmanager]]，本页只记实体侧；king_cane 分色工厂细节归 staffs 页、腐化账本归 corruption 页，本页只记本体接线与差异。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `staff_lunarplant` | `fn`：pristine 段（bank/build `staff_lunarplant`、`rangedweapon`/`magicweapon`/`show_broken_ui`/`weapon` 标签、`isbroken` net_bool）→ 主模拟 finiteuses／planardamage／damagetypebonus／inspectable／inventoryitem／SetupComponents（equippable＋weapon 0 伤 8-10 射程＋弹体）／setbonus／MakeForgeRepairable | 物品＋HANDS 装备＋远程武器；破损态动态移除并恢复 equippable/weapon；[工厂](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L159-L233)、[SetupComponents](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L87-L97) |
| `staff_lunarplant_fx` | `fxfn`：FX 实体，装备时跟随 `swap_object`、漂浮时跟随本休 `swap_spear` | 纯 FX（`persists=false`）；生成与父挂细节见 [[reference/prefabs/staff_lunarplant_fx]]；[fxfn](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L235-L269)、[SetFxOwner](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L12-L30) |
| `staff_tornado` | `staff_fn`：pristine 段（`nopunch`/`quickcast` 标签、controller 攻击距离取 ACTIONS.CASTSPELL.distance）→ 主模拟 finiteuses 15（耗尽即 Remove）／inspectable／inventoryitem／equippable／spellcaster | 物品＋HANDS 装备；施法在出手点与目标点 15% 处生成 tornado；[工厂](../../../raw/current/scripts/prefabs/staff_tornado.lua#L55-L116)、[spawntornado](../../../raw/current/scripts/prefabs/staff_tornado.lua#L19-L32) |
| `tornado` | `tornado_fn`：AnimState FinalOffset 2＋`spinLoop` 循环音＋InventoryPhysics 后 RemovePhysicsColliders → 主模拟 knownlocations／locomotor（走 25×.33、跑 25）→ `SGtornado`＋tornadobrain → `SetDuration(TORNADO_LIFETIME)` | 世界实体（风系召唤物）；`persists=false` 不进存档；`WINDSTAFF_CASTER` 记录施放者；[工厂](../../../raw/current/scripts/prefabs/staff_tornado.lua#L132-L173)、[Brain 装载](../../../raw/current/scripts/prefabs/staff_tornado.lua#L118) |
| `king_cane` | `fn`：pristine 段（`weapon`/`rechargeable`/`shadowlevel`/`trader`/`give_dolongaction`/`gemsocket` 标签）→ 主模拟 inspectable（无宝石 getstatus EMPTY）／inventoryitem／equippable（walkspeedmult 1.3、is_magic_dapperness＋按腐化分档 dapperness）／weapon（KING_CANE_DAMAGE）／rechargeable／shadowlevel 4／trader 嵌宝石／tradable／stalkerinspectable／corruption／itemmimicconfig | 物品＋HANDS 装备＋武器；本体无 finiteuses，宝石形态经 stafffns 装载（详见 staffs 页）；[工厂](../../../raw/current/scripts/prefabs/king_cane.lua#L257-L370)、[嵌宝石](../../../raw/current/scripts/prefabs/king_cane.lua#L148-L171) |
| `king_cane_fx` | `fxfn`：网络化跟随 FX，携带腐蚀层 corruptfx 与宝石层 gemfx（客户端本地生成） | 见 [[reference/prefabs/king_cane_fx]]；[fxfn](../../../raw/current/scripts/prefabs/king_cane.lua#L539-L589) |
| `yoth_knightstick` | `fn`：pristine 段（`weapon`/`gallopstick` 标签、漂浮换装）→ 主模拟 inspectable／inventoryitem／equippable／fueled（USAGE 燃料 6 天、no_sewing、耗尽即 Remove）／weapon（CANE_DAMAGE）／fuel（LARGE_FUEL）＋可燃可蔓延 | 物品＋HANDS 装备＋武器；燃料只在持有者移动时消耗；[工厂](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L44-L111) |
| `moonstorm_static` | `fn`：障碍物理＋Light＋DynamicShadow＋`moonstorm_static`/`soulless` 标签 → 主模拟 health 100（nofadeout）／combat＋受击／死亡表现＋`ms_stormchanged` 自杀监听／inspectable | 世界静电球（Wagstaff NPC 实验载体）；`persists=false`；[工厂](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L58-L118) |
| `moonstorm_static_nowag` | `nowag_fn`：同上，另覆写 `static_ball_empty` 空球符号、trader 收 `wagstaff_tool_1..5`、`need_tool`/`need_tool_over` 事件、`PlayInitAnimation` 四面向两面的登场动画 | 玩家捕捉 roamer 后的实验球（检视图鉴名同 MOONSTORM_STATIC）；`persists=false`；[工厂](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L170-L245) |
| `moonstorm_static_item` | `itemfn`：`finish_idle` 动画＋`moonstorm_static` 标签＋漂浮 → 主模拟 tradable／inspectable／inventoryitem／upgrader（SPEAR_LIGHTNING）＋醒睡循环音 | 库存产物（**不可装备**）；`playermade` 变体只改库存图标；[工厂](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L281-L325)、[MakePlayerMade](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L264-L269) |
| `moonstorm_static_catcher` | `fn_catcher`：`empty_idle` 空球动画＋`moonstormstatic_catcher` 标签 → 主模拟 stackable 10／inspectable／inventoryitem／equippable／moonstormstaticcatcher（捕捉成功后自删） | 物品＋HANDS 装备（**本文件唯一的玩家装备主体**）；[工厂](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L355-L398) |
| `moonstorm_static_roamer` | `fn_roamer`：角色物理（去碰撞体）＋Light＋pristine `moonstormstaticcapturable` 标签 → 客户端 hudindicatable → 主模拟 inspectable／locomotor（walkspeed 8、allowocean）／moonstormstaticcapturable／`SGmoonstormstatic`＋moonstormstaticbrain／锯齿游走任务 | 世界实体（风暴内游走静电，可捕捉）；[工厂](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L481-L544) |

### 名称核验

官方名（简中 `languages/chinese_s.po`，键为 `STRINGS.NAMES.*`）：`STAFF_LUNARPLANT`＝亮茄魔杖（Brightshade Staff）[L349964-L349966](../../../raw/current/scripts/languages/chinese_s.po#L349964-L349966)、`STAFF_TORNADO`＝天气风向标（Weather Pain）[L349969-L349971](../../../raw/current/scripts/languages/chinese_s.po#L349969-L349971)、`TORNADO`＝龙卷风 [L350484-L350486](../../../raw/current/scripts/languages/chinese_s.po#L350484-L350486)、`KING_CANE`＝国王魔杖（King's Staff）[L344209-L344211](../../../raw/current/scripts/languages/chinese_s.po#L344209-L344211)、`YOTH_KNIGHTSTICK`＝滑稽马（Shtick Horse）[L353019-L353021](../../../raw/current/scripts/languages/chinese_s.po#L353019-L353021)、`MOONSTORM_STATIC`／`_NOWAG`＝能量静电 [L345719-L345721](../../../raw/current/scripts/languages/chinese_s.po#L345719-L345721)、[L345734-L345736](../../../raw/current/scripts/languages/chinese_s.po#L345734-L345736)、`MOONSTORM_STATIC_ITEM`＝约束静电 [L345729-L345731](../../../raw/current/scripts/languages/chinese_s.po#L345729-L345731)、`MOONSTORM_STATIC_CATCHER`＝静电约束仪 [L345724-L345726](../../../raw/current/scripts/languages/chinese_s.po#L345724-L345726)、`MOONSTORM_STATIC_ROAMER`＝未约束的静电 [L345739-L345741](../../../raw/current/scripts/languages/chinese_s.po#L345739-L345741)。任务措辞"亮茄法杖／龙卷风魔杖／王杖／骑士杖"为俗称，一并收入 aliases。破损态检视借用 `BROKEN_FORGEDITEM` 键（staff_lunarplant.nameoverride，[L144](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L144)）。

### 配方与获取

| 注册名 | 获取 | 证据 |
|---|---|---|
| `staff_lunarplant` | 配方 purebrilliance 3＋lunarplant_husk 6，`TECH.LUNARFORGING_TWO`＋`lunar_forge` 站；修理材料 `lunarplant_kit`（同站 purebrilliance 1＋lunarplant_husk 1） | [recipes.lua L851](../../../raw/current/scripts/recipes.lua#L851)、[kit L855](../../../raw/current/scripts/recipes.lua#L855)、[kit 绑定](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56) |
| `staff_tornado` | 配方 goose_feather 10＋lightninggoathorn 1＋gears 1，`TECH.SCIENCE_TWO` | [recipes.lua L490](../../../raw/current/scripts/recipes.lua#L490) |
| `king_cane` | 无配方；远古大门 `SpawnShroudenLootSetPiece` 仪式布景落地，与 [[reference/prefabs/queen-torch|女王火炬]]、`mask_charlie_bosshat` 同组（Charlie 遗物） | [atrium_gate.lua L711](../../../raw/current/scripts/prefabs/atrium_gate.lua#L711)、[L715](../../../raw/current/scripts/prefabs/atrium_gate.lua#L715) |
| `yoth_knightstick` | 配方 lucky_goldnugget 6，`TECH.KNIGHTOFFERING_THREE`，动作 `PERDOFFERING`（骑士之年神龛供奉） | [recipes.lua L1099](../../../raw/current/scripts/recipes.lua#L1099)、[分类](../../../raw/current/scripts/recipes_filter.lua#L685-L686) |
| `moonstorm_static_catcher` | 配方 moonglass 2＋wagpunk_bits 1，`TECH.WAGPUNK_WORKSTATION_TWO`（产品映射） | [recipes.lua L1020](../../../raw/current/scripts/recipes.lua#L1020) |
| 静电球三种与 `moonstorm_static_item` | 只能由实验流程生成：Wagstaff NPC 路径在其 Brain 实验阶段放 `moonstorm_static`；无 NPC 路径放 `moonstorm_static_roamer`；实验完成替换为 `moonstorm_static_item` | [wagstaff_npcbrain.lua L61](../../../raw/current/scripts/brains/wagstaff_npcbrain.lua#L61)、[manager L528-L535](../../../raw/current/scripts/components/moonstormmanager.lua#L528-L535)、[finished](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L41-L47) |

皮肤：`king_cane` 1 款（`king_cane_scorpion`）、`staff_tornado` 1 款（`tornado_stick_crow`），其余注册无 `PREFAB_SKINS` 键即无皮肤 [king_cane 清单](../../../raw/current/scripts/prefabskins.lua#L1110-L1113)、[staff_tornado 清单](../../../raw/current/scripts/prefabskins.lua#L1801-L1803)。皮肤显示名未逐条核验。

## 关联机制导航

### 亮茄魔杖（staff_lunarplant）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：weapon` | 原版 `SetDamage(0)`、射程 8-10、弹体 `brilliance_projectile_fx`、OnAttack 唤醒＋仇恨 | 主模拟；普通伤害与位面伤害在 combat 中分别结算，原版普通伤害配置为 0 | [[reference/components/weapon#关键接口]]；[配置](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L92-L96) |
| `组件：planardamage／damagetypebonus` | 位面伤害 10；对 `shadow_aligned` 目标 ×2（加成乘区） | 主模拟；`noplanarhitfx=true` 关闭位面命中特效 | [[reference/components/planardamage#关键接口]]、[[reference/components/damagetypebonus#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L213-L217)、[标志](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L230) |
| `组件：finiteuses＋forgerepairable` | 50 次；`MakeForgeRepairable` 把耗尽回调改为进入破损态（掉落出装备槽）而非移除，用 `lunarplant_kit` 修理恢复 | 主模拟；修理经 forgerepair 材料名 `lunarplant` 匹配 | [[reference/components/finiteuses#关键接口]]；[修理工厂](../../../raw/current/scripts/standardcomponents.lua#L1851-L1888)、[OnBroken/OnRepaired](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L138-L157)、[Repair 匹配](../../../raw/current/scripts/components/forgerepairable.lua#L58-L73) |
| `组件：setbonus` | `SetSetName(EQUIPMENTSETNAMES.LUNARPLANT)`；头＋身两件同名 setbonus 齐备时整套启用 | 主模拟；弹跳加成实际不走本组件（见陷阱） | [[reference/components/setbonus#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L224-L225)、[集合名](../../../raw/current/scripts/constants.lua#L2205-L2211) |
| `关联 Prefab：brilliance_projectile_fx／brilliance_projectile_blast_fx` | weapon 弹射物：速度 15、射程 25、命中弹跳 5 次（戴 `lunarplanthat` 7 次）后自删，命中生成 blast 闪色 FX | 弹跳目标偏好与命中结算详见 [[reference/prefabs/brilliance_projectile_fx]] | [弹体依赖声明](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L6-L10)、[取弹跳数](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L23-L30) |
| `API／数据：TUNING` | USES 50、PLANAR_DAMAGE 10、VS_SHADOW_BONUS 2、BOUNCES 5／SETBONUS 7 | `wilson_attack`=34×1 推导：weapon 0＋位面 10，对暗影目标等效 20 | [L184](../../../raw/current/scripts/tuning.lua#L184)、[L390-L393](../../../raw/current/scripts/tuning.lua#L390-L393)、[wilson_attack](../../../raw/current/scripts/tuning.lua#L37) |

### 天气风向标与龙卷风（staff_tornado／tornado）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：spellcaster` | canuseontargets＋canonlyuseonworkable＋canonlyuseoncombat＋quickcast，SpellFn=spawntornado；施法消耗 1 次 | 施法门禁与动作链见 [[reference/components/spellcaster#原版法杖的施法模式]] | [配置](../../../raw/current/scripts/prefabs/staff_tornado.lua#L105-L111)、[消耗](../../../raw/current/scripts/prefabs/staff_tornado.lua#L31) |
| `组件：finiteuses` | 15 次，`SetOnFinished(inst.Remove)` 耗尽直接删除（无破损态） | 主模拟 | [配置](../../../raw/current/scripts/prefabs/staff_tornado.lua#L92-L95) |
| `SG：SGtornado` | `destroystuff` 每 5 帧扫半径 3：对战斗目标 7 点风伤（`wind`）；玩家施放且 PvP 关闭时先排除 `player`/`possessedbody`，通过后仅对 `player` 标签乘 `PVP_DAMAGE_MOD`；工作分支是伤害分支的 `elseif`，要求 `CanBeWorked`＋支持的工作动作并做 2 点工作；排除 `INLIMBO`／`tornado_immune` 与施放者本人 | idle/walk 等状态时间线调用 [L117](../../../raw/current/scripts/stategraphs/SGtornado.lua#L117)、[L178-L183](../../../raw/current/scripts/stategraphs/SGtornado.lua#L178-L183)；`tornado_immune` 原版无授予者 | [destroystuff](../../../raw/current/scripts/stategraphs/SGtornado.lua#L60-L98) |
| `Brain：tornadobrain` | Leash＋Wander 在 `knownlocations` 的 "target" 记录点周围游走 | 服务端；注册 [L118](../../../raw/current/scripts/prefabs/staff_tornado.lua#L118) | [brain](../../../raw/current/scripts/brains/tornadobrain.lua#L16-L24) |
| `组件：locomotor／knownlocations` | 走速 25×.33、跑速 25；目标点存 "target" 供 Brain 使用 | 服务端 | [挂载](../../../raw/current/scripts/prefabs/staff_tornado.lua#L157-L161) |
| `复用：perdfan` | `tornado` 注册被 perdfan 复用：`SetDuration(PERDFAN_TORNADO_LIFETIME)`（2 秒）＋同样记录 WINDSTAFF_CASTER，每次耗 2 次 | 羽毛扇同文件 `featherfan`/`perdfan` 两注册 | [fans.lua L58-L76](../../../raw/current/scripts/prefabs/fans.lua#L58-L75)、[L169-L170](../../../raw/current/scripts/prefabs/fans.lua#L169-L170)、[时长](../../../raw/current/scripts/tuning.lua#L3647) |
| `API／数据：TUNING` | TORNADOSTAFF_USES 15、TORNADO_LIFETIME 5、TORNADO_DAMAGE 7、TORNADO_WALK_SPEED 25 | 单位：秒／次／生命值 | [L3031-L3034](../../../raw/current/scripts/tuning.lua#L3031-L3034) |

### 国王魔杖（king_cane）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：trader（嵌宝石）` | `ItemTradeTest`：拒收"precious"系（蛋白石 `opalpreciousgem` → WRONGGEM），其余须命中 stafffns 键且带 `gem` 标签；OnGemGiven 掉出旧宝石再装新，播 `telebase_gemplace` | 主模拟；换宝石时 cleargemstaff 清旧效果、`stafffns` 按 `GetGemId` 键以 notinit=true 装载新效果（工厂细节归 staffs 页） | [测试](../../../raw/current/scripts/prefabs/king_cane.lua#L107-L116)、[换装](../../../raw/current/scripts/prefabs/king_cane.lua#L173-L187)、[工厂装载](../../../raw/current/scripts/prefabs/staff_common.lua#L887-L906) |
| `组件：rechargeable` | 未腐化态 `dodischarge=false`，共用施法函数两分支都不命中——**无次数/充能消耗**（精神由 staffsanity 照扣、腐化照常累积）；腐化态 `dodischarge=true` 改走 `Discharge(KING_CANE_*_COOLDOWN)`（紫 90s、橙 10s、黄 90s、绿 120s）；红蓝不施法、走弹体攻击，两个 0 只是占位；放电中加 `nomagiccast` 禁施法并摘 `rangedlighter`/`extinguisher`，充满回调恢复 | 主模拟；`nomagiccast` 由 CASTSPELL/BLINK/CORRUPTNIGHTMARE 动作检查 | [充放电回调](../../../raw/current/scripts/prefabs/king_cane.lua#L118-L132)、[分支](../../../raw/current/scripts/prefabs/staff_common.lua#L368-L372)、[动作门禁](../../../raw/current/scripts/actions.lua#L3445-L3450)、[冷却](../../../raw/current/scripts/tuning.lua#L9758-L9768) |
| `组件：corruption` | 监听 `onspellcast`/`onblink`/`weapononprojectilelaunched` 喂 `DoDelta(KING_CANE_CORRUPTION[gem])`（红蓝 3、橙 5、紫黄 15、绿 20）；三回调驱动 FX 分档、`shadow_item` 标签、shadowdominance、trader 开关；自然衰减 -1/12s，满 100 腐化后停更锁定 | 账本与 CorruptEnt 分发链归 [[reference/components/corruption]]；解除腐化唯一入口是月亮石 moonbase | [接线](../../../raw/current/scripts/prefabs/king_cane.lua#L189-L218)、[监听](../../../raw/current/scripts/prefabs/king_cane.lua#L355-L357)、[腐化增量](../../../raw/current/scripts/tuning.lua#L9747-L9755)、[moonbase 消费](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L466) |
| `组件：shadowlevel／shadowdominance` | 本体 shadowlevel 默认等级 4（供暗影相关系统）；腐化态动态加 shadowdominance 组件，解除时移除 | 主模拟 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/king_cane.lua#L339-L340)、[动态](../../../raw/current/scripts/prefabs/king_cane.lua#L196-L205) |
| `组件：equippable` | walkspeedmult 1.3；`is_magic_dapperness`＋按腐化分档的 GetDappernessFn（LOW/MED/HIGH 对应 CRAZINESS_TINY/SMALL/MED，均负值） | 主模拟 | [配置](../../../raw/current/scripts/prefabs/king_cane.lua#L325-L330)、[dapperness](../../../raw/current/scripts/prefabs/king_cane.lua#L224-L230)、[CRAZINESS](../../../raw/current/scripts/tuning.lua#L2223-L2225) |
| `组件：weapon` | 本体近战 KING_CANE_DAMAGE（=wilson_attack×.5，即 17）；红蓝宝石形态被改为 damage 0＋弹体（同火/冰杖），其余宝石形态保留近战 | 主模拟；装载/清除见 staff_common | [本体](../../../raw/current/scripts/prefabs/king_cane.lua#L332-L333)、[清除重置](../../../raw/current/scripts/prefabs/staff_common.lua#L900-L902) |
| `API／数据：itemmimicconfig` | `AddActionForceMimicReveal(ACTIONS.GIVE)`——假宝石 GIVE 时强制鉴伪 | 主模拟 | [配置](../../../raw/current/scripts/prefabs/king_cane.lua#L361-L362) |
| `复用：moonbase／stalkerinspectable` | tradable 注释"for moonbase"：月亮石接受 king_cane 插入为子实体（SetFxOwner）；腐化杖由 moonbase 满月充能后 `MakeUncorrupted`；stalkerinspectable 供 STALKER_NPC 评述 | 主模拟 | [收杖](../../../raw/current/scripts/prefabs/moonbase.lua#L311)、[L348-L355](../../../raw/current/scripts/prefabs/moonbase.lua#L348-L355)、[[reference/components/stalkerinspectable#关键接口]] |
| `关联 Prefab：king_cane_fx` | 本体生成 1 个网络化 fx（含 corruptfx/gemfx 本地层），`SetGemType`/`SetFXLevel`/`DoGemShine` 联动 | FX 层级、net 变量与端见 [[reference/prefabs/king_cane_fx]] | [生成](../../../raw/current/scripts/prefabs/king_cane.lua#L306-L313) |

### 滑稽马（yoth_knightstick）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：fueled` | USAGE 燃料 6 天（`YOTH_KNIGHTSTICK_PERISHTIME`）、`no_sewing`（缝纫包不可修）、耗尽 `inst.Remove`；`_onlocomote` 只在持有者前进时 `StartConsuming`，停下即停耗 | 主模拟；装备时挂 locomote 监听、卸下摘除并 StopConsuming | [配置](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L81-L85)、[locomote](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L93-L101)、[穿脱](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L6-L40) |
| `API／数据：gallopstick 标签` | SGwilson 奔跑态读 `EquipHasTag("gallopstick")` 切疾驰动画与速度：移动 0.6s 起步，之后每 16 帧一次 gallop，倍率从 1.15 线性升至 1.6（上限 30 次）；急转积累超 60 旋转压力触发 `predict_gallop_trip` 摔倒，摔倒/滑倒/撞墙额外扣燃料（-1%/-2% ×6 天） | 双端状态表现（客户端预测）；速度计算共享于 player_common_extensions | [标签消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L506)、[倍率](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1208-L1215)、[摔倒](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2558-L2582)、[滑倒损耗](../../../raw/current/scripts/stategraphs/SGwilson.lua#L814-L817)、[常量](../../../raw/current/scripts/tuning.lua#L9309-L9319) |
| `组件：weapon／fuel` | 近战 CANE_DAMAGE（17）；本体可作燃料（fuelvalue LARGE_FUEL＝6 段）入火 | 主模拟 | [weapon](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L87-L88)、[fuel](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L90-L91)、[常量](../../../raw/current/scripts/tuning.lua#L373) |
| `组件：burnable／propagator／hauntable` | MakeSmallBurnable＋MakeSmallPropagator（可被点燃并蔓延）、MakeHauntableLaunch | 主模拟 | [调用](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L103-L106) |

### 月暴静电族（moonstorm_static）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：moonstormstaticcatcher（catcher 侧）` | 装备后对带 `moonstormstaticcapturable` 标签目标给出 `DIVEGRAB`（priority 3、distance 5、rangecheck 7）；`Catch` 校验距离 1＋双方半径后调目标 OnCaught，catcher 自删 | 装备态才生效；`nomagiccast` 无关 | [[reference/components/moonstormstaticcatcher#关键接口]]；[动作收集](../../../raw/current/scripts/componentactions.lua#L2400-L2404)、[动作定义](../../../raw/current/scripts/actions.lua#L703)、[fn](../../../raw/current/scripts/actions.lua#L6594-L6602)、[Catch](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L15-L30) |
| `组件：moonstormstaticcapturable（roamer 侧）` | 被瞄准/解除瞄准推 `moonstormstaticcapturable_targeted`/`_untargeted`（SG 变速反应），被捕捉回调 `OnCaught_roamer`：ReplacePrefab 为 `moonstorm_static_nowag` 并推 `ms_moonstormstatic_roamer_captured` | 服务端；pristine 标签供客户端动作收集 | [[reference/components/moonstormstaticcapturable#关键接口]]；[捕捉](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L464-L470)、[SG 事件](../../../raw/current/scripts/stategraphs/SGmoonstormstatic.lua#L17-L26) |
| `实验流程（世界组件）` | roamer 由 [[reference/components/moonstormmanager]] 的实验变体生成（击败天境守护者后的无 NPC 路径）；捕捉后进入"守卫＋工具＋90s 计时"流程：`nowag` 球收 `wagstaff_tool_1..5` 恢复计时，到点 `finished()` → 替换为 `moonstorm_static_item`（玩家捕捉来源带 `playermade`）；被击杀或移除则实验失败 | 计时/工具投放/守卫逻辑归 moonstormmanager 页；Wagstaff NPC 路径的 `moonstorm_static` 球同用 `finished()` | [manager L528-L566](../../../raw/current/scripts/components/moonstormmanager.lua#L528-L566)、[收工具](../../../raw/current/scripts/components/moonstormmanager.lua#L641-L647)、[结束](../../../raw/current/scripts/components/moonstormmanager.lua#L476-L506)、[失败](../../../raw/current/scripts/components/moonstormmanager.lua#L514-L519) |
| `组件：health／combat（三球）` | health 100（`MOONSTORM_SPARK_HEALTH`，nofadeout）；受击播 hit、死亡播 explode 后 Remove（实验未完成）；`ms_stormchanged`（MOONSTORM 结束）1 秒后若已不在风暴内则 Kill | 服务端；`soulless` 标签 | [配置](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L99-L105)、[风暴自杀](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L49-L56)、[常量](../../../raw/current/scripts/tuning.lua#L6235) |
| `游走与衰变（roamer）` | brain 限定在月暴风暴区内游走（距 8）；`OnZigZagUpdate` 每 0.25-0.5s 抖动重定位＋随机 0.5/1/1.5 倍速；离开风暴区 `StartDecay`（3+rand 秒后 decay 动画自删，休眠即删），回区取消 | 服务端；Brain CheckPointFn 拒绝风暴区外落点 | [brain](../../../raw/current/scripts/brains/moonstormstaticbrain.lua#L12-L36)、[游走](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L437-L462)、[衰变](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L418-L427) |
| `组件：hudindicatable（roamer）` | 客户端 HUD 指示器：需查看者带 `moonstormevent_detector`（月暴风镜装备态授予，[hats.lua](../../../raw/current/scripts/prefabs/hats.lua#L2271)），近于 75（MAX_INDICATOR_RANGE×1.5）、在视锥内且可见 | 仅客户端组件 | [注册](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L507-L510)、[条件](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L472-L479)、[常量](../../../raw/current/scripts/tuning.lua#L3093) |
| `组件：upgrader（item）` | `upgradetype = UPGRADETYPES.SPEAR_LIGHTNING`：对 wathgrithr 战斗长矛（upgradeable 同类型）使用即充能为奔雷形态，充能矛限薇格弗德技能树 `spear_lightning_upgradeuser` | 主模拟；矛侧细节与升级链见 [[reference/prefabs/event-weapons]] | [upgrader](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L311-L312)、[矛侧](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L500-L507)、[门禁](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L536-L537) |
| `消费：配方与建造` | `moonstorm_static_item` 作材料：`security_pulse_cage`（脉冲笼）与 `moon_device_construction2` 建造阶段 | 主模拟 | [L1021](../../../raw/current/scripts/recipes.lua#L1021)、[L1531](../../../raw/current/scripts/recipes.lua#L1531) |

## 典型接入

### 调整亮茄魔杖的伤害与弹跳

- 前提与选择：模组 `modmain.lua` 全局改值即可，五个键都在 `TUNING`；弹跳数在弹体 OnThrown 时读取，改值对已生成弹体无效、对新攻击生效。注意伤害结构：原版在 `SetupComponents` 把 weapon 设为 0，位面伤害在构造时写入 `planardamage`；combat 会同时结算 weapon 普通伤害与位面伤害，所以给 weapon `SetDamage` **会**直接加在输出上，改 TUNING 的位面键只影响之后新造的实例。
- 生效与撤销：TUNING 键在 prefab 构造与弹体运行时读取，`AddGamePostInit` 覆盖后新 spawned 实体即用新值；卸载模组恢复原值。

```lua
-- modmain.lua：调整亮茄魔杖数值（示例改值）
local TUNING = GLOBAL.TUNING
AddGamePostInit(function()
    TUNING.STAFF_LUNARPLANT_PLANAR_DAMAGE = 15   -- 位面伤害（原 10）
    TUNING.STAFF_LUNARPLANT_VS_SHADOW_BONUS = 3  -- 对 shadow_aligned 加成（原 2）
    TUNING.STAFF_LUNARPLANT_BOUNCES = 8          -- 弹跳次数（原 5；套装判定见下）
end)
```

- 调整与验证：弹跳套装判定**不读 setbonus 组件**，直接判头部 `lunarplanthat`（[OnThrown](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L23-L30)），改套装加成须同时覆盖 `STAFF_LUNARPLANT_SETBONUS_BOUNCES` 的消费条件或该判定本身；对暗影加成消费链见 [[reference/components/damagetypebonus#关键接口]]。

### 为指定武器新增可升级充能材料（upgrader／upgradeable 配对）

- 前提与选择：模组自定义"能量静电"式材料时复用原版结构——材料侧 `upgrader`＋目标侧 `upgradeable`，`upgradetype` 一致只是第一层；动作收集还遍历 `UPGRADETYPES` 并要求 `<type>_upgrader`／`<type>_upgradeuser`／`<type>_upgradeable` 三组标签，服务端 `Upgrader:CanUpgrade` 再核类型与操作者标签，`Upgradeable:CanUpgrade` 默认从第 1 阶起、上限 3 阶、每阶 5 次；自定义 `canupgradefn` 只是附加条件，返回 true 也不能绕过阶段上限。原版参照 `moonstorm_static_item` → `spear_wathgrithr`。
- 生效与撤销：材料使用时按目标 `upgradeable` 的回调消耗并升级；`upgradetype` 不匹配的目标不会出现使用动作。

```lua
-- modmain.lua：自定义类型须登记进 UPGRADETYPES，并补齐三组标签
GLOBAL.UPGRADETYPES.MYSTUFF = "mystuff"

AddPrefabPostInit("mystaticball", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:AddTag("mystuff_upgrader")
    inst:AddComponent("upgrader")
    inst.components.upgrader.upgradetype = "mystuff"
end)

AddPrefabPostInit("myweapon", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:AddTag("mystuff_upgradeable")
    inst:AddComponent("upgradeable")
    inst.components.upgradeable.upgradetype = "mystuff"
end)

AddPlayerPostInit(function(inst)
    inst:AddTag("mystuff_upgradeuser")
end)
```

- 调整与验证：升级成功由 `Upgradeable:Upgrade` 消耗材料（堆叠取 1）并推进阶段；不满足阶段/标签时动作不出现或返回原因。升级值、角色门禁（restrictedtag）与升级后形态参照 [spear_wathgrithr.lua](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L500-L537)；矛侧完整机制（充能形态、技能树门禁）见 [[reference/prefabs/event-weapons]]。

## 常见陷阱与限制

- **误区：staff_tornado 的 `castingstate = "castspell_tornado"` 生效**：全 scripts 仅此一处赋值，无任何消费端，SGwilson 也没有该状态——本 build 该字段无效，实际施法走通用 CASTSPELL／quickcast 路径。[赋值](../../../raw/current/scripts/prefabs/staff_tornado.lua#L111)、[全库无消费端]
- **误区：亮茄魔杖伤害 10 是普通伤害**：原版 weapon damage 配置为 0，10 是位面伤害（无视护甲），对 `shadow_aligned` 目标再 ×2；图鉴同样按 planardamage=10 记录。[scrapbook](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1438)、[组件配置](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L213-L217)
- **边界：弹跳套装判定绕过 setbonus 组件**：弹体直接读头部 `lunarplanthat` 决定 5/7 次，staff 的 setbonus 组件不参与该判定；给其他帽子做"套装弹跳"须仿写头部检查。[判定](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L27)
- **边界：国王魔杖未腐化时不耗次数/充能**：`dodischarge=false` 且无 finiteuses，staff_common 的两分支都不命中，但精神与腐化仍照常结算；满腐化后每次施法才按 `KING_CANE_*_COOLDOWN` 放电，放电中 `nomagiccast` 禁施法直到充回。[分支](../../../raw/current/scripts/prefabs/staff_common.lua#L368-L372)、[门禁](../../../raw/current/scripts/actions.lua#L3445-L3450)
- **边界：tornado 的 `tornado_immune` 标签无原版授予者**：SG 只消费该标签做排除，全库无写入点——想保护单位要由模组自己加标签。[排除表](../../../raw/current/scripts/stategraphs/SGtornado.lua#L58)
- **边界：滑稽马耗尽即消失且不可缝补**：`DepletedFn=inst.Remove`＋`no_sewing=true`；燃料只在移动时消耗，但疾驰摔倒/滑倒按总寿命 1%／2% 一次性倒扣。[配置](../../../raw/current/scripts/prefabs/yoth_knightstick.lua#L81-L85)、[损耗](../../../raw/current/scripts/stategraphs/SGwilson.lua#L814-L817)
- **边界：三个静电球都不进存档**：`moonstorm_static`/`_nowag`/`_roamer` 均 `persists=false`，实验中断（存档重载）后静电球消失，manager 侧实验态由计时器与事件重建；只有产物 `moonstorm_static_item` 可存档（playermade 位落档）。[标志](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L90)、[L242](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L242)、[item 存档](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L270-L279)
- **边界：`playermade` 只改外观**：玩家捕捉来源的产物覆盖符号并换用 `moonstorm_static_catcher_item` 图标，全库无 gameplay 消费端。[MakePlayerMade](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L264-L269)
- **边界：国王魔杖拒收蛋白石**：`ItemTradeTest` 对 prefab 含 "precious" 段（如 `opalpreciousgem`）返回 WRONGGEM，因此 king_cane 无 opal 形态、合并依赖时只跳过 `staffcoldlight`；yellow 的 `stafflight` 仍随依赖表预载。[测试](../../../raw/current/scripts/prefabs/king_cane.lua#L107-L116)、[依赖排除](../../../raw/current/scripts/prefabs/king_cane.lua#L23-L31)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。核验五文件全文与 12 项注册、staff_common 七分色工厂的装载/清除分支、brilliance_projectile_fx 弹跳链、SGtornado destroystuff 与状态调用点、tornadobrain、SGmoonstormstatic、moonstormstaticbrain、moonstormstaticcatcher/capturable 组件、moonstormmanager 实验流程、wagstaff_npcbrain 生成点、corruption/rechargeable/setbonus/forgerepairable/spellcaster/equippable 消费端、SGwilson gallopstick 三处消费与摔倒事件、spear_wathgrithr 升级段、配方（recipes/recipes_filter）、TUNING 全部引用键、皮肤清单、moonbase 收杖与解腐化分支、.po 官方名 11 键。未覆盖：SGtornado 全部状态机逐帧、moonstorm_spark／lightning prefab、Wagstaff NPC SG、皮肤资源包内容、featherfan 全文。
- 游戏运行：未运行；弹跳表现、龙卷风伤害频率、疾驰倍率手感、静电实验全流程均为源码推导值。
- 结构检查：2026-11-07 validate_page.py 实测 188 行 / 39627 字节（complex 档 480 行 / 64 KiB 内），0 issue；validate_source_links.py --strict 0 error 0 warning（156 处行引用、33 个源文件）；validate_source_hashes.py check 通过（本页未入源码哈希基线，属预期，未 record）。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]；同族施法物品 [[reference/prefabs/staffs|基础法杖族]]、[[reference/prefabs/yellowstaff|唤星者魔杖]]、[[reference/prefabs/opalstaff|唤月者魔杖]]。
