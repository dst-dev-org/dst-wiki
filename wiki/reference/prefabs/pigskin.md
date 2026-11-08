---
title: "猪皮（pigskin）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "猪类掉落的皮革：猪人／守卫按 3:1 抽 1 件、疯猪与月光猪固定出表、猪王与精英猪无掉落；另含猪头回收、配方与 HORRIBLE 食性边界。"
aliases: ["pigskin", "猪皮", "Pig Skin", "猪人皮"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pigskin.lua", "prefabs/pigman.lua", "prefabs/pigking.lua", "prefabs/pigelite.lua", "prefabs/pigtorch.lua", "prefabs/gargoyles.lua", "prefabs/moonbase.lua", "components/moonbeastspawner.lua", "prefabs/hats.lua", "actions.lua", "prefabs/stickheads.lua", "prefabs/wanderingtrader.lua", "stategraphs/SGpig.lua", "stategraphs/SGwerepig.lua", "stategraphs/SGmoonpig.lua", "components/lootdropper.lua", "components/edible.lua", "components/eater.lua", "brains/pigbrain.lua", "components/tradable.lua", "cooking.lua", "containers.lua", "entityscript.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 猪皮（pigskin）

## 快速判断

- 职责与适用：`prefabs/pigskin.lua` 直接注册的皮革：40 叠、金值 1、可烧；另挂 `edible`（`HORRIBLE`，默认 10／10／0），图鉴另归为 food。[注册](../../../raw/current/scripts/prefabs/pigskin.lua#L47)、[挂件](../../../raw/current/scripts/prefabs/pigskin.lua#L27-L42)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1122)
- 优先使用：查掉落看[猪类掉落实况](#猪类掉落实况)；查配方看[配方与消费者](#配方与消费者)；查"能不能吃／能不能下锅"看[注册与构造](#注册与构造)。
- 重要副作用：普通猪人与守卫**权重 3:1 抽 1 件**（推导 25%）；疯猪／科学怪猪／月光猪改固定表；**猪王与精英猪无 `lootdropper`，不掉猪皮**。[普通表](../../../raw/current/scripts/prefabs/pigman.lua#L357-L360)、[固定表](../../../raw/current/scripts/prefabs/pigman.lua#L555-L575)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L672-L717)
- 修改边界：猪人／猪王／守卫／精英的完整行为归各自页面；雨伞与猪皮包见 [[reference/prefabs/umbrella]]、[[reference/prefabs/piggyback]]（S2）。[猪人注册](../../../raw/current/scripts/prefabs/pigman.lua#L1047-L1049)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pigskin` | 无包装工厂：`fn()` 直接构造；公共端（Transform／AnimState／Network／物理／动画／floater）→ `SetPristine` → 非主模拟返回 → 主模拟挂件 | 单注册、无皮肤、无 placer。[工厂](../../../raw/current/scripts/prefabs/pigskin.lua#L6-L45) |

- 公共端：bank／build 都是 `pigskin`、播 `idle`；`MakeInventoryFloatable(inst, "med", nil, 0.77)`＝尺寸 `med`、缩放 0.77。[外形](../../../raw/current/scripts/prefabs/pigskin.lua#L15-L19)、[参数](../../../raw/current/scripts/standardcomponents.lua#L1772-L1782)
- 主模拟：`stackable`（`STACK_SIZE_SMALLITEM`＝40）→ `inspectable` → `MakeSmallBurnable`（默认烧 10 秒）→ `MakeSmallPropagator` → `MakeHauntableLaunchAndIgnite`（点燃分支被注释，只弹飞）→ `tradable`（`goldvalue = GOLD_VALUES.MEAT`＝1）→ `inventoryitem` → `edible`（`foodtype = FOODTYPE.HORRIBLE`）。[挂件](../../../raw/current/scripts/prefabs/pigskin.lua#L27-L42)、[金值](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[助燃分支](../../../raw/current/scripts/standardcomponents.lua#L1204-L1213)
- **能吃，但玩家吃不了、也下不了锅**：默认三维 10／10／0、`HORRIBLE` 食性以及“玩家 OMNI 不含它”的完整口径以 [[reference/prefabs/cave-drops]] 为主章节，组件语义见 [[reference/components/edible]]、[[reference/components/eater]]；砂锅只收 `IsCookingIngredient` 表内条目，pigskin 不在其中。[默认值](../../../raw/current/scripts/components/edible.lua#L19-L24)、[砂锅](../../../raw/current/scripts/containers.lua#L373-L375)、[判定](../../../raw/current/scripts/cooking.lua#L185-L187)

### 名称核验

`STRINGS.NAMES.PIGSKIN`＝**猪皮**（Pig Skin）；本文件不改 prefab 名、不用 nameoverride（S1）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L346939-L346942)

## 猪类掉落实况

| 宿主／形态 | 掉落内容 | 判定与条件 |
|---|---|---|
| 猪人 `pigman`（普通态） | 1 件：`meat` 权重 3 ／ `pigskin` 权重 1 | `numrandomloot = 1`，按权重抽 1 件（推导：猪皮 1/4）；每次变回普通态都会重设。[接线](../../../raw/current/scripts/prefabs/pigman.lua#L357-L360)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 猪人守卫 `pigguard` | 同上 3:1 | `SetGuardPig` 设表；守卫由猪火炬的 spawner 生成。[接线](../../../raw/current/scripts/prefabs/pigman.lua#L484-L487)、[生成](../../../raw/current/scripts/prefabs/pigtorch.lua#L149) |
| 疯猪态（`werepig`，仍是 `pigman`） | 固定 `meat×2＋pigskin×1` | `SetWerePig` 在变身处调用，全掉。[表](../../../raw/current/scripts/prefabs/pigman.lua#L555-L575) |
| 科学怪猪态 `franken_pig`（仍是 `pigman`） | 固定 `monstermeat×2＋pigskin×1`（万圣节另加糖果） | 万圣节 `OnInit` 按 `HN_MONSTER_CHANCE` 抽中且未穿装备时转换。[表](../../../raw/current/scripts/prefabs/pigman.lua#L609)、[转换](../../../raw/current/scripts/prefabs/pigman.lua#L312-L321) |
| `moonpig`（猪人的月光形态，独立注册） | 固定 `meat×2＋pigskin×1` | 独立 `moon()` 工厂，死亡状态直接 `DropLoot`。[表](../../../raw/current/scripts/prefabs/pigman.lua#L1030-L1031)、[死亡](../../../raw/current/scripts/stategraphs/SGmoonpig.lua#L38-L52) |
| 石化疯猪（`gargoyle_werepig*`） | 默认只掉 `moonrocknugget`；月亮石充能后分两支：低耐久碎裂换成 `meat` 1＋`pigskin` 0.5＋`moonrocknugget` 0.5，高耐久复苏成 `moonpig`（掉 meat×2＋pigskin×1） | 默认表 `gargoyles_loot`；月亮石 `moonchargepre` 到点调 `moonbeastspawner:Start()`，再对范围内石像 `Reanimate`。[默认](../../../raw/current/scripts/prefabs/gargoyles.lua#L192-L193)、[启动](../../../raw/current/scripts/prefabs/moonbase.lua#L435-L446)、[范围](../../../raw/current/scripts/components/moonbeastspawner.lua#L192-L200)、[低耐久](../../../raw/current/scripts/prefabs/gargoyles.lua#L133-L147)、[高耐久](../../../raw/current/scripts/prefabs/gargoyles.lua#L97-L119)、[表](../../../raw/current/scripts/prefabs/gargoyles.lua#L13-L18) |
| 猪头 `pighead`（世界道具） | 锤击：`ceil(4×0.5)`＝2 猪皮＋2 树枝；拆解法杖：4＋4 | 掉落走 `droprecipeloot` 与 `DeconstructRecipe`。[挂接](../../../raw/current/scripts/prefabs/stickheads.lua#L126-L137)、[拆解表](../../../raw/current/scripts/recipes.lua#L1600)、[锤击比例](../../../raw/current/scripts/tuning.lua#L155) |
| 猪王 `pigking`／精英猪 `pigelite1`～`4` | **无** | 猪王只有 `pointofinterest`／`inspectable`／`trader`／`hauntable`／`minigame`，没有 `health` 与 `lootdropper`；精英猪组件段同样没有 `lootdropper`。[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L672-L717)、[精英](../../../raw/current/scripts/prefabs/pigelite.lua#L203-L254) |

- **死亡与燃烧**：猪人走 `SGpig` 的 `death` → `DropDeathLoot`，疯猪走 `SGwerepig` 的同名调用；若结算时仍在燃烧，`pigskin` 没有 `pigskin_cooked`／`cookedpigskin` 变体，会被替换成 `ash`，肉则换成 `*_cooked`。[猪人](../../../raw/current/scripts/stategraphs/SGpig.lua#L169-L182)、[疯猪](../../../raw/current/scripts/stategraphs/SGwerepig.lua#L47-L57)、[公共入口](../../../raw/current/scripts/entityscript.lua#L2180-L2192)、[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)

## 配方与消费者

材料与科技逐条核 `recipes.lua`。

- **防具与衣物**：雨伞 1（twigs 6＋silk 2，`TECH.SCIENCE_ONE`）；猪皮包 4（silk 6＋rope 2，`TECH.SCIENCE_TWO`）；橄榄球头盔 1（rope 1，`SCIENCE_TWO`）；鳞甲 3（dragon_scales 1＋armorwood 1，`SCIENCE_TWO`）；清凉夏装 2（rope 1＋feather_robin 3，`SCIENCE_ONE`）。[L525](../../../raw/current/scripts/recipes.lua#L525)、[L517](../../../raw/current/scripts/recipes.lua#L517)、[L504](../../../raw/current/scripts/recipes.lua#L504)、[L498](../../../raw/current/scripts/recipes.lua#L498)、[L550](../../../raw/current/scripts/recipes.lua#L550)
- **帽子**：时髦的护目镜 1（gold 1，`SCIENCE_ONE`）；沙漠护目镜 1（goggleshat 1，`TECH.LOST`）；刮地皮头盔 3（beefalowool 5＋townportaltalisman 1，`TECH.LOST`）。[L538](../../../raw/current/scripts/recipes.lua#L538)、[L539](../../../raw/current/scripts/recipes.lua#L539)、[L587](../../../raw/current/scripts/recipes.lua#L587)
- **武器、乐器与骑乘**：火腿棒 1（twigs 2＋meat 2，`SCIENCE_TWO`）；独奏乐器 2（gold 2＋nightmarefuel 4，`MAGIC_TWO`）；鞍具 4（beefalowool 4＋gold 4，`SCIENCE_TWO`）。[L480](../../../raw/current/scripts/recipes.lua#L480)、[L520](../../../raw/current/scripts/recipes.lua#L520)、[L682](../../../raw/current/scripts/recipes.lua#L682)
- **建筑**：猪屋 4（boards 4＋cutstone 3，`SCIENCE_TWO`）；皇家地毯 10（kelp 20＋beefalowool 15；拆解返 10 猪皮）。[L582](../../../raw/current/scripts/recipes.lua#L582)、[L1523](../../../raw/current/scripts/recipes.lua#L1523)、[L1638](../../../raw/current/scripts/recipes.lua#L1638)
- **弹弓与其它**：猪皮弹弓皮筋 1（`TECH.NONE`，`builder_skill="walter_slingshot_modding"`）；弹药袋 4（silk 6＋gold 2，`TECH.NONE`，`builder_skill="walter_ammo_bag"`）；战士变身涂鸦 1（monstermeat 2＋silk 1，`TECH.SPIDERCRAFT_ONE`）。[L235](../../../raw/current/scripts/recipes.lua#L235)、[L250](../../../raw/current/scripts/recipes.lua#L250)、[L329](../../../raw/current/scripts/recipes.lua#L329)
- **喂给会吃 HORRIBLE 的目标**：猪人吃猪皮进背包并加 192 秒忠诚（守卫除外，随后被它吃掉）；眼面具（`eater` OMNI＋HORRIBLE、`fedbyall`）按 4.0／1.75 吸收修甲，1 张＝10×4.0＋10×1.75＝57.5 点（推导；帽体见 [[reference/prefabs/hats-armor]]）。[猪人接受](../../../raw/current/scripts/prefabs/pigman.lua#L62-L106)、[常量](../../../raw/current/scripts/tuning.lua#L446)、[进食](../../../raw/current/scripts/brains/pigbrain.lua#L86-L89)、[眼面具](../../../raw/current/scripts/prefabs/hats.lua#L2321-L2358)、[回调](../../../raw/current/scripts/prefabs/hats.lua#L2329-L2340)、[FEED](../../../raw/current/scripts/actions.lua#L3768-L3791)
- **流浪商人**：3 beefalowool → 1 猪皮（`product="pigskin"`）；反向收 3 猪皮 → gears／红宝石／蓝宝石。[猪皮标价](../../../raw/current/scripts/recipes.lua#L1002)、[回收价](../../../raw/current/scripts/recipes.lua#L996-L1006)、[货单](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L22-L40)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | `foodtype = HORRIBLE` | 主模拟；玩家 OMNI 不含 HORRIBLE | [[reference/components/edible]]；[挂载](../../../raw/current/scripts/prefabs/pigskin.lua#L41-L42) |
| 组件：`tradable` | `goldvalue = 1`：猪王收 1 猪皮给 1 金块 | 主模拟；猪王 `AcceptTest` 读 `goldvalue > 0` | [[reference/components/tradable]]；[挂载](../../../raw/current/scripts/prefabs/pigskin.lua#L36-L37)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L572-L606) |
| 组件：`burnable`／`propagator` | `MakeSmallBurnable`（10 秒）＋`MakeSmallPropagator` | 主模拟 | [[reference/components/burnable]]、[[reference/components/propagator]]；[调用](../../../raw/current/scripts/prefabs/pigskin.lua#L32-L33) |
| 关联 Prefab：`pigman`／`pigguard`／`moonpig` | 掉落宿主（同一文件三注册） | 击杀结算 | [注册](../../../raw/current/scripts/prefabs/pigman.lua#L1047-L1049) |
| 关联 Prefab：`pigking` | 消费者（金块兑换），不掉猪皮 | 交易时 | [[reference/components/trader]]；[接受](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606) |
| 关联 Prefab：`pighead` | 回收来源：锤击 2／拆解 4 | 世界道具 | [注册](../../../raw/current/scripts/prefabs/stickheads.lua#L179) |

## 典型接入

### 让猪王不再收猪皮

- 前提与选择：服务端 `modmain.lua`。猪王的接受判定与付款都现读 `tradable.goldvalue`，改成 0 即退出"可换金块"。
- 生效与撤销：`AddPrefabPostInit("pigskin", ...)` 对之后生成的实例生效，读档重建时也会重跑；卸载模组后新实例恢复 1。
- 原版依据：`AcceptTest` 返回 `goldvalue > 0`，付款按 `for k = 1, goldvalue` 循环。[接受](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[付款](../../../raw/current/scripts/prefabs/pigking.lua#L74-L78)

```lua
-- modmain.lua：猪皮不再能换金块（猪王拒绝收）
AddPrefabPostInit("pigskin", function(inst)
    if inst.components.tradable ~= nil then
        inst.components.tradable.goldvalue = 0
    end
end)
```

- 调整与验证：金值是整数件数，0 即"不可交易"；只改猪皮。未运行，只核对了读取点。

## 常见陷阱与限制

- **误区：图鉴标 food 就能吃。** 图鉴确实写 `hungervalue=10`、`healthvalue=10`、`foodtype="HORRIBLE"`，但玩家的 OMNI 食组不含 HORRIBLE，EAT 候选不生成；这是"有 edible 但玩家吃不了"的边界。[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1122)、[候选](../../../raw/current/scripts/componentactions.lua#L2672-L2688)
- **边界：不能下锅。** 砂锅只收 `IsCookingIngredient` 的表内条目，pigskin 不在其中。[砂锅](../../../raw/current/scripts/containers.lua#L373-L375)
- **误区：猪王／精英猪也掉猪皮。** 两者都没有 `lootdropper`；猪王连 `health` 都没有。猪王只收猪皮（1 金块），不掉猪皮。[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L672-L717)、[精英](../../../raw/current/scripts/prefabs/pigelite.lua#L203-L254)
- **边界：普通猪只掉 1 件。** `numrandomloot = 1` 且权重 3:1，所以猪皮与肉不会同时掉；只有疯猪／科学怪猪／月光猪是固定多件。[普通表](../../../raw/current/scripts/prefabs/pigman.lua#L357-L360)
- **边界：烧死猪人不掉猪皮。** 燃烧中死亡时 `pigskin` 被替换成 `ash`。[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)
- **未核验**：未运行游戏。25%、猪头锤击 2 张与复苏路径为源码推导，未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `pigskin.lua` 全文；`pigman.lua` 的六段工厂与三处注册；`pigking.lua` 组件、接受与付款；`pigelite.lua` 组件段；`gargoyles.lua` 的默认表与 `brokenwerepig_loot`；`stickheads.lua` 的猪头；`wanderingtrader.lua` 货单；三套死亡状态与 `DropDeathLoot`；`lootdropper` 的加权与燃烧转换；`edible`／`eater`／`cooking`／`containers` 的食用与下锅边界；`recipes.lua` 全部含 pigskin 的配方；`tuning` 金值与锤击比例；`chinese_s.po` 名称。
- 游戏运行：未运行。概率、件数与拆解数量均为源码推导（`total_day_time`＝480 秒）。
- 复审修订：2026-11-10，R1：P2-4 石化分支、P3-5 HORRIBLE 口径＋眼面具、P3-10 图鉴因果。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 已有消费者页：[[reference/prefabs/umbrella]]、[[reference/prefabs/piggyback]]、[[reference/prefabs/hambat]]、[[reference/prefabs/hats-environment]]、[[reference/prefabs/hats-armor]]、[[reference/prefabs/clothing]]、[[reference/prefabs/armor-boss]]；组件：[[reference/components/lootdropper]]、[[reference/components/edible]]、[[reference/components/tradable]]、[[reference/components/eater]]

返回 [[reference/prefabs/index|Prefab 索引]]。
