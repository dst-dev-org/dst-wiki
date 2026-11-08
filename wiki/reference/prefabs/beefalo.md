---
title: "皮弗娄牛（beefalo）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "皮弗娄牛宿主：牛群成员与中心漂移、剃毛／刷毛的周期与上限、三天牛毛再生、成牛普通与燃烧死亡掉落；驯化、骑乘、牛铃与鞍只给入口。"
aliases: ["beefalo", "beefalo_carry", "皮弗娄牛", "牛", "牛群", "刷牛毛", "剃牛毛", "牛毛再生"]
tags: [prefab, creature, loot]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/beefalo.lua", "prefabs/beefaloherd.lua", "prefabs/babybeefalo.lua", "prefabs/beef_bell.lua", "prefabs/saddle.lua", "prefabs/beefalo_groomer.lua", "brains/beefalobrain.lua", "stategraphs/SGBeefalo.lua", "components/beard.lua", "components/brushable.lua", "components/herd.lua", "components/herdmember.lua", "components/lootdropper.lua", "components/domesticatable.lua", "components/rideable.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "componentutil.lua", "entityscript.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# 皮弗娄牛（beefalo）

## 快速判断

- 职责与适用：`prefabs/beefalo.lua` 注册的成年皮弗娄牛（S1）宿主，另有同文件辅助注册 `beefalo_carry`。本页覆盖牛群成员关系与群体中心漂移、剃毛／刷毛、牛毛再生、成牛死亡掉落；驯化、骑乘、牛铃与鞍只给入口。[注册](../../../raw/current/scripts/prefabs/beefalo.lua#L1349-L1350)
- 优先使用：查“牛群怎么走、为什么散开”看[牛群与迁移](#牛群与迁移)；查“什么时候能剃／刷、给几份”看[剃毛与刷毛](#剃毛与刷毛)与[牛毛再生](#牛毛再生)；查死亡产物看[死亡掉落](#死亡掉落)。
- 重要副作用：剃毛一次 3 份牛毛并把牛剃秃、服从 −1；刷毛一次最多 1 份且不剃毛；成牛在非保尸、非 `file_load` 路径死亡保底 4 肉＋3 牛毛，牛角 33%；`DropLoot` 结算时仍在燃烧则肉变熟肉、牛毛与牛角变灰；充能暗影铃保尸会先熄火且不执行掉落，骑乘中死亡要等甩下骑手才进死亡状态。[剃毛](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L443)、[死亡表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[保尸条件](../../../raw/current/scripts/prefabs/beefalo.lua#L1055-L1063)
- 修改边界：牛铃、鞍、皮弗娄牛美妆台、装扮台、骑乘与驯化数值归各自页或后续批次；本页不展开倾向成长、服从衰减、鞍具耐久与骑乘 SG。[驯化入口](../../../raw/current/scripts/prefabs/beefalo.lua#L530-L555)、[骑乘组件](../../../raw/current/scripts/components/rideable.lua#L1-L40)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `beefalo` | `beefalo()`：实体→Transform／AnimState／DynamicShadow／MiniMap／Network→`MakeCharacterPhysics(100, .5)`→bank `beefalo`、build `beefalo_build`→pristine 标签→`SetPristine`；非主模拟返回，主模拟挂组件并设 Brain／SG。[工厂](../../../raw/current/scripts/prefabs/beefalo.lua#L1065-L1130) | 成年大型生物；pristine 标签 `beefalo`／`animal`／`largecreature`／`bearded`／`trader`／`herdmember`／`saddleable`／`domesticatable`／`saltlicker`／`has_beard`；生命 1000、默认伤害 34、走 1.0／跑 7。[标签](../../../raw/current/scripts/prefabs/beefalo.lua#L1090-L1113)、[生命与伤害](../../../raw/current/scripts/tuning.lua#L869-L878)、[战斗应用](../../../raw/current/scripts/prefabs/beefalo.lua#L1156-L1158)、[速度](../../../raw/current/scripts/tuning.lua#L890-L897)、[移动应用](../../../raw/current/scripts/prefabs/beefalo.lua#L1221-L1223) |
| `beefalo_carry` | `beefalo_carry()`：Transform／AnimState／Network→bank `beefalo`、build `beefalo_build`、播 `idle_carry`→`Transform:SetNoFaced()`→`SetPristine`；主模拟只设 `persists=false`。[辅助工厂](../../../raw/current/scripts/prefabs/beefalo.lua#L1322-L1346) | 无标签、无组件、无掉落；本版 scripts 内未找到它的 `SpawnPrefab` 生成点，不能当作可交互生物或物品。用途未核验。[注册](../../../raw/current/scripts/prefabs/beefalo.lua#L1350-L1350) |

- **名称核验**：`STRINGS.NAMES.BEEFALO` 为 **皮弗娄牛**（S1）；本 prefab 没有 `SetPrefabNameOverride`，注册名与显示名一致。[po](../../../raw/current/scripts/languages/chinese_s.po#L339264-L339267)
- **状态文案**：`GetStatus` 的优先级是——死亡时按是否手持牛铃给 `DEAD_MYPARTNER`／`DEAD`；有 leader 时给 `MYPARTNER`／`FOLLOWER`；无 leader 才按 `beard.bits==0` 给 `NAKED`，或按 `tendency` 给 `DOMESTICATED`／`RIDER`／`ORNERY`／`PUDGY`。这只是查看文案，不是额外状态机。[GetStatus](../../../raw/current/scripts/prefabs/beefalo.lua#L374-L391)

## 牛群与迁移

皮弗娄牛通过 `herdmember` 加入 `beefaloherd`：成年牛在构造末尾 `Enable(true)`，pristine 阶段已带 `herdmember` 标签；`herdmember` 的默认 `herdprefab` 是 `beefaloherd`。[成员启用](../../../raw/current/scripts/prefabs/beefalo.lua#L1243-L1245)、[默认群名](../../../raw/current/scripts/components/herdmember.lua#L19-L29)、[成员标签](../../../raw/current/scripts/prefabs/beefalo.lua#L1100-L1101)

- 群节点侧：`beefaloherd` 把成员标签设为 `beefalo`（另有 `migratory` 分支，见下），收集范围 40、更新范围 20；自身 `herd.maxsize` 用组件默认 12，繁殖另有 `BEEFALOHERD_MAX_IN_RANGE=16` 的范围上限。[群配置](../../../raw/current/scripts/prefabs/beefaloherd.lua#L113-L120)、[默认上限](../../../raw/current/scripts/components/herd.lua#L15-L18)、[繁殖上限](../../../raw/current/scripts/tuning.lua#L941-L942)
- 中心更新：`Herd:OnUpdate` 的周期是构造时 `math.random()*2+6` 取得的固定值（每实例只掷一次，不是每轮重掷），即该实例每 6–8 秒执行一次。它在更新范围 20 内、且成员不在战斗中（`updateposincombat` 默认 false）时取成员坐标平均，写回群节点，再把群坐标写进每个成员的 `knownlocations.herd`。[更新间隔](../../../raw/current/scripts/components/herd.lua#L31-L34)、[中心计算](../../../raw/current/scripts/components/herd.lua#L195-L241)
- 成员侧：牛 Brain 的 Wander 直接以 `knownlocations.herd` 为目标，另有 loiter 锚点；因此“迁移”是**成员向中心游荡、中心又随成员移动**的反馈漂移，不是按季节切换固定营地。[Wander](../../../raw/current/scripts/brains/beefalobrain.lua#L89-L100)、[群目标](../../../raw/current/scripts/brains/beefalobrain.lua#L252-L254)
- 合并与空群：每次更新还会尝试 `MergeNearbyHerds`；空群由 `beefaloherd` 的 `SetOnEmptyFn(inst.Remove)` 删除。合并会把成员先登记到新群再移除旧节点，不能当作无损搬群 API。[合并](../../../raw/current/scripts/components/herd.lua#L172-L193)、[空群](../../../raw/current/scripts/prefabs/beefaloherd.lua#L121-L123)
- 繁殖与幼牛：春季发情 3 天、间隔 20 天，幼牛延迟 1.5 总日±0.5 总日且只在离屏生成；`babybeefalo` 成长到成体时 `SpawnPrefab("beefalo")`、把新牛加回原群并删除幼牛。[发情与产仔](../../../raw/current/scripts/prefabs/beefaloherd.lua#L125-L139)、[时间参数](../../../raw/current/scripts/tuning.lua#L879-L884)、[成体替换](../../../raw/current/scripts/prefabs/babybeefalo.lua#L91-L100)

**`migratory` 分支不是现行迁移机制**：`beefaloherd` 只在实体带 `migratory` 标签时把成员标签改成 `beefalo_migratory`，但本版 scripts 内没有找到给群节点加该标签的调用；正常世界生成的群走 `beefalo`。不能把 `beefalo_migratory` 当成默认成员标签。[分支](../../../raw/current/scripts/prefabs/beefaloherd.lua#L113-L118)

## 剃毛与刷毛

| 路径 | 门禁与调用 | 产出与复位 |
|---|---|---|
| 剃毛（SHAVE） | 目标带 `bearded`、执行者不骑乘、手持带 `shaver` 的工具；牛的 `CanShaveTest` 还要求**睡眠**，有牛铃主人时只允许主人剃；`Beard:Shave` 按 `oldbits−newbits` 掉落奖品。 | `beard.prize="beefalowool"`、初始 `bits=3` → 一次 3 份。剃到 `daysgrowth<=0` 触发 `Reset` → `OnResetBeard`：移除 `has_beard`、`brushable:SetBrushable(false)`、服从 −1、进入 `shaved` 外观。[动作门槛](../../../raw/current/scripts/componentactions.lua#L1716-L1723)、[shaver 检查](../../../raw/current/scripts/actions.lua#L2833-L2844)、[牛门禁](../../../raw/current/scripts/prefabs/beefalo.lua#L445-L460)、[剃毛结算](../../../raw/current/scripts/components/beard.lua#L107-L166)、[复位](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L443) |
| 刷毛（BRUSH） | 目标带 `brushable`；`ACTIONS.BRUSH.fn` 拒绝目标战斗中或死亡，再调 `brushable:Brush`。刷毛不要求睡眠。 | `brushable.prize="beefalowool"`、`max=1`；产出先 `SpawnPrefab` 再塞进刷毛者背包；推 `brushed` 事件，`OnBrushed` 仅在 `numprizes>0` 时给驯化与服从。刷毛不改变 `beard.bits`。[动作](../../../raw/current/scripts/actions.lua#L4205-L4216)、[标签门禁](../../../raw/current/scripts/componentactions.lua#L1082-L1086)、[配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[结算](../../../raw/current/scripts/components/brushable.lua#L36-L56)、[收益](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L491) |

**周期与上限（刷毛）**：`CalculateNumPrizes` 返回 `elapsed<=0 ? 0 : min(floor(elapsed/cyclesperprize), max)`。宿主只写 `regrowthdays=1`、`max=1`、`prize`，没有写 `cyclesperprize`（默认 0）；`regrowthdays` 在 `brushable.lua` 中没有读取点。按 Lua 5.1 浮点语义，`elapsed<=0` 返回 0，`elapsed>0` 时 `elapsed/0=inf`、`min(inf,1)=1`，所以同一世界日 0、跨天恰 1 份（源码推导，未运行验证；既有页也按此口径，见 [[reference/prefabs/living-tools]] 与 [[reference/components/brushable]]）。[公式](../../../raw/current/scripts/components/brushable.lua#L31-L34)、[宿主字段](../../../raw/current/scripts/prefabs/beefalo.lua#L1146-L1147)、[既有口径](../../../raw/current/scripts/components/brushable.lua#L5-L13)

**产出事件**：剃毛在 `Beard:Shave` 末尾推 `shaved`；刷毛在 `Brush` 末尾推 `brushed` 并调用 `onbrushfn`；长毛由 SG 的 `hair_growth`／`hair_growth_pop` 状态完成，不另发物品事件。[剃毛事件](../../../raw/current/scripts/components/beard.lua#L161-L166)、[刷毛事件](../../../raw/current/scripts/components/brushable.lua#L52-L56)、[长毛状态](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L804-L838)

## 牛毛再生

- 计时起点不是“剃毛后 3 秒”，而是世界日回调：`beard` 组件在 `cycles` 世界状态变化时给 `daysgrowth+1`，并调用该天注册的回调；牛在构造时注册 `AddCallback(TUNING.BEEFALO_HAIR_GROWTH_DAYS, OnHairGrowth)`，该常量＝3 天。[日回调](../../../raw/current/scripts/components/beard.lua#L7-L10)、[注册](../../../raw/current/scripts/prefabs/beefalo.lua#L1142-L1143)、[常量](../../../raw/current/scripts/tuning.lua#L898-L899)
- 初始 `daysgrowth=HAIR_GROWTH_DAYS+1=4`、`bits=3`，所以新牛默认是有毛状态；剃毛把 `daysgrowth` 归 0 后，下一次回调在第 3 天触发。`OnHairGrowth` 只在 `bits==0` 时置 `hairGrowthPending=true`；有 `rideable` 组件时调用 `rideable:Buck()`（把骑手甩下）。[初始值](../../../raw/current/scripts/prefabs/beefalo.lua#L1136-L1139)、[OnHairGrowth](../../../raw/current/scripts/prefabs/beefalo.lua#L467-L474)
- 落地执行：空闲状态看到 `hairGrowthPending` 进入 `hair_growth` → `hair_growth_pop`；后者把 `beard.bits` 设回 3、加回 `has_beard`、`brushable:SetBrushable(true, true)`（重置 `lastbrushcycle`）、`ApplyBuildOverrides`，再清掉 pending。[SG 入口](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L120-L143)、[长毛落地](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L820-L838)
- **边界**：`AddCallback(0, OnShaved)` 只是外观回调，不负责长毛；长毛完全依赖第 3 天的 `OnHairGrowth` 与 SG 状态，不是定时器。[回调表](../../../raw/current/scripts/prefabs/beefalo.lua#L1142-L1143)

## 死亡掉落

| 路径 | 条件 | 结果 |
|---|---|---|
| 普通死亡 | `SGBeefalo` 的 `death` 状态在非“保尸”且非 `file_load` 分支调用 `DropDeathLoot`；`lootdropper` 用 `SetChanceLootTable('beefalo')`。若死亡时被骑乘，先 `Buck(true)`，等 `OnRiderChanged` 甩下骑手后才进 `death` 并结算。 | 表项逐条独立判定：`meat` 4 条 1.00、`beefalowool` 3 条 1.00、`horn` 1 条 0.33。因此保底 4 肉＋3 牛毛，牛角 33%，不是“四选一”。[SG](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L628-L656)、[表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[接线](../../../raw/current/scripts/prefabs/beefalo.lua#L1169-L1170)、[逐项判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[骑乘死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L688-L692)、[甩下后进死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L806-L809) |
| 骑乘中死亡 | `OnDeath` 只推 `Buck(true)`；SG 不处理骑乘中的 `death` 事件，`OnRiderChanged` 在 `newrider==nil` 且 `health:IsDead()` 时才进 `death`。 | 掉落内容仍按进入 `death` 时的普通／燃烧／保尸路径；若在甩下前火已灭，`DropLoot` 结算时不再按燃烧改写。 | [OnDeath](../../../raw/current/scripts/prefabs/beefalo.lua#L688-L692)、[OnRiderChanged](../../../raw/current/scripts/prefabs/beefalo.lua#L806-L809) |
| 燃烧死亡 | `DropLoot` 结算时 `burnable:IsBurning()` 为真且无 `fueled`（牛没有）→ 进入燃烧转换；不是“死亡瞬间”读一次。 | `meat` 走 `cooked` 前缀变成 `cookedmeat`；`beefalowool` 与 `horn` 没有熟制变体，落到 `ash`。`BURNED_LOOT_OVERRIDES` 没有这三项的特例。[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530) |
| 保尸（暗影牛铃／读档） | `ShouldKeepCorpse` 为真（跟随的 `shadowbell` 有充能的 `rechargeable`），或死亡原因是 `file_load`（读档时已死）。 | `death` 状态走保尸分支：先 `burnable:Extinguish()`，不执行 `DropDeathLoot`；牛可被复活，同时停掉胡须生长与饥饿、加 `deadcreature`／`give_dolongaction`。[条件](../../../raw/current/scripts/prefabs/beefalo.lua#L1055-L1063)、[读档死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L671-L673)、[death 事件](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L50-L52)、[保尸分支](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L642-L655)、[复活接口](../../../raw/current/scripts/prefabs/beefalo.lua#L493-L495)、[暗影铃](../../../raw/current/scripts/prefabs/beef_bell.lua#L420-L421) |
| 幼牛 | 幼牛是独立注册 `babybeefalo`，不套用成牛表。 | 三阶段固定表：baby 3 小肉＋1 牛毛、toddler 4 小肉＋2 牛毛、teen 3 肉＋2 牛毛；详细表见 [[reference/prefabs/beefalowool]]。[阶段表](../../../raw/current/scripts/prefabs/babybeefalo.lua#L18-L20) |

- 死亡后 `inst.persists=false`，非保尸分支在 2 秒后 `ErodeAway`；这不是“尸体可锤”。[SG](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L652-L655)、[死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L686-L686)
- `DropDeathLoot` 是通用分叉：`health.is_corpsing` 为真时先 `GenerateLoot`、再交给 `deathloothandler:StoreLoot`（尸体暂存）；否则直接 `lootdropper:DropLoot(pos)`（`DropLoot` 内部再取 `GenerateLoot`）。`SGBeefalo` 明确未启用 corpse 状态，牛走直接 `DropLoot` 分支；不能把这两句串成“先 GenerateLoot 再 DropLoot”的一条链。[DropDeathLoot](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[corpse 状态注释](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L1017-L1017)、[EntityHasCorpse](../../../raw/current/scripts/componentutil.lua#L1472-L1475)

## 驯化、骑乘、牛铃与鞍（入口）

| 入口 | 本页只给什么 | 去何处 |
|---|---|---|
| 驯化与倾向 | `domesticatable`、三种倾向、服从／驯化增减；驯化成功 `DoDomestication` 会 `herdmember:Enable(false)`（退出牛群），回野生再启用。 | 见 [[reference/components/domesticatable]]；[挂载](../../../raw/current/scripts/prefabs/beefalo.lua#L1215-L1216)、[驯化/回野](../../../raw/current/scripts/prefabs/beefalo.lua#L530-L563) |
| 骑乘与鞍 | `rideable`、`saddleable` 标签、服从门槛；鞍物品由 `MakeSaddle` 注册（`saddle_basic` 等），配方牛毛 4＋猪皮 4＋金块 4。 | 见 [[reference/components/rideable]]；[骑乘挂载](../../../raw/current/scripts/prefabs/beefalo.lua#L1197-L1201)、[鞍注册](../../../raw/current/scripts/prefabs/saddle.lua#L407-L412)、[配方](../../../raw/current/scripts/recipes.lua#L682-L682)；鞍页 Wiki 待建 |
| 牛铃 | 注册 `beef_bell` 与 `shadow_beef_bell`；`SetBeefBellOwner` 把铃设为牛 leader，跟随/召回走 follower 链；有铃主人的牛剃毛只允许主人；充能暗影铃可保尸。 | [注册](../../../raw/current/scripts/prefabs/beef_bell.lua#L428-L430)、[铃主人](../../../raw/current/scripts/prefabs/beefalo.lua#L200-L228)；牛铃页 Wiki 待建 |
| 皮弗娄牛美妆台 `beefalo_groomer` | 结构 `beefalo_groomer` 与可部署套件 `beefalo_groomer_item`；配方木板 4＋牛毛 2。 | [注册](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L275-L276)、[配方](../../../raw/current/scripts/recipes.lua#L688-L688)、[po](../../../raw/current/scripts/languages/chinese_s.po#L339299-L339302)；页面 Wiki 待建 |

## 关联机制导航

| 类型／对象 | 本页如何使用 | 条件、端与证据 |
|---|---|---|
| 组件：`herdmember` | 成年牛加入 `beefaloherd`；默认群名、构造延迟、`Enable` 开关 | 驯化时禁用、回野时启用 | [成员](../../../raw/current/scripts/prefabs/beefalo.lua#L1243-L1245)、[组件](../../../raw/current/scripts/components/herdmember.lua#L19-L29)；[[reference/components/herdmember]] |
| 组件：`herd` | 群节点保存成员、中心、更新范围与合并；成员坐标写 `knownlocations.herd` | 中心每 6–8 秒更新 | [更新](../../../raw/current/scripts/components/herd.lua#L195-L241)；[[reference/components/herd]] |
| 组件：`beard` | 牛毛奖品、`bits`、日回调、剃毛与复位 | 剃毛需睡眠；初始 bits 3 | [配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1135-L1143)；[[reference/components/beard]] |
| 组件：`brushable` | 刷毛奖品、上限、`lastbrushcycle` 与 `brushed` 事件 | 剃秃后 `SetBrushable(false)`，长毛后恢复 | [配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)；[[reference/components/brushable]] |
| 组件：`lootdropper` | 共享概率表与燃烧转换；非保尸时 `DropDeathLoot` | 直接掉落；`DropLoot` 结算时仍燃烧才转换 | [表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)；[[reference/components/lootdropper]] |
| 组件：`domesticatable`／`rideable` | 驯化倾向与骑乘门槛；本页只给入口 | 驯化后退出牛群 | [驯化](../../../raw/current/scripts/prefabs/beefalo.lua#L1215-L1216)、[骑乘](../../../raw/current/scripts/prefabs/beefalo.lua#L1197-L1201)；[[reference/components/domesticatable]]、[[reference/components/rideable]] |
| 组件：`health`／`burnable` | 生命 1000、死亡路径；大型可燃烧角色 | `DropLoot` 结算时仍燃烧才触发转换 | [生命](../../../raw/current/scripts/prefabs/beefalo.lua#L1162-L1165)、[可燃](../../../raw/current/scripts/prefabs/beefalo.lua#L1218-L1219) |
| SG／Brain：`SGBeefalo`／`beefalobrain` | 死亡、长毛、剃秃、睡眠、游荡等状态与行为 | 实际执行端 | [SG](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L120-L143)、[Brain](../../../raw/current/scripts/brains/beefalobrain.lua#L257-L258) |
| 关联 Prefab：`beefaloherd`／`babybeefalo`／`beef_bell`／`beefalo_groomer` | 群节点、幼牛、铃、皮弗娄牛美妆台；不是本注册的变体 | 见各入口 | [群](../../../raw/current/scripts/prefabs/beefaloherd.lua#L102-L146)、[幼牛](../../../raw/current/scripts/prefabs/babybeefalo.lua#L111-L200) |
| 资源：`beefalo_basic.zip`／`beefalo_build.zip` 等 | 成年牛动画与 build；另有 shaved、baby、domestic、personality、carrat 覆盖 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/beefalo.lua#L3-L32) |

## 常见陷阱与限制

- **误区：刷毛和剃毛都能随时做。** 剃毛要求牛在睡眠；刷毛不要求睡眠，但要求 `brushable` 标签（有毛）且目标不在战斗、未死亡。剃秃后 `brushable` 被关闭，3 天后再开。[剃毛门禁](../../../raw/current/scripts/prefabs/beefalo.lua#L445-L460)、[刷毛动作](../../../raw/current/scripts/actions.lua#L4205-L4216)、[复位](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L443)
- **误区：刷毛“每天一份”是宿主配置的。** 宿主写的是 `regrowthdays=1`，但组件读 `cyclesperprize`，默认 0；不要照抄 `regrowthdays` 当周期字段。跨天 1 份／同日 0 由源码公式推出（`elapsed<=0`→0；`elapsed>0`→`min(inf,1)=1`），未运行验证。[字段](../../../raw/current/scripts/prefabs/beefalo.lua#L1146-L1147)、[读取](../../../raw/current/scripts/components/brushable.lua#L31-L34)
- **误区：剃毛只掉 1 份。** 初始 `bits=3`，一次剃秃按 3 个 bit 掉 3 份；刷毛才受 `max=1` 限制。[bits](../../../raw/current/scripts/prefabs/beefalo.lua#L1137-L1137)、[Shave](../../../raw/current/scripts/components/beard.lua#L144-L155)
- **边界：非保尸普通死亡不是“四选一”。** 4 肉、3 牛毛、0.33 牛角是三条独立表项；牛毛保底 3，牛角才是概率。保尸分支不执行 `DropDeathLoot`，这三项都不掉。[表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[结算](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[保尸分支](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L642-L655)
- **边界：燃烧改写读 `DropLoot` 结算当下，不是死亡瞬间。** 结算时仍燃烧才把肉变 `cookedmeat`、牛毛与牛角变灰；骑乘中延迟结算的牛，若甩下前火已灭，就按非燃烧掉落。[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)
- **边界：暗影牛铃保尸时没有死亡掉落。** `ShouldKeepCorpse` 为真或 `data.cause=="file_load"` 时，SG 走保尸分支（先熄火），不进 `DropDeathLoot`；不要把“有保尸”与“照常掉落”同时写。[条件](../../../raw/current/scripts/prefabs/beefalo.lua#L1055-L1063)、[读档](../../../raw/current/scripts/prefabs/beefalo.lua#L671-L673)、[SG](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L642-L655)
- **边界：`beefalo_carry` 不是第二只牛。** 它没有标签与组件，scripts 内也没有生成点；不要把它写进掉落或骑乘链。[工厂](../../../raw/current/scripts/prefabs/beefalo.lua#L1322-L1346)
- **边界：牛群“迁移”不是定时换营地。** 中心由 20 格内、非战斗成员的平均坐标驱动；成员被战斗、距离或驯化状态拉开时，中心不会瞬移。[中心](../../../raw/current/scripts/components/herd.lua#L195-L241)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/beefalo.lua` 全文（两个注册、标签、主模拟组件、剃毛／刷毛／死亡回调）、`prefabs/beefaloherd.lua` 全文、`prefabs/babybeefalo.lua` 全文、`components/herd.lua`／`herdmember.lua`／`beard.lua`／`brushable.lua`／`lootdropper.lua`、`stategraphs/SGBeefalo.lua` 的 idle／shaved／hair_growth／death 段、`brains/beefalobrain.lua` 的 herd／loiter／wander 段、`actions.lua` 的 BRUSH／SHAVE 入口、`entityscript.lua` 的 `DropDeathLoot`、`componentutil.lua` 的 `EntityHasCorpse`、`prefabs/beefalo.lua` 的骑乘死亡与甩下后进 `death` 段、`prefabs/beef_bell.lua` 与 `prefabs/saddle.lua`／`beefalo_groomer.lua` 的注册段、`recipes.lua` 的鞍与皮弗娄牛美妆台配方、`tuning.lua` 的牛参数与时间基数、`chinese_s.po` 的名称。
- 游戏运行：未运行。未核验：剃毛／长毛在睡眠与骑乘中的边界、暗影牛铃保尸的复活流程、燃烧死亡时各表项的逐项转换、牛群中心在真实寻路中的漂移速度。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 牛毛物品与消费者：[[reference/prefabs/beefalowool]]
- 刷毛组件：[[reference/components/brushable]]
- 剃毛与胡须：[[reference/components/beard]]
- 牛群：[[reference/components/herd]]、[[reference/components/herdmember]]
- 驯化与骑乘：[[reference/components/domesticatable]]、[[reference/components/rideable]]
- 掉落组件：[[reference/components/lootdropper]]
- 返回 [[reference/prefabs/index|Prefab 索引]]
