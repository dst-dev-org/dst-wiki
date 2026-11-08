---
title: "蘑菇树孢子与月亮孢子（spore_small、spore_medium、spore_tall、spore_moon）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "四种孢子的注册与构造：蘑菇树开花期用 periodicspawner 喷发对应颜色的孢子，孢子是 SGspore＋sporebrain 的地面实体、靠 workable 拾取，落地拆叠、90 秒基准与手持倍率、拥挤判死、可燃与传热，以及月亮孢子改走 SGmoonspore 的爆炸差异。"
aliases: ["spore_small", "spore_medium", "spore_tall", "spore_moon", "绿色孢子", "红色孢子", "蓝色孢子", "月亮孢子", "蘑菇孢子", "mushtree_spores"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mushtree_spores.lua", "prefabs/mushtree_moonspore.lua", "prefabs/mushtree.lua", "prefabs/mushtree_moon.lua", "stategraphs/SGspore.lua", "stategraphs/SGmoonspore.lua", "brains/sporebrain.lua", "prefabs/mushroom_farm.lua", "components/periodicspawner.lua", "components/workable.lua", "components/perishable.lua", "components/stackable.lua", "components/burnable.lua", "components/propagator.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# 蘑菇树孢子与月亮孢子（spore_small、spore_medium、spore_tall、spore_moon）

本页写四个注册的孢子本体与树侧喷发入口。蘑菇帽、蘑菇树本体与树掉落、蘑菇农场的完整种植链已在别处，本页只给入口。

## 快速判断

- 职责与适用：三种彩色孢子（`spore_small`／`spore_medium`／`spore_tall`）是蘑菇树开花期喷出的**地面实体**：有物理、有脑、靠 `workable` 的 `NET` 动作收进背包，能堆叠、会腐坏、能烧；`spore_moon` 是同一族的月亮变体，但改了 SG、不能拿、会爆。[工厂](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L97-L202)、[月亮孢子](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L99-L182)
- 优先使用：查「树怎么喷」看[喷发：蘑菇树的 periodicspawner](#喷发蘑菇树的-periodicspawner)；查「拿在手里能放多久」看[计时：定义与结算](#计时定义与结算)；查月亮孢子为什么不能捡看[月亮孢子差异](#月亮孢子差异)。能种进蘑菇农场的只有三种彩色孢子（月亮孢子没有 `inventoryitem`，拿不起来），映射见 [[reference/prefabs/mushrooms#世界蘑菇与获得路径]]。
- 重要副作用：孢子落地时会把整叠拆成单件、重新打开拥挤检测，并把 `workable` 的 `workleft` 补回 1；半径 20 内同 tag 实体超过 10 个时，任何一个孢子会把自己直接判到 0% 鲜度随后消失。[落地](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L67-L89)、[拥挤](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L47-L56)
- 修改边界：树的开花期由季节驱动（小树春、中树夏、高树冬），本页只写喷发与孢子行为；蘑菇树本体、树掉落、蘑菇帽与蘑菇农场数值归各自页面。

## 注册与构造

四个注册里三个共用 `makespore(data)` 工厂，`spore_moon` 是独立文件里的独立工厂。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spore_tall`（蓝色孢子） | `makespore(data.tall)`：`build = "mushroom_spore_blue"`、光照色 `{111/255, 111/255, 227/255}`；返回表第 1 项。[L204-L206](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L204-L206) | 与 `spore_small`／`spore_medium` 共用 bank `mushroom_spore`、物理与全部组件；差别只有 build 与灯光颜色。[data](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L13-L30) |
| `spore_medium`（红色孢子） | `makespore(data.medium)`：`build = "mushroom_spore_red"`、光照色 `{197/255, 126/255, 126/255}`。[L205-L205](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L205-L205) | 同左；`MUSHTREE_SPORE_RED` 这个模块级常量指向它。[常量](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L2-L4) |
| `spore_small`（绿色孢子） | `makespore(data.small)`：`build = "mushroom_spore"`、光照色 `{146/255, 225/255, 146/255}`。[L206-L206](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L206-L206) | 同左；三个注册共用同一份 `assets`（三张 ANIM）。[资源](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L6-L11) |
| `spore_moon`（月亮孢子） | `mushtree_moonspore.lua` 的 `fn()`：build `mushroom_spore_moon`／bank `spore_moon`，灯光 0.5 强度、0.5 半径；无 `inventoryitem`、无 `locomotor`、无脑。[L99-L182](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L99-L182) | 独立注册名 `spore_moon`（文件名与注册名不同）；见[月亮孢子差异](#月亮孢子差异)。[注册](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L184-L184) |

三个彩色注册的共用构造：`MakeCharacterPhysics(inst, 1, .5)`、bank `mushroom_spore`、播 `flight_cycle`、灯光初始关闭、影子先关，两端都有 `show_spoilage`＋`spore` 两个 tag（S3）；主模拟才挂 `inspectable`／`knownlocations`／`tradable`／`locomotor`（`walkspeed = 2`、关掉地面速度倍率与 creep 触发）／`inventoryitem`（`canbepickedup = false`）／`workable`／`perishable`／`stackable`／`burnable`／`propagator`，最后 `SetStateGraph("SGspore")`＋`SetBrain(brain)`。[L106-L198](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L106-L198)

### 名称核验

`spore_tall`＝**蓝色孢子**（Blue Spore）、`spore_medium`＝**红色孢子**（Red Spore）、`spore_small`＝**绿色孢子**（Green Spore）、`spore_moon`＝**月亮孢子**（Lunar Spore）。**名字与树高不是同一顺序**：小树给绿、中树给红、高树给蓝（S2）。[四个键](../../../raw/current/scripts/languages/chinese_s.po#L349929-L349947)

## 喷发：蘑菇树的 periodicspawner

三棵树各自把孢子写进 `tree_data`：`small.spore = "spore_small"`、`medium.spore = "spore_medium"`、`tall.spore = "spore_tall"`；同一张表里还带着各自的季节、掉落与砍伐次数（S4）。[L309-L351](../../../raw/current/scripts/prefabs/mushtree.lua#L309-L351)

构造时只登记不启动：`AddComponent("periodicspawner")` → `SetPrefab(data.spore)` → `SetOnSpawnFn(onspawnfn)` → `SetDensityInRange(MUSHSPORE_MAX_DENSITY_RAD, MUSHSPORE_MAX_DENSITY)` → `StopSpores(inst)`（清 `_sporetime` 并 `Stop()`）。[L768-L772](../../../raw/current/scripts/prefabs/mushtree.lua#L768-L772)

开花的进入／退出是唯一开关：`swapbuild` 在把 `treestate` 切成 `BLOOMING` 时调 `StartSpores`，切成 `NORMAL` 时调 `StopSpores`；树桩分支里同样 `StopSpores`。`startchange` 还会在进入开花时按当前动画长度补一次 `forcespore`，避免刚开花要等一整个周期。[L451-L485](../../../raw/current/scripts/prefabs/mushtree.lua#L451-L485)、[桩分支](../../../raw/current/scripts/prefabs/mushtree.lua#L577-L605)

周期长度：`periodicspawner` 默认 `basetime = 40`、`randtime = 60`，`Start` 取 `basetime + math.random() * randtime`，即**40～100 秒**一次；被酸雨灌注时 `acid_initialize` 把它改成 `SetRandomTimes(30, 30, false)`（30～60 秒），恢复后 `acid_reset` 再设回 40／60。[默认值](../../../raw/current/scripts/components/periodicspawner.lua#L14-L29)、[取时](../../../raw/current/scripts/components/periodicspawner.lua#L89-L96)、[酸雨两处](../../../raw/current/scripts/prefabs/mushtree.lua#L123-L128)、[恢复](../../../raw/current/scripts/prefabs/mushtree.lua#L147-L152)

两套门要分开看。**生成侧**是 `periodicspawner` 的密度门：`SetDensityInRange(20, 10)` 让 `TrySpawn` 在**宿主周围 20 格**内累计**同 prefab** 实体，`count >= density` 才放弃本次——已有 9 个仍会再产 1 个，它不是全场景稳态上限；酸雨灌注期间 `acid_initialize` 把上限抬到 `MUSHSPORE_MAX_DENSITY * 1.5`＝15，雨停 `acid_reset` 恢复 10。**孢子侧**是拥挤检测：查**所有 `spore` tag**（四色共用），超过 10 个才判死。两套集合不同，数量也不互相封顶。[生成门](../../../raw/current/scripts/components/periodicspawner.lua#L139-L170)、[接线](../../../raw/current/scripts/prefabs/mushtree.lua#L768-L772)、[酸雨 15](../../../raw/current/scripts/prefabs/mushtree.lua#L123-L126)、[恢复 10](../../../raw/current/scripts/prefabs/mushtree.lua#L147-L150)、[拥挤检测](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L47-L56)、[常量](../../../raw/current/scripts/tuning.lua#L826-L830)

生成被密度挡住时 `TrySpawn` 只返回 `false`、不带重试时间，`DoSpawn` 拿到 `nil` 就按 `basetime + math.random() * randtime` 重排——普通 40～100 秒、酸雨 30～60 秒，**不存在 10 秒左右的密度重试**；`basetime × 0.25` 那条 10 秒重试只用于 prefab 无效、离屏限制或 `spawnTest` 失败。[TrySpawn](../../../raw/current/scripts/components/periodicspawner.lua#L117-L170)、[DoSpawn](../../../raw/current/scripts/components/periodicspawner.lua#L199-L203)

出生点与家：`onspawnfn` 播 `cough`＋`idle_loop`、放喷发声，把树的坐标记成孢子的 `knownlocations` 的 `"home"`（脑靠它游荡），再用 `FindWalkableOffset` 把孢子挪到树旁可走点。[L186-L199](../../../raw/current/scripts/prefabs/mushtree.lua#L186-L199)、[脑](../../../raw/current/scripts/brains/sporebrain.lua#L24-L36)

离屏：树的 `onentitysleep` 会 `Stop()`，`onentitywake` 时若已过目标时间就 `ForceNextSpawn()`，否则 `LongUpdate` 把差值补上——**离屏不刷孢子，醒来最多补一次**。[L422-L449](../../../raw/current/scripts/prefabs/mushtree.lua#L422-L449)、[补时](../../../raw/current/scripts/components/periodicspawner.lua#L207-L219)

## 地面实体：SGspore、脑与拾取

`SGspore` 的初始状态是 `takeoff`（播 `cough_out`，第 14 帧才开灯与影子），之后在 `idle`／`moving` 之间切；`death` 状态播 `land`、第 45 帧关灯与影子、动画结束才 `Remove()`。[L109-L132](../../../raw/current/scripts/stategraphs/SGspore.lua#L109-L132)、[初态](../../../raw/current/scripts/stategraphs/SGspore.lua#L137-L137)

`sporebrain` 只做两件事：找 10 格内最近的 `player`／`character`／`monster` 跟过去（跟随距离 2／5／10），找不到就绕 `"home"` 游荡（半径 20）。它不会主动攻击，也没有战斗组件。[L14-L36](../../../raw/current/scripts/brains/sporebrain.lua#L14-L36)

"捡起"不是 `Pick`：孢子挂 `inventoryitem` 但 `canbepickedup = false`，真正的入口是 `workable` 的 `NET` 动作——`SetWorkLeft(1)`、`SetOnFinishCallback(onworked)`，`onworked` 里直接 `worker.components.inventory:GiveItem(inst, nil, inst:GetPosition())` 并放音效。[L155-L161](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L155-L161)、[回调](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L99-L104)

发光与读档：灯光平时关，`takeoff` 第 14 帧才开；`OnLoad` 只做一件事——把灯与影子打开，免得读档后孢子不亮。[L118-L130](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L118-L130)、[读档](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L91-L95)

## 计时：定义与结算

| 项目 | 定义处 | 结算与条件 |
|---|---|---|
| 基准鲜度 | `SetPerishTime(TUNING.SEG_TIME * 3)`＝90 秒，随后立即 `StartPerishing()`（S1）。[L163-L166](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L163-L166) | 采集册同值 `perishable=90`；tuning 另有同值的 `MUSHSPORE_PERISH_TIME`，孢子这边是直写 `SEG_TIME * 3`。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1437-L1440)、[常量](../../../raw/current/scripts/tuning.lua#L826-L826) |
| 地面 | `ondropped` 把 `localPerishMultiplyer` 设回 1。[L67-L72](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L67-L72) | 地面无持有者时结算再乘 `PERISH_GROUND_MULT`＝1.5，所以**实际约 60 秒**（90÷1.5）（S5）。[倍率](../../../raw/current/scripts/tuning.lua#L2083-L2083)、[结算](../../../raw/current/scripts/components/perishable.lua#L127-L133) |
| 手持／容器 | `onputininventory` 里 `SetLocalMultiplier(TUNING.SEG_TIME * 3 / TUNING.PERISH_SLOW)`＝90÷7200＝**0.0125**，并取消拥挤检测。[L58-L65](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L58-L65) | 乘的是腐坏速率不是总时长：90÷0.0125＝7200 秒＝**15 天**（`PERISH_SLOW`＝15 天，S5）；冰箱等容器再乘自己的倍率。[常量](../../../raw/current/scripts/tuning.lua#L2049-L2049)、[localPerishMultiplyer](../../../raw/current/scripts/components/perishable.lua#L196-L198) |
| 拥挤 | 每 `MUSHSPORE_DENSITY_CHECK_TIME + math.random() * MUSHSPORE_DENSITY_CHECK_VAR`＝15～30 秒查一次（首次 1～16 秒），半径 20 内 `spore` tag 超过 `MUSHSPORE_MAX_DENSITY`＝10 个就 `SetPercent(0)`。[L47-L56](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L47-L56)、[首查](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L193-L194) | 判死走 `depleted`：关 `workable`、推 `death`、删 `spore` tag（不再被别人的密度检测算进）、`persists = false`、3 秒兜底 `Remove`。[L34-L45](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L34-L45) |

鲜度条按 `perishtime`＝90 秒显示，手持时只是掉得慢；不要用"总时长 15 天"去反推 `perishtime`。[GetPercent](../../../raw/current/scripts/components/perishable.lua#L215-L221)

拥挤判死的时序：命中后先 `SetPercent(0)`（剩余时间置 0，并在已有更新任务上重启 `StartPerishing`），**下一拍**（`start_dt` 为 0～2 秒随机）的 `Update` 才判 ≤0 并 `Perish`，随后才走 `depleted`（`death`／`pop`、删 `spore` tag、3 秒兜底 `Remove`）。所以从判到拥挤到实体消失还隔着一个更新间隔。[设百分比](../../../raw/current/scripts/components/perishable.lua#L223-L233)、[排更新](../../../raw/current/scripts/components/perishable.lua#L262-L271)、[depleted](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L34-L45)

## 燃烧与落地拆叠

- 可燃：`burnable` 的 `SetFXLevel(1)`、`SetBurnTime(1)`、`AddBurnFX("fire", Vector3(0,0,0), "spore")`，三个回调都用 `DefaultBurnFn`／`DefaultBurntFn`／`DefaultExtinguishFn`；`propagator` 单独配置 `acceptsheat = true`、`flashpoint = 1`、`decayrate = 0.5`、`damages = false`——会传热但自身不造成火焰伤害。[L170-L183](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L170-L183)
- 落地拆叠：`ondropped` 里 `while StackSize() > 1 do` 反复 `stackable:Get()`，对每件调 `inventoryitem:OnDropped()` 并 `Teleport` 到同一格；因此地面上的孢子永远是单件，堆叠只存在于背包。[L74-L84](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L74-L84)
- 落地还补 `workable:SetWorkLeft(1)`：被捡过一次后实体仍能再被收。[L70-L72](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L70-L72)

## 蘑菇农场

带 `spore` tag 的孢子可以当蘑菇农场的种子（S6）；映射（`spore_tall→blue_cap` 等）、产量与 Wormwood 差异见 [[reference/prefabs/mushrooms#世界蘑菇与获得路径]]，本页不重复。孢子侧只补一条：用孢子种出的作物满产（6 份）时，有 `MUSHROOMFARM_SPAWN_SPORE_CHANCE`＝50% 概率再吐一枚同色孢子。[准入](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L265)、[吐孢子](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L145-L159)、[常量](../../../raw/current/scripts/tuning.lua#L1662-L1663)

## 月亮孢子差异

`spore_moon` 与三种彩色孢子共用的只有 `spore` tag、`workable` 的 `NET` 动作、`inspectable`／`stackable`／`burnable`／`propagator`、`MakeHauntablePerish` 与拥挤检测，其余都不同：

| 项目 | 彩色孢子 | `spore_moon` | 证据 |
|---|---|---|---|
| 图／脑 | `SGspore`＋`sporebrain` | `SGmoonspore`，无脑 | [图](../../../raw/current/scripts/stategraphs/SGmoonspore.lua#L98-L98)、[脑缺位](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L163-L182) |
| 持有 | `inventoryitem`（不可点拾，只能 `NET` 收） | 无 `inventoryitem`，收不了 | [L155-L156](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L155-L156)、[L163-L182](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L163-L182) |
| 鲜度 | 90 秒基准 | `MOONSPORE_PERISH_TIME`＝`seg_time * 0.25`＝7.5 秒 | [L141-L144](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L141-L144)、[常量](../../../raw/current/scripts/tuning.lua#L5695-L5698) |
| `NET` 的结果 | 进背包 | `PushEvent("pop")`：不产出物品，改在 3 格内做一次范围攻击 | [回调](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L33-L36)、[爆炸](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L50-L53) |
| 靠近 | 无反应 | 3.5 格内有 `player`／`monster`／`character` 且对方带 `_combat` 就 `pre_pop`（26 帧）再爆；**月亮蘑菇帽产出的孢子已被置 `_alwaysinstantpops`，同一判定直接 `pop`、跳过这 26 帧** | [普通门禁](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L61-L79)、[帽产置位](../../../raw/current/scripts/prefabs/hats.lua#L1937-L1939)、[pre_pop](../../../raw/current/scripts/stategraphs/SGmoonspore.lua#L19-L32) |

爆炸是 `combat:DoAreaAttack(inst, MOONSPORE_ATTACK_RANGE, …)`，半径 3、默认伤害 `MOONSPORE_DAMAGE`＝10，排除表里含 `spore`／`flight`／`playerghost` 等；**鲜度见底同样走 `pop`**，所以地上的月亮孢子到点也会炸。[伤害](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L163-L164)、[到点](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L14-L31)

月亮孢子有**三条入口**，别只写树：①**月亮蘑菇树**——构造时 `SetPrefab("spore_moon")` 后立刻 `Stop()`，本文件里没有周期启动；只有在被砍（`workleft > 0`）时 `ForceNextSpawn()`，`DoSpawn` 之后才按 40～100 秒继续喷，砍成树桩再 `Stop()`。②**月亮蘑菇帽**（`moon_mushroomhat`）——装备时 `periodicspawner:Start()`，`SetRandomTimes(MUSHROOMHAT_MOONSPORE_TIME, …)`＝1.5～1.75 秒一枚，生成回调给孢子置 `_alwaysinstantpops = true`；受击还会按 `MUSHROOMHAT_MOONSPORE_RETALIATION_SPORE_COUNT` 排队 3 次 `TrySpawn`。③**蘑菇地精**（`mushgnome`）——构造时 `SetPrefab("spore_moon")`、`SetRandomTimes(MUSHGNOME_SPORESPAWN_MIN, MAX)`（2／4 → 2～6 秒一轮）并立即 `Start()`；攻击状态在 25～36 帧逐帧 `spawn_spore`（12 枚，落点半径 1～2）；死亡共享表另有 3 个必出＋2 个各 50% 的 `spore_moon`，且它的 `onspawnfn` 不置 `_alwaysinstantpops`。所以“月亮孢子来自月亮树”只覆盖其中一条；帽产会跳过 `pre_pop`，地精与树产走普通邻近判定。[树入口](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L310-L314)、[砍树触发](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L155-L166)、[帽装备启动](../../../raw/current/scripts/prefabs/hats.lua#L1871-L1889)、[帽产回调](../../../raw/current/scripts/prefabs/hats.lua#L1937-L1973)、[地精周期](../../../raw/current/scripts/prefabs/mushgnome.lua#L185-L199)、[地精攻击生成](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L199-L240)、[地精掉落表](../../../raw/current/scripts/prefabs/mushgnome.lua#L18-L29)、[DoSpawn 重启](../../../raw/current/scripts/components/periodicspawner.lua#L199-L203)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：workable | `ACTIONS.NET`、`workleft = 1`；`onworked` 才是拾取入口 | 仅主模拟；落地补回 1 | [[reference/components/workable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L158-L161) |
| 组件：perishable | 90 秒基准；手持 0.0125 倍；见底走 `depleted` | 仅主模拟；拥挤命中先 `SetPercent(0)`，下一拍才 `Perish` | [[reference/components/perishable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L163-L166) |
| 组件：stackable | 默认 20（`STACK_SIZE_MEDITEM`），落地强制拆单 | 仅主模拟；背包内才成叠 | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/tuning.lua#L82-L82)、[拆叠](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L74-L84) |
| 组件：burnable／propagator | 1 秒烧完、火焰 FX 绑 `spore` 符号；传热但 `damages = false` | 仅主模拟（端分界之后才挂）；月亮孢子用同一套，只把 FX 符号换成 `spore_body` | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L170-L183) |
| 组件：periodicspawner（树） | 产物按树的 `data.spore`，密度 20／10（酸雨 15），开花才 `Start` | 仅主模拟；离屏停、醒来补一次 | [[reference/components/periodicspawner#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushtree.lua#L768-L772) |
| SG／Brain：`SGspore`／`sporebrain` | 初态 `takeoff`，游荡＋跟随最近角色；无攻击 | `SetStateGraph`／`SetBrain` 在端分界之后，只有主模拟副本挂载；月亮孢子换 `SGmoonspore` | [SG](../../../raw/current/scripts/stategraphs/SGspore.lua#L28-L137)、[脑](../../../raw/current/scripts/brains/sporebrain.lua#L24-L36) |
| API／数据：`MUSHSPORE_*` | 生成门：宿主 20 格内同 prefab 上限 10（酸雨 15）；拥挤门：全部 `spore` tag >10；检测 15～30 秒（首查 1～16 秒） | 主模拟与孢子自身都读同一组常量 | [常量](../../../raw/current/scripts/tuning.lua#L826-L830) |
| 关联 Prefab：`mushtree_small`／`mushtree_medium`／`mushtree_tall`／`mushtree_moon` | 喷发宿主；预载表里各自声明对应孢子 | 声明不等于生成，实际由 `periodicspawner` 产 | [预载](../../../raw/current/scripts/prefabs/mushtree.lua#L843-L862)、[月亮树](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L348-L358) |
| 关联 Prefab：`moon_mushroomhat` | 装备期 `periodicspawner` 产 `spore_moon`（1.5～1.75 秒一枚），生成即置 `_alwaysinstantpops`；受击另排队 3 次 `TrySpawn` | 主模拟；帽的其余组件归 [[reference/prefabs/hats-environment#地形改造与孢子链]] | [装备启动](../../../raw/current/scripts/prefabs/hats.lua#L1871-L1889)、[帽产回调](../../../raw/current/scripts/prefabs/hats.lua#L1937-L1973) |
| 关联 Prefab：`mushgnome` | 构造即 `periodicspawner:Start()` 产 `spore_moon`（2～6 秒）；攻击 25～36 帧逐帧生成；死亡表含 3 必出＋2×50% | 主模拟；`onspawnfn` 不置 `_alwaysinstantpops`，走普通邻近判定 | [周期](../../../raw/current/scripts/prefabs/mushgnome.lua#L185-L199)、[攻击生成](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L79-L85)、[掉落](../../../raw/current/scripts/prefabs/mushgnome.lua#L18-L29) |
| 关联 Prefab：`mushroom_farm` | 孢子作种子，映射到三种蘑菇帽 | 主模拟；见[蘑菇农场](#蘑菇农场) | [[reference/prefabs/mushrooms#世界蘑菇与获得路径]]；[映射](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L33-L38) |
| 资源：`anim/mushroom_spore*.zip` | 三种彩色孢子共用 bank `mushroom_spore`，build 分三份；月亮孢子另有 `spore_moon`／`mushroom_spore_moon` | 原版资源，不随页分发 | [彩色资源](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L6-L11)、[月亮资源](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L1-L5) |

## 常见陷阱与限制

- **误区：孢子能直接捡。** `inventoryitem` 的 `canbepickedup` 被设为 `false`，点地不会拾取；动作是 `NET`（虫网类），入口在 `workable`。[L155-L156](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L155-L156)、[L158-L161](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L158-L161)
- **边界：手持倍率乘的是腐坏速率。** `SetLocalMultiplier(0.0125)` 让"每秒掉多少鲜度"变小，`perishtime` 仍是 90 秒；15 天是 90÷0.0125 推导出来的等效时长。[L58-L65](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L58-L65)、[结算](../../../raw/current/scripts/components/perishable.lua#L127-L133)
- **边界：地面上活不到 90 秒。** 无持有者时还有 `PERISH_GROUND_MULT`＝1.5，实际约 60 秒；把孢子放在地上等它"慢慢过期"会提前消失。[倍率](../../../raw/current/scripts/tuning.lua#L2083-L2083)
- **误区：堆叠的孢子落地还是一叠。** `ondropped` 主动拆单，所以地面永远是散件；想省实体个数不能靠"丢一叠"。[L74-L84](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L74-L84)
- **边界：拥挤是按 tag 数，不按同 prefab。** 检测参数是 `SPORE_TAGS = {"spore"}`，所以绿／红／蓝／月亮孢子互相挤；月亮孢子**只有捕捉（`onworked`）与 `depleted` 会删 `spore` tag**——邻近分支只发 `pop`／`preparedpop`，状态机在动画结束才 `Remove()` 实体，本身不删 tag，所以它消失前仍可能被别人的密度检测算进去。[L47-L56](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L47-L56)、[捕捉与 depleted](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L14-L36)、[邻近分流](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L61-L79)、[pop 状态](../../../raw/current/scripts/stategraphs/SGmoonspore.lua#L34-L56)
- **边界：月亮孢子捡不到也留不久。** 它没有 `inventoryitem`、只有 7.5 秒基准（地面约 5 秒），`NET` 与到点都会爆炸，别把它当可收集材料。[L141-L144](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L141-L144)、[L163-L182](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L163-L182)
- **未核验**：本页未运行游戏；拥挤阈值的实战密度、蘑菇农场用孢子与用帽子的产量差异、月亮孢子爆炸的实际命中范围都只按源码推导（S7）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `mushtree_spores.lua` 的 `makespore`、四个回调（`depleted`／`checkforcrowding`／`onpickup`／`ondropped`）、三注册返回表；`mushtree_moonspore.lua` 的 `fn()`、`onworked`／`onpopped`、邻近检测与 `depleted`；`mushtree.lua` 的 `tree_data`、`StartSpores`／`StopSpores`／`swapbuild`／`onspawnfn` 与 `periodicspawner` 接线；`mushtree_moon.lua` 的 `spore` 配置与砍树触发；`SGspore`／`SGmoonspore`／`sporebrain`；`mushroom_farm.lua` 的 `spore_to_cap`／`StartGrowing`／`ongrow`／`accepttest`；`periodicspawner` 的默认值、密度与 `LongUpdate`；`perishable` 的倍率结算；`tuning.lua` 与 `languages/chinese_s.po` 的相关键；`scrapbookdata.lua` 四行采集册交叉核对。
- 游戏运行：未运行。所有秒数与天数为 tuning 推导（`total_day_time`＝480 秒、`seg_time`＝30 秒），非实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回执。
- 典型接入：本页没有已核验的完整模组案例（拾取、计时、喷发入口都已在正文接口段给全），按模板删除该节。
- 修订（2026-11-09，检视 P2）：密度稳态改为 10（酸雨期 15）；补月亮孢子邻近判定的 `_combat` 前置；蘑菇农场节压缩为两句并只保留孢子侧的 50% 吐孢子；`sources` 补 `screens/redux/scrapbookdata.lua`。
- 修订（2026-11-10，复验 R03／R12）：月亮孢子改**三条入口**（补蘑菇地精：构造启动周期喷发 2～6 秒、攻击 25～36 帧逐帧生成、死亡表 3 必出＋2×50%，其 `onspawnfn` 不置 `_alwaysinstantpops`）；陷阱限定删 `spore` tag 只发生在捕捉与 `depleted`，邻近分支只发 `pop`、动画结束才移除实体。
- 修订（2026-11-10，外部检视 R03／R12）：补月亮孢子第二条来源「月亮蘑菇帽」（装备期 periodicspawner 1.5～1.75 秒、`_alwaysinstantpops` 跳过 `pre_pop`、受击排队 3 次）；把生成门（宿主 20 格同 prefab、酸雨 15）与拥挤门（全部 `spore` tag >10）分开写；删除「密度挡住按 10 秒重试」的错误说法（`TrySpawn` 返回不带重试时间，`DoSpawn` 按正常随机间隔重排）；补 `SetPercent(0)` 到 `Perish` 的时序与月亮帽导航行。

## 相关链接

- 蘑菇帽、树掉落与农场映射：[[reference/prefabs/mushrooms]]
- 喷发机制：[[reference/components/periodicspawner]]
- 拾取与鲜度：[[reference/components/workable]]、[[reference/components/perishable]]
- 火焰传播：[[reference/components/burnable]]、[[reference/components/propagator]]
- 格罗姆与花：[[reference/prefabs/glommer-flower]]

返回 [[reference/prefabs/index|Prefab 索引]]。
