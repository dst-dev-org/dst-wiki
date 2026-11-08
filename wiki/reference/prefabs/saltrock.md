---
title: "盐晶（saltrock）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/saltrock.lua 单注册的海盐物品（简中“盐晶”）：堆叠 40、ELEMENTAL 食性 1、可交易但 goldvalue 为 0、默认浮水、可放进陷阱诱饵槽（鸟侧还需 CanEat，盐晶不吸引鸟）与鼹鼠诱饵；preservative 只接'对着食物右击撒盐'动作（夹取前 +50 个百分点、按目标叠数除并夹到 100%，满鲜目标仍扣件），repairer 走 SALT 的 finiteuses 通道给美味舔盐块补 80 次（480/6）。"
aliases: ["saltrock", "盐晶", "Salt Crystals", "海盐"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/saltrock.lua", "components/preservative.lua", "components/repairer.lua", "components/bait.lua", "components/trap.lua", "components/eater.lua", "components/edible.lua", "components/snowmandecoratable.lua", "constants.lua", "components/finiteuses.lua", "components/repairable.lua", "components/tradable.lua", "actions.lua", "componentactions.lua", "components/birdspawner.lua", "brains/molebrain.lua", "brains/rockybrain.lua", "brains/slurtlebrain.lua", "brains/dustmothbrain.lua", "prefabs/dustmoth.lua", "prefabs/slurtle.lua", "prefabs/rocky.lua", "prefabs/mole.lua", "components/perishable.lua", "standardcomponents.lua", "prefabs/saltstack.lua", "prefabs/polly_rogers.lua", "prefabs/moonrocknugget.lua", "stategraphs/SGsalty_dog.lua", "prefabs/meatrack2.lua", "components/dryingracksaltcollector.lua", "prefabs/saltlick.lua", "prefabs/saltbox.lua", "prefabs/pigking.lua", "messagebottletreasures.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 盐晶（saltrock）

## 快速判断

- 职责与适用：`prefabs/saltrock.lua` 单注册 1 件 item，官方简中"盐晶"（[chinese_s.po L348930-L348932](../../../raw/current/scripts/languages/chinese_s.po#L348930-L348932)）。堆叠 40、默认浮水、`molebait` 标签、`bait` 组件、`preservative`（+0.5）与 SALT `repairer` 都在 master 段挂载；资源是 `anim/salt.zip`，bank／build 同为 `salt`（[L1-L4](../../../raw/current/scripts/prefabs/saltrock.lua#L1-L4)、[L37-L39](../../../raw/current/scripts/prefabs/saltrock.lua#L37-L39)）。
- 优先使用：要判断"盐晶能不能保鲜／怎么补盐舔块"看下文"保鲜与修复口径"；陷阱诱饵口径见"获得路径"后的陷阱条目；配方消耗看 recipes.lua 清单。
- 重要副作用：`preservative.percent_increase = 0.5` 只在 `APPLYPRESERVATIVE` 动作里结算，目标堆叠越大单次收益越小（除以目标 `stacksize`）；它**不是**盐罐（`saltbox`）那种容器级保鲜（[actions.lua L4620-L4645](../../../raw/current/scripts/actions.lua#L4620-L4645)、[saltbox.lua L79-L87](../../../raw/current/scripts/prefabs/saltbox.lua#L79-L87)）。
- 修改边界：本页不覆盖盐舔块（`saltlick`／`saltlick_improved`）本体机制、珍珠晾肉架（`meatrack2`）的晾制机制与咸狗行为 AI；这些只在获得／修复条目里写到消费所需。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `saltrock` | `fn()` 单工厂：网络段建实体 + `MakeInventoryPhysics` + `SetRayTestOnBB(true)` + bank/build/`idle` + `pickupsound = "rock"` → 加 `molebait` 标签 → `MakeInventoryFloatable` → `SetPristine` → 非 master 返回 → master 段先算配方缓存再挂 8 个组件 → 文件末 `Prefab("saltrock", fn, assets)` | 无变体、无皮肤；`MakeInventoryFloatable` 在 `SetPristine` **前**（两端都浮水），与月岩的 `SetSinks(true)` 相反（[L26-L51](../../../raw/current/scripts/prefabs/saltrock.lua#L26-L51)、[L81](../../../raw/current/scripts/prefabs/saltrock.lua#L81)） |

master 段实际挂载：

- `bait`（[L57](../../../raw/current/scripts/prefabs/saltrock.lua#L57)）、`inspectable`、`inventoryitem`、`tradable`（[L58-L60](../../../raw/current/scripts/prefabs/saltrock.lua#L58-L60)）。`tradable` 不设 `goldvalue`，保持 0（[tradable.lua L1-L7](../../../raw/current/scripts/components/tradable.lua#L1-L7)），猪王只收 `> 0`（[pigking.lua L604-L605](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605)）。
- `edible`：`FOODTYPE.ELEMENTAL`、`hungervalue = 1`；未写 `healthvalue`／`sanityvalue`，即沿用组件默认 health 10、sanity 0（[L62-L64](../../../raw/current/scripts/prefabs/saltrock.lua#L62-L64)、[edible.lua L19-L24](../../../raw/current/scripts/components/edible.lua#L19-L24)）。
- `stackable`：`TUNING.STACK_SIZE_SMALLITEM` = 40（[L66-L67](../../../raw/current/scripts/prefabs/saltrock.lua#L66-L67)、[tuning.lua L83](../../../raw/current/scripts/tuning.lua#L83)）。
- `preservative`：`percent_increase = TUNING.SALTROCK_PRESERVE_PERCENT_ADD` = 0.5（[L69-L70](../../../raw/current/scripts/prefabs/saltrock.lua#L69-L70)、[tuning.lua L4114](../../../raw/current/scripts/tuning.lua#L4114)）；组件默认 `divide_effect_by_stack_size = true`（[preservative.lua L1-L9](../../../raw/current/scripts/components/preservative.lua#L1-L9)）。
- `repairer`：`MATERIALS.SALT`、只设 `finiteusesrepairvalue = TUNING.SALTLICK_IMPROVED_MAX_LICKS / CACHED_SALT_LICK_IMPROVED_RECIPE_COST`（[L72-L74](../../../raw/current/scripts/prefabs/saltrock.lua#L72-L74)），健康／work 两条为 0，因此只产生 `finiteuses_salt` 标签（[repairer.lua L8-L24](../../../raw/current/scripts/components/repairer.lua#L8-L24)）。
- `MakeHauntableLaunchAndSmash`（[L76](../../../raw/current/scripts/prefabs/saltrock.lua#L76)）：smash 分支在本快照被注释，实际只有 Launch（[standardcomponents.lua L1071-L1105](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)）。
- 未挂：`fuel`、`perishable`、`snowmandecor`（全文 81 行内无这些组件；雪人装饰数据表也没有 saltrock，[snowmandecoratable.lua L5-L42](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L42)）。

## 配方缓存与修复口径

- 模块级缓存：文件顶部 `local CACHED_SALT_LICK_IMPROVED_RECIPE_COST = nil`（[L6](../../../raw/current/scripts/prefabs/saltrock.lua#L6)），首次 master 侧生成盐晶时写入一次（[L53-L55](../../../raw/current/scripts/prefabs/saltrock.lua#L53-L55)），之后该进程内不再重算——模组若在首个盐晶生成后才改配方，本值仍是旧值。
- 取数逻辑：读 `AllRecipes.saltlick_improved`，配方或其 `ingredients` 为 nil 时返回默认 6；否则累加 `ingredient.type == "saltrock"` 的金额，和 `> 0` 才采用，否则同样回落 6（[L8-L24](../../../raw/current/scripts/prefabs/saltrock.lua#L8-L24)）。当前配方是 2 板 + 6 盐晶（[recipes.lua L585](../../../raw/current/scripts/recipes.lua#L585)），故缓存值 = 6。
- 修复量：`SALTLICK_IMPROVED_MAX_LICKS` = 480（[tuning.lua L8270](../../../raw/current/scripts/tuning.lua#L8270)），480 / 6 = **80 次/件**；目标是美味舔盐块（`saltlick_improved`）的 `finiteuses`，其上限也是 480（[saltlick.lua L216-L221](../../../raw/current/scripts/prefabs/saltlick.lua#L216-L221)）。普通盐舔块（`saltlick`）不挂 `repairable`，盐晶修不了它。
- 门禁与扣件：客户端出现 REPAIR 需目标 `repairable_salt` + `finiteusesrepairable`（由 `finiteuses` 在未满时打开，[finiteuses.lua L4](../../../raw/current/scripts/components/finiteuses.lua#L4)）与手持物 `finiteuses_salt` 同时成立（[componentactions.lua L1688-L1696](../../../raw/current/scripts/componentactions.lua#L1688-L1696)）；结算只进 `finiteuses` 分支，满值 `GetPercent() >= 1` 直接拒绝且不消耗（[repairable.lua L135-L142](../../../raw/current/scripts/components/repairable.lua#L135-L142)）。

## 保鲜口径

- 消费者只有 `APPLYPRESERVATIVE`：右击持有盐晶对目标食物施用时，**客户端门禁**要求目标 `health == nil`、带 `fresh`／`stale`／`spoiled` 之一、带 `cookable`、且不是 `deployable`／`smallcreature`（[componentactions.lua L1574-L1582](../../../raw/current/scripts/componentactions.lua#L1574-L1582)）；**服务端 `fn`** 复核查 `health == nil`、带 `fresh`／`stale`／`spoiled` 之一、带 `cookable`、不是 `deployable`，同样不再查 `smallcreature`（[actions.lua L4622-L4625](../../../raw/current/scripts/actions.lua#L4622-L4625)）。
- 结算：先把目标 `perishable` 的百分比设为 `当前 + percent_increase / 目标 stacksize`（目标可堆叠时），`SetPercent` 会夹到 0～1（[perishable.lua L223-L232](../../../raw/current/scripts/components/perishable.lua#L223-L232)），随后才从背包移出该件并 `Remove`（[actions.lua L4627-L4641](../../../raw/current/scripts/actions.lua#L4627-L4641)）。所以 0.5 是**夹取前 +50 个百分点**：4 个一叠每件只 +12.5 个百分点；90% 的食物升到 100%（只涨 10 个百分点）；已经 100% 的目标照样会被扣掉 1 件。取件失败时 `fn` 返回 false，但已写入的百分比**不回滚**。
- 与舔盐块 REPAIR 的区别：撒盐不看目标是否已满、也不回滚；`saltlick_improved` 的 REPAIR 走 `repairable` 的 finiteuses 分支，满耐久直接返回 false、不扣盐晶（[repairable.lua L135-L142](../../../raw/current/scripts/components/repairable.lua#L135-L142)）。两条路径互不影响。
- 与盐罐的区别：`saltbox` 用 `preserver` 组件按容器倍率统一减缓腐坏（[saltbox.lua L79-L87](../../../raw/current/scripts/prefabs/saltbox.lua#L79-L87)），盐晶只是它的建造材料，不参与该倍率；不要把 `preservative` 当成"放进容器就生效"。
- 其他"保鲜"字样不属本组件：`freshen_salt` 是 `repairer` 在 `perishrepairpercent > 0` 时给自己（即手持修复材料）加的标签（[repairer.lua L17-L19](../../../raw/current/scripts/components/repairer.lua#L17-L19)），客户端据此要求目标带 `fresh`／`stale`／`spoiled`（[componentactions.lua L1692](../../../raw/current/scripts/componentactions.lua#L1692)）；本 prefab 该值为 0，不会加此标签，不要把修复标签当成防腐。

## 获得路径

- **盐柱（`saltstack`）采掘**：海上浮盐柱，`MINE` 采掘，`workleft` 档位 10/6/3/0 对应 full／med／low／empty（[saltstack.lua L60](../../../raw/current/scripts/prefabs/saltstack.lua#L60)、[L287-L291](../../../raw/current/scripts/prefabs/saltstack.lua#L287-L291)）；每降一档掉该档表，一次挖穿多档时 `DropLoots` 会把跨过的档位表逐张结算（[L62-L85](../../../raw/current/scripts/prefabs/saltstack.lua#L62-L85)）：full 档 1 枚必得 + 50%，med／low 档各 1 枚必得（[L13-L28](../../../raw/current/scripts/prefabs/saltstack.lua#L13-L28)）。
- **生长计时**：每次 `OnWork` 都会重排生长计时，不是只有采空才启动（[saltstack.lua L151-L154](../../../raw/current/scripts/prefabs/saltstack.lua#L151-L154)）；`Grow` 时若船体压在柱上则本轮跳过增长，未满仍会重排（[L133-L145](../../../raw/current/scripts/prefabs/saltstack.lua#L133-L145)）。计时走 `worldsettingstimer`：时长 = `SALTSTACK_GROWTH_FREQUENCY` + 随机 × `SALTSTACK_GROWTH_FREQUENCY_VARIANCE`，再除以正的世界设置倍率；只有 `SALTSTACK_GROWTH_ENABLED` 且 `REGROWTH_TIME_MULTIPLIER > 0` 时该计时才启用（[L121-L131](../../../raw/current/scripts/prefabs/saltstack.lua#L121-L131)、[L296-L302](../../../raw/current/scripts/prefabs/saltstack.lua#L296-L302)、[tuning.lua L3219](../../../raw/current/scripts/tuning.lua#L3219)、[L4117-L4119](../../../raw/current/scripts/tuning.lua#L4117-L4119)）。
- **珍珠晾肉架（`meatrack2`）副产**：带 `driedsalticon.collects` 的晾干物进入槽位时（`itemget`）先给该槽记一份盐并显示盐图标（[meatrack2.lua L395-L409](../../../raw/current/scripts/prefabs/meatrack2.lua#L395-L409)、[dryingracksaltcollector.lua L12-L22](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L22)）；把晾干物**取走**时（`itemlose` → `hermit_DoItemTaken`）才 `RemoveSalt` 并生成 1 枚盐晶——槽已空则放进槽里，槽里有别的物品则掉在地上（[L413-L428](../../../raw/current/scripts/prefabs/meatrack2.lua#L413-L428)、[L430-L440](../../../raw/current/scripts/prefabs/meatrack2.lua#L430-L440)）；拆架时把存量一次性倒出（[L445-L456](../../../raw/current/scripts/prefabs/meatrack2.lua#L445-L456)）。机制归 [[reference/components/dryingracksaltcollector]]。
- **咸狗（`salty_dog`，注册在 polly_rogers.lua）**：计数 `salty` 在水中按 `SALTY_DOG_TIME_TO_SALT` 递增、上限 `SALTY_DOG_MAX_SALT_COUNT`，离水暂停（[polly_rogers.lua L168-L193](../../../raw/current/scripts/prefabs/polly_rogers.lua#L168-L193)）；"抖盐"动作每个时间点按 0.2–0.8 概率抖 1 枚、末帧清空剩余（[SGsalty_dog.lua L96-L140](../../../raw/current/scripts/stategraphs/SGsalty_dog.lua#L96-L140)、[polly_rogers.lua L123-L151](../../../raw/current/scripts/prefabs/polly_rogers.lua#L123-L151)）。
- 其他：漂流瓶宝藏按 `saltrock = {5, 8}` 抽 5–8 枚（[messagebottletreasures.lua L54](../../../raw/current/scripts/messagebottletreasures.lua#L54)），本条只记表值，未核验其触发链。

## 主要消费

- **配方**（量按 [recipes.lua](../../../raw/current/scripts/recipes.lua)）：盐调味料 3→2（[L104](../../../raw/current/scripts/recipes.lua#L104)）、精制粉尘 1（[L472](../../../raw/current/scripts/recipes.lua#L472)）、盐罐 10（[L564](../../../raw/current/scripts/recipes.lua#L564)）、美味舔盐块 6（[L585](../../../raw/current/scripts/recipes.lua#L585)）、贝壳编织者 9 条（7 条各 1：[L980-L986](../../../raw/current/scripts/recipes.lua#L980-L986)；`salty_doghat` 5：[L987](../../../raw/current/scripts/recipes.lua#L987)；`hermitcrab_shell` 3：[L988](../../../raw/current/scripts/recipes.lua#L988)）、冬季盛宴腌鲱鱼 1（[L1440](../../../raw/current/scripts/recipes.lua#L1440)）。
- **修复**：美味舔盐块每次 +80 次耐久（见上节）；盐晶不修其他任何 SALT 目标——全 scripts 检索 `repairable.repairmaterial = MATERIALS.SALT` 只有 `saltlick.lua` 一处（[saltlick.lua L216-L218](../../../raw/current/scripts/prefabs/saltlick.lua#L216-L218)）。
- **陷阱诱饵**：`bait` 组件让盐晶能通过 `ACTIONS.BAIT` 放进带 `canbait` 标签的陷阱（[componentactions.lua L1046-L1050](../../../raw/current/scripts/componentactions.lua#L1046-L1050)、[trap.lua L412-L425](../../../raw/current/scripts/components/trap.lua#L412-L425)）；诱饵槽是否空闲由 `Bait:IsFree` 判定（[bait.lua L42-L44](../../../raw/current/scripts/components/bait.lua#L42-L44)）。鸟选落点的两条分支共用外层门禁（鸟自身有 `eater` 且未被 `ignorebait` 排除，[birdspawner.lua L536-L537](../../../raw/current/scripts/components/birdspawner.lua#L536-L537)）：可食诱饵分支要求 `eater:CanEat` 且物品带 `bait`——盐晶是 ELEMENTAL，鸟吃不掉，所以不会成为可吃诱饵（[L541-L549](../../../raw/current/scripts/components/birdspawner.lua#L541-L549)）；陷阱分支要求陷阱 `isset`、未 `issprung`、`targettag` 为空或与鸟标签匹配，**且** `IsDangerNearby` 为假，再按 `BIRD_TRAP_CHANCE` 掷中，与诱饵是否可食无关（[L550-L559](../../../raw/current/scripts/components/birdspawner.lua#L550-L559)）。所以"放盐晶进鸟陷阱"只占诱饵槽、不吸引鸟。
- **鼹鼠诱饵**：`molebait` 标签 + `bait` 组件同时命中鼹鼠的偷饵检索（`IsMoleBait` 要求 `bait` 组件，`TAKEBAIT_MUST_TAGS = {"molebait"}`，[molebrain.lua L56-L73](../../../raw/current/scripts/brains/molebrain.lua#L56-L73)），地上的盐晶会被鼹鼠顺走。
- **进食者（食性允许 ≠ 行为会吃）**：普通角色 `OMNI` 食性不含 `ELEMENTAL`（[constants.lua L2049-L2063](../../../raw/current/scripts/constants.lua#L2049-L2063)、[eater.lua L18-L24](../../../raw/current/scripts/components/eater.lua#L18-L24)）。组件层 `SetDiet` 含 `ELEMENTAL` 的生物都**允许**吃（鼹鼠 [mole.lua L229](../../../raw/current/scripts/prefabs/mole.lua#L229)、岩石龙虾 [rocky.lua L436](../../../raw/current/scripts/prefabs/rocky.lua#L436)、蜗牛龟 [slurtle.lua L150](../../../raw/current/scripts/prefabs/slurtle.lua#L150)、尘蛾 [dustmoth.lua L257](../../../raw/current/scripts/prefabs/dustmoth.lua#L257)），但是否去取食看各自行为入口：岩石龙虾的觅食以 `edible_ELEMENTAL` 为目标标签（[rockybrain.lua L83](../../../raw/current/scripts/brains/rockybrain.lua#L83)）、蜗牛龟按 `eater:CanEat` 搜地面食物（[slurtlebrain.lua L37-L50](../../../raw/current/scripts/brains/slurtlebrain.lua#L37-L50)）；尘蛾的进食与交易还要求 `dustmothfood` 标签（[dustmoth.lua L62-L66](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66)、[dustmothbrain.lua L57-L77](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L77)）、鼹鼠偷饵要求 `molebait`（[molebrain.lua L64-L73](../../../raw/current/scripts/brains/molebrain.lua#L64-L73)），这里要分开看：盐晶**有** `bait` 与 `molebait`（[saltrock.lua L41-L57](../../../raw/current/scripts/prefabs/saltrock.lua#L41-L57)），会被鼹鼠选作偷饵；但它**没有** `dustmothfood`，尘蛾的日常取食（[dustmothbrain.lua L57-L85](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L85)）与交易（[dustmoth.lua L62-L66](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66)）都不会选它。吃到时按食物组件参数回 1 饱食、10 生命（`hungervalue` 显式设 1、`healthvalue` 取默认 10），实际回复还受进食者 absorption 等修饰。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：preservative | 0.5 新鲜度补偿；按目标堆叠均摊 | 服务端；右击动作触发一次消耗 1 件 | [[reference/components/preservative]]；[actions.lua L4620-L4645](../../../raw/current/scripts/actions.lua#L4620-L4645) |
| 组件：repairer | 只给 `finiteuses_salt`（80/件） | 服务端；缓存配方成本 | [[reference/components/repairer]]；[saltrock.lua L72-L74](../../../raw/current/scripts/prefabs/saltrock.lua#L72-L74) |
| 组件：repairable／finiteuses | 美味舔盐块的修复目标侧 | 服务端；未满才可修 | [[reference/components/repairable]]；[saltlick.lua L216-L221](../../../raw/current/scripts/prefabs/saltlick.lua#L216-L221) |
| 组件：bait／trap | 诱饵槽接线；鸟侧可食诱饵要 `CanEat`，陷阱分支要求陷阱状态、`targettag`（可空）与周边无危险 | 服务端；陷阱需 `canbait` | [[reference/components/bait]]、[[reference/components/trap]]；[birdspawner.lua L534-L560](../../../raw/current/scripts/components/birdspawner.lua#L534-L560) |
| 组件：edible／tradable／stackable | ELEMENTAL 1、无 goldvalue、40／堆 | 服务端；玩家吃不到、猪王不收 | [[reference/components/eater]]、[[reference/components/tradable]]；[saltrock.lua L57-L67](../../../raw/current/scripts/prefabs/saltrock.lua#L57-L67) |
| API／数据：`CACHED_SALT_LICK_IMPROVED_RECIPE_COST` | 文件级缓存，首件生成时定值 | 服务端进程级；非存档字段 | [saltrock.lua L6-L24](../../../raw/current/scripts/prefabs/saltrock.lua#L6-L24) |
| 关联 Prefab：`saltbox` | 建造材料；容器级 `preserver` 与盐晶无关 | 服务端；配方 10 盐晶 | [saltbox.lua L79-L87](../../../raw/current/scripts/prefabs/saltbox.lua#L79-L87) |
| 关联 Prefab：`saltlick_improved` | 修复目标；配方消耗 6 | 服务端；上限 480 次 | [saltlick.lua L209-L224](../../../raw/current/scripts/prefabs/saltlick.lua#L209-L224) |
| 关联 Prefab：`saltstack`／`meatrack2`／`salty_dog` | 主要获得源 | 采掘／晾制副产／宠物抖盐 | [[reference/components/dryingracksaltcollector]]；[saltstack.lua L13-L28](../../../raw/current/scripts/prefabs/saltstack.lua#L13-L28) |
| 关联 Prefab：`moonrocknugget` | 同批矿物材料对照：同为 ELEMENTAL、40 堆，月岩沉水、盐晶浮水 | 服务端 | [[reference/prefabs/moonrocknugget]]；[moonrocknugget.lua L40](../../../raw/current/scripts/prefabs/moonrocknugget.lua#L40) |
| 资源 | bank／build 均为 `salt`，包 `anim/salt.zip` | 两端；无皮肤分支 | [saltrock.lua L1-L4](../../../raw/current/scripts/prefabs/saltrock.lua#L1-L4) |

## 常见陷阱与限制

- **误区：放进容器就能防腐**：`preservative` 只被 `APPLYPRESERVATIVE` 读取（全 scripts 检索该组件仅 `actions.lua` 三处 + prefab 挂载处），盐罐的保鲜来自 `preserver`（[actions.lua L4621-L4631](../../../raw/current/scripts/actions.lua#L4621-L4631)、[saltbox.lua L86-L87](../../../raw/current/scripts/prefabs/saltbox.lua#L86-L87)）。
- **误区：0.5 就是半条命**：结算写的是**夹取前**的百分比增量，且除以目标 `stacksize`：4 个一叠每件只 +12.5 个百分点；90% 的食物一件只把它推到 100%（实得 10 个百分点）；对已经 100% 的食物，动作候选仍然成立、盐晶照样被扣（[actions.lua L4627-L4638](../../../raw/current/scripts/actions.lua#L4627-L4638)、[perishable.lua L223-L232](../../../raw/current/scripts/components/perishable.lua#L223-L232)、[componentactions.lua L1574-L1582](../../../raw/current/scripts/componentactions.lua#L1574-L1582)）。
- **边界：放盐晶进鸟陷阱不吸引鸟**：两条分支都在同一外层门禁内——鸟自身有 `eater` 且未被 `ignorebait` 排除（[birdspawner.lua L536-L537](../../../raw/current/scripts/components/birdspawner.lua#L536-L537)）。可食诱饵分支还要求 `eater:CanEat`、物品带 `bait`、未被持有、周边无危险，且**鸟带 `floater` 或落点可通行**；盐晶是 ELEMENTAL 不可食，所以不会被选（[L541-L549](../../../raw/current/scripts/components/birdspawner.lua#L541-L549)）。鸟仍可能落到陷阱上，但那是同层另一分支：陷阱 `isset`、未 `issprung`、`targettag` 为空**或**鸟带该标签，再按 `BIRD_TRAP_CHANCE` 掷中、周边无危险（[L550-L559](../../../raw/current/scripts/components/birdspawner.lua#L550-L559)）。
- **边界：配方成本只算一次**：`CACHED_SALT_LICK_IMPROVED_RECIPE_COST` 在首件盐晶生成时定值，热改配方不会让已运行的世界重算；默认 6 是"配方读不到"的回退，不是常量（[saltrock.lua L8-L24](../../../raw/current/scripts/prefabs/saltrock.lua#L8-L24)、[L53-L55](../../../raw/current/scripts/prefabs/saltrock.lua#L53-L55)）。
- **边界：满耐久拒绝修复 ≠ 撒盐**：`finiteuses` 满时 `repairable` 分支返回 false、动作不扣盐晶，残耐久不足 80 时也只补到上限（[repairable.lua L135-L142](../../../raw/current/scripts/components/repairable.lua#L135-L142)）；撒盐走 `APPLYPRESERVATIVE`，满鲜目标仍会被扣件、且写入的百分比不回滚，两条路径不能互相套用。
- **边界：作祟只弹飞**：`MakeHauntableLaunchAndSmash` 的 smash 段被注释，盐晶不会因作祟碎裂（[standardcomponents.lua L1084-L1101](../../../raw/current/scripts/standardcomponents.lua#L1084-L1101)）。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/saltrock.lua` 全文（81 行，含模块级缓存函数与取数分支）；`APPLYPRESERVATIVE` 动作与客户端门禁；`saltlick_improved` repairable／finiteuses 与 480 上限；`finiteuses` 的 repairable 开关；`saltstack` 分档掉落与生长；`meatrack2` 盐槽增删；`salty_dog` 计数与抖盐时间轴；`bait`／`trap`／`birdspawner`／`molebrain` 诱饵链；`saltbox` 的 preserver；recipes 14 条；`chinese_s.po` 名称键。未核验：漂流瓶宝藏触发链（仅记表值）、`SALTSTACK_GROWTH_FREQUENCY` 的实际分钟数、咸狗作为宠物/事件的获取条件、`driedsalticon.collects` 具体由哪些晾干物设置。
- R2 修订（2026-11-10，主代理复审 S-1／S-2、终报 1–3 条）：edible 默认 health 10 改正（进食回 10 生命）；贝壳编织者改 9 条并写明 987／988 的量；珍珠晾肉架改为「入槽记盐 → 取走才出盐」；`freshen_salt` 归 repairer 侧标签；`APPLYPRESERVATIVE` 的 `smallcreature` 限定为客户端门禁。
- 外部检视修订（2026-11-10，R11–R13）：进食者改为「食性允许 ≠ 行为入口」并补尘蛾 `dustmothfood`／鼹鼠 `molebait` 门禁；撒盐改为「夹取前百分点 + 按叠数除 + 夹到 100% + 满鲜仍扣件 + 失败不回滚」，与舔盐块 REPAIR 分开；盐柱补「每次 OnWork 重排计时／船体压住跳过／世界设置与正倍率门禁／跨档结算」；鸟诱饵段收窄到外层门禁（eater、`ignorebait`）＋可食诱饵分支（`CanEat`、`bait`、未持有、危险度、落点可通行）＋陷阱分支（`isset`、未 `issprung`、`targettag` 可空、`BIRD_TRAP_CHANCE`）。
- 定向复验修订（2026-11-10，R11／R13）：R11 改准为「盐晶有 `bait`／`molebait`（会被鼹鼠选作偷饵）、缺 `dustmothfood`（尘蛾取食与交易都不选）」；R13 补外层门禁同时罩住两条分支、陷阱分支允许 `targettag` 为空或匹配；二轮复验再补：陷阱分支的 `IsDangerNearby` 为必要条件（去掉「只看」）、可食诱饵分支的通行检查允许「鸟带 `floater`」时跳过。
- 结构检查：validate_page 通过（0 issue，standard 档内）；validate_source_links 通过（113 处引用、0 error／0 warning）；未加 --check-indexes。

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|原版 Prefab 索引]]。
