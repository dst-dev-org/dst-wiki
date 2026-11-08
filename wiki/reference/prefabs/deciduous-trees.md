---
title: "桦栗树（deciduoustree 六注册）"
created: "2026-11-08"
updated: "2026-11-10"
type: prefab
summary: "deciduoustrees.lua 的六个注册：随机阶段的 deciduoustree 与显式 short／normal／tall、烧毁的 _burnt、树桩 _stump；growable 三阶段与春季加速、按季节切换叶色与产出、三档 CHOP 与分阶段战利品、怪物桦栗树分支、砍倒加工成桩、DIG 挖桩返 log、烧毁转 charcoal、雪覆盖与月雹积累。"
aliases: ["deciduoustree", "deciduoustree_normal", "deciduoustree_tall", "deciduoustree_short", "deciduoustree_burnt", "deciduoustree_stump", "桦栗树", "桦栗树桩", "烧毁的桦栗树", "毒桦栗树", "桦栗树怪"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/deciduoustrees.lua", "prefabs/planted_tree.lua", "components/lootdropper.lua", "components/growable.lua", "components/plantregrowth.lua", "standardcomponents.lua", "simutil.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 桦栗树（deciduoustree 六注册）

## 快速判断

- 职责与适用：`deciduoustrees.lua` 用同一个 `makefn` 产出六个注册，差别只在阶段参数与初始状态；所有形态都把 `inst.prefab` 改写成 `deciduoustree`，所以显示名、`plantregrowth` 计时键与 `searchTag` 全部落在同一个名字上。[注册段](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1460-L1465)、[改名](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1317)
- 优先使用：改生长与季节看[生长与季节](#生长与季节)；改砍伐次数与掉落看[砍伐与掉落](#砍伐与掉落)；改烧毁／树桩看[烧毁、树桩与消失](#烧毁树桩与消失)。种下入口与放置器见 [[reference/prefabs/acorn#种植放置预览与生长]]，本页只写树。
- 重要副作用：砍倒会换成同名的树桩（保留 `monster` 标记），树桩只能 DIG 一次并掉 1 根 `log`（怪物形态掉 `livinglog`）；砍倒的树桩与烧毁株都带 `decay` 计时器（世界生成直接放置的 `deciduoustree_stump` 不带）。到期时只有实体**休眠**才清扫半径 6 内的散落 `log`／`acorn`／`charcoal`，且这批散落物总共只保留一件、其余删除；醒着则只生成 `small_puff` 烟雾，随后删除本体。[make_stump](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L482-L526)、[OnTimerDone](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1007-L1028)
- 修改边界：`acorn`／`acorn_cooked` 的数值与放置器见 [[reference/prefabs/acorn]]；`log` 与木头链见 [[reference/prefabs/basic-gathered-materials]]；叶片 FX、月雹与光照链路只写接线，不展开。本页只写树。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deciduoustree` | `tree(name, "normal", 0)`；`stage == 0` 时 `SetStage(math.random(1, 3))` | 世界生成用的默认名，随机落在 short／normal／tall 三阶段之一；叶色另由季节决定。[随机阶段](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1380-L1383) |
| `deciduoustree_short` | `stage = 1` → `SetShort` | `short_anims`、CHOP 5、倒塌掉落 1 `log`、月雹小档。[SetShort](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L362-L371) |
| `deciduoustree_normal` | `stage = 2` → `SetNormal` | `normal_anims`、CHOP 10、掉落 2 `log`、月雹中档；非强制 `StartMonster` 只命中该阶段。[SetNormal](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L384-L391)、[怪物前置](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L830-L832) |
| `deciduoustree_tall` | `stage = 3` → `SetTall` | `tall_anims`、CHOP 15、掉落 3 `log`（带 acorn 的 build 多 1 个）、月雹大档。[SetTall](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L402-L409) |
| `deciduoustree_burnt` | `data = "burnt"` → 主模拟末尾 `OnBurnt(inst, true)` | 直接进 `burnt` 状态：无 `growable`、无 `shelter`、1 次 CHOP 后给 `charcoal`。[烧毁初始化](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1436-L1438) |
| `deciduoustree_stump` | `data = "stump"` → 建完即 `AddTag("stump")` 并重配组件 | 无 `growable`、无 `shelter`、物理碰撞移除、DIG 1 次掉 `log`；该注册不起 `decay`（`decay` 只由砍倒的 `make_stump` 与烧毁的 `OnBurnt` 启动）。[树桩分支](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1413-L1430) |

### 共同的构造与端分界

- 资源与依赖：`tree_leaf_short／normal／tall／monster` 四套 bank，加 trunk／green／red／orange／yellow／poison 六个 leaves build；prefabs 依赖含 `log`、`acorn`、`charcoal`、`livinglog`、`nightmarefuel`、落叶 FX 与 `deciduous_root`。[assets](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1-L41)
- 公共端：`MakeObstaclePhysics(.25)`、`SetDeploySmartRadius(DEPLOYSPACING.DEFAULT/2)`、tags `plant`／`tree`／`birchnut`／`cattoyairborne`／`deciduoustree`／`shelter`，bank 固定 `tree_leaf`、build 固定 `tree_leaf_trunk_build`，叶片靠 `swap_leaves` 覆盖。[构造](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1288-L1315)
- 端分界：`MakeSnowCoveredPristine` 与 `SetPristine()` 之后 `if not TheWorld.ismastersim then return inst end`；SG、燃烧、growable、workable、lootdropper 全在主模拟。[分界](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1324-L1333)、[组件段](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1342-L1398)
- 命名：`SetPrefabName(GetBuild(inst).prefab_name)` 把六注册统一成 `deciduoustree`，引擎再用 `STRINGS.NAMES.DECIDUOUSTREE` 填 `name`（`SetPrefabName` 内 `self.name = self.name or STRINGS.NAMES[upper(prefab)]`）；怪物形态不改名，只多一个 `POISON` 检查状态。[命名](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1317)、[默认命名](../../../raw/current/scripts/entityscript.lua#L840-L844)、[状态](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L782-L787)

## 生长与季节

`acorn` 种下先生成 `acorn_sapling`，树苗计时结束后 `SpawnPrefab("deciduoustree")` 再调 `tree:growfromseed()`，由 `handler_growfromseed` 把新树定在 `stage 1`。[树苗长树](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L80)、[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L185)、[growfromseed](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L746-L780)

| 阶段 | 时长（`DECIDUOUS_GROW_TIME`） | 进入时做什么 |
|---|---|---|
| `short` | 1.5×`day_time`±0.5×`day_time`＝450±150 秒（300–600） | `SetShort`：改 `short_anims`、CHOP 5、月雹小档、`short_loot` |
| `normal` | 5×`day_time`±2×`day_time`＝1500±600 秒（900–2100） | `SetNormal`：CHOP 10、月雹中档、`normal_loot` |
| `tall` | 5×`day_time`±2×`day_time`＝1500±600 秒 | `SetTall`：CHOP 15、月雹大档、`tall_loot` |

- `loopstages = true`：`tall` 之后再回到 `short`（`loopstages_start` 未设，默认 1），所以树会一直循环长大。[阶段表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L420-L426)、[配置](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1380-L1386)
- `springgrowth = true`：每段时长在春天乘 `SPRING_GROWTH_MODIFIER`＝0.75；其它季节原样。[SpringGrowthMod](../../../raw/current/scripts/simutil.lua#L563-L565)、[常量](../../../raw/current/scripts/tuning.lua#L2284)
- 阶段推进走 `Growable:DoGrowth`：先 `pregrowfn`，再 `SetStage`（触发 `fn`），再 `StartGrowing` 取下一段时长；`StartGrowing` 用 `stagedata.time(inst, stage, stagedata)` 取秒数，因此 `SetShort`／`SetNormal`／`SetTall` 的副作用发生在计时开始之前。[DoGrowth](../../../raw/current/scripts/components/growable.lua#L121-L151)、[取时长](../../../raw/current/scripts/components/growable.lua#L57-L78)

### 季节分支

`OnSeasonChanged` 只处理非怪物、非树桩、非烧毁的树，把 `target_leaf_state` 设成季节对应值再走 `OnChangeLeaves`；`OnCyclesChanged` 在距季节结束 ≤3 天时提前 0.1–3×`day_time` 秒切换到下一季的颜色。[季节回调](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1157-L1183)、[提前切换](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1185-L1238)

| 季节／状态 | `leaf_state` 与 build | 外观与产出 |
|---|---|---|
| 春、夏 | `normal`，build `normal` | 绿色叶，`SetMultColour(color)` 用 0.5–1 的随机明度；`shelter` 生效；正常掉 `acorn`。[叶色](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L305-L329) |
| 秋 | `colorful`，build 从 `red`／`orange`／`yellow` 随机 | 落叶特效每 `MIN_SWAY_FX_FREQUENCY`–`MAX_SWAY_FX_FREQUENCY` 秒一次；`tall` 且 `colorful` 时砍倒额外掉 1 个 `acorn`。[秋季特效](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L231-L240)、[额外桦栗果](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L601-L618) |
| 冬 | `barren`，build `barren`、`leavesbuild = nil` | 光杆：无 `shelter`、无叶 FX、无 `acorn`（`drop_acorns = false`），砍倒仍按阶段掉 `log`。[barren build](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L56-L66)、[拔 shelter](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L336-L340) |
| 毒（怪物） | `poison`，build `poison` | 紫色叶，战利品换 `livinglog`＋`acorn`，详见[怪物分支](#怪物分支)。[毒 build](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L100-L110) |

- 换叶是动画驱动：从 `barren` 恢复时播 `growleaves`，秋／冬切换播 `dropleaves`，`animover` 时才真正换 build、换掉落表并 `Sway`；打叶子的瞬间不会立刻改掉落。[换叶流程](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L289-L348)
- `GrowLeavesFn` 在切叶完成时按当前 `growable.stage` 重设掉落表（short／normal／tall 各一份），所以季节切换同时改写「砍倒给几个 log」。[按阶段设表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L263-L271)
- 世界刚生成时 `OnInitSeason` 立即定叶；`POPULATING` 阶段季节还没就绪，所以构造后再补一次 `DoTaskInTime(0, OnInitSeason)`。[季节初始化](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1241-L1250)、[补一次](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1432-L1443)

## 砍伐与掉落

`workable` 用 `ACTIONS.CHOP`，工作量按阶段在 `SetShort`／`SetNormal`／`SetTall` 里写：5／10／15，怪物形态 `DECIDUOUS_CHOPS_MONSTER`＝18。[工作量](../../../raw/current/scripts/tuning.lua#L815-L818)、[怪物工作量](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L789-L793)

| build | `short_loot` | `normal_loot` | `tall_loot` |
|---|---|---|---|
| `normal`／`red`／`orange`／`yellow` | `log`×1 | `log`×2 | `log`×3＋`acorn`×1 |
| `barren` | `log`×1 | `log`×2 | `log`×3 |
| `poison` | `livinglog`＋`acorn` | `livinglog`＋`acorn`×2 | `livinglog`＋`acorn`×3 |

- 每次 `CHOP` 命中：播 `chop_<阶段>`、掉一片落叶（`chopfx`）、按镜头摆动；怪物形态改走 SG 的 `chop_pst` 并放受伤音。[chop_tree](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L428-L459)
- 砍倒：`chop_down_tree` 先判怪物化，再按玩家相对镜头的左右播 `fallleft`／`fallright`，把战利品丢在倒向一侧并震屏，最后 `PushAnimation(stump)`＋`make_stump`。[chop_down_tree](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L601-L625)
- `tall` 且 `colorful`（秋）时，除 `tall_loot` 里的 `acorn` 外**再** `SpawnLootPrefab("acorn")` 1 个，所以秋色的 `tall` 被砍倒时共得 2 个 `acorn`。[额外桦栗果](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L606-L617)

### 怪物分支

- 触发条件：非怪物、`leaf_state ~= "barren"`、`stage == 3`、存活天数 ≥ `DECID_MONSTER_MIN_DAY`＝3 的 `tall` 树被砍倒时掷一次骰：季节概率 秋 0.15／春 0.08／夏 0.033／冬 0，再乘天数阈值修正（`{20, 35, 70}` 对应 `{.2, .5, 1, 1.12}`），海狸乘 0、伐木工乘 1.5，最后过 `TryLuckRoll`。[判定](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L530-L556)、[常量](../../../raw/current/scripts/tuning.lua#L2901-L2914)
- 「3–4 棵」只是**预约上限**：命中后在半径 30 内找带 `birchnut`、非 `stump`／`burnt`／`monster`／FX 且没有在途 `monster_start_task`／`monster_stop_task`／`domonsterstop_task` 的树，按 `math.random(3, 4)` 最多给这么多棵各排 1–4 秒后的 `delayed_start_monster`；这些树到点后还要过 `StartMonster` 的阶段门禁，过不了就不变身，所以不是必定变 3–4 棵。[扩散](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L557-L574)
- 变身：非强制路径只接受 `normal` 阶段（`inst.anims == normal_anims`）且非 `barren` 的树，先 `DoGrowth` 长到 `tall`，换 bank `tree_leaf_monster`、走 `transform_in`，加 `plantcreature`／`combat`／`deciduoustreeupdater`，工作量改 18；`force` 为真（作祟、读档）时跳过阶段检查，门禁不通过时到点直接不变身。[StartMonster](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L830-L851)、[DoStartMonster](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L789-L823)
- 怪物树被砍倒：额外各 `AddChanceLoot` 一次 `livinglog` 与 `nightmarefuel`，概率 `DECID_MONSTER_ADDITIONAL_LOOT_CHANCE`＝0.2；同时清掉 `combat`／`deciduoustreeupdater`／`plantcreature`。[额外掉落](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L577-L599)、[常量](../../../raw/current/scripts/tuning.lua#L2913)
- 退回普通树：`StopMonster` 撤组件、换回 `tree_leaf` bank、按季节重定叶色并 `DoGrowth`；`acorn` 半径 30 内的安抚走的就是它。[StopMonster](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L874-L897)

## 烧毁、树桩与消失

- 燃烧：`MakeLargeBurnable(inst, TUNING.TREE_BURN_TIME)`＝45 秒、`SetFXLevel(5)`、`extinguishimmediately = false`；怪物形态点燃会走 SG 的 `burning_pre` 并放一圈 `SpawnIgniteWave`。[可燃](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1349-L1354)、[点燃回调](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L899-L906)
- 烧毁转化（`onburntchanges`）：删 `growable`、去 `shelter`／`cattoyairborne`／`monster`，`SetLoot({})` 清空整张战利品表（概率表一并清掉），只在 build 的 `drop_acorns` 为真时 `AddChanceLoot("acorn", .1)`；把 `workable` 改成 1 次 CHOP 且完成回调换成 `chop_down_burnt_tree`、播 `burnt`、3 帧后彻底摘掉 `burnable`／`propagator`。[onburntchanges](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L651-L687)
- 烧毁株的掉落分两条路径，且每次 `DropLoot` 都重新 `GenerateLoot`、把机会表整张重掷：① 等 10 秒自然掉落——`tree_burnt` 先 `OnBurnt`，再由 `OnAcornTask` 调一次 `DropLoot`，除 10% `acorn` 机会（实体带 `burnt` tag，`DropLoot` 的通用转换会把它落成 `acorn_cooked`）外，`lootdropper` 的 `burnt` 规则（`HasTag("burnt")` 且 `math.random() < .4`）还会额外掷 40% `charcoal`；② 先砍——`chop_down_burnt_tree` 先 `SpawnLootPrefab("charcoal")` 必给 1 个，再 `DropLoot` 重掷同一张表（同样 10% `acorn`＋40% `charcoal`），并 `Cancel` 掉待执行的 10 秒任务，动画结束删除。[tree_burnt](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L732-L744)、[烧毁砍伐](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L627-L640)、[burnt 规则](../../../raw/current/scripts/components/lootdropper.lua#L252-L253)、[熟化转换](../../../raw/current/scripts/components/lootdropper.lua#L418-L441)
- 树桩：`make_stump` 把 `burnable`／`propagator` 换成 Small 版、移除原 `workable` 与物理碰撞、加 `stump` tag、换树桩小地图图标、取消怪物叶色任务、停 `growable`，并起 `decay` 计时器。[make_stump](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L482-L526)
- 挖桩：`workable(ACTIONS.DIG)` 工作量 1，完成时 `SpawnLootPrefab(inst.monster and "livinglog" or "log")` 后删除——怪物树桩给 `livinglog`，并**不**退还 `acorn`。[dig_up_stump](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L461-L464)、[树桩 workable](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L514-L517)
- 消失：树桩与烧毁株都靠 `decay` 计时器，时长 `DECIDUOUS_REGROWTH.DEAD_DECAY_TIME`＝30 天，随机 ±50%；到期时若实体在睡眠，会把半径 6 内的 `log`／`acorn`／`charcoal` 只留一件、其余删除；醒着则只生成 `small_puff` 烟雾，最后都 `Remove`。[计时器](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1007-L1028)、[常量](../../../raw/current/scripts/tuning.lua#L3207-L3211)
- 自然扩散：树挂 `plantregrowth`，产物 `acorn_sapling`、搜索 tag `deciduoustree`、间隔 `DECIDUOUS_REGROWTH.OFFSPRING_TIME`＝3 天，但该计时只在春天累加（`DECIDIOUS_REGROWTH_TIME_MULT × (非春 ? 0 : 1)`）。[接线](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1358-L1361)、[季节倍率](../../../raw/current/scripts/components/plantregrowth.lua#L33-L35)

## 雪与月雹

- 雪覆盖是外观层：`MakeSnowCoveredPristine` 预插 `snow` 符号并隐藏，`MakeSnowCovered` 按 `TheWorld.state.issnowcovered` 显示／隐藏，同时给主模拟挂 `lunarhailbuildup`。[雪覆盖](../../../raw/current/scripts/standardcomponents.lua#L835-L855)、[接线](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1411)
- 月雹积累按阶段给容量：`short` 为小档、`normal` 中档、`tall` 大档——对应总工作量 2／3／4 与月玻璃 1／2／3；`make_stump`（树桩）与读档时的烧毁株会 `RemoveLunarHailBuildup` 摘掉组件（刚烧毁、尚未读档的树仍挂着）。[档位](../../../raw/current/scripts/standardcomponents.lua#L901-L920)、[常量](../../../raw/current/scripts/tuning.lua#L7911-L7916)、[阶段赋值](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L362-L409)
- 换阶段只改容量，不重置已有积累；要清空得退化到树桩／烧毁分支。[RemoveLunarHailBuildup](../../../raw/current/scripts/standardcomponents.lua#L892-L899)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`growable` | 三阶段表、`loopstages`、`springgrowth`、`magicgrowable`；`DoGrowth` 推进并重设掉落表 | 主模拟；阶段数决定 CHOP 与月雹容量 | [[reference/components/growable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1380-L1386) |
| 组件：`workable` | `CHOP` 回调 `chop_tree`、完成回调 `chop_down_tree`；树桩换成 `DIG` | 主模拟；`SetWorkLeft` 按阶段改写 | [[reference/components/workable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1369-L1372) |
| 组件：`lootdropper` | 按 build／阶段 `SetLoot`，怪物与烧毁用 `AddChanceLoot` | 主模拟；换叶、换阶段、烧毁都会重设 | [[reference/components/lootdropper#关键接口]]；[设置](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L263-L271) |
| 组件：`plantregrowth` | 自然扩散 `acorn_sapling`，仅春季推进 | 主模拟；按 `inst.prefab`＝`deciduoustree` 取倍率 | [[reference/components/plantregrowth#关键接口]]；[接线](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1358-L1361) |
| 组件：`simplemagicgrower` | 园艺书类道具的连续催长，止于第 3 阶段 | 主模拟 | [[reference/components/simplemagicgrower#关键接口]]；[接线](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1388-L1389) |
| 组件：`lunarhailbuildup` | 月雹积累与月玻璃产出，按阶段给容量 | 主模拟；树桩移除，烧毁株读档时移除 | [档位](../../../raw/current/scripts/standardcomponents.lua#L901-L920) |
| 组件：`burnable`／`propagator`／`hauntable` | 45 秒大火＋中档蔓延；作祟可提前砍一刀或极低概率强制变怪物 | 主模拟；烧毁后摘掉燃烧组件 | [Haunt](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1257-L1282)、[helper](../../../raw/current/scripts/standardcomponents.lua#L188-L228) |
| SG：`SGdeciduoustree` | 摆动、受击、燃烧、变身与 `empty` 状态 | 主模拟；`SetStateGraph` 后立刻 `empty` | [接线](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1342-L1344) |
| 关联 Prefab：`acorn_sapling` | 自然扩散与种下产物，长大回 `deciduoustree` | 主模拟 | [[reference/prefabs/acorn]]；[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L185) |
| 关联 Prefab：`log`／`livinglog`／`charcoal` | 砍倒、挖桩、烧毁的产物 | 主模拟 | `log` 见 [[reference/prefabs/basic-gathered-materials]]、`livinglog` 见 [[reference/prefabs/driftwood-livinglog]]、`charcoal` 见 [[reference/prefabs/ashes-and-charcoal]]；[掉落表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L43-L111) |
| 资源：anim | `tree_leaf_short／normal／tall／monster` 与六个 leaves build | 六注册共用 | [assets](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1-L18) |

## 典型接入

### 只把桦栗树的生长时间压到 1/4（不动松树和巨石枝）

- 前提与选择：服务端 `modmain.lua` 直接改 `TUNING.DECIDUOUS_GROW_TIME` 每一档的 `base`／`random`（含未被阶段引用的 `old` 档）。阶段时长函数是在**每次进入阶段时**才读这张表，所以不必挂 `AddPrefabPostInit`，也不必改 `growth_stages`。
- 生效与撤销：只影响之后才起算的阶段计时：已经跑着的当前阶段不缩短，`tall`→`short` 的循环照常。撤掉模组、重进世界后表回到默认值（`tuning.lua` 在启动时重建）。
- 原版依据：三段的 `time` 都是 `GetRandomWithVariance(TUNING.DECIDUOUS_GROW_TIME[i].base, ...)`，读表时机在 `Growable:StartGrowing`。[阶段表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L420-L426)、[取时长](../../../raw/current/scripts/components/growable.lua#L57-L78)

```lua
-- modmain.lua：桦栗树各阶段时长压到 1/4（春季的 0.75 仍会再乘一次）
local SCALE = 0.25
for _, stage in ipairs(GLOBAL.TUNING.DECIDUOUS_GROW_TIME) do
    stage.base = stage.base * SCALE
    stage.random = stage.random * SCALE
end
```

- 调整与验证：单位是秒；`short` 会从 450±150 秒变成 112.5±37.5 秒。验证看新进入阶段的树：`inst.components.growable.targettime - GetTime()` 应落在新区间，且松树（`EVERGREEN_GROW_TIME`）不受影响。

## 常见陷阱与限制

- **误区：六个注册是六个显示名。** 全部走 `SetPrefabName("deciduoustree")`，实显都是「桦栗树」，阶段只体现在动画与掉落；要区分形态请看 `growable.stage` 与 `inst.build`。[改名](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1317)
- **边界：怪物形态仍是同一个 prefab。** `plantcreature`／`combat` 是运行期加的，砍倒后会摘掉；与怪物形态相关的存档字段只有 `monster` 与 `monster_start_offset`（另有 `build`／`leaf_state`／`burnt`／`stump` 等常规字段）。[存档](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1034-L1090)
- **边界：冬季不产桦栗果。** `barren` build 的 `drop_acorns = false`，且 `tall` 额外 `acorn` 只在 `colorful` 时给；冬天砍 `tall` 只有 3 个 `log`。[barren](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L56-L66)、[额外桦栗果](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L606-L617)
- **边界：树桩不能恢复成树。** `make_stump` 只 `StopGrowing`，`growable` 组件仍在（仅 `deciduoustree_stump` 注册会移除它），唯一出路是 DIG 或等 `decay` 到期删除；`plantregrowth` 的扩散是另生树苗，不是原地复活。[make_stump](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L482-L526)
- **边界：烧毁株的收益只有烤桦栗果与木炭，且木炭分两段。** `SetLoot({})` 清掉原表后只留 10% `acorn` 机会（实际落为 `acorn_cooked`）；10 秒自然掉落或被砍时的 `DropLoot` 每次都会再按 `burnt` 规则掷 40% `charcoal`，先砍还会先 `SpawnLootPrefab("charcoal")` 必给 1 个。不要指望烧完还能拿 `log`。[onburntchanges](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L651-L687)、[burnt 规则](../../../raw/current/scripts/components/lootdropper.lua#L252-L253)、[熟化转换](../../../raw/current/scripts/components/lootdropper.lua#L418-L441)
- **边界：桦栗树自然再生只在春天推进。** `TimeMultipliers["deciduoustree"]` 在非春返回 0，扩散计时不走，所以秋冬不会冒新树苗。[季节倍率](../../../raw/current/scripts/components/plantregrowth.lua#L33-L35)
- **未核验**：怪物树的战斗数值（`DECID_MONSTER_DAMAGE` 等）属于 `deciduoustreeupdater` 与 SG，本页只核对了变身与掉落分支；`spook` 与活动期皮肤未覆盖。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检烧毁双路径、`decay` 清理与怪物预约上限）。核对 `deciduoustrees.lua` 全文（`builds`、三阶段 setter、换叶与换季、`chop_tree`／`chop_down_tree`／`make_stump`／`dig_up_stump`、怪物分支、烧毁链、雪与月雹、存档字段、六注册）、`planted_tree.lua` 的 `growtree` 与 `acorn_sapling` 注册、`growable.lua` 的 `DoGrowth`／`StartGrowing`、`plantregrowth.lua` 的季节倍率、`standardcomponents.lua` 的雪与月雹 helper、`simutil.lua` 的 `SpringGrowthMod`、`tuning.lua` 的 `DECIDUOUS_GROW_TIME`／`DECIDUOUS_CHOPS_*`／`DECIDUOUS_REGROWTH`／`DECID_MONSTER_*`／`TREE_BURN_TIME`／`SPRING_GROWTH_MODIFIER`／`LUNARHAIL_BUILDUP_*`。
  名称核验：`STRINGS.NAMES.DECIDUOUSTREE`＝桦栗树（键名没有第二个下划线，`DECIDUOUS_TREE` 不存在）；六个注册共用该键。[po](../../../raw/current/scripts/languages/chinese_s.po#L342005-L342007)
  未覆盖：怪物树的战斗数值与 SG 细节、`deciduous_root`、活动期皮肤、`leafpile` 与其他掉落来源。
- 游戏运行：未运行。秒数由 `day_time`＝300 秒推导；怪物概率为源码常量，未做实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 均通过；Lua 示例已过语法解析。

## 相关链接

- 种下入口与树苗：[[reference/prefabs/acorn]]
- 木头与采集材料：[[reference/prefabs/basic-gathered-materials]]
- 同批木料与燃烧残留：[[reference/prefabs/driftwood-livinglog]]、[[reference/prefabs/ashes-and-charcoal]]
- 机制页：[[reference/components/growable]]、[[reference/components/workable]]、[[reference/components/lootdropper]]、[[reference/components/plantregrowth]]

返回 [[reference/prefabs/index|Prefab 索引]]。
