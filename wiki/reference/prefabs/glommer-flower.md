---
title: "格罗姆花与格罗姆（glommerflower、glommer）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "格罗姆雕像满月放花、非满月收花的获得链，格罗姆花作为 leader 与格罗姆 follower 的绑定方式，以及黏液周期、死亡掉落、理智光环、睡眠／离场条件与格罗姆死后花转死花的分支。"
aliases: ["glommerflower", "glommer", "格罗姆花", "格罗姆", "格罗姆的黏液", "格罗姆翅膀", "格罗姆雕像", "死花"]
tags: [prefab, creature, item, aura]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/glommerflower.lua", "prefabs/glommer.lua", "prefabs/statueglommer.lua", "brains/glommerbrain.lua", "stategraphs/SGglommer.lua", "components/leader.lua", "components/follower.lua", "components/periodicspawner.lua", "components/sanityaura.lua", "components/sleeper.lua", "components/perishable.lua", "components/pickable.lua", "components/lootdropper.lua", "components/health.lua", "stategraphs/commonstates.lua", "componentutil.lua", "standardcomponents.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 格罗姆花与格罗姆（glommerflower、glommer）

本页覆盖两个注册加一个获得宿主：`glommerflower`（花，物品＋`leader` 宿主）、`glommer`（格罗姆，飞行生物＋`follower`）、`statueglommer`（格罗姆雕像，只负责满月放花）。雕像不另建页，凡与花／格罗姆无关的雕像细节（照明 netvar、镐挖动画分档）只在此带过。本页在注册清单里对应的是 `glommerflower` 与 `glommer` 两个主注册；`statueglommer` 只作获得宿主，`glommerfuel`／`glommerwings` 是格罗姆的产物，物品侧留给各自页面承接，本页只在关联导航给入口（R19）。

## 快速判断

