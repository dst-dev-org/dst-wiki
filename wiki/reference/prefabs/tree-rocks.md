---
title: "巨石枝（tree_rock 系列七注册）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "tree_rocks.lua 的七个注册：tree_rock 随机变体、tree_rock1／tree_rock2 两套 build 与各自的 _short／_normal 显式阶段；growable 两阶段与春季加速、tree_rock_seed 种下后的 growfromseed、plantregrowth 的春季扩散与 skipCanPlantCheck、5 次 CHOP 换 1 根树枝与 5 件藤蔓战利品、砍倒后的石块化与 6 次 MINE、燃烧 3 秒转石块、雪覆盖与月雹。"
aliases: ["tree_rock", "tree_rock1", "tree_rock2", "tree_rock1_short", "tree_rock1_normal", "tree_rock2_short", "tree_rock2_normal", "巨石枝", "巨石枝树", "巨石枝树苗"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/tree_rocks.lua", "prefabs/tree_rock_data.lua", "prefabs/tree_rock_seed.lua", "prefabs/planted_tree.lua", "prefabs/books.lua", "components/lootdropper.lua", "components/growable.lua", "components/plantregrowth.lua", "map/rooms/cave/vents.lua", "standardcomponents.lua", "simutil.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 巨石枝（tree_rock 系列七注册）

## 快速判断

- 职责与适用：`tree_rocks.lua` 的 `MakeRockTree` 用同一套 `fn` 产出七个注册——两套 build（`rock1`／`rock2`）各带 `tree_rock1`／`tree_rock2` 随机阶段、`tree_rock1_short`／`_normal`、`tree_rock2_short`／`_normal` 显式阶段，再加一个完全随机的 `tree_rock`。它是「先当树砍、砍倒变成可挖石块」的两段式资源。[注册段](../../../raw/current/scripts/prefabs/tree_rocks.lua#L799-L808)
- 优先使用：改阶段与再生看[生长与再生](#生长与再生)；改砍伐、藤蔓与采查看[砍伐、藤蔓与采掘](#砍伐藤蔓与采掘)；改燃烧、雪与月雹看[燃烧、雪与月雹](#燃烧雪与月雹)。种子与放置器见 [[reference/prefabs/acorn#种植放置预览与生长]]，本页只写树。
- 重要副作用：砍倒（CHOP 5 次）只给 1 根 `twigs` 与最多 5 件藤蔓战利品（抽到 `EMPTY` 的位置为空），树本身**变成石块**（`boulder`）：换 `ACTIONS.MINE`、6 次、战利品表换成 `tree_rock1_mine`；石头与 1 个 `tree_rock_seed` 都在石块阶段结算——正常 MINE 耗尽走 `tree_rock1_mine`，倒下砸到 `tree_rock_breaker` 时改走 `tree_rock1_mine_break`，两表都必给种子；烧毁态的石块会让种子按 `BURNED_LOOT_OVERRIDES` 转成 `ash`。[砍倒](../../../raw/current/scripts/prefabs/tree_rocks.lua#L633-L650)、[石块化](../../../raw/current/scripts/prefabs/tree_rocks.lua#L424-L455)、[崩落分支](../../../raw/current/scripts/prefabs/tree_rocks.lua#L548-L552)
- 修改边界：`tree_rock_seed` 的物品数值、放置器与两段部署校验见 [[reference/prefabs/acorn]]；本页只写树本身（阶段、再生、砍伐、燃烧）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tree_rock1` | `MakeRockTree("tree_rock1","rock1")`；`stage` 传 `nil` → `math.random(1, 2)` | bank `tree_rock`、build `tree_rock_normal`、`prefab_name = "tree_rock1"`；倒下砸击半径 2、伤害 200。[rock1](../../../raw/current/scripts/prefabs/tree_rocks.lua#L155-L166)、[随机阶段](../../../raw/current/scripts/prefabs/tree_rocks.lua#L754-L756) |
| `tree_rock2` | 同上，`MakeRockTree("tree_rock2","rock2")` | bank `tree_rock2`、`SetBuild` 仍是 `tree_rock_normal`、`prefab_name = "tree_rock2"`；倒下砸击半径 3.25、伤害 100。[rock2](../../../raw/current/scripts/prefabs/tree_rocks.lua#L167-L178) |
| `tree_rock1_short` | `stage = 1` | 强制 `short_anims`、月雹小档；CHOP 与掉落同 build。[阶段表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L318-L335) |
| `tree_rock1_normal` | `stage = 2` | 强制 `normal_anims`、月雹中档 |
| `tree_rock2_short` | `stage = 1` | 与 `tree_rock1_short` 同阶段，只是 bank 与砸击参数换成 rock2 |
| `tree_rock2_normal` | `stage = 2` | 同上 |
| `tree_rock` | 不传 build、不传 stage | build 用 `GetRandomKey(builds)`、阶段再 `math.random(1, 2)`，即四个组合等概率；世界生成用的默认名。[随机变体](../../../raw/current/scripts/prefabs/tree_rocks.lua#L715-L721) |

### 共同的构造与命名

- 资源是四套动画包 `tree_rock_short`／`tree_rock_normal` 与 `tree_rock2_short`／`tree_rock2_normal`；两套 `builds` 的 `build` 都取 `tree_rock_normal`，`bank` 才是 `tree_rock`／`tree_rock2`。数据脚本 `tree_rock_data.lua` 与音效包 `rifts6` 一起声明。[assets](../../../raw/current/scripts/prefabs/tree_rocks.lua#L1-L15)
- 公共端：`MakeObstaclePhysics(0.25)`、`SetBuild(GetBuild(inst).build)`、`SetBank(GetBuild(inst).bank)`、tags `tree`／`rock_tree`／`shelter`／`nodangermusic`，并 `MakeSnowCoveredPristine`。注意它**没有** `plant` tag。[构造](../../../raw/current/scripts/prefabs/tree_rocks.lua#L711-L733)
- 命名：先 `SetPrefabName(GetBuild(inst).prefab_name)`（于是 `inst.prefab` 变成 `tree_rock1`／`tree_rock2`，供 `plantregrowth` 取倍率），再 `SetPrefabNameOverride("tree_rock")`，最终显示名统一取 `STRINGS.NAMES.TREE_ROCK`＝巨石枝。[命名](../../../raw/current/scripts/prefabs/tree_rocks.lua#L720-L722)、[显示名回落](../../../raw/current/scripts/entityscript.lua#L687-L692)
- 同一段还 `AddTag(GetBuild(inst).prefab_name)`，注释写明「used by regrowth」——它与 `plantregrowth` 的 `searchTag = "rock_tree"` 是两套 tag，别混用。[tag](../../../raw/current/scripts/prefabs/tree_rocks.lua#L722)、[再生搜索](../../../raw/current/scripts/prefabs/tree_rocks.lua#L762-L766)
- 端分界：`SetPristine()` 后 `if not TheWorld.ismastersim then return inst end`；`lootdropper`／`workable`／`growable`／`plantregrowth`／燃烧全在主模拟。[分界](../../../raw/current/scripts/prefabs/tree_rocks.lua#L733-L737)
- 主模拟组件顺序：`lootdropper`（`tree_rock1_chop`）→`workable`（CHOP）→`growable`→`plantregrowth`→`inspectable`（`boulder` 时状态 `CHOPPED`）→燃烧／蔓延→`MakeSnowCovered`＋月雹小档→`SetupVineLoot`→`MakeHauntableWork`。[组件段](../../../raw/current/scripts/prefabs/tree_rocks.lua#L745-L791)

## 生长与再生

种下链：`tree_rock_seed` → `tree_rock_sapling`（显式 `TREE_ROCK.SAPLING_GROW_TIME`＝4 天±1 天）→ 计时结束 `SpawnPrefab` 一个 `tree_rock1_short`／`tree_rock2_short` 再 `growfromseed()`，由 `handler_growfromseed` 定到 `stage 1` 并播 `grow_seed_to_short`。[树苗注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L190)、[长树](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L80)、[growfromseed](../../../raw/current/scripts/prefabs/tree_rocks.lua#L693-L698)、[部署校验](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L24-L38)

| 阶段 | 时长（`TREE_ROCK.GROW_TIME`） | 进入时做什么 |
|---|---|---|
| `short` | 1.5×`day_time`±0.5×`day_time`＝450±150 秒（300–600） | `SetShort`：`short_anims`、月雹小档、`Sway` |
| `normal` | 5×`day_time`±2×`day_time`＝1500±600 秒（900–2100） | `SetNormal`：`normal_anims`、月雹中档、`Sway` |

- 只有两段：`growth_stages[build]` 里 `tall`／`old` 两段被整段注释掉，长到 `normal` 就停在最后一个阶段（`loopstages = false`）。[阶段表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L318-L350)、[配置](../../../raw/current/scripts/prefabs/tree_rocks.lua#L754-L760)、[推进](../../../raw/current/scripts/components/growable.lua#L121-L151)
- `springgrowth = true`：每段时长在春天乘 `SPRING_GROWTH_MODIFIER`＝0.75。[SpringGrowthMod](../../../raw/current/scripts/simutil.lua#L563-L565)、[常量](../../../raw/current/scripts/tuning.lua#L2284)
- `magicgrowable = true` 但**没有** `domagicgrowthfn`，也没有 `simplemagicgrower`：园艺书类道具走到 `growable:DoGrowth()` 这条兜底分支，效果是 `DoGrowth()` 推进一步（已在末段时只是重进 `normal`，完全停下后返回 false），而不是「一口气长满」。[书的分支](../../../raw/current/scripts/prefabs/books.lua#L66-L81)、[标记](../../../raw/current/scripts/prefabs/tree_rocks.lua#L759)
- `growable` 与 `plantregrowth` 是两条独立链：前者管自身长大，后者管在附近**生成新的树苗**。再生参数：`SetRegrowthRate(TREE_ROCK_REGROWTH.OFFSPRING_TIME)`＝8 天、`SetProduct("tree_rock_sapling")`、`SetSearchTag("rock_tree")`、`SetSkipCanPlantCheck(true)`。[接线](../../../raw/current/scripts/prefabs/tree_rocks.lua#L762-L766)、[常量](../../../raw/current/scripts/tuning.lua#L8930-L8934)
- `skipCanPlantCheck` 的后果：`GetSpawnPoint` 里跳过 `map:CanPlantAtPoint`（`GROUND_HARD` 检查），但仍要求 `CanPlacePrefabFilteredAtPoint`、不在路上（`RoadManager:IsOnRoad`）、落点 3 格内无实体、8 格内无 `structure`／`wall`，且落点五半径内同名 tag 实体少于 5 个——所以巨石枝能扩散到岩地与路边，却挤不进建筑区。[GetSpawnPoint](../../../raw/current/scripts/components/plantregrowth.lua#L183-L206)、[密度检查](../../../raw/current/scripts/components/plantregrowth.lua#L229-L241)
- 再生计时按 `inst.prefab` 累加：`tree_rock1`／`tree_rock2` 的倍率都是 `TREE_ROCK_REGROWTH_TIME_MULT`＝1，全年无季节折扣；`tree_rock` 因 `SetPrefabName` 已落到具体 build 名，不会漏配。[倍率表](../../../raw/current/scripts/components/plantregrowth.lua#L54-L60)、[常量](../../../raw/current/scripts/tuning.lua#L8936)

## 砍伐、藤蔓与采掘

### 砍倒前：CHOP 5 次

- `workable` 用 `ACTIONS.CHOP`、`SetWorkLeft(TUNING.TREE_ROCK.CHOP)`＝5、回调 `OnChop`，完成回调 `OnChopDown`；战利品表在构造时设为 `tree_rock1_chop`。[工作配置](../../../raw/current/scripts/prefabs/tree_rocks.lua#L748-L752)、[表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L36-L39)
- 每斧：播 `chop_<阶段>`、推 `sway`、在树身随机高度生成 `tree_rock_chop` 碎屑，并唤醒半径 `LEIF_REAWAKEN_RADIUS`＝20 内睡着的 `leif`。[OnChop](../../../raw/current/scripts/prefabs/tree_rocks.lua#L594-L613)
- `tree_rock1_chop` 只有 `twigs`×1（1.00）：砍倒必得 1 根树枝，**没有** `log`。[战利品表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L36-L39)

### 砍倒：藤蔓战利品与砸击

- `OnChopDown` 播 `fall_pre`、`DropLoot`（那根 `twigs`）、`SpawnVineLoot`，然后立刻 `MakeRock(inst, true)` 转成石块；真正的砸击伤害在 `animover` 里结算。[OnChopDown](../../../raw/current/scripts/prefabs/tree_rocks.lua#L633-L650)
- `OnAnimOver` 三条分支：带 `tree_rock_breaker` tag → `fall_break`，战利品表换成 `tree_rock1_mine_break` 并 `persists = false` 后删除；带 `tree_rock_bouncer` tag 或戴 `hardarmor` 头盔 → `fall_bounce`，11 帧后砸地；其余 → `fall_miss`，4 帧后砸地。[OnAnimOver](../../../raw/current/scripts/prefabs/tree_rocks.lua#L535-L568)、[判定](../../../raw/current/scripts/prefabs/tree_rocks.lua#L504-L527)
- 砸地 `OnRockFall`：对范围内带 `_combat` 的目标按 `PlayerDamageMod(v, damage, PLAYERDAMAGEPERCENT = 1/4)` 结算并推 `knockback`；rock1 半径 2／200 伤害，rock2 半径 3.25／100 伤害。[OnRockFall](../../../raw/current/scripts/prefabs/tree_rocks.lua#L483-L502)、[参数](../../../raw/current/scripts/tuning.lua#L8902-L8912)

### 藤蔓战利品（`SetupVineLoot`）

- 时机：构造时 `DoTaskInTime(0, SetupVineLoot)` 先决定 5 件「挂着的东西」；已有 `vine_loot` 或已经是 `boulder` 时直接返回（读档时由 `OnLoad` 恢复同一批）。[调用](../../../raw/current/scripts/prefabs/tree_rocks.lua#L789)、[SetupVineLoot](../../../raw/current/scripts/prefabs/tree_rocks.lua#L130-L147)
- 抽样：`GetVineLoots` 用 `weighted_random_choices` 抽 `NUM_VINE_LOOT`＝5 件；权重表按脚下地块的拓扑来源（layout → room → task）映射到区域表，映射不到时回落到 `DEFAULT`（`rocks` 20 与各色宝石 0.02–0.5）。[抽样](../../../raw/current/scripts/prefabs/tree_rocks.lua#L102-L128)、[默认表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L1-L10)
- 加权修正：`WEB_CREEP`（蛛网地皮）追加 `silk`＝总权重×0.25；`CAVE` 追加 `wormlight`＝总权重×巢穴等级×0.006；`VENT_AREA` 在暗影裂隙激活时切到 `VENT_AREA_SHADOW_RIFT` 表。[修正表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L266-L290)、[区域改写](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L488-L503)
- 外观：每件按 `VINE_LOOT_DATA` 的 `build`／`symbol` 覆盖 `swap_gem_i`，抽到 `EMPTY` 就 `Hide` 该符号——「树上挂什么」在砍倒前就能看出来。[外观](../../../raw/current/scripts/prefabs/tree_rocks.lua#L137-L146)、[映射表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L234-L260)
- 兑现：`SpawnVineLoot` 先把掉落速度调到 2–4，再逐个 `SpawnLootPrefab`（跳过 `EMPTY`），最后清空字段；读档只保存 `vine_loot` 数组，不保存已生成的实体。[SpawnVineLoot](../../../raw/current/scripts/prefabs/tree_rocks.lua#L615-L631)、[存档](../../../raw/current/scripts/prefabs/tree_rocks.lua#L655-L686)

### 砍倒后：石块 MINE 6 次

- `MakeRock` 是「树 → 石块」的唯一入口：熄火、摘 `burnable`／`propagator`／`workable`／`shelter`，把 `hauntable` 换成 `MakeHauntableWork`，加 `boulder` tag、去 `tree` tag，`growable:StopGrowing()`，战利品表改 `SetChanceLootTable("tree_rock1_mine")`，再挂 `ACTIONS.MINE`、`SetWorkLeft(TUNING.TREE_ROCK.MINE)`＝6、`savestate = true`；物理只有在 `no_change_physics` 为假时才 `RemovePhysicsColliders`＋`ChangeToObstaclePhysics(inst, 1)`。[MakeRock](../../../raw/current/scripts/prefabs/tree_rocks.lua#L424-L455)
- 石块阶段的净次序是「`boulder` 状态 → workable／采掘表替换 → 碰撞变化 → 掉落结算」：`OnChopDown` 先 `DropLoot`（仍是 `tree_rock1_chop` 的 1 根 `twigs`）并 `SpawnVineLoot`，再 `MakeRock(inst, true)` 完成状态与组件替换但**暂不改碰撞**；倒地动画结束、`OnRockFall` 才 `ChangeToObstaclePhysics(inst, 1)`，石块掉落要等 MINE 耗尽（或崩落分支）才结算；崩落分支不改碰撞，`DropLoot` 后等 animover 删除。所以砍倒瞬间还不是「已换好碰撞体的石块」，砍倒表也不含石头与种子。[OnChopDown](../../../raw/current/scripts/prefabs/tree_rocks.lua#L633-L650)、[OnRockFall](../../../raw/current/scripts/prefabs/tree_rocks.lua#L483-L502)
- 挖掘进度决定外观：`PlayRockAnimation` 按 `MINE` 的 2/3 与 1/3 两个阈值切 `fall_full_normal`／`fall_med_normal`／`fall_low_normal`（烧过的用 `burnt_*_normal`）。[动画阈值](../../../raw/current/scripts/prefabs/tree_rocks.lua#L360-L378)、[常量](../../../raw/current/scripts/tuning.lua#L8898-L8900)
- 挖完：生成 `rock_break_fx`、`DropLoot`、`Remove`；`tree_rock1_mine` 必给 3 个 `rocks`＋1 `nitre`＋1 `flint`，另有 0.25 `nitre`／0.6 `flint` 追加，以及 1 个 `tree_rock_seed`。[OnMine](../../../raw/current/scripts/prefabs/tree_rocks.lua#L397-L418)、[表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L41-L52)
- 两套 build 共用 `tree_rock1_mine` 表（名字带 1，但 rock2 也用），按 build 区分的只有倒塌砸击参数与 bank。[共用表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L447)
- `boulder` 是存档字段：`OnSave` 存 `boulder`／`burnt`／`vine_loot`，`OnLoad` 据此重建石块或恢复藤蔓，并保留 `workleft`。[存档](../../../raw/current/scripts/prefabs/tree_rocks.lua#L655-L686)
- 世界生成也会直接给出「已是石块」的巨石枝：洞穴喷口房间按 `TREE_ROCK.BOULDER_GEN_CHANCE`＝1/3 掷骰，命中即写 `boulder = true`，再按 roll 决定 `workleft`：抽到高段给 `MINE_MED`＝4、中段给 `MINE_LOW`＝2，未写 `workleft` 的分支沿用构造时的 5（砍倒后新建的采掘组件才是 6）。[RandomRockTreeState](../../../raw/current/scripts/map/rooms/cave/vents.lua#L19-L30)

## 燃烧、雪与月雹

- 燃烧：`MakeMediumBurnable(inst, TUNING.TREE_ROCK.BURN_TIME)`＝3 秒、`SetFXLevel(5)`、完成回调 `OnBurnt`、`MakeSmallPropagator`；3 秒是**燃烧时长**，不是烧毁后的存活时间。[燃烧](../../../raw/current/scripts/prefabs/tree_rocks.lua#L781-L784)、[常量](../../../raw/current/scripts/tuning.lua#L8910)
- 烧毁：`OnBurnt` 加 `burnt` tag，播 `fall_pre_burnt`，同时挂 `animover` 与从 `OnBurnt` 起算的 0.5 秒 `MakeRock`（两者并列，不等动画）；这次调用不带 `no_change_physics`，所以 0.5 秒时就换成可 MINE 的 `boulder` 并改碰撞，外观换 `burnt_*`，战利品仍是 `tree_rock1_mine`。[OnBurnt](../../../raw/current/scripts/prefabs/tree_rocks.lua#L570-L587)
- 雪覆盖：`MakeSnowCoveredPristine` 预插并隐藏 `snow` 符号，`MakeSnowCovered` 按 `TheWorld.state.issnowcovered` 显隐，同时给主模拟挂 `lunarhailbuildup`。[雪](../../../raw/current/scripts/standardcomponents.lua#L835-L855)
- 月雹：巨石枝固定小档 `SetLunarHailBuildupAmountSmall`——总工作量 2、月玻璃 1；阶段切换会重设容量（小／中档），但不会重置已积累的量。[档位](../../../raw/current/scripts/prefabs/tree_rocks.lua#L787)、[常量](../../../raw/current/scripts/tuning.lua#L7911-L7912)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`growable` | 两阶段表、`loopstages = false`、`springgrowth`、`magicgrowable`（无 `domagicgrowthfn`） | 主模拟；砍倒后 `StopGrowing` | [[reference/components/growable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/tree_rocks.lua#L754-L760) |
| 组件：`plantregrowth` | 8 天一次在附近生 `tree_rock_sapling`，搜索 tag `rock_tree`、跳过 `CanPlantAtPoint` | 主模拟；按 `inst.prefab` 取倍率 | [[reference/components/plantregrowth#关键接口]]；[接线](../../../raw/current/scripts/prefabs/tree_rocks.lua#L762-L766) |
| 组件：`workable` | 树形态 `CHOP` 5 次，石形态 `MINE` 6 次且 `savestate` | 主模拟；`MakeRock` 会重建组件 | [[reference/components/workable#关键接口]]；[两次挂载](../../../raw/current/scripts/prefabs/tree_rocks.lua#L748-L752) |
| 组件：`lootdropper` | `tree_rock1_chop` → 砍倒时换成 `tree_rock1_mine`；崩落分支切 `tree_rock1_mine_break` | 主模拟；按状态换表 | [[reference/components/lootdropper#关键接口]]；[换表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L447)、[崩落](../../../raw/current/scripts/prefabs/tree_rocks.lua#L548) |
| 组件：`lunarhailbuildup` | 月雹积累，固定小档 2／1 | 主模拟；`MakeRock` 不摘该组件 | [档位](../../../raw/current/scripts/standardcomponents.lua#L901-L906) |
| 组件：`burnable`／`propagator`／`hauntable` | 3 秒中火＋小档蔓延；作祟只挂 `MakeHauntableWork` 的冷却与失败回调，`WorkedBy` 分支整段被注释、恒返回 false，当前 build 作祟不推进 CHOP／MINE | 主模拟；树态与石块态都挂了 `hauntable`，但作祟无工作效果 | [燃烧](../../../raw/current/scripts/prefabs/tree_rocks.lua#L781-L784)、[树态挂载](../../../raw/current/scripts/prefabs/tree_rocks.lua#L791)、[石块态挂载](../../../raw/current/scripts/prefabs/tree_rocks.lua#L432-L433)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1107-L1122) |
| 关联 Prefab：`tree_rock_sapling` | 种下产物，长大成 `tree_rock1_short`／`tree_rock2_short` | 主模拟；`SAPLING_GROW_TIME` 4 天±1 天 | [注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L190) |
| 关联 Prefab：`tree_rock_seed` | 种子来源：正常 MINE 耗尽掉 `tree_rock1_mine`，崩落破坏掉 `tree_rock1_mine_break`，两表都必给 1 个（烧毁态会转 `ash`）；也可种 | 见 [[reference/prefabs/acorn#获得路径]] | [两表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L41-L65) |
| 数据：`tree_rock_data` | 藤蔓权重表、区域映射与额外修正 | 只读；按世界拓扑查询 | [返回表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L505-L515) |
| 资源：anim | `tree_rock_short`／`normal`、`tree_rock2_short`／`normal` 四套动画包（`SetBuild` 只用 `tree_rock_normal`） | 七注册共用 | [assets](../../../raw/current/scripts/prefabs/tree_rocks.lua#L1-L15) |

## 典型接入

### 把巨石枝的砍倒产物换成 2 根树枝

- 前提与选择：服务端 `modmain.lua`，改**共享战利品表** `LootTables.tree_rock1_chop` 而不是实例字段。实例级 `SetLoot` 会一直留在 `lootdropper.loot` 上，等 `MakeRock` 把树变成石块后，MINE 完还会再掉一次那些树枝。
- 生效与撤销：`LootTables` 是 `components/lootdropper.lua` 加载时建立的全局表，`GenerateLoot` 每次按名字现查，所以改一次就对新旧实例都生效；因为无法确定组件文件与 modmain 的先后，这里把赋值放进 postinit，第一次生成巨石枝时才写。撤模组后表随进程重建回到原值。
- 原版依据：砍倒时 `OnChopDown` 先 `DropLoot`（读 `chanceloottable = "tree_rock1_chop"`），再 `SpawnVineLoot`；随后 `MakeRock` 才把表换成 `tree_rock1_mine`。[OnChopDown](../../../raw/current/scripts/prefabs/tree_rocks.lua#L633-L650)、[换表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L447)、[表注册](../../../raw/current/scripts/components/lootdropper.lua#L21-L24)

```lua
-- modmain.lua：巨石枝砍倒给 2 根树枝（藤蔓与石块战利品不变）
local ROCK_TREES = {
    "tree_rock", "tree_rock1", "tree_rock2",
    "tree_rock1_short", "tree_rock1_normal",
    "tree_rock2_short", "tree_rock2_normal",
}

local function BuffChopLoot(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    GLOBAL.LootTables.tree_rock1_chop = {
        { "twigs", 1 },
        { "twigs", 1 },
    }
end

for _, name in ipairs(ROCK_TREES) do
    AddPrefabPostInit(name, BuffChopLoot)
end
```

- 调整与验证：表项格式是 `{ prefab, chance }`，两条 `chance = 1` 就是必掉两根。七个注册共用这张表，所以写一次即可；重复挂 postinit 只是保证在任一变体首次生成时都已生效。验证时砍倒看地面掉落数，并确认石块形态挖完仍是 `rocks`／`nitre`／`flint`＋`tree_rock_seed`。

## 常见陷阱与限制

- **误区：砍倒就给石头。** `tree_rock1_chop` 只有 1 根 `twigs`，`rocks`／`nitre`／`flint` 与种子都要等石块阶段；种子有两条并列且都必给的路径——正常 MINE 耗尽走 `tree_rock1_mine`（种子 1.00），倒下砸到 `tree_rock_breaker` 时 `OnAnimOver` 先把表换成 `tree_rock1_mine_break`（种子同样 1.00）再 `DropLoot`，随后 `persists = false` 并在下一个 animover 移除。崩落表的石头更少（`rocks` 1.00＋两个 0.50，`nitre` 0.25＋0.10，`flint` 1.00＋0.20），但种子权重仍是 1.00；与 [[reference/prefabs/acorn#获得路径]] 已核实的「巨石枝树掉落」一致。[表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L36-L65)、[崩落](../../../raw/current/scripts/prefabs/tree_rocks.lua#L548-L552)
- **边界：两套 build 共用 `tree_rock1_mine` 与 `tree_rock1_chop`。** 表名带 `1` 但 rock2 也在用，按 build 区分的只有砸击半径／伤害与 bank。[共用](../../../raw/current/scripts/prefabs/tree_rocks.lua#L447)、[build](../../../raw/current/scripts/prefabs/tree_rocks.lua#L153-L179)
- **边界：`tree_rock` 不是第三个 build。** 它只是「build 与阶段都随机」的注册；实例的 `inst.prefab` 会被改写成 `tree_rock1`／`tree_rock2`，所以按 `inst.prefab == "tree_rock"` 判断会永远失败。[改名](../../../raw/current/scripts/prefabs/tree_rocks.lua#L720-L721)
- **边界：`skipCanPlantCheck` 不是「随便长」。** 它只跳过 `CanPlantAtPoint` 的硬地检查；路面仍会被 `RoadManager` 拒，8 格内有建筑、或落点五半径内已有 5 株同 tag 实体也会被拒。[GetSpawnPoint](../../../raw/current/scripts/components/plantregrowth.lua#L183-L206)
- **边界：`magicgrowable` 只让书能推进一步。** 没有 `domagicgrowthfn`，园艺书不会像对作物那样连跳多级。[书的兜底](../../../raw/current/scripts/prefabs/books.lua#L66-L81)
- **边界：烧完不是消失，而是变石块。** `OnBurnt` 播 `fall_pre_burnt` 后同时排 `animover` 与 0.5 秒的 `MakeRock`；0.5 秒一到就变 `boulder` 并改碰撞，仍是 6 次 MINE、表仍是 `tree_rock1_mine`，只是动画换 `burnt_*`；但烧毁实体带 `burnt` tag，`DropLoot` 会把表里的 `tree_rock_seed` 按 `BURNED_LOOT_OVERRIDES` 变成 `ash`，见 [[reference/prefabs/acorn#常见陷阱与限制]]。[OnBurnt](../../../raw/current/scripts/prefabs/tree_rocks.lua#L570-L587)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4529)、[转换](../../../raw/current/scripts/components/lootdropper.lua#L418-L441)
- **未核验**：`collapse_small`／`tree_rock_fall` 只在 prefabs 依赖里出现，未见实际生成点；砸击伤害为源码推导，未实测。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检种子双路径、作祟与砍倒／烧毁顺序）。核对 `tree_rocks.lua` 全文（七注册、builds、两阶段、`OnChop`／`OnChopDown`／`OnAnimOver`／`OnRockFall`、`MakeRock`／`OnMine`、`SetupVineLoot`／`SpawnVineLoot`、`OnBurnt`、`OnSave`／`OnLoad`、雪与月雹）、`tree_rock_data.lua` 的权重表／区域映射／额外修正、`planted_tree.lua` 的 `tree_rock_sapling` 注册与 `growtree`、`tree_rock_seed.lua` 的部署校验、`plantregrowth.lua` 的 `GetSpawnPoint` 与倍率表、`growable.lua` 的 `DoGrowth`／`StartGrowing`、`books.lua` 的 magicgrowable 分支、`standardcomponents.lua` 的雪／月雹／可燃 helper、`simutil.lua` 的 `SpringGrowthMod`，以及 `tuning.lua` 的 `TREE_ROCK`／`TREE_ROCK_REGROWTH`／`TREE_ROCK_REGROWTH_TIME_MULT`／`SPRING_GROWTH_MODIFIER`／`LUNARHAIL_BUILDUP_*`。
  名称核验：`STRINGS.NAMES.TREE_ROCK`＝巨石枝；`TREE_ROCK1`／`TREE_ROCK2` 没有独立 `NAMES` 键，七注册靠 `SetPrefabNameOverride("tree_rock")` 统一显示为巨石枝。[po](../../../raw/current/scripts/languages/chinese_s.po#L350575-L350577)
  未覆盖：洞穴喷口房间以外的世界生成布点、`tree_rock_fall`／`collapse_small` 的实际生成点、砸击伤害实测。
- 游戏运行：未运行。秒数由 `day_time`＝300 秒推导；砸击半径／伤害与再生天数为源码常量，未做实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 均通过；Lua 示例已过语法解析。

## 相关链接

- 种子与放置器：[[reference/prefabs/acorn]]
- 同族树与木材：[[reference/prefabs/deciduous-trees]]、[[reference/prefabs/basic-gathered-materials]]
- 机制页：[[reference/components/growable]]、[[reference/components/plantregrowth]]、[[reference/components/workable]]、[[reference/components/lootdropper]]

返回 [[reference/prefabs/index|Prefab 索引]]。
