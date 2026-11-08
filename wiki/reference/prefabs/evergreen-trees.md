---
title: "常青树族（evergreen 15 注册）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/evergreens.lua 的单个 tree() 工厂产出 15 个注册：三套 build（normal／sparse／twiggy）各四阶段、多枝树独有 twiggy_old，以及预置烧毁的 evergreen_burnt 与树桩 evergreen_stump；四阶段循环生长、按阶段换 CHOP 工作量与战利品、砍倒换桩并 DIG 返 log、烧毁后双重掉落、树精生成、石化换石块、季节与世界设置门禁的再生、蜡封与雪覆盖。"
aliases: ["evergreen", "evergreen_normal", "evergreen_tall", "evergreen_short", "evergreen_sparse", "evergreen_sparse_normal", "evergreen_sparse_tall", "evergreen_sparse_short", "twiggytree", "twiggy_normal", "twiggy_tall", "twiggy_short", "twiggy_old", "evergreen_burnt", "evergreen_stump", "常青树", "臃肿常青树", "多枝树", "松树", "常青树桩", "烧毁的常青树"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/evergreens.lua", "prefabs/planted_tree.lua", "prefabs/leif.lua", "prefabs/rocks.lua", "prefabs/waxed_plants.lua", "prefabs/waxed_plant_common.lua", "prefabs/forest.lua", "components/growable.lua", "components/plantregrowth.lua", "components/petrifiable.lua", "components/forestpetrification.lua", "components/lunarhailbuildup.lua", "components/lootdropper.lua", "components/desolationspawner.lua", "standardcomponents.lua", "worldsettings_overrides.lua", "simutil.lua", "componentutil.lua", "actions.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 常青树族（evergreen 15 注册）

## 快速判断

- 职责与适用：`prefabs/evergreens.lua` 只有一个 `tree(name, build, stage, data)` 工厂，用 3 条 `builds` 配置产出 15 个注册——常青树 4 个、臃肿常青树 4 个、多枝树 5 个，外加预置烧毁与树桩各 1 个；注册的差异来自 build（战利品、产物、树精）与 stage（决定动画、工作量与月雹档；容量只在 `SetStage` 时改写），而不是各自的 `fn`。[工厂](../../../raw/current/scripts/prefabs/evergreens.lua#L816-L987)、[注册表](../../../raw/current/scripts/prefabs/evergreens.lua#L989-L1005)
- 优先使用：改生长与阶段看[生长与阶段](#生长与阶段)；改砍伐、挖桩与掉落看[砍伐与掉落](#砍伐与掉落)；改烧毁与衰亡看[烧毁衰亡与挖桩](#烧毁衰亡与挖桩)；树精看[树精生成](#树精生成)；石化看[石化](#石化)；再生、雪与蜡封看[再生雪与蜡封](#再生雪与蜡封)。
- 重要副作用：砍倒后原地换同 prefab 树桩（DIG 1 次得 1 个 `log`，多枝树桩一样），砍倒产生的树桩与烧毁株都起 `decay`（世界生成直放的 `evergreen_stump` 不带）；烧毁株除统一 40% `charcoal` 外，先砍必给 1 个木炭并再掷一次；普通与臃肿常青树可被森林石化，多枝树不能。[砍倒](../../../raw/current/scripts/prefabs/evergreens.lua#L498-L549)、[烧毁砍伐](../../../raw/current/scripts/prefabs/evergreens.lua#L152-L165)、[石化接线](../../../raw/current/scripts/prefabs/evergreens.lua#L917-L922)
- 修改边界：`pinecone`／`twiggy_nut`／`log` 的物品数值与种植链不在本页（见 [[reference/prefabs/basic-gathered-materials]]）；石化后的 `rock_petrified_tree*` 数值归 `rocks.lua`；蜡封后换 `<prefab>_waxed` 注册，属蜡封植物。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `evergreen` | `tree("evergreen", "normal", 0)`；`stage == 0` 时 `SetStage(math.random(1, 3))` | 世界生成用的默认名；`inst.build = "normal"`，显示名 常青树。[随机阶段](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L899) |
| `evergreen_normal` | `stage = 2` → `SetNormal` | CHOP 10、`normal_loot`＝2 `log`＋1 `pinecone`。[SetNormal](../../../raw/current/scripts/prefabs/evergreens.lua#L270-L285) |
| `evergreen_tall` | `stage = 3` → `SetTall` | CHOP 15、`tall_loot`＝3 `log`＋2 `pinecone`。[SetTall](../../../raw/current/scripts/prefabs/evergreens.lua#L293-L307) |
| `evergreen_short` | `stage = 1` → `SetShort` | CHOP 5、`short_loot`＝1 `log`。[SetShort](../../../raw/current/scripts/prefabs/evergreens.lua#L246-L261) |
| `evergreen_sparse` | `tree("evergreen_sparse", "sparse", 0)` | build `evergreen_new_2`、产物 `lumpy_sapling`、显示名 臃肿常青树；所有阶段都不掉松果。[sparse 配置](../../../raw/current/scripts/prefabs/evergreens.lua#L73-L86) |
| `evergreen_sparse_normal` | `stage = 2` | 与普通版同为工作量 10，`normal_loot`＝2 `log` |
| `evergreen_sparse_tall` | `stage = 3` | 工作量 15，`tall_loot`＝3 `log` |
| `evergreen_sparse_short` | `stage = 1` | 工作量 5，`short_loot`＝1 `log` |
| `twiggytree` | `tree("twiggytree", "twiggy", 0)` | build `twiggy_build`、产物 `twiggy_nut_sapling`、显示名 多枝树；不加 `evergreens`／`petrifiable`／`shelter`，改加 `renewable`。[twiggy 配置](../../../raw/current/scripts/prefabs/evergreens.lua#L87-L100)、[标签分支](../../../raw/current/scripts/prefabs/evergreens.lua#L830-L843) |
| `twiggy_normal` | `stage = 2` | CHOP 10、`normal_loot`＝1 `log`＋1 `twigs`＋1 `twiggy_nut` |
| `twiggy_tall` | `stage = 3` | CHOP 15、`tall_loot`＝1 `log`＋2 `twigs`＋2 `twiggy_nut` |
| `twiggy_short` | `stage = 1` | CHOP 5、`short_loot`＝1 `twigs` |
| `twiggy_old` | `stage = 4` → `SetOld` | 唯一显式的老树注册：CHOP 1、战利品空表、`old_anims`。[SetOld](../../../raw/current/scripts/prefabs/evergreens.lua#L315-L334) |
| `evergreen_burnt` | `tree("evergreen_burnt", "normal", 0, "burnt")` → 主模拟末尾 `OnBurnt(inst)` | 阶段先随机 1–3，仍短暂保有 `growable`；`OnBurnt` 的组件改写延迟 0.5 秒，外观立即进 `burnt`。[burnt 分支](../../../raw/current/scripts/prefabs/evergreens.lua#L972-L974) |
| `evergreen_stump` | `tree("evergreen_stump", "normal", 0, "stump")` | 建完即去物理碰撞、加 `stump`、去 `shelter`／`growable`、换小号可燃与蔓延、装 DIG workable；**不起** `decay`。[stump 分支](../../../raw/current/scripts/prefabs/evergreens.lua#L953-L970) |

### 共同的构造与命名

- 资源：普通与臃肿共用 `evergreen_new`／`evergreen_new_2` 与 `evergreen_short` bank；多枝树用 `twiggy_build`／`twiggy`。依赖表列 `log`／`pinecone`／`charcoal`／`leif`／`leif_sparse`、四个石化石块与 FX、`small_puff` 等。[assets](../../../raw/current/scripts/prefabs/evergreens.lua#L1-L55)
- 公共端（两端都执行）：`MakeObstaclePhysics(.25)`、`SetDeploySmartRadius(DEPLOYSPACING_RADIUS[DEFAULT] / 2)`、`plant`／`tree` 与 build 级标签、`inst.build`、动画 bank／build、`SetPrefabName`、`AddTag(prefab_name)`、`MakeSnowCoveredPristine`、`SetPristine()`。[构造](../../../raw/current/scripts/prefabs/evergreens.lua#L820-L864)
- 命名：`SetPrefabName(GetBuild(inst).prefab_name)` 把三套 build 的 `inst.prefab` 统一成 `evergreen`／`evergreen_sparse`／`twiggytree`，引擎再取 `STRINGS.NAMES[upper(prefab)]` 填 `name`；所以 15 个注册只有三个实显名（常青树、臃肿常青树、多枝树），烧毁与树桩注册也显示 常青树。[SetPrefabName](../../../raw/current/scripts/entityscript.lua#L840-L844)
- 端分界：`SetPristine()` 之后 `if not TheWorld.ismastersim then return inst end`；颜色、燃烧、workable、lootdropper、growable、plantregrowth、timer、petrifiable、hauntable、雪与蜡封全在主模拟。[分界](../../../raw/current/scripts/prefabs/evergreens.lua#L864-L870)

## 生长与阶段

`growable` 用同一张四阶段表（每套 build 一份），推进顺序：`pregrowfn`→`SetStage`（触发 `fn`）→按下一段取时长。`loopstages = true` 且未设 `loopstages_start`，第 4 阶段后回第 1 阶段。[growable](../../../raw/current/scripts/components/growable.lua#L121-L151)[阶段表](../../../raw/current/scripts/prefabs/evergreens.lua#L348-L380)、[配置](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L899)

| 阶段 | 普通／臃肿（`EVERGREEN_GROW_TIME`，秒） | 多枝树（`TWIGGY_TREE_GROW_TIME`，秒） | 进入时做什么 |
|---|---|---|---|
| `short` | 450±150 秒 | 同 | `SetShort`：工作量 5、`short_loot`、月雹小档、非多枝树补 `shelter` |
| `normal` | 1500±600 秒 | 900±300 秒 | `SetNormal`：工作量 10、`normal_loot`、月雹中档 |
| `tall` | 1500±600 秒 | 900±300 秒 | `SetTall`：工作量 15、`tall_loot`、月雹大档 |
| `old` | 300±150 秒 | 1500±150 秒 | `SetOld`：工作量 1、战利品只剩松果（或空表）、摘掉 `shelter`、换 `old_anims` |

- 老树不是终点：`old` 的计时结束后 `GetNextStage` 回到 `short`，所以形态会「高→老→矮」无限循环；老树每下只扣 1 点工作，而 `chop_tree` 有三档早退：休眠或缺 `growable` 时直接返回（连音效都没有），老树放完音效即返回（跳过动画、松针 FX 与唤醒），其余才继续。[提前返回](../../../raw/current/scripts/prefabs/evergreens.lua#L391-L402)、[下一阶段](../../../raw/current/scripts/components/growable.lua#L93-L101)
- 春季加速：`springgrowth = true`，每段时长在春天乘 `SPRING_GROWTH_MODIFIER`＝0.75，其它季节原样。[SpringGrowthMod](../../../raw/current/scripts/simutil.lua#L563-L565)、[常量](../../../raw/current/scripts/tuning.lua#L2284)
- 口径：表内是秒（`day_time`＝300 秒），春季倍率只乘这些阶段时长；再生与 `decay` 另用完整昼夜 `total_day_time`＝480 秒计（5／8／5 天与 30 天），不要和阶段秒数混算。[单位](../../../raw/current/scripts/tuning.lua#L19-L29)、[再生常量](../../../raw/current/scripts/tuning.lua#L3187-L3205)
- 种下的树苗（`pinecone_sapling`／`lumpy_sapling`／`twiggy_nut_sapling`）先走 `PINECONE_GROWTIME`（225±75 秒），计时到才 `SpawnPrefab` 对应 `*_short` 并 `growfromseed()` 定阶段 1。[树苗](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L186)、[growfromseed](../../../raw/current/scripts/prefabs/evergreens.lua#L571-L576)
- 月雹容量的写入顺序有个边界：容量 setter 在 `SetStage` 里，而 `lunarhailbuildup` 要到之后的 `MakeSnowCovered` 才挂，所以**新生成**树用组件默认中档（3／2），直到下一次阶段推进才改写；读档重跑 `SetStage` 才对齐。`SetOld` 不写容量；树桩与烧毁株读档会移除组件。[挂载顺序](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L939)、[组件默认](../../../raw/current/scripts/components/lunarhailbuildup.lua#L4-L8)、[读档设阶段](../../../raw/current/scripts/components/growable.lua#L249-L254)

## 砍伐与掉落

- 工作量按阶段写在 `SetShort`／`SetNormal`／`SetTall`：5／10／15（`EVERGREEN_CHOPS_*`），老树 1。动作层按 `tool:GetEffectiveness`×`workmultiplier` 结算每次扣减；树不设 `workmultiplierfn`，差异只来自工具与角色加成。[工作量](../../../raw/current/scripts/tuning.lua#L803-L805)、[效率结算](../../../raw/current/scripts/actions.lua#L1703-L1734)
- 战利品表在换阶段时整体替换（`SetLoot`），下表是源码 `builds` 的原始表；`twiggy_nut` 只出现在多枝树自己的表里，普通与臃肿永远不掉它。[loot 表](../../../raw/current/scripts/prefabs/evergreens.lua#L66-L100)

| build | `short_loot` | `normal_loot` | `tall_loot` | `old`（`SetOld`） |
|---|---|---|---|---|
| `normal` | `log`×1 | `log`×2＋`pinecone`×1 | `log`×3＋`pinecone`×2 | `pinecone`×1 |
| `sparse` | `log`×1 | `log`×2 | `log`×3 | 空表 |
| `twiggy` | `twigs`×1 | `log`×1＋`twigs`×1＋`twiggy_nut`×1 | `log`×1＋`twigs`×2＋`twiggy_nut`×2 | 空表 |

- 砍倒：`chop_down_tree` 按玩家相对镜头的左右选 `fallleft`／`fallright`（玩家在右则向左倒），把整表战利品丢在倒向一侧，再按 build 延迟震屏（普通／臃肿任何阶段 0.4 秒；多枝树只在非 short（stage>1）时 `20*FRAMES`（约 0.67 秒）），最后 `make_stump`＋推 `stump_*` 动画。[chop_down_tree](../../../raw/current/scripts/prefabs/evergreens.lua#L498-L549)
- 换桩：`make_stump` 把小号可燃／蔓延换回来、移除原 `workable` 与物理碰撞、摘 `shelter` 与月雹、停 `growable`（组件仍在）、加 `stump`、换小地图图标、装 1 次 DIG 的 `workable`，并起 `decay` 计时。[make_stump](../../../raw/current/scripts/prefabs/evergreens.lua#L459-L487)
- 挖桩：DIG 完成只 `SpawnLootPrefab("log")` 1 个再删除；多枝树桩也给 `log`，**不**退 `twigs`／`twiggy_nut`。[dig_up_stump](../../../raw/current/scripts/prefabs/evergreens.lua#L137-L140)

## 烧毁、衰亡与挖桩

- 燃烧：`MakeLargeBurnable(inst, TUNING.TREE_BURN_TIME)`＝45 秒，`SetFXLevel(5)`，`MakeMediumPropagator`；烧完回调 `tree_burnt`。[可燃](../../../raw/current/scripts/prefabs/evergreens.lua#L873-L877)、[常量](../../../raw/current/scripts/tuning.lua#L1685)
- `OnBurnt` 的改写（非立即路径延迟 0.5 秒）：熄火并摘 `burnable`／`propagator`／`growable`／`hauntable`／`petrifiable`、摘 `shelter`、把作祟换成 `MakeHauntableWork`、`SetLoot({})`，仅当 build 的 `drop_pinecones` 为真时加 10% `pinecone`（`drop_pinecones_twiggy` 从未置真，多枝树没有）、`workable` 改成 1 次 CHOP 且完成回调换 `chop_down_burnt_tree`；同时加 `burnt`、播 `burnt_*`、换小地图图标、起 `decay` 计时。[OnBurnt](../../../raw/current/scripts/prefabs/evergreens.lua#L171-L215)
- 自然掉落：`tree_burnt` 在 `OnBurnt` 后排 10 秒 `onpineconetask`，到点 `DropLoot(pt)`（落点偏移 1 单位）并置 `burntcone`。此处与 `chop_down_burnt_tree` 的 `DropLoot()` 都是**默认列表**调用：未传 `prefabs` 先 `GenerateLoot`，40% `charcoal` 规则在其中，故各掷一次；显式列表 `DropLoot(pt, prefabs)` 跳过生成、不追加。[tree_burnt](../../../raw/current/scripts/prefabs/evergreens.lua#L551-L569)、[默认](../../../raw/current/scripts/components/lootdropper.lua#L415-L416)、[规则](../../../raw/current/scripts/components/lootdropper.lua#L252-L254)、[显式](../../../raw/current/scripts/prefabs/corpses.lua#L493)
- 先砍：`chop_down_burnt_tree` 先 `SpawnLootPrefab("charcoal")` 必给 1 个，再 `DropLoot()` 重掷同一张机会表（同样无参、默认列表；可再吃 40% 木炭与 10% 松果，后者仅普通树），取消待执行的 10 秒任务，并在 `animover` 删除。两条路径都可能掉，一棵最多结算两次。[chop_down_burnt_tree](../../../raw/current/scripts/prefabs/evergreens.lua#L152-L165)
- 衰亡：`decay` 时长取 `regrowth_tuning.DEAD_DECAY_TIME`（普通／臃肿／多枝树都落在 30 天）±50%；到期时若实体在睡眠，按 `FindEntities` 顺序保留半径 6 内第一个命中的一件（`log`／`pinecone`／`twigs`／`twiggy_nut`／`charcoal` 之一），其余删除；醒着则只生成 `small_puff` 烟雾，随后删除本体。[计时器](../../../raw/current/scripts/prefabs/evergreens.lua#L700-L721)、[常量](../../../raw/current/scripts/tuning.lua#L3189-L3199)
- 离屏与读档：`OnEntitySleep` 摘燃烧与 inspectable；燃烧中的树睡眠时标 `burnt` 并摘 `growable`／`petrifiable`；唤醒时 `tree_burnt` 重跑（`burntcone` 防重排）。读档 `onload` 恢复 `build`，`stump` 走 `make_stump`、`burnt` 走 `OnBurnt(inst, true)`。[睡眠](../../../raw/current/scripts/prefabs/evergreens.lua#L626-L640)、[唤醒](../../../raw/current/scripts/prefabs/evergreens.lua#L642-L687)、[读档](../../../raw/current/scripts/prefabs/evergreens.lua#L598-L624)

## 树精生成

- 只有 `builds[n].leif` 有值的 build 会抽树精：`normal`→`leif`、`sparse`→`leif_sparse`，`twiggy` 没有这个字段。[配置](../../../raw/current/scripts/prefabs/evergreens.lua#L70-L84)
- 触发点在被砍倒的那棵树：`chop_down_tree` 用砍伐者的存活天数（或世界天数）过 `TUNING.LEIF_MIN_DAY`＝3 的门禁，再取 `LEIF_PERCENT_CHANCE`＝1/75，海狸乘 `BEAVER_LEIF_CHANCE_MOD`＝0、伐木工乘 `WOODCUTTER_LEIF_CHANCE_MOD`＝1.5，最后过 `TryLuckRoll(chopper, chance, LuckFormulas.SpawnLeif)`（无幸运值时结果等于 chance）。[判定](../../../raw/current/scripts/prefabs/evergreens.lua#L530-L548)、[常量](../../../raw/current/scripts/tuning.lua#L1049-L1058)、[修正](../../../raw/current/scripts/tuning.lua#L4641-L4644)、[幸运](../../../raw/current/scripts/componentutil.lua#L2237-L2251)
- 命中后的次数是「最多」而不是保证：循环上界写作 `(days <= 30 and 1) or math.random(days <= 80 and 2 or 3)`，即 ≤30 天只试 1 次、≤80 天试 1–2 次、更久试 1–3 次；每次在半径 `LEIF_MAXSPAWNDIST`＝15 内找一个替补树，找不到就跳过。[循环](../../../raw/current/scripts/prefabs/evergreens.lua#L539-L547)
- 替补条件：没有 `noleif`、仍有 `growable`、阶段 ≤3（老树不能变）、带 `evergreens`＋`tree` 且不在 `leif`／`stump`／`burnt` 里——因此普通树被砍时可能把旁边的**臃肿**常青树变成 `leif_sparse`，树精种类取被替换树自己的 build。[寻找目标](../../../raw/current/scripts/prefabs/evergreens.lua#L436-L440)
- 替换过程：`TransformIntoLeif` 先置 `noleif`、按阶段取 `leifscale`（0.7／1／1.25）、记下砍伐者，再在 1–4 秒内（`1+math.random()*3`）`SpawnPrefab`、继承树的多重颜色与缩放、把树原地删除并让树精播 `spawn`。[变身](../../../raw/current/scripts/prefabs/evergreens.lua#L489-L494)、[生成](../../../raw/current/scripts/prefabs/evergreens.lua#L442-L457)
- 砍树联动（非多枝树，且未命中上述早退）：半径 `LEIF_REAWAKEN_RADIUS`＝20 内找 `leif`，睡着的排 `math.random()` 秒后唤醒，再对每只 `combat:SuggestTarget(chopper)`——只是**建议目标**，不是强制仇恨。[chop_tree](../../../raw/current/scripts/prefabs/evergreens.lua#L407-L419)
- 作祟：`onhauntevergreen` 要求 `HAUNT_CHANCE_SUPERRARE`＝0.001、过 `find_leif_spawn_target`（无 `noleif`、有 `growable`、阶段 ≤3）且不带 `burnt`／`stump`。故：砍过几刀但没倒的树仍会被变（无 `stump`）；老态阶段 4 不能变；正在燃烧但还没加 `burnt` 的树也能变——「未烧」指没有 `burnt` tag。特例不中才回落 `onhauntwork`：有 `workable` 时按 `HAUNT_CHANCE_OFTEN` 推进 1 点工作。[作祟](../../../raw/current/scripts/prefabs/evergreens.lua#L792-L814)、[常量](../../../raw/current/scripts/tuning.lua#L3055)

## 石化

- 只有非多枝树注册会加 `petrifiable`（并设 `SetPetrifiedFn(onpetrifiedfn_evergreen)`）与 `TransformIntoLeif`；`petrifiable` 标签在公共端预先加上，作为「来自组件」的优化标记。[挂载](../../../raw/current/scripts/prefabs/evergreens.lua#L917-L922)、[标签](../../../raw/current/scripts/prefabs/evergreens.lua#L837-L840)
- 触发是世界级的森林石化：`forest.lua` 挂 `forestpetrification`，按 `PETRIFICATION_CYCLE`（默认 0.6–0.9 年）随机排期；未过 20% 判定（无树或树过多）先重试 `MAX_RETRIES`＝5 次，找到的森林树数必须少于已登记 `petrifiable` 总数的 20% 才整片石化；成功或重试耗尽后的重排期减半为 0.3–0.45 年（`MIN_YEARS`／`MAX_YEARS`×0.5）。[阈值](../../../raw/current/scripts/components/forestpetrification.lua#L21)、[重试](../../../raw/current/scripts/components/forestpetrification.lua#L23)、[判定与重排](../../../raw/current/scripts/components/forestpetrification.lua#L224-L235)、[初始排期](../../../raw/current/scripts/components/forestpetrification.lua#L207-L211)、[挂载](../../../raw/current/scripts/prefabs/forest.lua#L666-L669)
- 阶段到产物的映射：1→`rock_petrified_tree_short`、2→`rock_petrified_tree_med`、3→`rock_petrified_tree_tall`、4→`rock_petrified_tree_old`，各配 FX：short→short、med→normal、tall→tall、old→old；石化时继承树的多重颜色，`POPULATING` 阶段直接换（不放 FX）。[映射](../../../raw/current/scripts/prefabs/evergreens.lua#L723-L754)
- 换出来的石块继承位置并删除原树，**没有回退路径**（树实体已不存在；`rock_petrified_tree*` 按 `rocks.lua` 的共享机会表结算 rocks／nitre／flint，不是 MINE 表）。[石块表](../../../raw/current/scripts/prefabs/rocks.lua#L138-L166)、[尺寸映射](../../../raw/current/scripts/prefabs/rocks.lua#L202-L228)
- 树桩不会变石块：`onpetrifiedfn_evergreen` 在有 `stump` 标记时只生成 `petrified_trunk_break_fx`（`stump_short` 缩 0.75、`stump_tall` 缩 1.2）后删除。[树桩分支](../../../raw/current/scripts/prefabs/evergreens.lua#L779-L789)

## 再生、雪与蜡封

- 自然扩散：树挂 `plantregrowth`，产物按 build 取 `pinecone_sapling`／`lumpy_sapling`／`twiggy_nut_sapling`，搜索 tag 就是 `prefab_name`，间隔取 `regrowth_tuning.OFFSPRING_TIME`：普通 5 天、臃肿 8 天；多枝树虽定义 `TWIGGY_TREE_REGROWTH`（8 天），`builds.twiggy` 的 `regrowth_tuning` 却指向 `EVERGREEN_REGROWTH`，实际 5 天。[接线](../../../raw/current/scripts/prefabs/evergreens.lua#L906-L910)、[常量](../../../raw/current/scripts/tuning.lua#L3189-L3205)
- 季节与世界设置：扩散计时按 `InternalTimes[prefab] += dt × 倍率 × REGROWTH_TIME_MULTIPLIER` 累积，普通常青树夏季×2、冬季为 0，臃肿常青树没有季节门禁（恒 ×1），多枝树用 `TWIGGYTREE_REGROWTH_TIME_MULT`；世界设置 `evergreen_regrowth`／`twiggytrees_regrowth` 分别写这两个值，`regrowth` 设置写总倍率。[倍率](../../../raw/current/scripts/components/plantregrowth.lua#L23-L35)、[累积](../../../raw/current/scripts/components/plantregrowth.lua#L62-L74)、[常青设置](../../../raw/current/scripts/worldsettings_overrides.lua#L2288-L2313)、[多枝设置](../../../raw/current/scripts/worldsettings_overrides.lua#L2314-L2339)、[总倍率](../../../raw/current/scripts/worldsettings_overrides.lua#L2185-L2210)
- 扩散不是必胜：到点先 `ResetGrowthTime`（失败也重置），再要求落点能种、非道路、3 半径内无实体、`BASE_RADIUS` 内无 `structure`／`wall`、落点半径内同 tag 实体 <5 才 `SpawnPrefab`。[落点与密度](../../../raw/current/scripts/components/plantregrowth.lua#L183-L242)
- 大灭绝后的世界级补充：`desolationspawner` 按 `evergreen`／`evergreen_sparse`／`twiggytree` 三类分别用 `DESOLATION_RESPAWN_TIME`（50 天）补苗，倍率与上表同源（普通夏季×2、冬季 0）。[重生](../../../raw/current/scripts/components/desolationspawner.lua#L170-L178)
- 雪覆盖是外观层：`MakeSnowCoveredPristine` 预插并隐藏 `snow` 符号，`MakeSnowCovered` 按 `TheWorld.state.issnowcovered` 显示／隐藏，并在主模拟顺手挂 `lunarhailbuildup`。[雪](../../../raw/current/scripts/standardcomponents.lua#L835-L855)
- 蜡封：`MakeWaxablePlant` 挂 `waxable`、`SetNeedsSpray()`（需蜡喷雾而非普通蜂蜡），`WAXED_PLANTS.WaxPlant` 生成 `<inst.prefab>_waxed`（`evergreen_waxed`／`evergreen_sparse_waxed`／`twiggytree_waxed`），按当前动画挑 `*_burnt_*`／`*_stump_*`／`*_idle_old` 键后删除原树；树桩与烧毁株同样可封。蜡封件自身行为不在本页。[挂载](../../../raw/current/scripts/prefabs/evergreens.lua#L937-L939)、[蜡封函数](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L543-L586)、[注册](../../../raw/current/scripts/prefabs/waxed_plants.lua#L356-L389)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`growable` | 四阶段表、`loopstages`、`springgrowth`、`magicgrowable`；`SetStage` 触发 setter | 主模拟；阶段决定工作量、战利品与月雹 | [[reference/components/growable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/evergreens.lua#L348-L380) |
| 组件：`workable` | `CHOP` 回调 `chop_tree`、完成 `chop_down_tree`；换桩后改 `DIG` | 主模拟；老树 1 点、树桩 1 次 DIG | [[reference/components/workable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/evergreens.lua#L883-L887) |
| 组件：`lootdropper` | 按 build／阶段 `SetLoot`，烧毁后清表＋10% 松果（仅普通常青树） | 主模拟；`burnt` 规则另掷 40% `charcoal` | [[reference/components/lootdropper#关键接口]]；[烧毁](../../../raw/current/scripts/prefabs/evergreens.lua#L184-L191) |
| 组件：`plantregrowth` | 产物 `*_sapling`、搜索 tag＝`prefab_name`、间隔 5／8／5 天 | 主模拟；普通树有冬季门禁 | [[reference/components/plantregrowth#关键接口]]；[接线](../../../raw/current/scripts/prefabs/evergreens.lua#L906-L910) |
| 组件：`petrifiable` | 非多枝树专属；`SetPetrifiedFn` 换石块或只留碎裂 FX | 主模拟；森林石化事件触发 | [[reference/components/petrifiable#关键接口]]、[[reference/components/forestpetrification]]；[挂载](../../../raw/current/scripts/prefabs/evergreens.lua#L917-L922) |
| 组件：`lunarhailbuildup` | 由 `MakeSnowCovered` 顺手挂上，阶段 setter 写容量 | 主模拟；树桩与读档后的烧毁株会移除 | [[reference/components/lunarhailbuildup#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L887-L920) |
| 组件：`simplemagicgrower` | 园艺书连续催长，止于第 3 阶段 | 主模拟 | [[reference/components/simplemagicgrower#关键接口]]；[接线](../../../raw/current/scripts/prefabs/evergreens.lua#L901-L902) |
| 组件：`burnable`／`propagator`／`hauntable` | 45 秒大火与中档蔓延；自定义 `onhauntevergreen` | 主模拟；烧毁／换桩后换成小号或无操作版本 | [[reference/components/burnable#关键接口]]、[[reference/components/hauntable#关键接口]]；[作祟](../../../raw/current/scripts/prefabs/evergreens.lua#L929-L930) |
| 组件：`timer` | 只用于 `decay` | 主模拟；树桩与烧毁株 | [[reference/components/timer#关键接口]]；[接线](../../../raw/current/scripts/prefabs/evergreens.lua#L912-L914) |
| 关联 Prefab：`leif`／`leif_sparse` | 砍倒抽签或作祟直接替换的树精 | 主模拟；由被替换树的 build 决定种类 | 源码 [leif.lua](../../../raw/current/scripts/prefabs/leif.lua#L216-L217)（Wiki 待建） |
| 关联 Prefab：`pinecone_sapling`／`lumpy_sapling`／`twiggy_nut_sapling` | 再生与种下的树苗，长大回各自 `*_short` | 主模拟 | [planted_tree](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L186)（Wiki 待建） |
| 关联 Prefab：`rock_petrified_tree_*`／`petrified_tree_fx_*` | 石化产物与阶段 FX | 主模拟；`POPULATING` 时不放 FX | [石块工厂](../../../raw/current/scripts/prefabs/rocks.lua#L465-L502) |

## 典型接入

### 只把常青树的生长时间压到一半（不动多枝树）

- 前提与选择：服务端 `modmain.lua` 直接改 `TUNING.EVERGREEN_GROW_TIME` 每档的 `base`／`random`。四段时长函数都在**进入该阶段时**才读表，所以不需要 `AddPrefabPostInit`；多枝树读的是另一张 `TWIGGY_TREE_GROW_TIME`，不受影响。
- 生效与撤销：只影响之后才起算的阶段，当前阶段不变；撤掉模组重进世界后回默认。
- 原版依据：四段 `time` 都是 `GetRandomWithVariance(data.grow_times[i].base, ...)`，读表时机在 `Growable:StartGrowing`。[阶段表](../../../raw/current/scripts/prefabs/evergreens.lua#L348-L380)、[取时长](../../../raw/current/scripts/components/growable.lua#L42-L79)

```lua
-- modmain.lua：常青树（普通／臃肿）各阶段时长减半；春季 0.75 仍会再乘一次
local SCALE = 0.5
for _, stage in ipairs(GLOBAL.TUNING.EVERGREEN_GROW_TIME) do
    stage.base = stage.base * SCALE
    stage.random = stage.random * SCALE
end
```

- 调整与验证：单位是秒（`day_time`＝300）；`short` 会从 450±150 秒变成 225±75 秒。验证看新进入阶段的树 `inst.components.growable.targettime - GetTime()` 落在新区间，且 `twiggy_short`／`twiggy_tall` 仍按原表。

## 常见陷阱与限制

- **误区：15 个注册是 15 个显示名。** 只有三个 `NAMES` 键（常青树／臃肿／多枝树）；阶段、树桩、烧毁共用 base 名，区分形态看 `inst.build` 与 `growable.stage`。[改名](../../../raw/current/scripts/prefabs/evergreens.lua#L855-L856)
- **误区：常青树长到 tall 就停。** 阶段表有第 4 段 `old`，`loopstages` 会继续 `tall→old→short`；老树只砍 1 下、普通树仅掉 1 个松果、臃肿与多枝树为空表。[阶段表](../../../raw/current/scripts/prefabs/evergreens.lua#L348-L380)、[SetOld](../../../raw/current/scripts/prefabs/evergreens.lua#L315-L334)
- **边界：新生成树的月雹容量不是阶段值。** 阶段 setter 早于 `MakeSnowCovered`，组件默认中档；读档会重跑 `SetStage` 才对齐。做「按阶段给容量」的断言前先确认不是刚生成的树。[顺序](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L939)、[默认值](../../../raw/current/scripts/components/lunarhailbuildup.lua#L4-L8)
- **边界：烧毁株能结算两次且每次重掷。** 10 秒自然掉落与「先砍」都会各自 `DropLoot`，两处都是默认列表调用，40% 木炭与普通树的 10% 松果各掷一次，不是一次性；先砍另必给 1 个木炭。[两路径](../../../raw/current/scripts/prefabs/evergreens.lua#L551-L569)、[先砍](../../../raw/current/scripts/prefabs/evergreens.lua#L152-L165)
- **边界：树桩不能复原。** `make_stump` 只 `StopGrowing`，`growable` 仍在（`evergreen_stump` 注册移除组件），唯一出路是 DIG 或等 `decay` 删除；`plantregrowth` 是另生树苗。[换桩](../../../raw/current/scripts/prefabs/evergreens.lua#L459-L487)
- **边界：多枝树的再生间隔不是 8 天。** `builds.twiggy` 复用 `EVERGREEN_REGROWTH.OFFSPRING_TIME`（5 天），只有 `desolationspawner` 的 `DESOLATION_RESPAWN_TIME` 用到 `TWIGGY_TREE_REGROWTH`。[配置](../../../raw/current/scripts/prefabs/evergreens.lua#L87-L100)、[常量](../../../raw/current/scripts/tuning.lua#L3201-L3205)
- **边界：烧毁／换桩后作祟不再推进工作。** `MakeHauntableWork`／`MakeHauntableIgnite` 分支已被 `#HAUNTFIX` 注释、恒返回 false（[hauntable](../../../raw/current/scripts/standardcomponents.lua#L1107-L1189)）；「作祟砍一刀」只来自自定义 `onhauntwork`。[自定义作祟](../../../raw/current/scripts/prefabs/evergreens.lua#L792-L799)
- **未核验**：树精战斗与行为、蜡封件交互、月雹积累节奏、蓝图与活动皮肤未覆盖。

## 验证记录

- 源码审阅：2026-11-10，buildid 26474873。通读 `evergreens.lua` 全部（15 注册、四阶段、砍伐／换桩／挖桩、烧毁双路径、读档与唤醒、树精、石化、再生、雪与蜡封）及相关组件、世界设置与 tuning。
  名称核验：`STRINGS.NAMES.EVERGREEN`＝常青树、`EVERGREEN_SPARSE`＝臃肿常青树、`TWIGGYTREE`＝多枝树；各阶段、烧毁、树桩变体均无独立键。[po](../../../raw/current/scripts/languages/chinese_s.po#L342490-L342497)、[po](../../../raw/current/scripts/languages/chinese_s.po#L351110-L351112)
  未覆盖：树精行为与战斗、蜡封件交互、月雹积累实测、皮肤与活动变体。
- 游戏运行：未运行。秒数按 `day_time`＝300 秒推算；概率与计时为源码常量，未实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 均通过（计数见交付回报）。

## 相关链接

- 木头与炭、灰：[[reference/prefabs/basic-gathered-materials]]、[[reference/prefabs/ashes-and-charcoal]]
- 同批木料：[[reference/prefabs/driftwood-livinglog]]
- 对比页：[[reference/prefabs/deciduous-trees]]、[[reference/prefabs/tree-rocks]]
- 机制页：[[reference/components/growable]]、[[reference/components/workable]]、[[reference/components/lootdropper]]、[[reference/components/plantregrowth]]、[[reference/components/petrifiable]]、[[reference/components/forestpetrification]]、[[reference/components/lunarhailbuildup]]

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|Prefab 索引]]。
