---
title: "蝙蝠棒（batbat）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版吸血手持武器 batbat：命中给未满血的攻击者回血 6.8 并扣理智（蝙蝠帽减半），伤害 42.5、耐久 75 仅攻击磨损、影子等级 2 供 Maxwell 影像随从增伤；同文件另注册两个皮肤命中 FX 代理。"
aliases: ["batbat", "蝙蝠棒", "Bat Bat", "吸血武器", "武器吸血"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/batbat.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/health.lua", "components/shadowlevel.lua", "components/combat.lua", "componentutil.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "standardcomponents.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "skin_strings.lua", "prefabs/hats.lua", "prefabs/waxwell.lua", "stategraphs/SGshadowwaxwell.lua", "prefabs/magicprototyper.lua", "map/layouts.lua", "map/pointsofinterest.lua", "worldgen_main.lua", "map/static_layouts/ruins_camp.lua", "map/static_layouts/skeleton_batfight.lua", "prefabs/treasurechest.lua", "scenarios/chest_cavesupplies.lua", "scenarios/chest_labyrinth.lua", "scenarios/chest_labyrinth_mimic.lua", "widgets/bloodover.lua", "widgets/healthbadge.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蝙蝠棒（batbat）

## 快速判断

- 职责与适用：`prefabs/batbat.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽，特色是 onattack 吸血（回血＋理智代价）与 shadowlevel 影子等级；无 tool，同文件无 quagmire 分支。同文件另注册 `batbat_bats`／`batbat_fantasy_fx` 两个皮肤命中 FX 代理（见[注册与构造](#注册与构造)）。[工厂](../../../raw/current/scripts/prefabs/batbat.lua#L58-L113)、[注册](../../../raw/current/scripts/prefabs/batbat.lua#L268-L270)
- 优先使用：改伤害／耐久／吸血／理智代价查下方数值表与[命中吸血流程](#命中吸血与理智消耗)；复刻"命中回血＋理智代价"读[典型接入](#修改蝙蝠棒的吸血与理智代价)；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：命中时若攻击者未满血且目标可汲命，给攻击者回 6.8 血并按 0.5 倍率扣理智（HEAD 槽带 `bathat` 标签的蝙蝠帽减半至 0.25）；耐久耗尽 `SetOnFinished(inst.Remove)` 整个实体移除。[drain 块](../../../raw/current/scripts/prefabs/batbat.lua#L47-L55)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/batbat.lua#L82-L86)）；皮肤命中 FX 经全局 `SKIN_FX_PREFAB` 在 onattack 内触发，本页不覆盖皮肤贴图内容与蝙蝠帽自身机制（回血光环等）。[SKIN_FX_PREFAB 写入](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `batbat` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `batbat_init_fn`（basic_init_fn，swap 符号 `swap_batbat`）。[注册](../../../raw/current/scripts/prefabs/batbat.lua#L268)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L193-L194) |
| `batbat_bats` | `batsfn`：FX 代理实体（Transform＋Network），`net_tinybyte` 同步 `variation` | 蝙蝠镰刀皮肤（`batbat_scythe`）的攻击命中 FX 代理；服务端 0.5 秒后自删、`persists = false`，非专用端在 0 帧任务生成本地视觉实体。[工厂](../../../raw/current/scripts/prefabs/batbat.lua#L166-L193) |
| `batbat_fantasy_fx` | `fantasyfn`：与 batsfn 同构，`net_tinybyte("batbat_fantasy.variation")` | 魔幻蝙蝠棒皮肤（`batbat_fantasy`）的攻击命中 FX 代理；本地实体改用 bank `batbat_fantasy`＋`SetSkin("batbat_fantasy")`。[工厂](../../../raw/current/scripts/prefabs/batbat.lua#L239-L266) |

`batbat` 共同初始化（[batbat.lua#L58-L113](../../../raw/current/scripts/prefabs/batbat.lua#L58-L113)）：实体组件＋`MakeInventoryPhysics` → bank／build `batbat`＋`idle` → 公共端标签 `dull`、`weapon`、`shadowlevel`（pristine 镜像注释）→ `MakeInventoryFloatable("large", 0.05, {0.8, 0.35, 0.8}, true, -27, {sym_build = "swap_batbat"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → equippable → shadowlevel（`SetDefaultLevel(2)`）→ `MakeHauntableLaunch`。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.BATBAT`＝蝙蝠棒（Bat Bat）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L339090-L339092)

### 数值与消耗

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 耐久上限 | `TUNING.BATBAT_USES` | 75（[tuning.lua#L180](../../../raw/current/scripts/tuning.lua#L180)） |
| 武器伤害 | `TUNING.BATBAT_DAMAGE`＝`wilson_attack`×1.25 | 42.5 生命值（源码推导，[tuning.lua#L349](../../../raw/current/scripts/tuning.lua#L349)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |
| 命中吸血 | `TUNING.BATBAT_DRAIN`＝`wilson_attack`×0.2 | 6.8 生命值/击，正 `DoDelta` 治疗攻击者（[tuning.lua#L350](../../../raw/current/scripts/tuning.lua#L350)） |
| 命中理智消耗 | `TUNING.BATBAT_SANITY_DRAIN_MULT`＝0.5；蝙蝠帽套装 `BATBAT_SETBONUS_SANITY_DRAIN_MULT`＝0.25 | −3.4／−1.7 理智/击（`−mult × BATBAT_DRAIN`，[tuning.lua#L9657-L9658](../../../raw/current/scripts/tuning.lua#L9658-L9659)） |
| 影子等级 | `TUNING.BATBAT_SHADOW_LEVEL` | 2（[tuning.lua#L2865](../../../raw/current/scripts/tuning.lua#L2865)） |

### 命中吸血与理智消耗

每次成功命中的服务端执行序：combat 结算调用 `weapon:OnAttack`（[combat.lua#L1224](../../../raw/current/scripts/components/combat.lua#L1224)）→ 先执行 onattack 回调（[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）→ 再按 `(attackwear or 1)×multipliers` 扣耐久。onattack（[batbat.lua#L35-L56](../../../raw/current/scripts/prefabs/batbat.lua#L35-L56)）分两块：

1. **皮肤 FX**：有皮肤且目标有 combat 组件并有效时，`SpawnPrefab` 该皮肤的 fx_prefab 挂到目标、`FollowSymbol` 目标 `hiteffectsymbol`，并调 `fx:OnFXSpawned(武器实例)` 决定动画变体（见下方皮肤小节）。
2. **吸血与理智**：`owner.components.health` 存在且 `IsHurt()`（未满血）且 `IsLifeDrainable(target)` 时，`health:DoDelta(TUNING.BATBAT_DRAIN, false, "batbat")`（[DoDelta 签名](../../../raw/current/scripts/components/health.lua#L613)）；再若有 sanity 组件，读 HEAD 槽装备按是否带 `bathat` 标签选倍率，`sanity:DoDelta(−mult × BATBAT_DRAIN)`。满血只跳过吸血与理智块；皮肤 FX 仍按皮肤与目标 combat 条件生成。

`IsLifeDrainable` 是 componentutil.lua 的全局函数：`(not target:HasAnyTag(NON_LIFEFORM_TARGET_TAGS) or target:HasTag("lifedrainable")) and not target:HasTag("nolifedrainable")`；`NON_LIFEFORM_TARGET_TAGS`＝{structure, wall, balloon, groundspike, smashable, deck_of_cards, nolife}，源码注释注明"Lifedrain (Batbat, mauler) uses this list"。[定义](../../../raw/current/scripts/componentutil.lua#L185-L188)、[表](../../../raw/current/scripts/componentutil.lua#L140-L148)

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 配方 | 洞穴蝙蝠翅膀（batwing）3＋活木 2＋紫宝石 1，`TECH.MAGIC_THREE`（＝{MAGIC=3}，暗影操控器 researchlab3 档）。[配方](../../../raw/current/scripts/recipes.lua#L481)、[TECH](../../../raw/current/scripts/constants.lua#L1250)、[科技树](../../../raw/current/scripts/tuning.lua#L1276-L1278)、[researchlab3](../../../raw/current/scripts/prefabs/magicprototyper.lua#L219) |
| 静态布局 | `ruins_camp` 放 1 个（`type = "batbat"`），注册键 `RuinsCamp`（全 scripts 未检索到按名消费端）；`skeleton_batfight` 放 1 个，注册在 `map/pointsofinterest.lua` 的 Cave POI 表，可被洞穴世界生成按区域随机选中（实际落点与频率未验证）。[ruins_camp 放置](../../../raw/current/scripts/map/static_layouts/ruins_camp.lua#L200-L205)、[RuinsCamp 注册](../../../raw/current/scripts/map/layouts.lua#L848)、[skeleton_batfight 放置](../../../raw/current/scripts/map/static_layouts/skeleton_batfight.lua#L83-L88)、[POI 注册](../../../raw/current/scripts/map/pointsofinterest.lua#L74-L77)、[随机抽取](../../../raw/current/scripts/worldgen_main.lua#L263-L280)、[生成入口](../../../raw/current/scripts/worldgen_main.lua#L351) |
| scenario 宝箱 | `chest_cavesupplies`（5%，耐久 33%–80%）、`chest_labyrinth`（5%，耐久 30%–50%）、`chest_labyrinth_mimic`（5%，还可能加 `itemmimic` 组件）；仅对应 scenario 启用时生效。[cavesupplies](../../../raw/current/scripts/scenarios/chest_cavesupplies.lua#L32-L34)、[labyrinth](../../../raw/current/scripts/scenarios/chest_labyrinth.lua#L46-L49)、[mimic](../../../raw/current/scripts/scenarios/chest_labyrinth_mimic.lua#L62-L70) |
| 数据依赖（非生成点） | `treasurechest.lua` 的 `pandora_scrapbook_adddeps` 收录 batbat，属图鉴依赖数据。[列表](../../../raw/current/scripts/prefabs/treasurechest.lua#L417-L435) |

配方过滤归 crafting 菜单 WEAPONS 与 MAGIC 两类。[WEAPONS](../../../raw/current/scripts/recipes_filter.lua#L1019-L1032)、[MAGIC](../../../raw/current/scripts/recipes_filter.lua#L1433-L1455)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(BATBAT_DAMAGE)`＋`onattack = onattack`（直接赋字段） | 主模拟；`OnAttack` 先回调后扣耐久；`attackwear` 缺省按 1 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/batbat.lua#L88-L90)、[回调](../../../raw/current/scripts/prefabs/batbat.lua#L35-L56) |
| 组件：finiteuses | 上限＝当前＝75；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/batbat.lua#L94-L98)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`、ARM_carry／ARM_normal 显隐、皮肤事件 | 主模拟；槽默认 HANDS；皮肤推 `equipskinneditem`＋`OverrideItemSkinSymbol` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/batbat.lua#L103-L105)、[onequip](../../../raw/current/scripts/prefabs/batbat.lua#L14-L24) |
| 组件：shadowlevel | `SetDefaultLevel(TUNING.BATBAT_SHADOW_LEVEL)`＝2 | 主模拟；pristine 加同名标签镜像 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/batbat.lua#L107-L108) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/batbat.lua#L100-L101) |
| 组件：floater | 公共端 `MakeInventoryFloatable("large", 0.05, {0.8, 0.35, 0.8}, true, -27, swap_data)` | 公共端；浮水换 `swap_batbat` 贴图 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/batbat.lua#L79-L80)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟时弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/batbat.lua#L110)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`IsLifeDrainable` | onattack 判定目标可否被汲命 | 命中结算时读取；同用者见 bat_boss.lua、shadow_battleaxe.lua、hats.lua（各自 grep 可证） | [定义](../../../raw/current/scripts/componentutil.lua#L185-L188)、[消费](../../../raw/current/scripts/prefabs/batbat.lua#L47) |
| API／数据：`dull` 标签 | `Combat:GetImpactSound` 按武器有无 `sharp` 选钝器命中音 | 命中穿甲结算时读取 | [[reference/components/combat#关键接口]]；[消费](../../../raw/current/scripts/components/combat.lua#L745) |
| 关联 Prefab：bat_bosscorpsehat（蝙蝠帽） | onattack 读 HEAD 槽装备的 `bathat` 标签切理智倍率（0.5→0.25），只影响理智不影响回血 | HEAD 槽装备时判定；快照中唯一加该标签的 prefab | [[reference/prefabs/hats]]（帽 FX 层）；[标签](../../../raw/current/scripts/prefabs/hats.lua#L6638)、[MakeHat 名映射](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7952) |
| 关联 Prefab：shadowwaxwell 系影像随从（Wiki 待建） | shadowlevel 消费端一：protector 形态伤害＝20＋4×领导者装备 shadowlevel 总和（batbat 装备态贡献 2）；消费端二：装备时按 level 播报（10 分钟同档抑制） | Maxwell 专属；只统计装备槽内物品 | [protector 增伤](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L145-L160)、[20＋4/级](../../../raw/current/scripts/tuning.lua#L2805-L2806)、[装备播报](../../../raw/current/scripts/prefabs/waxwell.lua#L230-L241) |
| 资源：anim／swap 动画包 | batbat.zip（bank＋build）／swap_batbat.zip（持握符号）；两 FX 共享 assets_bats：bat_tree_fx.zip＋PKGREF batbat_scythe.dyn | 静态声明 | [声明](../../../raw/current/scripts/prefabs/batbat.lua#L2-L12) |

`bathat` 标签另有 HUD 消费（bloodover.lua:73、healthbadge.lua:381，控血量徽章表现），归蝙蝠帽页维护：[bloodover](../../../raw/current/scripts/widgets/bloodover.lua#L73)、[healthbadge](../../../raw/current/scripts/widgets/healthbadge.lua#L381)。

### 皮肤（命中 FX 与接线）

皮肤接线与常见手持物一致：`GetSkinBuild()` 非空时装备／卸装推 `equipskinneditem`／`unequipskinneditem`，持握贴图用 `OverrideItemSkinSymbol`（[onequip](../../../raw/current/scripts/prefabs/batbat.lua#L14-L24)）。已注册 2 种皮肤；其 `fx_prefab` 在 CreatePrefabSkin 声明并由 prefabskin 写入全局 `SKIN_FX_PREFAB[皮肤名]`（[写入](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)），onattack 按当前皮肤名取第一个 FX 注册名在目标命中点上生成：

| 皮肤（官方简中名） | fx_prefab | 命中效果（已核验构造） | 注册证据 |
|---|---|---|---|
| 蝙蝠镰刀（batbat_scythe，Elegant／HALLOWED） | `batbat_bats` | 本地实体 bank `batbat_scythe`＋build `bat_tree_fx` 播 `bat1..bat4` 扇翅动画＋递减拍翼音，挂在目标命中符号上 | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1717-L1728)、[PlayBatFX](../../../raw/current/scripts/prefabs/batbat.lua#L122-L148) |
| 魔幻蝙蝠棒（batbat_fantasy，Woven／FANTASY） | `batbat_fantasy_fx` | 同构；bank `batbat_fantasy`＋`SetSkin("batbat_fantasy")` | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1704-L1715)、[PlayFantasyFX](../../../raw/current/scripts/prefabs/batbat.lua#L195-L221) |

变体机制（两 FX 共用模式）：服务端在武器实体上维护洗牌袋 `_batfxvariations`／`_fantasyfxvariations`（{1,2,3,4}），每次从袋首两个中随机取一个再放回袋尾，降低连续重复；代理实体 0.5 秒后删除，本地视觉实体播完动画（animover）自删，专用服务器不生成本地视觉。[OnBatFXSpawned](../../../raw/current/scripts/prefabs/batbat.lua#L150-L164)。皮肤显示名：[skin_strings.lua](../../../raw/current/scripts/skin_strings.lua#L555-L556)、[.po](../../../raw/current/scripts/languages/chinese_s.po#L391315-L391322)。

## 典型接入

### 修改蝙蝠棒的吸血与理智代价

- 前提与选择：服务端 `modmain.lua`；吸血与理智都在 onattack 回调内，`Weapon:SetOnAttack`（[weapon.lua#L53](../../../raw/current/scripts/components/weapon.lua#L53)）会整体替换默认回调，所以改其中一项必须重写整个回调。`IsLifeDrainable`、`SKIN_FX_PREFAB`、`TUNING.*` 都是游戏全局，modmain 可直接读。
- 生效与撤销：postinit 只影响新实例；onattack 回调不存档，已有实例下次加载即用新回调。停用模组后恢复原回调。
- 原版依据：原回调先皮肤 FX 后吸血；重写时保留皮肤块可避免皮肤命中特效丢失。[原回调](../../../raw/current/scripts/prefabs/batbat.lua#L35-L56)

```lua
-- modmain.lua：蝙蝠棒每击回血 10、理智代价固定 2（不受蝙蝠帽影响），保留皮肤命中 FX
AddPrefabPostInit("batbat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.weapon:SetOnAttack(function(inst, attacker, target)
        local skin_fx = GLOBAL.SKIN_FX_PREFAB[inst:GetSkinName()]
        if skin_fx ~= nil and skin_fx[1] ~= nil and target ~= nil
            and target.components.combat ~= nil and target:IsValid() then
            local fx = GLOBAL.SpawnPrefab(skin_fx[1])
            if fx ~= nil then
                fx.entity:SetParent(target.entity)
                fx.entity:AddFollower():FollowSymbol(target.GUID, target.components.combat.hiteffectsymbol, 0, 0, 0)
                if fx.OnFXSpawned ~= nil then
                    fx:OnFXSpawned(inst)
                end
            end
        end
        if attacker ~= nil and attacker.components.health ~= nil
            and attacker.components.health:IsHurt() and GLOBAL.IsLifeDrainable(target) then
            attacker.components.health:DoDelta(10, false, "batbat")
            if attacker.components.sanity ~= nil then
                attacker.components.sanity:DoDelta(-2)
            end
        end
    end)
end)
```

- 调整与验证：回血单位为生命值/击、理智为点/击。只调数值不必重写回调：吸血与理智倍率在每次命中时读 `TUNING`（运行时改 `TUNING.BATBAT_DRAIN` 等键立即影响所有实例）；伤害与耐久在实例构造时读表（改 `TUNING.BATBAT_DAMAGE`／`BATBAT_USES` 只对新实例生效，或 postinit 调 `SetDamage`／`SetMaxUses`）。

## 常见陷阱与限制

- **误区：满血也能吸血**：整个 drain 块要求 `owner.components.health:IsHurt()`，满血攻击既不回血也不扣理智；`BATBAT_DRAIN` 也不给目标扣血，是纯治疗。[条件](../../../raw/current/scripts/prefabs/batbat.lua#L47)
- **边界：对不可汲命目标无吸血**：目标带 structure／wall／balloon／groundspike／smashable／deck_of_cards／nolife 任一标签（除非另带 `lifedrainable`），或带 `nolifedrainable` 时不触发；建筑、墙、气球等常见刷耐久对象因此不回血。[判定](../../../raw/current/scripts/componentutil.lua#L185-L188)
- **边界：吸血先于磨损结算**：onattack 先于耐久结算执行，回调内读 `finiteuses` 拿到的是本次磨损前的耐久；把耐久打到 0 的最后一击仍会先回血，随后才 `usesdepleted`＋Remove。[顺序](../../../raw/current/scripts/components/weapon.lua#L105-L121)
- **边界：耐久耗尽即整个实体 Remove**：`SetOnFinished(inst.Remove)`，无掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/batbat.lua#L98)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：皮肤命中 FX 只在有皮肤时出现**：基础 batbat 命中无特效；皮肤 FX 还要求目标有 combat 组件且有效，无 combat 目标只吸血不出特效。[条件](../../../raw/current/scripts/prefabs/batbat.lua#L36-L46)
- **边界：影子等级无普适加成**：`shadowlevel` 在快照内只被 Maxwell 影像随从增伤与装备播报消费，普通角色装备无效果；不要把它当通用暗影伤害加成。[消费](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L154-L160)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 FX 与吸血门槛分离、skeleton_batfight POI 注册）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/batbat.lua 全文（三注册、onattack 两块、两个 FX 代理工厂、装备皮肤接线）、weapon／finiteuses／equippable／shadowlevel／health 关键方法、`IsLifeDrainable` 与 `NON_LIFEFORM_TARGET_TAGS`、`dull` 消费端（combat GetImpactSound）、shadowlevel 消费端（SGshadowwaxwell protector 增伤、waxwell 装备播报）、蝙蝠帽（bat_bosscorpsehat，MakeHat 名映射＋`bathat` 标签）、tuning 七键与 wilson_attack、配方与 recipes_filter 两分类、RuinsCamp 注册、三个 scenario 宝箱、treasurechest 图鉴依赖、两皮肤注册与 SKIN_FX_PREFAB 写入、floater／MakeHauntableLaunch helper；全 scripts grep "batbat" 复核其余命中（debugcommands、consolescreen 注释、misc_items 皮肤礼包数据、scrapbook 数据）均非生成点。
- 游戏运行：未运行。未覆盖：42.5 伤害与 6.8 吸血为源码推导（wilson_attack＝34），非运行测量；皮肤实装外观与变体随机观感、RuinsCamp／skeleton_batfight 布局是否实际进入世界生成、scenario 宝箱实机掉率均未验证。
- 结构检查：validate_page.py（0 error，153 行／21802 字节，standard 档）与 validate_source_links.py --strict（81 条源码引用、29 个源文件，0 error 0 warning）实际执行通过；lua 围栏经括号／块配对粗检。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
