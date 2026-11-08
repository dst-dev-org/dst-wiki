---
title: "活树根种植与树精召唤（livingtree_root、leif_idol；附 lunar_seed）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "三件常被当成「特殊树种子」的物品的真实入口：livingtree_root 走 DEPLOYMODE.PLANT 生成 livingtree_sapling 并在约两段 900±45 秒后长成显示名仍为「完全正常的树」的 livingtree_halloween，挖苗时机决定回根还是回活木；leif_idol 不可种，只有点燃烧毁（每件 2 的上限、按当时堆叠数倍乘）与当燃料（每件 2 个）两条唤醒树精并按额度预约转换的路；lunar_seed 是天体珠宝，无 deployable，靠戴满它的天体守卫帽用地面充能月玻璃种出进化守卫。"
aliases: ["livingtree_root", "livingtree_sapling", "livingtree_halloween", "leif_idol", "lunar_seed", "livingtree_root_placer", "完全正常的树根", "完全正常的树苗", "完全正常的树", "树精守卫雕像", "天体珠宝", "月树种子", "树精雕像", "活树根"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/livingtree_root.lua", "prefabs/livingtree_root_planted.lua", "prefabs/livingtree_halloween.lua", "prefabs/lunar_seed.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/leif_idol.lua", "prefabs/leif.lua", "prefabs/evergreens.lua", "prefabs/pigking.lua", "prefabs/skilltree_woodie.lua", "prefabs/witchbroom.lua", "prefabs/madscience_lab.lua", "prefabs/hats.lua", "containers.lua", "brains/leifbrain.lua", "components/deployable.lua", "components/map.lua", "components/stackable.lua", "components/tradable.lua", "components/burnable.lua", "components/growable.lua", "prefabutil.lua", "recipe.lua", "recipes.lua", "components/builder_replica.lua", "standardcomponents.lua", "componentutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 活树根种植与树精召唤（livingtree_root、leif_idol；附 lunar_seed）

## 快速判断

- 职责与适用：三件常被当作「特殊树种子」的物品里只有一件能种。`livingtree_root`（完全正常的树根）走 `DEPLOYMODE.PLANT`，种下生成 `livingtree_sapling`（完全正常的树苗），两段 `LIVINGTREE_YOUNG_GROW_TIME` 后长成 `livingtree_halloween`（显示名仍是「完全正常的树」）。`leif_idol`（树精守卫雕像）不可种，是伍迪技能解锁的配方物品，只有「点燃烧毁」与「当燃料」两条召唤树精的路。`lunar_seed`（天体珠宝）没有 `deployable`，不是月树种子，而是天体守卫的掉落物与天体守卫帽的容器件。[活树根](../../../raw/current/scripts/prefabs/livingtree_root.lua#L143-L160)、[树精雕像](../../../raw/current/scripts/prefabs/leif_idol.lua#L126-L176)、[天体珠宝](../../../raw/current/scripts/prefabs/lunar_seed.lua#L6-L52)
- 优先使用：种活树与挖回看[活树根的种植与本树苗](#活树根的种植与本树苗)；改雕像召唤看[树精雕像的两个入口](#树精雕像的两个入口)；核对天体珠宝看[天体珠宝的真实身份](#天体珠宝的真实身份)；月树本体与月树苗见 [[reference/prefabs/moontree]]。
- 重要副作用：活树根种下会像松果一样安抚半径 16 内的树精；树精雕像烧毁时先走 `DefaultBurntFn`（海上不产灰、灰烬按堆叠封顶）再唤醒半径 20 内的树精，并在 10 半径内最多预约「2 × 当时堆叠数」棵候选（常青树变成树精、桦栗树走延迟怪物化，两种结果）；当燃料放进火里则固定按 2 棵算。[安抚](../../../raw/current/scripts/prefabs/livingtree_root.lua#L50-L69)、[烧毁](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L109)、[当燃料](../../../raw/current/scripts/prefabs/leif_idol.lua#L111-L124)
- 修改边界：树精本体（`leif`／`leif_sparse`）的数值、Brain 与掉落只作关系说明；活树长成后的容器／砍伐只列入口；天体守卫帽的完整机制、月玻璃消耗与宠物链细节不在本页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `livingtree_root` | `fn`：公共段 → `SetPristine` → 主模拟；主模拟末尾 `ondropped` 起心跳任务 | tag `deployedplant`；`MakeInventoryFloatable("small",0.2)`；`scrapbook_specialinfo="PLANTABLE"`；另挂 `useabletargeteditem`（对女巫扫帚生效）。[构造](../../../raw/current/scripts/prefabs/livingtree_root.lua#L91-L157) |
| `livingtree_root_placer` | `MakePlacer("livingtree_root_placer","livingtree_root","livingtree_root","placer")` | 与实物同 bank／build，动画名是 `placer`（不是 `idle_planted`）；placer 带 `CLASSIFIED`／`NOCLICK`／`placer` 标签且不存档。[注册](../../../raw/current/scripts/prefabs/livingtree_root.lua#L159-L160)、[MakePlacer](../../../raw/current/scripts/prefabutil.lua#L1-L19) |
| `livingtree_sapling` | `livingtree_root_planted.lua` 的 `fn`：tag `plant` → `SetPristine` → 主模拟 | 显示名 完全正常的树苗；`scrapbook_proxy="livingtree"`；`SetDeploySmartRadius(DEFAULT/2)`＝1。[构造](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L55-L107) |
| `livingtree_halloween` | `livingtree_halloween.lua` 的 `fn`：外形与容器共用，`SetPrefabNameOverride("livingtree")` | 名称取 `NAMES.LIVINGTREE`＝完全正常的树，所以种出来的树不显示「万圣夜」字样。[改名](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L303-L325) |
| `leif_idol` | `fn`：公共段 → `SetPristine` → 主模拟 | 显示名 树精守卫雕像；没有 `deployable`、没有 placer，不属于种植链。[构造](../../../raw/current/scripts/prefabs/leif_idol.lua#L126-L176) |
| `lunar_seed` | `seedfn`：带 `DynamicShadow`／`Light` 的发光物品，公共段 → `SetPristine` → 主模拟 | 显示名 **天体珠宝**；tag `lunarseed`；无 `deployable`。[构造](../../../raw/current/scripts/prefabs/lunar_seed.lua#L6-L50) |

### 名称核验

`NAMES.LIVINGTREE_ROOT`＝完全正常的树根、`LIVINGTREE_SAPLING`＝完全正常的树苗、`LIVINGTREE`＝完全正常的树、`LEIF_IDOL`＝树精守卫雕像、`LUNAR_SEED`＝天体珠宝。也就是说「月树种子」不是这件的简中名；月树苗的实显名是「月树苗」，归 `moonbutterfly_sapling`。[树根](../../../raw/current/scripts/languages/chinese_s.po#L344765-L344767)、[树苗](../../../raw/current/scripts/languages/chinese_s.po#L344770-L344772)、[树](../../../raw/current/scripts/languages/chinese_s.po#L344760-L344762)、[雕像](../../../raw/current/scripts/languages/chinese_s.po#L344690-L344692)、[珠宝](../../../raw/current/scripts/languages/chinese_s.po#L344930-L344932)

## 活树根的种植与本树苗

### 门禁与部署

- 走 `DEPLOYMODE.PLANT`：`Map:CanDeployPlantAtPoint` 要求陆地且不在 `GROUND_HARD`，再按 `DEPLOYSPACING_RADIUS[DEPLOYSPACING.DEFAULT]`＝2 查空位（`DEPLOYSPACING.DEFAULT` 的值是 0，2 出自半径表）——与 [[reference/prefabs/tree-seeds#种植门禁与落点]] 同一套规则。[PLANT 分支](../../../raw/current/scripts/components/deployable.lua#L112-L113)、[地块判定](../../../raw/current/scripts/components/map.lua#L167-L175)
- `ondeploy`：`stackable:Get()` → `Physics:Teleport(pt)` → `SpawnPrefab("livingtree_sapling")` → 播 `plant_tree` → 删原件；随后遍历半径 `LEIF_PINECONE_CHILL_RADIUS`＝16 内的 `leif`。[ondeploy](../../../raw/current/scripts/prefabs/livingtree_root.lua#L41-L70)
- 安抚抽签与松果逐行相同：近档是 `dist² < 5²`（**严格小于 5**）取基础概率 0.33，其余半径 16 内取 0.15；两者都只是 `TryLuckRoll(deployer, chance, LuckFormulas.LeifChill)` 的基础输入，实际概率还受操作者运气修正；抽中后还要目标有 `sleeper` 才 `GoToSleep(1000)`（1000 是睡眠接口的时长参数）并记 `pacify_forest` 成就；失败只播一次 `taunt_VO`。[常量](../../../raw/current/scripts/tuning.lua#L1060-L1064)、[幸运式](../../../raw/current/scripts/componentutil.lua#L2237-L2251)

### 树苗的三段生长

`growable` 在进入阶段时跑该段的 `fn`，下一段时长由新阶段的 `time` 现算；`StartGrowing` 读的是**当前**阶段的 `time`。[调度](../../../raw/current/scripts/components/growable.lua#L42-L79)、[推进](../../../raw/current/scripts/components/growable.lua#L121-L151)

| 阶段 | 时长（`GetRandomWithVariance`） | 进入时做什么 | 证据 |
|---|---|---|---|
| 1 | `LIVINGTREE_YOUNG_GROW_TIME`＝900 秒 ±5%＝±45 秒 | 播 `idle_planted_flask`；掉落表仍是 `{"livingtree_root"}` | [阶段表](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L33-L38)、[常量](../../../raw/current/scripts/tuning.lua#L2889-L2893) |
| 2 | 同上（900±45 秒） | 播 `idle_planted`、`growfn` 播 `burst`；掉落表改 `{"livinglog"}` | [阶段表](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L40-L42) |
| 3 | 无 `time` | `StopGrowing` → `DoTaskInTime(0, growtree)` → 生成 `livingtree_halloween` 并 `growfromseed()` | [阶段表](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L45-L45)、[growtree](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L14-L21) |

- 挖掘：树苗挂 `workable(ACTIONS.DIG)` 1 次，完成回调 `DropLoot` 后删除自己——所以**挖早回根、挖晚回活木**：阶段 1 给 `livingtree_root`，阶段 2 起只给 1 个 `livinglog`。[挖苗](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L50-L53)、[掉落表](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L91-L97)
- 可燃：`MakeSmallBurnable(TUNING.SMALL_BURNTIME)`＋`MakeSmallPropagator`，作祟是 `MakeHauntableIgnite`（点燃分支已被注释，见[常见陷阱](#常见陷阱与限制)）。[接线](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L99-L102)

### 长成的树

种出来的是 `livingtree_halloween`，不是世界生成的 `livingtree`。它挂着 2 个阶段、但**只有一段生长**：种下即阶段 1（`young`），`growfromseed` 只负责 `StartGrowing` 并播 `grow_seed_to_young`；young→old 用一段 `LIVINGTREE_YOUNG_GROW_TIME`＝900±45 秒，进入阶段 2（`old`）时该段没有 `time`、`SetGrowth` 直接 `StopGrowing`，此后不再生长。工作量为 `LIVINGTREE_YOUNG_WORK`＝15／`LIVINGTREE_WORK`＝20，砍倒掉落 young 1 个、old 2 个 `livinglog`。[阶段表](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L55-L65)、[收尾](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L37-L47)、[种下](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L242-L247)、[常量](../../../raw/current/scripts/tuning.lua#L2889-L2893)

- 构造时先按 `workable:SetWorkLeft(20)` 写死，进入阶段后由 `SetGrowth` 改成该段的值。[工作量](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L337-L341)
- 容器：`WidgetSetup("livingtree_halloween")`、tag `fridge`，但 `canbeopened` 只在万圣夜为真；非活动期打开会失败。[容器](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L343-L350)
- 眼睛火焰、`sanityaura`（-`SANITYAURA_MED`）与容器一样只在万圣夜启用；`MakeLargeBurnable`＋`MakeMediumPropagator` 恒在。[分叉](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L316-L356)

### 活树根的其它入口

- **女巫扫帚附魔**：`useabletargeteditem` 只接受 `witchbroom`（且不带 `flyingbroom`），`MakeLiving` 把扫帚变成活体并补满耐久；完整链条见 [[reference/prefabs/tools-misc]]。[判定](../../../raw/current/scripts/prefabs/livingtree_root.lua#L78-L89)、[消费](../../../raw/current/scripts/prefabs/witchbroom.lua#L94-L114)
- **燃料**：`LARGE_FUEL`＝180 秒（`seg_time`×6；90 是 `MED_LARGE_FUEL`），`ontaken` 只对 `campfire` 播烟效果，不改燃料值。[燃料](../../../raw/current/scripts/prefabs/livingtree_root.lua#L127-L132)
- **心跳动画**：丢在地上时每 15＋0–15 秒播一次 `idle`，捡起即取消；这是纯表现，不影响任何判定。[心跳](../../../raw/current/scripts/prefabs/livingtree_root.lua#L15-L38)
- **来源**：①挖起阶段 1 的 `livingtree_sapling`（见上）；②万圣夜实验台 `halloween_experiment_root` 的产物表只含 1 个 `livingtree_root`，生成数按 1–3 加权。[实验台](../../../raw/current/scripts/prefabs/madscience_lab.lua#L45-L75)

## 树精雕像的两个入口

### 获得与构造

- 配方 `leif_idol`：3 个 `cutgrass`＋2 个 `livinglog`＋5 个 `nightmarefuel`，`TECH.NONE`，但带 `builder_skill="woodie_human_treeguard_max"`；该技能在伍迪技能树的「treeguard」第三格，注释写明「可制作 leif_idol」。[配方](../../../raw/current/scripts/recipes.lua#L172-L172)、[技能节点](../../../raw/current/scripts/prefabs/skilltree_woodie.lua#L449-L454)
- 门禁落在 `skilltreeupdater:IsActivated(recipe.builder_skill)`：只有激活了该技能的人物能解锁，其它角色即使 `TECH.NONE` 也做不出来。[校验](../../../raw/current/scripts/components/builder_replica.lua#L305-L316)、[登记](../../../raw/current/scripts/recipe.lua#L136-L140)
- 本体：堆叠 `STACK_SIZE_LARGEITEM`＝10；`fuel`＝`LARGE_FUEL`＝180 秒且 `SetOnTakenFn(OnFuelTaken)`；`MakeSmallBurnable(TUNING.TINY_BURNTIME)`＝3 秒并把 `OnBurnt`／`OnIgnite` 换成自定义函数；`MakeHauntableLaunchAndIgnite`；对模组开放 `WakeUpNearbyLeifs`／`SpawnNewLeifs`／`CanTransformIntoLeifTest`。[组件](../../../raw/current/scripts/prefabs/leif_idol.lua#L148-L171)、[常量](../../../raw/current/scripts/tuning.lua#L7825-L7826)

### 入口 A：点燃烧毁

1. `OnIgnite(inst, source, doer)` 播 `livinglog_burn` 并记下 `_igniter`：先试 `source`、再试 `doer`，必须是有 `combat` 组件的实体实例，否则留空。[点火](../../../raw/current/scripts/prefabs/leif_idol.lua#L89-L93)、[回调签名](../../../raw/current/scripts/components/burnable.lua#L352-L377)
2. 烧完 `OnBurnt`：先 `DefaultBurntFn`（非海上生成 `ash`，堆叠取 `min(ash 上限, 原堆叠)`，然后删实体），再取**当时的** `stackable:StackSize()` 当倍数。[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[顺序](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L109)
3. `doer` 取 `_igniter`，没有就取半径 15 内最近玩家；随后 `WakeUpNearbyLeifs`＋`SpawnNewLeifs(x,y,z,doer,stacksize)`。

### 入口 B：当燃料放进火里

`fuel:SetOnTakenFn(OnFuelTaken)` 在投进火堆时触发：播同一音效、用**火堆**的位置、`doer` 取半径 15 内最近玩家，然后 `WakeUpNearbyLeifs`＋`SpawnNewLeifs`（倍数参数缺省 → 1）。[入口](../../../raw/current/scripts/prefabs/leif_idol.lua#L111-L124)

### 唤醒与生成

| 步骤 | 实际条件 | 证据 |
|---|---|---|
| 唤醒 | `FindEntities(x,y,z,TUNING.LEIF_REAWAKEN_RADIUS)` 半径 20 内所有 `leif` 标签实体；只在 `sleeper:IsAsleep()` 时排一个 `math.random()` 秒的 `WakeUp`；有 `doer` 时对每只 `combat:SuggestTarget(doer)`（只是建议目标） | [唤醒](../../../raw/current/scripts/prefabs/leif_idol.lua#L44-L58)、[常量](../../../raw/current/scripts/tuning.lua#L1060-L1064) |
| 生成数 | `TUNING.LEIF_IDOL_NUM_SPAWNS`＝**每件 2**，乘倍数后作为本次预约上限；只有真命中一棵带 `TransformIntoLeif` 或 `StartMonster` 的候选才减 1，减到 0 立刻 `break`；候选遍历完但额度未归零时不 `break`，直接把剩余额度返回——所以是上限，不是保证产出 | [入口](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L83)、[常量](../../../raw/current/scripts/tuning.lua#L7825-L7826) |
| 候选筛选 | 半径 `LEIF_IDOL_SPAWN_RADIUS`＝10；必须 `tree`；至少带 `evergreens` 或 `birchnut` 之一；排除 `leif`／`fire`／`stump`／`burnt`／`monster`／`FX`／`NOCLICK`／`DECOR`／`INLIMBO` | [标签表](../../../raw/current/scripts/prefabs/leif_idol.lua#L10-L14) |
| 合格判定 | 常青树：`not target.noleif` 且有 `growable` 且 `stage <= 3`；桦栗树：`leaf_state ~= "barren"` 且没有 `monster`／`monster_start_task`／`monster_stop_task`／`domonsterstop_task` | [判定](../../../raw/current/scripts/prefabs/leif_idol.lua#L16-L33) |
| 执行 | 常青树走 `ent:TransformIntoLeif(doer)`：先置 `noleif`、记下砍伐者，`DoTaskInTime(1 + math.random()*3)` 后 `SpawnPrefab` 并删树（同样 1–4 秒，只是结果换成 `leif`／`leif_sparse`）；桦栗树走 `ent:StartMonster()`，`DoTaskInTime(math.random(1,4))` 后变怪物桦栗树 | [分派](../../../raw/current/scripts/prefabs/leif_idol.lua#L65-L80)、[延迟与 noleif](../../../raw/current/scripts/prefabs/evergreens.lua#L489-L494)、[spawn_leif](../../../raw/current/scripts/prefabs/evergreens.lua#L442-L457) |

## 天体珠宝的真实身份

- 构造：`AnimState` 用 `lunar_seed` bank／build 并播循环 `idle`，`SetBloomEffectHandle` 与四个 `SetSymbolLightOverride` 做发光；`MakeInventoryFloatable(nil, 0.13, 0.9)`（尺寸走默认 `small`）；tag `lunarseed`。[发光与漂浮](../../../raw/current/scripts/prefabs/lunar_seed.lua#L16-L31)
- 主模拟只有 `inspectable`／`inventoryitem`／`tradable`（**未设 `goldvalue`**）／`stackable`（未设上限 → 组件默认 20）／`MakeHauntableLaunch`，另把动画帧随机一次；**没有** `deployable`、`burnable`、`edible`、`fuel`。[组件](../../../raw/current/scripts/prefabs/lunar_seed.lua#L33-L49)
- 因为 `goldvalue` 是 0，猪王不收（判据 `goldvalue > 0` 或万圣夜糖果）。[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L572-L580)
- 来源：天体守卫（`alterguardian_phase4_lunarrift`）的共享战利品表给 5 个 `lunar_seed`，权重都是 1.0。[战利品表](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L46-L52)
- 实际用途在天体守卫帽：容器 5 格，`itemtestfn` 只收带 `spore` 或 `lunarseed` 标签的物品。[容器](../../../raw/current/scripts/containers.lua#L1821-L1824)
- 装满 5 个珠宝（`lunarseedscount == GetNumSlots()`）只是**达到启动条件**：`lunarseedplanting_start` 还要求 `equippable:IsEquipped()`，没戴在头上不会开周期任务（tick 里还会再查 `petleash` 是否存在、该宠物槽是否已满）。数量同时决定额外物理伤害与位面转化（3 个起满档 0.25）。[计数与加成](../../../raw/current/scripts/prefabs/hats.lua#L3375-L3387)、[常量](../../../raw/current/scripts/tuning.lua#L6213-L6216)
- 「种植」不是种树，材料是**充能**月玻璃：tick 先要求持有者有 `petleash` 且 `IsFullForPrefab("gestalt_guard_evolved")` 为假，在帽主 8 半径内找 `moonglass_charged`；再以该片为中心、4 半径内累计**未被预约**的 `moonglass_charged` 堆叠数，凑够 `GESTALT_EVOLVED_PLANTING_MOONGLASS_REQUIREMENT`＝3 才预约宠物、给这些片打 `_pending_lunarseedplant`，并投出 `gestalt_evolved_planting_visual_projectile`。投射物到达时重新找料，移除时若未被标记失败再找一次料，确认后才按 3 件扣除并 `SpawnPetAt` 生成 `gestalt_guard_evolved`。普通 `moonglass` 不够资格——筛选按 `moonglass_piece` 标签取实体后还要求 `prefab == "moonglass_charged"`。这段代码只移除月玻璃，未见移除珠宝的调用（检索范围：`hats.lua` 的 `lunarseedplanting_*` 与 `OnRemove` 分支）。[查找](../../../raw/current/scripts/prefabs/hats.lua#L3205-L3222)、[消耗与生成](../../../raw/current/scripts/prefabs/hats.lua#L3236-L3260)、[周期](../../../raw/current/scripts/prefabs/hats.lua#L3345-L3347)、[常量](../../../raw/current/scripts/tuning.lua#L8796-L8800)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`deployable` | 只有 `livingtree_root` 有：PLANT＋`ondeploy` | 仅主模拟持有 | [[reference/components/deployable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/livingtree_root.lua#L143-L145) |
| 组件：`growable` | 树苗 3 段（`growoffscreen=true`）、成树 2 段 | 主模拟 | [[reference/components/growable#关键接口]]；[阶段](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L83-L87) |
| 组件：`workable` | 树苗 1 次 DIG；成树 20／15 次 CHOP | 主模拟 | [[reference/components/workable#关键接口]]；[挖苗](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L94-L97)、[砍伐](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L337-L341) |
| 组件：`fuel`／`burnable` | 活树根 180 秒燃料（`LARGE_FUEL`）＋小号可燃；雕像 180 秒燃料＋**3 秒**可燃并改写 `OnBurnt`／`OnIgnite` | 主模拟；雕像烧毁会召唤 | [[reference/components/fuel]]、[[reference/components/burnable#关键接口]]；[雕像](../../../raw/current/scripts/prefabs/leif_idol.lua#L156-L171) |
| 组件：`stackable` | 活树根 20、雕像 10、珠宝默认 20；雕像烧毁时读堆叠当倍数 | 主模拟与 replica；20 是组件默认上限 | [[reference/components/stackable#关键接口]]；[活树根](../../../raw/current/scripts/prefabs/livingtree_root.lua#L122-L123)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L34) |
| 组件：`useabletargeteditem` | 活树根对女巫扫帚调用 `MakeLiving` | 主模拟；只在目标 `prefab == "witchbroom"` 且非飞行态 | [[reference/prefabs/tools-misc]]；[接线](../../../raw/current/scripts/prefabs/livingtree_root.lua#L147-L149) |
| 组件：`sleeper`（树精侧） | 安抚与唤醒都改它的睡眠；树精自身 `SetResistance(3)` | 主模拟 | [[reference/components/sleeper]]；[树精](../../../raw/current/scripts/prefabs/leif.lua#L178-L179) |
| SG／Brain：`SGLeif`／`leifbrain` | 树精被唤醒后的行为：`AttackWall`→`ChaseAndAttack`→`Wander` | 主模拟 | [Brain](../../../raw/current/scripts/brains/leifbrain.lua#L12-L22)、[SG 与 Brain](../../../raw/current/scripts/prefabs/leif.lua#L145-L199)（Wiki 待建） |
| 关联 Prefab：`leif`／`leif_sparse` | 雕像召唤的结果；种类由被替换的树 build 决定 | 主模拟；`TransformIntoLeif` 在树侧 | [[reference/prefabs/evergreen-trees#树精生成]]；[注册](../../../raw/current/scripts/prefabs/leif.lua#L208-L217) |
| 关联 Prefab：`livingtree_halloween` | 活树根种出的成树；显示名借 `livingtree` | 主模拟 | [构造](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L303-L325)（Wiki 待建） |
| 关联 Prefab：`moonglass_charged`／`gestalt_guard_evolved` | 天体珠宝的「种植」目标与产物 | 主模拟；宠物链（`petleash`） | [查找](../../../raw/current/scripts/prefabs/hats.lua#L3205-L3222)（Wiki 待建） |
| 资源：`anim/livingtree_root.zip`／`anim/lunar_seed.zip`／`anim/leif_idol.zip` | 物品、树苗与 placer 共用动画包 | 原版资源，不随页分发 | [活树根 assets](../../../raw/current/scripts/prefabs/livingtree_root.lua#L3-L13)、[珠宝 assets](../../../raw/current/scripts/prefabs/lunar_seed.lua#L1-L4) |

## 典型接入

### 改树精雕像的召唤数量

- 前提与选择：服务端 `modmain.lua` 直接改 `TUNING.LEIF_IDOL_NUM_SPAWNS`。`SpawnNewLeifs` 每次调用都现读该常量，所以不用 prefab postinit，也不用改组件。
- 生效与撤销：影响所有 `leif_idol`。燃烧路径把当时堆叠数当倍数、燃料路径倍数恒为 1，所以最多预约「2 × 堆叠」与「2」棵；示例改成 4 后即「4 × 堆叠」与「4」棵。两件一叠烧毁：原版上限 4、改后 8；燃料入口：原版 2、改后 4。撤掉模组后回默认 2。
- 原版依据：`SpawnNewLeifs` 第一行就是 `TUNING.LEIF_IDOL_NUM_SPAWNS * (multiplier or 1)`；数量再受半径 10 内合格树数限制，不够就少生成。[入口](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L83)、[常量](../../../raw/current/scripts/tuning.lua#L7825-L7826)

```lua
-- modmain.lua：树精雕像每次最多召唤 4 个（默认 2）
GLOBAL.TUNING.LEIF_IDOL_NUM_SPAWNS = 4
```

- 调整与验证：分两种情形验收同一次改动。①**合格树充足**（普通常青树／可怪物化的桦栗树都在 10 半径内）：改值 4 后，烧一叠 2 个应最多预约 8 次、燃料入口最多 4 次；原版对应 4 与 2。②**合格树不足**（只有 2 棵）：两种入口都只预约 2 棵，遍历耗尽即结束、**剩余额度保留**（`break` 只在额度归零时发生），这不代表改动失效——上限只在树够时才体现。另注意常青树是 1–4 秒后生成树精、桦栗树同延迟但变怪物桦栗树，两者都只在目标真有 `TransformIntoLeif`／`StartMonster` 时才消耗额度。

## 常见陷阱与限制

- **误区：`lunar_seed` 是月树种子，种下能长月树。** 它没有 `deployable`，既不能种也不能烧；月树苗 `moonbutterfly_sapling` 由月树再生、荒漠化重生与月蛾放生产生（见 [[reference/prefabs/moontree]]）。[无部署](../../../raw/current/scripts/prefabs/lunar_seed.lua#L33-L49)
- **误区：`leif_idol` 是「种树精」的种子。** 它没有 `deployable`，只有点燃与当燃料两个入口；两个入口都需要附近有合格树可换，否则只唤醒已有的树精。[构造](../../../raw/current/scripts/prefabs/leif_idol.lua#L148-L171)
- **边界：两条路径的数量口径不同，且都只是上限。** 每件基础量是 2：烧毁再乘当时堆叠数（一叠 10 个即 20 棵的预约上限），当燃料倍数恒 1（2 次）；两者都受半径 10 与候选数量限制——候选不足时只是遍历耗尽、剩余额度保留，`break` 只在额度归零时发生；也只在命中真正可转换的候选时才减额度。[烧毁](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L109)、[燃料](../../../raw/current/scripts/prefabs/leif_idol.lua#L111-L124)
- **边界：两条路都延迟 1–4 秒，但结果不同。** 常青树走 `TransformIntoLeif`（置 `noleif` → 1–4 秒后生成树精并删树），桦栗树走 `StartMonster()`（1–4 秒后变成怪物桦栗树），所以「雕像一烧完就变」不成立。[分派](../../../raw/current/scripts/prefabs/leif_idol.lua#L65-L80)
- **边界：老树、`noleif` 树与燃烧中的树不能变。** 判定要求 `growable.stage <= 3` 且 `target.noleif` 为假；`noleif` 一旦在 `TransformIntoLeif` 里被置真就不再清除，同一棵树不会被换第二次。候选排除表还含 `fire`／`burnt`／`stump`，而点火时 `burnable` 就给实体加 `fire` 标签，所以燃烧中的树当场被排除，不存在「正在烧但还能换」的窗口。[判定](../../../raw/current/scripts/prefabs/leif_idol.lua#L16-L33)、[排除表](../../../raw/current/scripts/prefabs/leif_idol.lua#L14-L14)、[noleif](../../../raw/current/scripts/prefabs/evergreens.lua#L489-L494)、[fire 标签](../../../raw/current/scripts/components/burnable.lua#L12-L15)
- **边界：烧毁先出灰再召唤。** `DefaultBurntFn` 在召唤之前执行并删除实体；海上不生成 `ash`，但召唤照常进行，所以「海上烧雕像」不会掉灰。灰烬堆叠取 `min(ash 上限, 原堆叠)`。[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- **边界：活树苗挖早有赚、挖晚亏。** 阶段 1 挖回 1 个 `livingtree_root`，阶段 2 起只回 1 个 `livinglog`。[掉落](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L40-L42)
- **边界：长出的是 `livingtree_halloween`。** 它靠 `SetPrefabNameOverride("livingtree")` 才显示「完全正常的树」；容器与眼睛火焰只在万圣夜可用，非活动期打开会失败。[改名](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L316-L325)
- **边界：作祟不点燃。** `MakeHauntableLaunchAndIgnite`（活树根、雕像）的点燃分支、`MakeHauntableIgnite`（树苗）的整个点火分支都带 `#HAUNTFIX` 注释，当前只弹飞（`MakeHauntableLaunchAndIgnite` 恒 `return false`）或什么都不做。[弹飞](../../../raw/current/scripts/standardcomponents.lua#L1204-L1215)、[点燃](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)
- **未核验**：游戏内运行；树精战斗、`leifbrain` 的实际决策；活树成长时的容器交互；月玻璃与宠物链的完整流程；活动皮肤（`livingtree_root_hallowed_nights` 只核到换图与换动画名）。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `livingtree_root.lua`、`livingtree_root_planted.lua`、`leif_idol.lua`、`lunar_seed.lua` 全文；核对 `livingtree_halloween.lua` 的阶段／容器／改名，`leif.lua` 的组件与掉落，`leifbrain.lua` 的节点顺序，`skilltree_woodie.lua` 与 `recipes.lua` 的技能配方门禁，`builder_replica.lua`／`recipe.lua` 的 `builder_skill` 校验，`madscience_lab.lua` 的实验产物表，`hats.lua` 与 `containers.lua` 的天体珠宝容器与种植流程，`alterguardian_phase4_lunarrift.lua` 的战利品表，`evergreens.lua` 的 `TransformIntoLeif`／`spawn_leif` 与 `pigking.lua` 的猪王门禁，以及 `deployable`／`map`／`burnable`／`growable`／`workable`／`stackable`／`tradable`／`standardcomponents` 的相关分支与 `tuning` 常量。
  名称核验：`NAMES.LIVINGTREE_ROOT`＝完全正常的树根、`LIVINGTREE_SAPLING`＝完全正常的树苗、`LIVINGTREE`＝完全正常的树、`LEIF_IDOL`＝树精守卫雕像、`LUNAR_SEED`＝天体珠宝；`livingtree_root_placer` 无 `NAMES` 键。
  未覆盖：树精战斗与 Brain 实测、容器交互、月玻璃消耗细节、活动皮肤。
- 游戏运行：未运行。秒数按 `day_time`＝300／`total_day_time`＝480 与 `seg_time`＝30 推算；概率为源码常量。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警（计数见校验输出与交付回报）。Lua 示例用 `luaparser` 4.2.0 解析通过（非游戏运行时）。

## 相关链接

- 树种与种植门禁：[[reference/prefabs/tree-seeds]]、[[reference/prefabs/acorn]]
- 常青树与树精：[[reference/prefabs/evergreen-trees]]
- 月树与月树苗：[[reference/prefabs/moontree]]
- 活木头与女巫扫帚：[[reference/prefabs/driftwood-livinglog]]、[[reference/prefabs/tools-misc]]
- 机制页：[[reference/components/deployable]]、[[reference/components/growable]]、[[reference/components/burnable]]、[[reference/components/fuel]]

返回 [[reference/prefabs/index|Prefab 索引]]。