- 职责与适用：花是格罗姆的 `leader` 宿主，格罗姆是它的 `follower`；绑定不是生成时写死，而是由雕像的满月流程建立、由 `follower:SetLeader` 在摘花时转移到花本体。[花挂 leader](../../../raw/current/scripts/prefabs/glommerflower.lua#L116-L117)、[摘花转移](../../../raw/current/scripts/prefabs/statueglommer.lua#L158-L164)
- 优先使用：查「什么时候有花、什么时候有格罗姆」看[获得：雕像的满月开关](#获得雕像的满月开关)；查跟随、标签与离场看[跟随、标签与睡眠](#跟随标签与睡眠)；查格罗姆死后的花看[死花分支](#死花分支onlosechild)；只想改黏液间隔，直接用 [[reference/components/periodicspawner#典型接入]] 里已核验的格罗姆示例。
- 重要副作用：花被摘走后格罗姆改跟花——花在玩家背包里时，`GetLeader()` 返回的是背包持有者，所以格罗姆会跟着**拿花的人**走；花掉在地上又回到花本体。[itemowner 分支](../../../raw/current/scripts/components/follower.lua#L316-L335)、[GetLeader](../../../raw/current/scripts/components/follower.lua#L59-L61)
- 重要副作用：格罗姆**死亡或被移除**（包括自己飞走）都会让绑定它的花立刻进入死态：挂腐坏组件（只设时长）与燃料、换图标、删 `glommerflower` tag；飞走不掉战利品，死亡才走掉落表。[死花回调](../../../raw/current/scripts/prefabs/glommerflower.lua#L7-L42)、[移除即回调](../../../raw/current/scripts/components/leader.lua#L170-L187)
- 修改边界：雕像的 `pickable` 只负责「显示／收起」花，花物品要到摘取时才由 `SpawnProductLoot` 生成；本页不展开雕像照明、黏液物品 `glommerfuel` 的施肥数值与格罗姆尸体 `glommercorpse` 本体（S6）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `glommerflower` | `fn()`：先建实体、动画（bank／build `glommer_flower`、`idle`）、小地图（`glommerflower.png`、优先级 5），加 3 个 tag，`MakeInventoryFloatable(inst, "med", nil, 0.7, nil, nil, swap_data)`，`SetPristine()` 之后才是端分界；主模拟依次挂 `leader`（同时设 `onremovefollower`）→ `inspectable`（`getstatus`）→ `inventoryitem`，再 `MakeHauntableLaunch`，注册 `OnPreLoad`／`OnSave`／`RefreshFlowerIcon`，末尾 `DoTaskInTime(0, OnInit)`。[L84-L132](../../../raw/current/scripts/prefabs/glommerflower.lua#L84-L132) | 物品＋`leader` 宿主；活态 tag `glommerflower`／`nonpotatable`／`irreplaceable`（S3）；`scrapbook_adddeps = {"glommer"}`；死态无独立注册，只靠删 tag 区分。[标签](../../../raw/current/scripts/prefabs/glommerflower.lua#L101-L103) |
| `glommer` | `fn()`：`MakeGhostPhysics(inst, 1, .5)`、四向朝向、小地图（`glommer.png`、优先级 5、`SetCanUseCache(false)`），加 5 个 tag，`MakeInventoryFloatable(inst, "med")`，`SetPristine()` 之后分端；主模拟按序挂 `maprevealable`（图标 `globalmapiconunderfog`）→ `inspectable` → `follower`（听 `startfollowing`／`stopfollowing`）→ `health`／`combat`／`knownlocations`／`lootdropper` → `sleeper` → `sanityaura` → `locomotor`（`walkspeed = 6`、`pathcaps = {allowocean = true}`）→ `periodicspawner`，再 `SetBrain(brain)`／`SetStateGraph("SGglommer")`，末尾 `MakeMediumFreezableCharacter(inst, "glommer_body")`。[L73-L157](../../../raw/current/scripts/prefabs/glommer.lua#L73-L157) | 飞行生物；tag `glommer`／`flying`／`lunar_aligned`／`ignorewalkableplatformdrowning`／`cattoyairborne`；`health` 未赋值，沿用组件默认 100；预载表另含 `glommerfuel`／`glommerwings`／`monstermeat`／`glommercorpse`／`globalmapiconunderfog`。[标签](../../../raw/current/scripts/prefabs/glommer.lua#L96-L100)、[预载](../../../raw/current/scripts/prefabs/glommer.lua#L7-L15)、[默认血量](../../../raw/current/scripts/components/health.lua#L64-L66) |
| `statueglommer`（获得宿主） | `fn()`：障碍物物理 0.75、`statue` tag、光照 netvar（`lightval`／`islighton`）、非专服加 `pointofinterest`；主模拟挂 `leader`（`onremovefollower = OnLoseChild`）→ `pickable`（`product = "glommerflower"`、`onpickedfn`／`makeemptyfn`／`makefullfn`）→ `workable`（`ACTIONS.MINE`、`TUNING.ROCKS_MINE`）→ `lootdropper`，`spawned = false` 后 `DoTaskInTime(0, OnInit)`。[L216-L293](../../../raw/current/scripts/prefabs/statueglommer.lua#L216-L293) | 满月／非满月由 `WatchWorldState("isfullmoon", OnIsFullmoon)` 驱动；`pickable` 只显示花，摘取时才 `SpawnPrefab("glommerflower")` 并交给摘取者；镐挖 6 下后掉 3 份大理石（`SetChanceLootTable('statueglommer')` 三条 1.00）。[世界状态](../../../raw/current/scripts/prefabs/statueglommer.lua#L144-L152)、[镐挖](../../../raw/current/scripts/prefabs/statueglommer.lua#L166-L177)、[掉落表](../../../raw/current/scripts/prefabs/statueglommer.lua#L14-L20) |

闹鬼接入只列与本页相关的三处（雕像其余细节不展开）：花 `MakeHauntableLaunch`、格罗姆 `MakeHauntablePanic`、雕像 `MakeHauntableWork`。[花](../../../raw/current/scripts/prefabs/glommerflower.lua#L122-L122)、[格罗姆](../../../raw/current/scripts/prefabs/glommer.lua#L155-L155)、[雕像](../../../raw/current/scripts/prefabs/statueglommer.lua#L290-L290)

三者的关系是「宿主 → 物品 → 生物」：雕像的 `pickable` 用 `canbepicked` 表示「架上有没有花」，`Regen()` 走 `makefullfn` 只切显示符号，`MakeEmpty()` 走 `makeemptyfn` 收起符号；花物品要到摘取时由 `Pickable:Pick` → `SpawnProductLoot` 生成，随后才调用雕像的 `OnPicked`。[Regen](../../../raw/current/scripts/components/pickable.lua#L386-L396)、[摘取顺序](../../../raw/current/scripts/components/pickable.lua#L535-L555)、[产物生成](../../../raw/current/scripts/components/pickable.lua#L493-L518)

### 名称核验

`glommerflower`＝**格罗姆花**（Glommer's Flower）、`glommer`＝**格罗姆**（Glommer）；掉落物 `glommerfuel`＝**格罗姆的黏液**、`glommerwings`＝**格罗姆翅膀**。四个键各命中一次，注册名与 `NAMES` 键同名（S2）。[GLOMMER／GLOMMERFLOWER](../../../raw/current/scripts/languages/chinese_s.po#L343309-L343317)、[GLOMMERFUEL／GLOMMERWINGS](../../../raw/current/scripts/languages/chinese_s.po#L343319-L343327)

## 获得：雕像的满月开关

满月与非满月由同一个回调分流：`OnInit` 在构造后立即 `WatchWorldState("isfullmoon", OnIsFullmoon)` 并主动跑一次 `OnIsFullmoon(inst, TheWorld.state.isfullmoon)`，所以读档时也会校正一次（S4）。[L136-L152](../../../raw/current/scripts/prefabs/statueglommer.lua#L136-L152)

`SpawnGland` 的五道门（按源码顺序）：①`pickable.canbepicked` 为真＝架上已有花，只补灯光并置 `spawned = true` 后返回；②`spawned` 已为真＝这个满月已处理过，直接返回；③先置 `spawned = true`，再在世界里按 tag 找 `glommerflower`，找到就返回（**一个世界同时只认一朵活花**）；④点亮雕像灯；⑤按 tag 找 `glommer`，没有或已死才 `SpawnGlommer`，最终拿到的那只（复用或新生成）都把 `ShouldLeaveWorld` 清掉；最后 `pickable:Regen()` 把花放回架上。[L84-L121](../../../raw/current/scripts/prefabs/statueglommer.lua#L84-L121)

`SpawnGlommer` 在雕像周围 `FindWalkableOffset`（半径 35、12 次尝试）找落点，`SpawnPrefab("glommer")` 后若其 `follower.leader` 不是雕像就 `SetLeader(inst)`，再把格罗姆传送到落点并面向雕像。[L70-L82](../../../raw/current/scripts/prefabs/statueglommer.lua#L70-L82)

`RemoveGland`（非满月）做三件事：`spawned = false`、`pickable:MakeEmpty()` 收起架上花、**且仅当世界里没有 `glommerflower` tag 实体**时把现存格罗姆的 `ShouldLeaveWorld` 置真。也就是说：花被人摘走拿在身上时，非满月不会赶走格罗姆；架上没人摘的那朵不是实体，不会阻挡离场判定。[L123-L134](../../../raw/current/scripts/prefabs/statueglommer.lua#L123-L134)

摘花把绑定从雕像转到花：`Pickable:Pick` 先生成花物品，再调 `OnPicked(inst, picker, loot)`；后者只在「找得到格罗姆且它当前 leader 不是这朵花」时先 `StopFollowing()` 再 `SetLeader(loot)`。若它此前正跟雕像，这次换 leader 会让雕像自己的 `leader` 掉一个 follower，于是雕像的 `OnLoseChild` 收花（`pickable:MakeEmpty()`），架上花与手上花不会同时显示。[OnPicked](../../../raw/current/scripts/prefabs/statueglommer.lua#L158-L164)、[雕像收花](../../../raw/current/scripts/prefabs/statueglommer.lua#L154-L156)、[RemoveFollower 链](../../../raw/current/scripts/components/leader.lua#L170-L187)

花被捡走后仍绑定的原因在 `follower:SetLeader`：新 leader 带 `inventoryitem` 时，组件会 `MakeComponentAnInventoryItemSource` 并把 `itemowner` 设成持有者的「祖父持有者」，`GetLeader()` 优先返回 `itemowner`；`ondropped` 时清空 `itemowner`，leader 回到花实体本身。[SetLeader](../../../raw/current/scripts/components/follower.lua#L293-L336)、[itemsource](../../../raw/current/scripts/componentutil.lua#L1777-L1840)、[GetLeader](../../../raw/current/scripts/components/follower.lua#L59-L61)

读档补绑：花的 `OnInit` 用 `TheSim:FindFirstEntityWithTag("glommer")` 找活着的格罗姆，若它当前 leader 不是这朵花就 `SetLeader(inst)`；这是「花被摘走后存档再读，格罗姆仍认得这朵花」的兜底。[L71-L82](../../../raw/current/scripts/prefabs/glommerflower.lua#L71-L82)、[触发](../../../raw/current/scripts/prefabs/glommerflower.lua#L129-L129)

## 跟随、标签与睡眠

- 同伴标记：`OnStartFollowing` 只在 leader 带 `glommerflower` tag 时加 `companion`，`OnStopFollowing` 无条件删；所以自制「花类」leader 若不带这个 tag，格罗姆照样跟随但不会被标成同伴。[L63-L71](../../../raw/current/scripts/prefabs/glommer.lua#L63-L71)
- 行为与状态：脑是 `glommerbrain`（跟随距离 0／4／6、游荡半径 10，`PriorityNode` 里先 panic 再跟随），图是 `SGglommer`（`idle`／`goo`／`bored`／`frozen`／`thaw`／`flyaway` 与 `CommonStates` 的走、睡、战斗、尸体状态）。[Follow](../../../raw/current/scripts/brains/glommerbrain.lua#L40-L52)、[图](../../../raw/current/scripts/stategraphs/SGglommer.lua#L270-L270)
- 睡眠：`sleeper.watchlight = true`（在洞穴里要「没被光照到」才睡）、`SetResistance(3)`、`testperiod = GetRandomWithVariance(6, 2)`＝4～8 秒；`ShouldSleep` 要求离 leader ≤7 **且非满月**，`ShouldWakeUp` 是默认判据或离 leader >14。[L125-L130](../../../raw/current/scripts/prefabs/glommer.lua#L125-L130)、[判据](../../../raw/current/scripts/prefabs/glommer.lua#L32-L41)、[默认判据](../../../raw/current/scripts/components/sleeper.lua#L94-L111)
- 离场：`ShouldLeaveWorld` 为真时，格罗姆**实体休眠（离屏）即 `Remove()`**；在屏内则由脑的 `WanderOff` 发 `GOHOME`，`SGglommer` 的 `flyaway` 在第 5 秒 `Remove()`。两条路都不走死亡、不掉战利品。[OnEntitySleep](../../../raw/current/scripts/prefabs/glommer.lua#L43-L47)、[WanderOff](../../../raw/current/scripts/brains/glommerbrain.lua#L30-L34)、[flyaway](../../../raw/current/scripts/stategraphs/SGglommer.lua#L196-L214)、[GOHOME 接线](../../../raw/current/scripts/stategraphs/SGglommer.lua#L3-L6)
- 存读：`ShouldLeaveWorld` 自己存档（`OnSave`／`OnLoad`），所以「已经决定离场」的状态能跨读档。[L49-L57](../../../raw/current/scripts/prefabs/glommer.lua#L49-L57)

## 数值与计时

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒，天数按 480 秒折算（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项目 | 定义处 | 结算与条件 |
|---|---|---|
| 黏液周期 | `periodicspawner.prefab = "glommerfuel"`、`basetime = TOTAL_DAY_TIME * 2`、`randtime = TOTAL_DAY_TIME * 2`，构造后立即 `Start()`。[L139-L144](../../../raw/current/scripts/prefabs/glommer.lua#L139-L144) | `Start` 取 `basetime + math.random() * randtime`，首轮与后续都是 **960～1920 秒＝2～4 天**；产物经 `OnSpawnFuel` 进 `goo` 状态（先 `Hide()`，30 帧后再 `Show()`）。[计时](../../../raw/current/scripts/components/periodicspawner.lua#L89-L96)、[goo](../../../raw/current/scripts/stategraphs/SGglommer.lua#L71-L101) |
| 掉落表 | `SetChanceLootTable('glommer')`，表内 3×`monstermeat`、1×`glommerwings`、2×`glommerfuel`，概率全 1.00。[L19-L27](../../../raw/current/scripts/prefabs/glommer.lua#L19-L27)、[接线](../../../raw/current/scripts/prefabs/glommer.lua#L122-L123) | 死亡由 `CommonStates` 的 `death` 状态调 `DropDeathLoot()`：常态直接落地；只有满足 `CanEntityBecomeCorpse`（SG 有 `corpse` 状态、未标 `nocorpse`，且尸体留存或月变异条件之一成立）时才先存进尸体。[掉落入口](../../../raw/current/scripts/stategraphs/commonstates.lua#L1446-L1466)、[分流](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[尸体判定](../../../raw/current/scripts/componentutil.lua#L1546-L1561) |
| 理智光环 | `sanityaura.aura = TUNING.SANITYAURA_TINY`＝`100/(seg_time*32)`≈0.1042。[L132-L133](../../../raw/current/scripts/prefabs/glommer.lua#L132-L133)、[常量](../../../raw/current/scripts/tuning.lua#L2205-L2205) | `GetAura` 在 `SANITY_EFFECT_RANGE`＝10 内按 `max(1, distsq)` 衰减：贴脸约 0.1042/秒（约 50/天），10 格处约 0.001。[衰减](../../../raw/current/scripts/components/sanityaura.lua#L25-L33)、[范围](../../../raw/current/scripts/tuning.lua#L2233-L2233) |
| 死花腐坏（只设不跑） | 死态只 `AddComponent("perishable")`＋`SetPerishTime(TUNING.PERISH_FAST)`＝6 天＝2880 秒，全文没有 `StartPerishing`。[L12-L14](../../../raw/current/scripts/prefabs/glommerflower.lua#L12-L14)、[常量](../../../raw/current/scripts/tuning.lua#L2046-L2046) | `perishable` 的周期任务只在 `StartPerishing()` 里创建，没调用就没有结算：`perishremainingtime` 恒等于 `perishtime`、`GetPercent()` 恒 1，永不 `Perish`，`PERISH_GROUND_MULT` 与冰箱倍率都不参与（S5 反例）。要让它真的烂见[典型接入](#让死花真正开始腐坏)。[任务创建](../../../raw/current/scripts/components/perishable.lua#L262-L275)、[百分比](../../../raw/current/scripts/components/perishable.lua#L215-L221) |
| 死花燃料／燃烧 | `fuel.fuelvalue = TUNING.MED_LARGE_FUEL`＝90；`MakeSmallBurnable(inst, TUNING.SMALL_BURNTIME)`＝7.5 秒；`MakeSmallPropagator(inst)`。[L38-L41](../../../raw/current/scripts/prefabs/glommerflower.lua#L38-L41)、[常量](../../../raw/current/scripts/tuning.lua#L1676-L1681) | 燃料值与燃烧时长在组件结算；活花没有 `fuel`／`burnable`，只有死花可当燃料烧。[小可燃工具](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |

## 死花分支（OnLoseChild）

触发链是「格罗姆没了」而不是「花到点了」：`leader:AddFollower` 时给 follower 挂了 `death` 与 `onremove` 两个监听，任一发生都会 `RemoveFollower`，而 `RemoveFollower` 在移出列表后调用宿主的 `onremovefollower`——花的这个回调就是 `OnLoseChild`。[AddFollower 挂监听](../../../raw/current/scripts/components/leader.lua#L189-L216)、[RemoveFollower 回调](../../../raw/current/scripts/components/leader.lua#L170-L187)、[花接线](../../../raw/current/scripts/prefabs/glommerflower.lua#L116-L117)

`OnLoseChild` 的守卫与顺序（缺一不可）：

1. 花已不带 `glommerflower` tag 就返回——死态只结算一次，读档重入也靠这个守卫。[L7-L10](../../../raw/current/scripts/prefabs/glommerflower.lua#L7-L10)
2. 加 `perishable` 并设 6 天、加 `show_spoilage`、**删 `glommerflower` tag**（源码注释直言「tag 就是死态的标记」）、播 `idle_dead`、`RefreshFlowerIcon()`、关小地图。[L12-L19](../../../raw/current/scripts/prefabs/glommerflower.lua#L12-L19)
3. 若正被持有，先 `RemoveFromOwner(true)` 再 `GiveItem` 回原容器／背包，并把 `ignoresound` 临时置真——注释说这是为了让 `show_spoilage` 刷新物品格图标，属于原版自认的 hack。[L21-L36](../../../raw/current/scripts/prefabs/glommerflower.lua#L21-L36)
4. 最后加 `fuel`（90）、`MakeSmallBurnable`、`MakeSmallPropagator`。[L38-L41](../../../raw/current/scripts/prefabs/glommerflower.lua#L38-L41)

图标与状态跟随 tag 走：`RefreshFlowerIcon` 默认取 `glommerflower`，没有 tag 时取 `glommerflower_dead`，再按皮肤名做 `gsub` 替换；`inspectable.getstatus` 在没有 tag 时返回 `DEAD`。`glommerflower_dead` 是 `INV_IMAGE` 资源，在 `assets` 里声明。[L48-L59](../../../raw/current/scripts/prefabs/glommerflower.lua#L48-L59)、[状态](../../../raw/current/scripts/prefabs/glommerflower.lua#L44-L46)、[资源](../../../raw/current/scripts/prefabs/glommerflower.lua#L1-L5)

存档只存一个布尔：`OnSave` 写 `data.deadchild = not inst:HasTag("glommerflower") or nil`，`OnPreLoad` 见到 `deadchild` 就直接重跑 `OnLoseChild(inst)`（回调里不使用 `child` 参数，所以预载时可以只传 `inst`）。[L61-L69](../../../raw/current/scripts/prefabs/glommerflower.lua#L61-L69)

死花不会开始腐坏：`perishable` 的计时任务只在 `StartPerishing()` 里创建，而 `OnLoseChild` 只设了总时长；因此鲜度条恒满、`GetPercent()` 恒 1、永远走不到 `Perish()`。这和「6 天后消失」是两回事——原版死花可以长期留作燃料，想让它真烂要自己补一次启动（见[典型接入](#让死花真正开始腐坏)）。[任务创建](../../../raw/current/scripts/components/perishable.lua#L262-L275)、[百分比](../../../raw/current/scripts/components/perishable.lua#L215-L221)、[本页调用](../../../raw/current/scripts/prefabs/glommerflower.lua#L12-L14)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：leader（花） | 花是宿主，`onremovefollower = OnLoseChild`；`AddFollower` 时接收 follower 的 `death`／`onremove` | 仅主模拟；死花后 `leader` 仍在，但已无 follower 可掉 | [[reference/components/leader#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommerflower.lua#L116-L117) |
| 组件：follower（格罗姆） | `SetLeader` 绑定花或雕像；leader 带 `inventoryitem` 时改跟持有者 | 仅主模拟；`startfollowing`／`stopfollowing` 驱动 `companion` 标签 | [[reference/components/follower#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommer.lua#L115-L117)、[转移](../../../raw/current/scripts/prefabs/statueglommer.lua#L158-L164) |
| 组件：periodicspawner（格罗姆） | 产物 `glommerfuel`、2～4 天一轮、构造即 `Start` | 仅主模拟；`OnSpawnFuel` 交给 SG `goo`，不在回调里生成新状态 | [[reference/components/periodicspawner#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommer.lua#L139-L144) |
| 组件：sanityaura（格罗姆） | `aura = SANITYAURA_TINY`，注册即加 `sanityaura` tag | 仅主模拟（组件与 tag 都在端分界之后）；由玩家 sanity 侧按距离读取，范围 10、按距离平方衰减 | [[reference/components/sanityaura#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommer.lua#L132-L133) |
| 组件：sleeper（格罗姆） | `watchlight`、抗性 3、4～8 秒测试周期，自定义睡／醒判据 | 仅主模拟；满月不睡、离 leader 太远会醒 | [[reference/components/sleeper#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommer.lua#L125-L130) |
| 组件：perishable（死花） | 死态才加，只设 6 天总时长，未 `StartPerishing` | 仅主模拟；因此不计时、鲜度恒 100%，地面／容器倍率都不生效 | [[reference/components/perishable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glommerflower.lua#L12-L14) |
| 组件：pickable（雕像） | 用 `canbepicked` 表示架上花，`Regen`／`MakeEmpty` 切显示符号 | 仅主模拟；摘取时才 `SpawnPrefab("glommerflower")` | [[reference/components/pickable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/statueglommer.lua#L268-L272) |
| SG／Brain：`SGglommer`／`glommerbrain` | 脑负责跟随与 `GOHOME`，图负责 `goo`／`flyaway`／睡／战斗／尸体 | `SetBrain`／`SetStateGraph` 都在端分界之后，只有主模拟副本挂载；`flyaway` 第 5 秒自删 | [brain](../../../raw/current/scripts/brains/glommerbrain.lua#L40-L52)、[SG](../../../raw/current/scripts/stategraphs/SGglommer.lua#L196-L214) |
| API／数据：`SetSharedLootTable('glommer')` | 掉落表在文件顶层注册，`lootdropper:SetChanceLootTable` 只引用 | 主模拟；表内每条 1.00 | [[reference/components/lootdropper#关键接口]]；[表](../../../raw/current/scripts/prefabs/glommer.lua#L19-L27) |
| 关联 Prefab：`glommerflower`／`glommer`／`glommerfuel` | 雕像 `prefabs` 表声明花与格罗姆；格罗姆 `prefabs` 表声明黏液、翅膀、怪物肉、尸体、`globalmapiconunderfog` | 声明不等于生成：黏液由 `periodicspawner` 产，花由 `pickable` 产 | [雕像预载](../../../raw/current/scripts/prefabs/statueglommer.lua#L7-L12)、[格罗姆预载](../../../raw/current/scripts/prefabs/glommer.lua#L7-L15) |
| 资源：`anim/glommer_flower.zip`、`anim/glommer.zip`、`sound/glommer.fsb` | 花 `bank`／`build` 都为 `glommer_flower`；格罗姆为 `glommer`；音效单独 `.fsb` | 原版资源，不随页分发 | [花资源](../../../raw/current/scripts/prefabs/glommerflower.lua#L1-L5)、[格罗姆资源](../../../raw/current/scripts/prefabs/glommer.lua#L1-L5) |

## 典型接入

### 让死花真正开始腐坏

- 前提与选择：原版 `OnLoseChild` 只 `SetPerishTime`，从未 `StartPerishing`（见[死花分支](#死花分支onlosechild)）。要在保留原版死态外观的同时让它按 6 天腐坏到 0%（原版没有 `onperishreplacement`，到 0% 只推 `perished`、不会自动消失，见 [`Perish`](../../../raw/current/scripts/components/perishable.lua#L277-L320)），最小改动是在模组 `modmain.lua` 里包住花自己的 `leader.onremovefollower` 与实体 `OnLoad`，死态补齐并**在组件 `OnLoad` 之后**补一次 `StartPerishing`；只需服务端。
- 生效与撤销：`AddPrefabPostInit` 在 `fn()` 之后运行，此时 `leader` 已挂好；`leader.onremovefollower` 入口先跑原版死态回调、`OnLoad` 入口先跑既有 `OnLoad`（若有；原厂只设 `OnPreLoad`／`OnSave`，见[工厂](../../../raw/current/scripts/prefabs/glommerflower.lua#L124-L125)）再启动计时，所以图标、tag、燃料仍由原版逻辑处理。补启动必须落在实体 `OnLoad`：读档顺序是 `OnPreLoad` → 各组件 `OnLoad` → 实体 `OnLoad`，从未启动的 `perishable` 存档为 `paused=true`，组件 `OnLoad` 只会 `StopPerishing`，早一步启动会被它取消。停用模组后的结果取决于**这份死花有没有被本示例启动过**，而不是当前是否装着模组：启用期间已经计时的死花，`perishable` 存的是 `time`、不写 `paused`（`OnSave` 里 `paused = updatetask == nil or nil`，运行中为 nil），停用后原版 `OnPreLoad` 仍会重建 `perishable`，组件 `OnLoad` 见 `paused` 为空就自己 `StartPerishing`，计时接着走完；从没启动过的死花（原版旧档）存 `paused=true`，加载后仍停表。可概括为「原版暂停旧档 → 启用模组 → 计时后存档 → 停用并加载」，最后一步会继续腐坏。
- 原版依据：`StartPerishing` 创建 `DoPeriodicTask`，`OnLoad` 按 `paused` 恢复；实体读档顺序见 `SetPersistData`。[任务](../../../raw/current/scripts/components/perishable.lua#L262-L275)、[组件存读](../../../raw/current/scripts/components/perishable.lua#L329-L349)、[实体顺序](../../../raw/current/scripts/entityscript.lua#L1988-L2013)

```lua
-- modmain.lua：给死掉的格罗姆花补上 StartPerishing（原版只设总时长）
AddPrefabPostInit("glommerflower", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    local function start_if_dead(flower)
        local perishable = flower.components.perishable
        if perishable ~= nil and not perishable:IsPerishing() then
            perishable:StartPerishing()
        end
    end

    local leader = inst.components.leader
    local onlosechild = leader ~= nil and leader.onremovefollower or nil
    if onlosechild ~= nil then
        leader.onremovefollower = function(flower, child)
            onlosechild(flower, child)
            start_if_dead(flower)
        end
    end

    local onload = inst.OnLoad
    inst.OnLoad = function(flower, data)
        if onload ~= nil then
            onload(flower, data)
        end
        start_if_dead(flower) -- 组件 OnLoad 之后补一次：老存档的死花也计时
    end
end)
```

- 调整与验证：把 `TUNING.PERISH_FAST` 换成别的天数即改时长；只要“死亡当场计时”可删掉 `OnLoad` 包装。`start_if_dead` 只对已有 `perishable` 的死花生效，活花不受影响。6 天是**定义值**；地面无其他修正时结算再乘 `PERISH_GROUND_MULT`＝1.5，约 4 天到 0%，到 0% 也只推 `perished`、原版没有替换物，不会自动删除（R10）。本示例未在游戏内运行。

## 常见陷阱与限制

- **误区：花是「种」在雕像上的实体。** 满月只做 `pickable:Regen()`（切符号），雕像上那朵花不是实体；`FindFirstEntityWithTag("glommerflower")` 找的是**被摘走的花物品**，不是雕像上的显示。[SpawnGland](../../../raw/current/scripts/prefabs/statueglommer.lua#L84-L121)、[Regen](../../../raw/current/scripts/components/pickable.lua#L386-L396)
- **边界：死花不挡下一次满月。** 死态删掉 `glommerflower` tag，`SpawnGland` 的「世界里已有活花」守卫因此漏过它，雕像会在下一个满月再放一朵；同时 `OnLoseChild` 的守卫靠同一 tag 保证死态只结算一次。[删 tag](../../../raw/current/scripts/prefabs/glommerflower.lua#L12-L15)、[守卫](../../../raw/current/scripts/prefabs/glommerflower.lua#L7-L10)
- **边界：格罗姆飞走也会让绑定的花变死花，但不掉战利品。** `flyaway` 与离屏自删都是 `Remove()`，走 `onremove` 分支；掉落只在 `death` 的 `DropDeathLoot` 里。[离场](../../../raw/current/scripts/stategraphs/SGglommer.lua#L196-L214)、[掉落](../../../raw/current/scripts/stategraphs/commonstates.lua#L1446-L1466)
- **边界：`companion` 不是永久标签。** 换 leader 到非花实体（例如读档后先跟到别处）就不会再加；`OnStopFollowing` 一律删除。[L63-L71](../../../raw/current/scripts/prefabs/glommer.lua#L63-L71)
- **边界：满月不睡。** `ShouldSleep` 里显式 `and not TheWorld.state.isfullmoon`，即使满足距离与夜晚也不会入睡。[L37-L41](../../../raw/current/scripts/prefabs/glommer.lua#L37-L41)
- **误区：死花会像普通食物一样烂掉。** 只挂组件与 6 天定义值，没有 `StartPerishing`，鲜度条恒满、`PERISH_GROUND_MULT` 与冰箱倍率都不参与；要让死花真的腐坏必须自己启动计时（[典型接入](#让死花真正开始腐坏)）。[任务创建](../../../raw/current/scripts/components/perishable.lua#L262-L275)、[百分比](../../../raw/current/scripts/components/perishable.lua#L215-L221)
- **未核验**：本页未运行游戏；格罗姆在实机中的跟随手感、满月花被摘后的再生成节奏、`glommercorpse` 的尸体留存条件都只按源码推导（S7）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `glommerflower.lua` 的 `fn()`、`OnLoseChild`、`OnInit`、`OnPreLoad`／`OnSave`、`RefreshFlowerIcon`；`glommer.lua` 的标签、`follower`／`sleeper`／`sanityaura`／`periodicspawner` 接线与掉落表；`statueglommer.lua` 的 `SpawnGland`／`RemoveGland`／`OnPicked`／`OnWorked`；`glommerbrain.lua` 与 `SGglommer.lua` 的跟随、`goo`、`flyaway`；`leader`／`follower`／`pickable`／`perishable`／`sanityaura`／`sleeper`／`periodicspawner` 的对应方法；`componentutil.lua` 的 `MakeComponentAnInventoryItemSource`、`entityscript.lua` 的 `DropDeathLoot`；`tuning.lua` 的鲜度、燃料、燃烧、光环、范围常量；`languages/chinese_s.po` 的名称键。
- 游戏运行：未运行。天数与秒数均为 tuning 推导（`total_day_time`＝480 秒、`seg_time`＝30 秒），非实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回执。
- 分档说明：本页覆盖 2 个注册＋1 个获得宿主、雕像满月／非满月状态机、leader／follower 双向绑定与死花分支，需要整体保留，故 `size_class: complex`。
- 修订（2026-11-09，检视 P1）：更正死花腐坏——原版只 `SetPerishTime`、全文件无 `StartPerishing`，死花不会腐坏也不会消失；删除“地面 4 天／见底删除”的推理，改记为陷阱，并在[典型接入](#让死花真正开始腐坏)给出补启动示例。
- 修订（同日，检视 P3）：`sources` 补 `components/health.lua`、`stategraphs/commonstates.lua`；格罗姆预载表补 `globalmapiconunderfog`；补闹鬼三项与本节分档说明。
- 修订（2026-11-10，复验 B/C）：典型接入的读档补启动从 `OnPreLoad` 改到实体 `OnLoad`——读档顺序为 `OnPreLoad` → 组件 `OnLoad` → 实体 `OnLoad`，未启动计时的 `perishable` 存档为 `paused=true`、组件 `OnLoad` 只 `StopPerishing`，原写法会被紧随其后的组件加载取消；`paused=false` 措辞改为 `paused` 为空；导航行格罗姆预载表补 `globalmapiconunderfog`。
- 修订（2026-11-10，复验措辞）：L118「按 6 天烂掉」改为「按 6 天腐坏到 0%（不会自动消失）」；L119「两个入口都先跑原版」改为「先跑既有 `OnLoad`（若有）」——原厂 glommerflower 只设 `OnPreLoad`／`OnSave`，没有实体 `OnLoad`。
- 修订（2026-11-10，外部检视 R10／R19）：停用模组后的存档复述改为「已启动过的死花存 `time`、不写 `paused`，停用后原版 `OnLoad` 仍会接着腐坏；从未启动的旧档仍停表」；6 天补注为定义值、地面约 4 天到 0%、到 0% 不自动删除；`aliases` 去掉 `glommerfuel`／`glommerwings`／`statueglommer` 三个精确别名，正文补「两个主注册＋雕像获得宿主」的口径句。

## 相关链接

- 绑定协议：[[reference/components/leader]]、[[reference/components/follower]]
- 周期产出与光环：[[reference/components/periodicspawner]]、[[reference/components/sanityaura]]
- 死花与睡眠：[[reference/components/perishable]]、[[reference/components/sleeper]]
- 蘑菇树孢子：[[reference/prefabs/mushtree-spores]]

返回 [[reference/prefabs/index|Prefab 索引]]。
