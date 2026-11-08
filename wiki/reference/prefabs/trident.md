---
title: "刺耳三叉戟（trident）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版海洋手持武器 trident：TECH.LOST 蓝图配方制作，近战伤害按攻击者位置分地面 51／海面 76.5，右键点海面触发水爆（伤害 85、可砸矿、掀鱼、拔海带、船漏水、照料农田），耐久 200 兼吃攻击磨损与水爆消耗；无 tool、无取水组件、不可作祟。"
aliases: ["trident", "刺耳三叉戟", "三叉戟", "Strident Trident"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/trident.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/tradable.lua", "components/spellcaster.lua", "components/builder.lua", "components/klaussackloot.lua", "components/shard_mermkingwatcher.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "recipes.lua", "recipes_filter.lua", "prefabs/blueprint.lua", "prefabs/crabking.lua", "prefabs/mermking.lua", "prefabs/wurt.lua", "prefabs/merm.lua", "prefabs/mermking_gear_buffs.lua", "prefabs/skilltree_wurt.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "skin_strings.lua", "stategraphs/SGwilson.lua", "prefabs/boat.lua", "fx.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 刺耳三叉戟（trident）

## 快速判断

- 职责与适用：`prefabs/trident.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽，特色是 spellcaster 点海面施法的水爆（`canuseonpoint_water`）；实体级 `spelltype = "MUSIC"`，动作文案走"演奏"；无 tool、无 wateryprotection／fillable／watersource（不能取水或当水壶），同文件无 quagmire 分支，也未调用 `MakeHauntableLaunch`（不可作祟）。[工厂](../../../raw/current/scripts/prefabs/trident.lua#L201-L279)、[注册](../../../raw/current/scripts/prefabs/trident.lua#L281)
- 优先使用：复刻"点水施法武器"按[水爆施法链](#水爆施法链)抄 spellcaster＋reticule 接线；改伤害／耐久／水爆数值查[数值与消耗](#数值与消耗)；给伍特阵营接入读[关联机制导航](#关联机制导航)的鱼人王条目；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：水爆分两段。实体分派（半径 2.75、`IsOnOcean(false)`）：health 受 85（combat 还须 `CanBeAttacked`）；`oceanfishable` 变投射物；海带拔起；inventoryitem 掀飞；MINE 砸 10。四点采样：落点外圈 4 点，点须海洋瓦片且非视觉地面，平台→中型漏水并推船，否则水柱。另以持有者为中心照料 20 内农田。
- 修改边界：标签、reticule、floater 挂公共端，组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/trident.lua#L233-L236)）；施法动作是客户端发起、服务端 `ACTIONS.CASTSPELL` 结算；本页不覆盖 `crab_king_waterspout` FX 内部与鱼人王技能树细节。[动作入口](../../../raw/current/scripts/actions.lua#L3445-L3468)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `trident` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备＋施法物；皮肤经 `trident_init_fn`。[注册](../../../raw/current/scripts/prefabs/trident.lua#L281)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L860-L868) |

共同初始化（[trident.lua#L201-L279](../../../raw/current/scripts/prefabs/trident.lua#L201-L279)）：Transform／AnimState／Network → `MakeInventoryPhysics` → 公共端标签 `allow_action_on_impassable`／`guitar`／`pointy`／`sharp`／`weapon` → 实体字段 `spelltype = "MUSIC"` → reticule（目标点 3.5、双摇杆参数、`validfn` 限海面）→ `SetBank("trident")`＋`SetBuild("trident")`＋`idle` → `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -9, {sym_build = "swap_trident"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → tradable → equippable → `inst.DoWaterExplosionEffect` 方法 → spellcaster。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.TRIDENT`＝刺耳三叉戟（Strident Trident），检视文案"等着轰轰轰吧！"。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L350589-L350592)、[.po 检视](../../../raw/current/scripts/languages/chinese_s.po#L20984-L20987)

### 数值与消耗

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 近战伤害（攻击者在地面上） | `TUNING.TRIDENT.DAMAGE`＝`wilson_attack`×1.5 | 51 生命值（源码推导，[tuning.lua#L5431](../../../raw/current/scripts/tuning.lua#L5431)） |
| 近战伤害（攻击者在海面） | `TUNING.TRIDENT.OCEAN_DAMAGE`＝`wilson_attack`×2.25 | 76.5 生命值（[tuning.lua#L5432](../../../raw/current/scripts/tuning.lua#L5432)；图鉴显示"51-76"，[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1541)） |
| 耐久上限 | `TUNING.TRIDENT.USES` | 200（[tuning.lua#L5433](../../../raw/current/scripts/tuning.lua#L5433)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次（[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121)） |
| 每次水爆消耗 | 加载期常量 `USES / SPELL.USE_COUNT`＝200/50 | 4 耐久点/次，全寿命 50 次水爆（[trident.lua#L139](../../../raw/current/scripts/prefabs/trident.lua#L139)、[L197](../../../raw/current/scripts/prefabs/trident.lua#L197)；`SPELL.USE_COUNT`＝50，[tuning.lua#L5436](../../../raw/current/scripts/tuning.lua#L5436)） |
| 水爆伤害 | `TUNING.TRIDENT.SPELL.DAMAGE`＝`wilson_attack`×2.5 | 85 生命值（[tuning.lua#L5438](../../../raw/current/scripts/tuning.lua#L5438)） |
| 水爆目标搜索半径 | `TUNING.TRIDENT.SPELL.RADIUS` | 2.75（施法时运行时读取；[tuning.lua#L5437](../../../raw/current/scripts/tuning.lua#L5437)） |
| 水爆采掘工作量 | `TUNING.TRIDENT.SPELL.MINES` | 10 工作量/次，仅 MINE 类 workable（[tuning.lua#L5439](../../../raw/current/scripts/tuning.lua#L5439)、[trident.lua#L129-L131](../../../raw/current/scripts/prefabs/trident.lua#L129-L131)） |
| 农田照料半径 | `TUNING.TRIDENT_FARM_PLANT_INTERACT_RANGE` | 20，以持有者为中心（[tuning.lua#L5442](../../../raw/current/scripts/tuning.lua#L5442)、[trident.lua#L189-L194](../../../raw/current/scripts/prefabs/trident.lua#L189-L194)） |

伤害分档读**攻击者**位置而非目标：`trident_damage_calculation` 用 `IsVisualGroundAtPoint(attacker:GetPosition())` 分档，站在岸上打海里目标仍按地面 51 结算。[回调](../../../raw/current/scripts/prefabs/trident.lua#L21-L24)、[FunctionOrValue 消费](../../../raw/current/scripts/components/weapon.lua#L94-L95)

### 获取途径（蓝图链）

| 途径 | 内容与证据 |
|---|---|
| 配方 | 一角鲸的角（gnarwail_horn，见 [[reference/prefabs/gnarwail_horn]]）3＋海带 4＋树枝 2，`TECH.LOST`（＝{MAGIC=10, SCIENCE=10, ANCIENT=10}，[constants.lua#L1324](../../../raw/current/scripts/constants.lua#L1324)）；任何原型器科技树都到不了 10 级，配方只能经蓝图习得。[配方](../../../raw/current/scripts/recipes.lua#L491) |
| 蓝图注册 | blueprint.lua 按 `AllRecipes` 给每个可蓝图的配方自动注册 `<配方名>_blueprint`，`trident_blueprint` 因 `TECH.LOST` 各级＞0 而注册；读取后 `teacher` 授予配方，`KnowsRecipe` 对已学配方短路返回 true，学过即可随处制作。[注册循环](../../../raw/current/scripts/prefabs/blueprint.lua#L239-L243)、[KnowsRecipe 短路](../../../raw/current/scripts/components/builder.lua#L883-L887) |
| 蓝图掉落：帝王蟹 | `SetSharedLootTable("crabking")` 固定掉 `trident_blueprint`（概率 1.00）。[掉落表](../../../raw/current/scripts/prefabs/crabking.lua#L82) |
| 蓝图掉落：克劳斯（Klaus） | Klaus 战利品卷积中 `giant_loot2`（9 项之一，含 `trident_blueprint`）有 50% 概率从中均匀抽 1 项，单次命中三叉戟蓝图为 `0.5×1/9=1/18`。[klaussackloot](../../../raw/current/scripts/components/klaussackloot.lua#L20)、[50% 抽取](../../../raw/current/scripts/components/klaussackloot.lua#L117-L120) |
| 作祟重掷边界 | `CanBlueprintRandomRecipe` 排除等级 ≥10 的配方（注释即"Exclude TECH.LOST"），普通蓝图作祟重掷不会变成三叉戟蓝图。[排除](../../../raw/current/scripts/prefabs/blueprint.lua#L45-L56) |

`trident` 本体无其他生成点：全 scripts 检索仅剩 debug 指令清单、prefablist／scrapbook／皮肤／strings 等数据文件与按 prefab 名判定的消费者（wurt／mermking 系），无战利品表或布局直接掉落实体。配方归 crafting 菜单 WEAPONS 类。[过滤](../../../raw/current/scripts/recipes_filter.lua#L1019-L1044)

### 皮肤

| 皮肤（官方简中名） | 稀有度 | 接线与差异 | 注册证据 |
|---|---|---|---|
| 力挽狂澜三叉戟（trident_merm，Tide-Turner Trident） | ProofOfPurchase | `trident_init_fn`→`basic_init_fn` `SetSkin("trident_merm", "trident")`，主模拟 `AddSkinSounds`；装备／卸装事件与持握换装走通用 [on_equipped](../../../raw/current/scripts/prefabs/trident.lua#L34-L45) | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L13328-L13336)、[init](../../../raw/current/scripts/prefabskin.lua#L860-L868)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L407954-L407957)、[英文表](../../../raw/current/scripts/skin_strings.lua#L3883) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(trident_damage_calculation)` 按攻击者位置分档；无 onattack／投射 | 主模拟；`GetDamage` 经 `FunctionOrValue` 每击调用回调；无 `attackwear` 按每次 1 磨损 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/trident.lua#L242-L243) |
| 组件：finiteuses | 上限＝当前＝200；未设 `SetConsumption`（动作消耗不触达）；`SetOnFinished(on_uses_finished)`：给持有者推 `toolbroke` 后 `Remove` 整个实体 | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/trident.lua#L247-L250)、[回调](../../../raw/current/scripts/prefabs/trident.lua#L26-L32) |
| 组件：spellcaster | `SetSpellFn(create_water_explosion)`＋`canuseonpoint_water = true`（实体得 `castonpointwater` 标签）；组件级 `spelltype` 未设 → 无施法者标签限制 | 主模拟组件；`CanCast` 点位路径：`IsOceanAtPoint` 且非被阻挡地面点 | [[reference/components/spellcaster#关键接口]]；[接线](../../../raw/current/scripts/prefabs/trident.lua#L272-L276)、[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L173-L174) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`、ARM_carry／ARM_normal 显隐、皮肤事件 | 主模拟；槽默认 HANDS；皮肤推 `equipskinneditem`／`unequipskinneditem` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/trident.lua#L266-L268)、[onequip](../../../raw/current/scripts/prefabs/trident.lua#L34-L45) |
| 组件：inspectable／inventoryitem／tradable | 检视、拾取（耗尽回调读 `owner`、水爆读 `GetGrandOwner`、落水 `SetLanded`）与可交易（默认 `goldvalue` 0） | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]、[[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/trident.lua#L254-L262) |
| 组件：reticule | `targetfn` 取玩家前方 3.5、`ease`／双摇杆参数、`validfn`＝海面且未被阻挡 | 公共端；客户端瞄准环显示 | [[reference/components/reticule#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/trident.lua#L218-L225) |
| 组件：floater | `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -9, swap_data)` | 公共端；浮水换 `swap_trident` 贴图 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/trident.lua#L231)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| API／动作：`ACTIONS.CASTSPELL` | 点海面动作入口：componentactions 按 `castonpointwater` 校验海面（排除驾驶／转舵中），action fn 调 `CanCast`→`CastSpell` 并推 `onspellcast` | 客户端发起、服务端结算；`spelltype="MUSIC"` 供 strfn 出"演奏"文案 | [点位筛选](../../../raw/current/scripts/componentactions.lua#L2193-L2225)、[action fn](../../../raw/current/scripts/actions.lua#L3445-L3468)、[strfn](../../../raw/current/scripts/actions.lua#L3442) |
| API／数据：`guitar` 标签 | SGwilson 的 CASTSPELL ActionHandler 按此选 `play_strum` 施法状态（弹奏动作） | 客户端 SG 状态选择 | [ActionHandler](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1304-L1312) |
| API／数据：`allow_action_on_impassable` 标签 | playeractionpicker／playercontroller 将装备／手持物此标签并入 `alwayspassable`，允许对海面等不可通行点给点动作 | 客户端动作筛选 | [picker](../../../raw/current/scripts/components/playeractionpicker.lua#L463)、[controller](../../../raw/current/scripts/components/playercontroller.lua#L5405) |
| API／数据：`sharp`／`pointy` 标签 | `sharp` 切命中音效档；`pointy` 作 Hutch 伤害反弹电池 | 命中结算／装入 Hutch 时读取 | [[reference/components/combat#关键接口]]；[音效](../../../raw/current/scripts/components/combat.lua#L745)、[电池](../../../raw/current/scripts/prefabs/hutch.lua#L138-L140) |
| 关联 Prefab：`trident_blueprint` | 教学蓝图（teach 后习得配方）；本体页唯一获取前置 | 掉落见[获取途径](#获取途径蓝图链) | （Wiki 待建）；[蓝图工厂](../../../raw/current/scripts/prefabs/blueprint.lua#L225-L233)、[OnTeach](../../../raw/current/scripts/prefabs/blueprint.lua#L40-L42) |
| 关联 Prefab：`crab_king_waterspout` | 四点采样中无平台的海面点生成，至多 4 个 | 声明于 `prefabs` 且实际 SpawnPrefab；纯 FX（fx.lua 注册表 bank `Bubble_fx`） | [生成](../../../raw/current/scripts/prefabs/trident.lua#L183-L185)、[FX 定义](../../../raw/current/scripts/fx.lua#L2177-L2183) |
| 关联 Prefab：`bullkelp_plant`／`bullkelp_root` | 水爆将海带连根拔起：爆出 product（继承湿痕、按 numtoharvest 堆叠）＋生成 bullkelp_root，本体 Remove | 目标为海面海带时 | [分支](../../../raw/current/scripts/prefabs/trident.lua#L98-L123) |
| 关联 Prefab：船（boat 系平台） | 四点采样命中平台时推 `spawnnewboatleak`（med_leak，向内偏移防滑出）＋ `boatphysics:ApplyForce` 推船；与实体伤害分派是两段 | boat.lua 监听该事件生成漏水 | [[reference/components/boatphysics#关键接口]]；[泄漏监听](../../../raw/current/scripts/prefabs/boat.lua#L779)、[施法侧](../../../raw/current/scripts/prefabs/trident.lua#L166-L181) |
| 关联 Prefab：鱼人王（mermking）＋伍特 | 供奉链： giver 需激活 `wurt_mermkingtrident` 技能且王未持三叉戟；献上后 `get_trident` 状态＋推 `onmermkingtridentadded`（掉出推 removed）；Wurt 监听后给自己和鱼人随从挂 `mermkingtridentbuff`（debuff `mermking_buff_trident`：`externaldamagemultipliers` +1.05）；跨分片由 shard_mermkingwatcher 转 `_anywhere` 事件；鱼人卫兵带 debuff 时按 0.33 幸运掷点三连击 | 伍特技能树分支；供奉与 buff 均主模拟 | [[reference/components/skilltreeupdater#关键接口]]；[受礼](../../../raw/current/scripts/prefabs/mermking.lua#L204-L224)、[供奉](../../../raw/current/scripts/prefabs/mermking.lua#L254-L256)、[升格](../../../raw/current/scripts/prefabs/wurt.lua#L189-L204)、[buff](../../../raw/current/scripts/prefabs/mermking_gear_buffs.lua#L49-L64)、[跨分片](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L49-L64)、[三连击](../../../raw/current/scripts/prefabs/merm.lua#L1261-L1266)、[技能节点](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L341) |
| 资源：anim／swap 动画包 | trident.zip（bank＋build）／swap_trident.zip（持握符号）；INV_IMAGE `trident` | 静态声明 | [声明](../../../raw/current/scripts/prefabs/trident.lua#L1-L6) |

## 典型接入

### 水爆施法链

- 前提与选择：理解原版接线再复刻或改参。完整链条（服务端结算）：右键海面点 → componentactions `spellcaster` 点位筛选（`castonpointwater` → `IsOceanAtPoint`，排除 `steeringboat`／`rotatingboat`）→ `ACTIONS.CASTSPELL` → `CanCast`（spellcaster L173-174 同样海面校验）→ `CastSpell(nil, pos, doer)` → `create_water_explosion(inst, nil, position)`。[点位筛选](../../../raw/current/scripts/componentactions.lua#L2213-L2221)、[action fn](../../../raw/current/scripts/actions.lua#L3465-L3467)
- 生效与撤销：施法只在 `GetGrandOwner()` 非空时继续（物品不在持有链直接返回，不耗耐久）；每次施法固定扣 4 耐久，与目标多少无关。[Owner 检查](../../../raw/current/scripts/prefabs/trident.lua#L141-L144)
- 原版依据：分两段——①实体分派（半径 2.75、`IsOnOcean(false)`）：health→`GetAttacked`（combat 须 `CanBeAttacked`）或 `DoDelta(-85)`；oceanfishable→投射物（无 complexprojectile 时掀飞）；海带拔起；inventoryitem 掀飞；MINE→`WorkedBy(10)`。②四点采样（`FX_RADIUS` 外圈 4 点，海洋瓦片且非视觉地面）：平台→中型漏水＋推船，否则水柱。[实体分派](../../../raw/current/scripts/prefabs/trident.lua#L68-L130)、[四点采样](../../../raw/current/scripts/prefabs/trident.lua#L146-L186)

### 调整三叉戟的近战与水爆数值

- 前提与选择：服务端 `modmain.lua`。近战伤害经 `SetDamage` 整体替换回调（原回调逻辑简单，重写成本低）；水爆参数在每次施法时运行时读 `TUNING`，modmain 环境直挂 `TUNING` 白名单，`TheWorld` 等须 `GLOBAL.` 前缀（[GLOBAL 字段](../../../raw/current/scripts/mods.lua#L289)、[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)）。
- 生效与撤销：postinit 只影响新实例；`TUNING.TRIDENT.SPELL.*` 改动立即影响所有实例的水爆结算。停用模组恢复原值。
- 原版依据：分档回调读攻击者位置（[trident.lua#L21-L24](../../../raw/current/scripts/prefabs/trident.lua#L21-L24)）；耐久在 finiteuses，两条磨损路径独立（见[陷阱](#常见陷阱与限制)）。

```lua
-- modmain.lua：地面近战 60／海面 90，耐久 300；水爆伤害 120、半径 4
AddPrefabPostInit("trident", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetDamage(function(w, attacker, target)
        local is_over_ground = GLOBAL.TheWorld.Map:IsVisualGroundAtPoint(attacker:GetPosition():Get())
        return is_over_ground and 60 or 90
    end)
    inst.components.finiteuses:SetMaxUses(300)
    inst.components.finiteuses:SetUses(300)
end)

-- 水爆结算每次施法时读表，运行时改 TUNING 即时生效
TUNING.TRIDENT.SPELL.DAMAGE = 120
TUNING.TRIDENT.SPELL.RADIUS = 4
```

- 调整与验证：伤害单位为生命值/击，工作量点/次，半径为世界单位；改 `TUNING.TRIDENT.USES`／`SPELL.USE_COUNT` 不会改变每次水爆消耗（加载期已算好 4，见陷阱）。

## 常见陷阱与限制

- **误区：伤害分档看目标位置**：分档读攻击者所处地面（`IsVisualGroundAtPoint(attacker:GetPosition())`），站在岸上打海中目标仍是 51。[回调](../../../raw/current/scripts/prefabs/trident.lua#L21-L24)
- **边界：两条磨损路径独立且水爆消耗是加载期常量**：攻击 1/次（`attackwear or 1`，吃 efficientuser 等乘区）、水爆 4/次（直接 `Use(COST_PER_EXPLOSION)`，无乘区）；`COST_PER_EXPLOSION` 与 FX 半径 `FX_RADIUS` 是文件加载期 local，运行时改 `TUNING.TRIDENT.USES`／`SPELL.USE_COUNT` 不影响每次消耗，但 `SPELL.RADIUS` 施法时运行时读取、改了即时生效。[常量](../../../raw/current/scripts/prefabs/trident.lua#L138-L139)、[施法读表](../../../raw/current/scripts/prefabs/trident.lua#L149)
- **边界：只能点海面施法**：`canuseonpoint_water` 分支要求 `IsOceanAtPoint`，地面点不给出动作；驾驶／转舵中的船也不能施法；瞄准环 `validfn` 同步海面校验。[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L173-L174)、[点位筛选](../../../raw/current/scripts/componentactions.lua#L2213-L2221)
- **边界：不是取水／浇水工具**：未挂 wateryprotection／fillable／watersource，不能灌水壶或直接浇地；农田照料是水爆时以**持有者**为中心、20 范围内 `tendable_farmplant` 实体执行 `farmplanttendable:TendTo(owner)` 的附带效果，与爆炸落点无关。[照料](../../../raw/current/scripts/prefabs/trident.lua#L189-L194)
- **边界：目标须在海洋上**：`FindEntities` 结果再经 `IsOnOcean(false)` 过滤，岸上贴边目标不吃水爆；有 combat 的还须 `CanBeAttacked(owner)` 才 `GetAttacked`，否则该目标不受伤；无 combat 的 health 目标走 `DoDelta` 且不触发攻击反应。[过滤](../../../raw/current/scripts/prefabs/trident.lua#L150-L153)、[分派](../../../raw/current/scripts/prefabs/trident.lua#L69-L77)
- **边界：耐久耗尽即整个实体移除**：先给持有者推 `toolbroke` 再 `Remove`，无残骸掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/trident.lua#L26-L32)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：不可作祟**：同文件未调用 `MakeHauntableLaunch`（对比 spear／batbat），无作祟行为；普通蓝图作祟也不会重掷出三叉戟蓝图。[注册](../../../raw/current/scripts/prefabs/trident.lua#L254-L279)、[重掷排除](../../../raw/current/scripts/prefabs/blueprint.lua#L45-L56)
- **边界：`TECH.LOST` 配方学过即可随处制作**：`KnowsRecipe` 对已学配方短路，不再校验科技树等级；但未习得前任何原型器都无法解锁（等级 10 不可达）。[短路](../../../raw/current/scripts/components/builder.lua#L883-L887)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/trident.lua 全文（工厂、伤害回调、水爆两段流程、装备皮肤接线、加载期常量）、weapon／finiteuses／spellcaster／equippable／inspectable／inventoryitem／tradable、tuning 九键、配方与 `TECH.LOST`、blueprint 注册／重掷／授学、KnowsRecipe、帝王蟹掉落与 Klaus 池抽取、componentactions／actions 入口、SGwilson `guitar`、boat 泄漏监听、fx 水柱、鱼人王链、皮肤注册与 .po 名称／文案；全 scripts grep "trident" 其余命中均为数据／调试，非生成点。
- 游戏运行：未运行。未覆盖：51／76.5／85 均为源码推导值（`wilson_attack`＝34）非运行测量；水爆实机观感（水柱落点、船漏偏移、抛鱼轨迹）、Klaus 实机抽取概率、皮肤实装外观未验证。
- 结构检查：validate_page.py（0 error，145 行／24489 字节，standard 档）与 validate_source_links.py --strict（97 条源码引用、35 个源文件，0 error 0 warning）实际执行通过。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
