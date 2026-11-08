---
title: 矿物原料与幸运金块（flint、nitre、rocks、marble、goldnugget、lucky_goldnugget）
created: "2026-11-08"
updated: "2026-11-10"
type: prefab
summary: 玩家物品栏形态的五种矿物原料与活动货币幸运金块：注册名与文件名的关系（物品态 vs 世界态）、挖矿／活动产出路径、堆叠／可食／燃料／进贡实际接线与 lucky_goldnugget 的专属消费链。
aliases: ["flint", "nitre", "rocks", "marble", "goldnugget", "lucky_goldnugget", "燧石", "硝石", "石头", "大理石", "金块", "幸运金块", "矿物原料", "inv_rocks", "inv_marble"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/flint.lua", "prefabs/nitre.lua", "prefabs/inv_rocks.lua", "prefabs/inv_marble.lua", "prefabs/goldnugget.lua", "prefabs/rocks.lua", "prefabs/marbletree.lua", "prefabs/pigking.lua", "prefabs/antlion.lua", "prefabs/knight.lua", "prefabs/chessjunk.lua", "prefabs/kitcoonden.lua", "prefabs/boatrace_start.lua", "prefabs/oceanfishdef.lua", "prefabs/mermking.lua", "prefabs/bundle.lua", "prefabs/worm.lua", "prefabs/worm_boss.lua", "prefabs/shadowthrall_mouth.lua", "brains/molebrain.lua", "components/yotb_stager.lua", "components/yotc_raceprizemanager.lua", "components/quaker.lua", "recipes.lua", "tuning.lua", "constants.lua", "components/stackable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 矿物原料与幸运金块（flint、nitre、rocks、marble、goldnugget、lucky_goldnugget）

## 快速判断

- 职责与适用：玩家物品栏形态的矿物原料——`flint`（燧石）、`nitre`（硝石）、`rocks`（石头）、`marble`（大理石）——与两种金块：常驻 `goldnugget`（金块）、YOT 活动季专属 `lucky_goldnugget`（幸运金块，与 goldnugget 同文件注册）。可借鉴：矿物物品的标准接线与"挖掘／活动产出→配方／进贡消费"链路。[注册行](../../../raw/current/scripts/prefabs/goldnugget.lua#L167-L168)
- 优先使用：查产出见[获取途径](#获取途径与入栏路径)；调参数查[数值表](#数值与参数)；lucky_goldnugget 的专属消费链见[关联机制导航](#关联机制导航)；原料精炼成 cutstone 见 [[reference/prefabs/basic-refined-materials]]。
- 重要副作用：flint／nitre／rocks／marble／goldnugget 带 `quakedebris` 标签（地震落石可砸毁）；nitre 是 `FUELTYPE.CHEMICAL` 燃料（灭火器、冷火等）；lucky_goldnugget 监听 `knockbackdropped` 做弹飞交互延迟。[quakedebris 消费](../../../raw/current/scripts/components/quaker.lua#L195-L230)、[knockback 监听](../../../raw/current/scripts/prefabs/goldnugget.lua#L162)
- 修改边界：本页只覆盖玩家物品形态的注册；世界态巨砾与大理石实体（rock1／rock2／marbletree 等）是别的注册名，只写到产出关系为止；活动行为（YOT 各季）只核验与两金块直接相关的接线。

## 注册与构造

六个注册中前五个各自成文件，`goldnugget` 与 `lucky_goldnugget` 同在 goldnugget.lua 末尾注册；骨架与前两族一致（公共端实体＋physics＋bank/build＋pickupsound＋标签 → `SetPristine` → 主模拟挂组件），但矿物族公共端多挂 `SoundEmitter`，两金块另挂 bloom 光照。

**注册名与文件名（易混，必读）**：物品态注册名是 `rocks` 与 `marble`；`inv_rocks.lua`／`inv_marble.lua` 只是文件名，不存在注册名 `inv_rocks`／`inv_marble`。世界态实体另用其他注册名：巨砾族 `rock1`／`rock2`／`rock_flintless(_med/_low)`／`rock_moon(_shell)`／`moonglass_rock`／`rock_petrified_tree(_med/_tall/_short/_old)` 全部注册在 prefabs/rocks.lua，大理石树注册名 `marbletree`。写代码引用 SpawnPrefab 时用注册名，不要用文件名。[inv_rocks 注册](../../../raw/current/scripts/prefabs/inv_rocks.lua#L82)、[inv_marble 注册](../../../raw/current/scripts/prefabs/inv_marble.lua#L48)、[rocks.lua 全部注册](../../../raw/current/scripts/prefabs/rocks.lua#L524-L536)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `flint` | 单文件单注册 `Prefab("flint", fn, assets)`；主模拟挂 edible、tradable、stackable、inspectable、inventoryitem（sinks）、snowmandecor、hauntable、bait | 标签 `molebait`／`renewable`／`quakedebris`；无 fuel；文件内留有注释掉的 sparkle 代码（`shine` L6-L13，未接线）。 [组件段](../../../raw/current/scripts/prefabs/flint.lua#L42-L63) |
| `nitre` | 同骨架；多挂 fuel（`fueltype=FUELTYPE.CHEMICAL`）与 secondary `FOODTYPE.NITRE` | 标签无 `renewable`；fuel 值 `MED_LARGE_FUEL`=90 秒；可作灭火器等 CHEMICAL 燃料。 [fuel 段](../../../raw/current/scripts/prefabs/nitre.lua#L41-L43)、[edible](../../../raw/current/scripts/prefabs/nitre.lua#L31-L34) |
| `rocks` | 注册名 `rocks`（文件 inv_rocks.lua）；主模拟挂 edible、tradable（`rocktribute=1`）、stackable、inspectable、inventoryitem（sinks）、bait、repairer、snowmandecor、hauntable；OnSave／OnLoad 保存 anim 变体；quagmire 模式 postinit 钩子 | 公共端随机取动画名 `f1`/`f2`/`f3`（L44-L45）并 PlayAnimation，存档经 `data.anim` 恢复（L8-L17）；标签多 `rocks`；修补 STONE `50/3`。 [变体保存](../../../raw/current/scripts/prefabs/inv_rocks.lua#L8-L17)、[组件段](../../../raw/current/scripts/prefabs/inv_rocks.lua#L47-L79) |
| `marble` | 注册名 `marble`（文件 inv_marble.lua）；主模拟挂 stackable、inspectable、inventoryitem（sinks）、snowmandecor、hauntable、bait | **无** edible／tradable／fuel；标签 `molebait`／`quakedebris`；固定 `PlayAnimation("anim")` 无变体。 [组件段](../../../raw/current/scripts/prefabs/inv_marble.lua#L31-L45) |
| `goldnugget` | 与 lucky 同文件：`Prefab("goldnugget", fn, assets)`；主模拟挂 edible、tradable（goldvalue 未设）、inspectable、inventoryitem（sinks）、stackable（无 maxsize）、bait、snowmandecor、hauntable（launch+smash）；构造即 `shine()` 并挂 `OnEntityWake` 周期闪光 | 标签 `molebait`／`quakedebris`；非 YOTP 活动期加 `minigameitem` 标签（L45-L47）；bloom `shaders/anim.ksh`。 [fn](../../../raw/current/scripts/prefabs/goldnugget.lua#L25-L76)、[shine 机制](../../../raw/current/scripts/prefabs/goldnugget.lua#L7-L23) |
| `lucky_goldnugget` | 同文件 `Prefab("lucky_goldnugget", luckyfn, assets)`（L168）；同骨架，但 `OverrideSymbol("nugget","gold_nugget","lucky_goldnugget")` 换外观、`scrapbook_overridedata` 同步图鉴 | tradable 设 `goldvalue=1`（goldnugget 未设）；**无** bait／snowmandecor；hauntable 仅 launch；YOTP 活动期才加 `minigameitem`（L130-L132，与 goldnugget 相反）；监听 `knockbackdropped`。 [luckyfn](../../../raw/current/scripts/prefabs/goldnugget.lua#L114-L165)、[OnKnockbackDropped](../../../raw/current/scripts/prefabs/goldnugget.lua#L88-L112) |

两金块的闪光是同一对 `shine`／`OnEntityWake`：进入视野 4-9 秒后播一次 `sparkle` 动画并回到 idle，睡眠时停、唤醒重排。金块未设 OnSave 恢复闪光状态，靠 `OnEntityWake` 重启。[shine](../../../raw/current/scripts/prefabs/goldnugget.lua#L7-L16)、[OnEntityWake](../../../raw/current/scripts/prefabs/goldnugget.lua#L19-L23)

### 获取途径与入栏路径

| 注册名 | 产出（已核验） |
|---|---|
| `flint`／`nitre`／`rocks` | 挖巨砾：`SetSharedLootTable('rock1')`（石头×3＋硝石＋燧石概率）、`'rock2'`（石头×3＋金块＋燧石）、`'rock_flintless*'`（纯石头）、`'rock_moon'`（石头＋燧石＋月岩 `moonrocknugget`，见 [[reference/prefabs/moonrocknugget]]）、`'rock_petrified_tree*'`（石头／硝石／燧石），OnWork 按 loot 表 DropLoot。 [loot 表](../../../raw/current/scripts/prefabs/rocks.lua#L65-L166)、[OnWork](../../../raw/current/scripts/prefabs/rocks.lua#L168-L200)、[月岩名称](../../../raw/current/scripts/languages/chinese_s.po#L345669-L345672) |
| `marble` | 挖大理石树 `marbletree`：loot 表 `'marble_tree'`（marble×1.0、×0.5）。大理石柱／雕像等其他来源本页未逐一枚举。 [loot 表](../../../raw/current/scripts/prefabs/marbletree.lua#L14-L17)、[接线](../../../raw/current/scripts/prefabs/marbletree.lua#L86) |
| `goldnugget` | 金脉巨砾 `'rock2'` loot 表；猪王 `cointoss`：按进贡物 `goldvalue` 每点 SpawnPrefab("goldnugget")；**非 YOTP（猪年）活动期**作为猪王小游戏投掷物——`MINIGAME_ITEM` 仅在 `IsSpecialEventActive(SPECIAL_EVENTS.YOTP)` 时切为 lucky_goldnugget，其他 YOT 活动（YOTB/YOTC/YOTH 等）期间仍是普通金块。 [rock2 表](../../../raw/current/scripts/prefabs/rocks.lua#L76-L85)、[cointoss](../../../raw/current/scripts/prefabs/pigking.lua#L74-L77)、[MINIGAME_ITEM 切换](../../../raw/current/scripts/prefabs/pigking.lua#L540) |
| `lucky_goldnugget` | 活动季专属：猪王 YOTP 小游戏（`MINIGAME_ITEM` 切换，[pigking.lua#L540](../../../raw/current/scripts/prefabs/pigking.lua#L540)、[投掷生成](../../../raw/current/scripts/prefabs/pigking.lua#L414-L428)）；幸运骑士掉落 redpouch_yoth 包 4 枚（[knight.lua#L290-L299](../../../raw/current/scripts/prefabs/knight.lua#L290-L299)）；棋子残骸 redpouch_yoth 包 1 枚（[chessjunk.lua#L147-L155](../../../raw/current/scripts/prefabs/chessjunk.lua#L147-L155)）；猫猫窝扔红包（[kitcoonden.lua#L119-L122](../../../raw/current/scripts/prefabs/kitcoonden.lua#L119-L122)）；赛船奖励（[boatrace_start.lua#L117-L123](../../../raw/current/scripts/prefabs/boatrace_start.lua#L117-L123)）；胡萝卜鼠赛跑奖品（YOTC=`year_of_the_carrat`，[constants.lua#L850](../../../raw/current/scripts/constants.lua#L850)；[yotc_raceprizemanager.lua#L32](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L32)）；牛王赛马奖品（[yotb_stager.lua#L995-L1009](../../../raw/current/scripts/components/yotb_stager.lua#L995-L1009)）；YOT 活动海鱼 `oceanfish_medium_6/7`：`SpecialEventSetup` 接线 loot／heavy_loot／oncooked_fn（[oceanfishdef.lua#L1089-L1094](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1089-L1094)）、烤鱼产物函数 `MEDIUM_YOT_ONCOOKED_FN` 烤出 2 枚（[oceanfishdef.lua#L149-L153](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L149-L153)）；鱼人王收活动鱼付幸运金块（[mermking.lua#L132-L137](../../../raw/current/scripts/prefabs/mermking.lua#L132-L137)）；蠕虫／影口 chance loot（[worm.lua#L261-L263](../../../raw/current/scripts/prefabs/worm.lua#L261-L263)、[shadowthrall_mouth.lua#L225-L229](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L225-L229)）、蠕虫 Boss 幸运掉落（[worm_boss.lua#L480](../../../raw/current/scripts/prefabs/worm_boss.lua#L480)）。基础红包 `redpouch` 内含 1 枚（[bundle.lua#L684](../../../raw/current/scripts/prefabs/bundle.lua#L684)）。 |

拾取与入栏链同前两族（PICKUP→GiveItem，见 [[reference/prefabs/basic-gathered-materials#获取途径与入栏路径]]）；六注册均 `SetSinks(true)`（落水下沉走打捞路径）。[flint sinks](../../../raw/current/scripts/prefabs/flint.lua#L52-L53)、[lucky sinks](../../../raw/current/scripts/prefabs/goldnugget.lua#L152-L153)

### 数值与参数

tuning 基准：`STACK_SIZE_SMALLITEM`=40、`STACK_SIZE_MEDITEM`=20（默认档）、燃料 [L1674-L1683](../../../raw/current/scripts/tuning.lua#L1674-L1683)。

| 注册名 | 堆叠上限 | 可食（foodtype／饥饿） | 燃料值（秒）／类型 | sinks | 修补 |
|---|---|---|---|---|---|
| `flint` | 40 | ELEMENTAL／1 | 无 fuel | 是 | 无 |
| `nitre` | 40 | ELEMENTAL＋secondary NITRE／2 | `MED_LARGE_FUEL`=90／CHEMICAL | 是 | 无 |
| `rocks` | 40 | ELEMENTAL／1 | 无 fuel | 是 | STONE／`REPAIR_ROCKS_HEALTH`=50/3≈16.67 |
| `marble` | 40 | 无 edible | 无 fuel | 是 | 无 |
| `goldnugget` | 默认 20 | ELEMENTAL／2 | 无 fuel | 是 | 无 |
| `lucky_goldnugget` | 默认 20 | ELEMENTAL／2 | 无 fuel | 是 | 无 |

`FUELTYPE.CHEMICAL` 的实际消费者（核验过的文件）：灭火器 `firesuppressor`、冷火 `coldfire`／`coldfirepit`、Winona 低档发电机 `winona_battery_low`。[FUELTYPE 定义](../../../raw/current/scripts/constants.lua#L2004)、[nitre fuel](../../../raw/current/scripts/prefabs/nitre.lua#L41-L43)

名称键经简中 .po `msgctxt` 核验：`NAMES.FLINT`=燧石、`NAMES.NITRE`=硝石、`NAMES.ROCKS`=石头、`NAMES.MARBLE`=大理石、`NAMES.GOLDNUGGET`=金块、`NAMES.LUCKY_GOLDNUGGET`=幸运金块。[FLINT](../../../raw/current/scripts/languages/chinese_s.po#L342890-L342892)、[NITRE](../../../raw/current/scripts/languages/chinese_s.po#L346190-L346192)、[ROCKS](../../../raw/current/scripts/languages/chinese_s.po#L348675-L348677)、[MARBLE](../../../raw/current/scripts/languages/chinese_s.po#L345020-L345022)、[GOLDNUGGET](../../../raw/current/scripts/languages/chinese_s.po#L343380-L343382)、[LUCKY_GOLDNUGGET](../../../raw/current/scripts/languages/chinese_s.po#L344845-L344847)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：inventoryitem | 拾取入栏；六注册均 `SetSinks(true)`（落水下沉） | 主模拟 | [[reference/components/inventoryitem#关键接口]]；[flint](../../../raw/current/scripts/prefabs/flint.lua#L52-L53) |
| 组件：stackable | 上限 40（四矿物）／20（两金块走组件默认） | 主模拟 | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/components/stackable.lua#L28) |
| 组件：edible | ELEMENTAL 可食（礁石类生物／矿食生物的类别入口；数值见上表）；nitre 另设 NITRE 二类（酸蝙蝠食物） | 主模拟 | [[reference/components/edible#关键接口]]；[NITRE 注释](../../../raw/current/scripts/constants.lua#L2039) |
| 组件：fuel | nitre 是唯一带燃料的：90 秒 CHEMICAL 燃料 | 仅 nitre；主模拟 | [[reference/components/fuel#关键接口]]；[nitre](../../../raw/current/scripts/prefabs/nitre.lua#L41-L43) |
| 组件：bait | flint／nitre／rocks／marble／goldnugget 挂 bait＋`molebait` 标签（鼹鼠拾取目标）；lucky_goldnugget 不挂 | 主模拟 | [[reference/components/bait#关键接口]]；[molebait 消费（鼹鼠脑 TAKEBAIT_MUST_TAGS）](../../../raw/current/scripts/brains/molebrain.lua#L64) |
| 组件：tradable | lucky 设 `goldvalue=1`（猪王收货后按值抛金块，即幸运金→普通金的兑换口；兑换受玩家门禁：`AcceptTest` 要求 `goldvalue>0`，且 merm 角色递非 pig_token 直接拒绝）；rocks `rocktribute=1`／cutstone 族进贡蚁狮；goldnugget 挂 tradable 但未设 goldvalue（猪王不收） | 主模拟 | [[reference/components/tradable#关键接口]]；[goldvalue 抛金](../../../raw/current/scripts/prefabs/pigking.lua#L74-L77)、[AcceptTest 含 merm 门禁](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮 rocktribute](../../../raw/current/scripts/prefabs/antlion.lua#L117-L141) |
| 组件：repairer | rocks 修补 STONE 类（值 50/3） | 仅 rocks；主模拟 | [[reference/components/repairer#关键接口]]；[inv_rocks](../../../raw/current/scripts/prefabs/inv_rocks.lua#L64-L66) |
| 组件：hauntable | `MakeHauntableLaunchAndSmash`（flint／nitre／rocks／marble 四矿物＋goldnugget，共 5 注册）；lucky 仅 `MakeHauntableLaunch` | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[lucky](../../../raw/current/scripts/prefabs/goldnugget.lua#L157) |
| 组件：snowmandecor | 雪人装饰：flint／nitre／rocks／marble／goldnugget 挂；lucky 不挂 | 主模拟 | Wiki 待建；[flint](../../../raw/current/scripts/prefabs/flint.lua#L55)、[lucky 缺席](../../../raw/current/scripts/prefabs/goldnugget.lua#L144-L162) |
| API／数据：`minigameitem` 标签 | 猪王小游戏道具标记：随从猪按该标签捡道具（pigelitebrain）、玩家/猪王 SG 判定 | goldnugget＝非 YOTP 活动期、lucky_goldnugget＝YOTP（猪年）活动期；公共端打标签 | [pigelitebrain](../../../raw/current/scripts/brains/pigelitebrain.lua#L23-L28)、[goldnugget](../../../raw/current/scripts/prefabs/goldnugget.lua#L45-L47)、[lucky](../../../raw/current/scripts/prefabs/goldnugget.lua#L130-L132) |
| API／数据：`knockbackdropped` 事件 | lucky 监听弹飞事件，按 `delayinteraction`／`delayplayerinteraction` 临时加 `knockbackdelayinteraction`／`NOCLICK`，防落点秒拾秒点；推送方：猪王抛币（pigking.lua#L328）、牛王领奖（yotb_stager.lua#L986）等 | 仅 lucky；主模拟监听 | [监听与回调](../../../raw/current/scripts/prefabs/goldnugget.lua#L78-L112)、[猪王推送](../../../raw/current/scripts/prefabs/pigking.lua#L328)、[牛王推送](../../../raw/current/scripts/components/yotb_stager.lua#L986) |
| API／数据：`quakedebris` 标签 | 地震落石（quaker）把带此标签的邻近物品直接砸毁（Remove＋碎裂 FX） | flint／nitre／rocks／marble／goldnugget；lucky 不带 | [quaker 消费](../../../raw/current/scripts/components/quaker.lua#L195-L230) |
| API／数据： loot 表 `SetSharedLootTable` | 巨砾族按表掉落本页物品；`SetSinks`+`quakedebris` 是余震清场组合 | 世界态 rock1/rock2 等与物品态的关系见[注册与构造](#注册与构造) | [[reference/components/lootdropper#关键接口]]；[rocks.lua loot 表](../../../raw/current/scripts/prefabs/rocks.lua#L65-L166) |
| 关联 Prefab：rock1／rock2／rock_flintless*／rock_moon*／rock_petrified_tree* | 巨砾世界实体（挖矿来源），注册在 prefabs/rocks.lua，Wiki 待建 | 世界态；被 MINE 工作挖毁后 DropLoot | [注册](../../../raw/current/scripts/prefabs/rocks.lua#L524-L536)、[baserock_fn](../../../raw/current/scripts/prefabs/rocks.lua#L243-L313) |
| 关联 Prefab：marbletree | 大理石树（挖矿来源），Wiki 待建 | 世界态 | [marbletree loot 接线](../../../raw/current/scripts/prefabs/marbletree.lua#L86) |
| 关联 Prefab：cutstone／gunpowder 等配方产物 | rocks→cutstone（石砖×3）、nitre→火药等精炼与配方消费 | 建造端 | [[reference/prefabs/basic-refined-materials]]、[cutstone 配方](../../../raw/current/scripts/recipes.lua#L460) |
| 关联 Prefab：redpouch／redpouch_yoth | lucky 的红包载体：基础 redpouch 内含 1 枚；活动掉落红包由 knight／chessjunk WrapItems 装入 | YOT 活动季 | [bundle.lua#L684](../../../raw/current/scripts/prefabs/bundle.lua#L684)、[knight](../../../raw/current/scripts/prefabs/knight.lua#L290-L299) |
| API／数据：YOT 活动配方族 | lucky 是全部 `PERDOFFERING`／季别 OFFERING 科技配方的通用货币（floatinglantern、dragonboat_pack、kitcoonden_kit、nightcaphat 等约 60 条），部分建筑拆除按 `DeconstructRecipe` 返还 | recipes.lua L1098-L1190／L1604-L1668；活动期科技 | [配方段](../../../raw/current/scripts/recipes.lua#L1098-L1190)、[拆除返还](../../../raw/current/scripts/recipes.lua#L1641-L1649) |
| 资源：anim 包 | flint.zip／nitre.zip／rocks.zip（f1-f3 动画）／marble.zip／gold_nugget.zip（bank goldnugget，lucky 用 OverrideSymbol 换符号）；两金块 bloom `shaders/anim.ksh` | 公共端声明 | [goldnugget assets](../../../raw/current/scripts/prefabs/goldnugget.lua#L1-L4)、[lucky 外观覆盖](../../../raw/current/scripts/prefabs/goldnugget.lua#L128) |

## 典型接入

### 借鉴"同文件注册常驻／活动双形态"

- 前提与选择：服务端 `modmain.lua`＋prefab 文件；`goldnugget.lua` 把常驻版与活动版放同一文件、同一 assets，用 `OverrideSymbol` 复用动画包只换符号，是"外观变体不新建动画资源"的标准样板。活动开关通过 `IsSpecialEventActive` 在构造期决定标签差异。
- 生效与撤销：两注册独立 SpawnPrefab；活动期判定在构造时执行，活动状态变化不会翻转已生成实体的 `minigameitem` 标签。
- 原版依据：luckyfn 的符号覆盖＋`scrapbook_overridedata`，及两 fn 中相反的 YOTP 判定。[luckyfn](../../../raw/current/scripts/prefabs/goldnugget.lua#L114-L165)、[事件判定对比](../../../raw/current/scripts/prefabs/goldnugget.lua#L45-L47)

```lua
-- modmain.lua：给自己的矿物加"挖矿产出"只需在源实体的 lootdropper 配置，无需新注册；
-- 引用物品态注册名时用 rocks（不是 inv_rocks）
local function SpawnMineralSample()
    local inst = GLOBAL.SpawnPrefab("rocks")  -- 注册名 rocks，文件是 inv_rocks.lua
    return inst
end
```

- 调整与验证：注册名以 `Prefab("注册名", fn, assets)` 第一参为准；goldnugget.lua 内两注册可从该文件末尾的返回值确认。[注册返回](../../../raw/current/scripts/prefabs/goldnugget.lua#L167-L168)

## 常见陷阱与限制

- **误区：`inv_rocks`／`inv_marble` 是注册名**：它们是文件名；注册名是 `rocks`／`marble`。用 `SpawnPrefab("inv_rocks")` 会得到 nil。世界态巨砾另用 `rock1`／`rock2` 等注册名，与物品态无同名关系。[inv_rocks 注册](../../../raw/current/scripts/prefabs/inv_rocks.lua#L82)、[rocks.lua 注册](../../../raw/current/scripts/prefabs/rocks.lua#L524-L536)
- **边界：goldnugget 与 lucky_goldnugget 不是皮肤或别名，是两个注册**：外观差一个 `OverrideSymbol`；`goldvalue`（0 vs 1）、bait／snowmandecor（有 vs 无）、hauntable（smash vs launch）、`minigameitem` 打点条件（非活动 vs 活动）、`knockbackdropped` 监听（无 vs 有）各不相同，postinit 按注册名分别接线。[对比](../../../raw/current/scripts/prefabs/goldnugget.lua#L25-L76)、[L114-L165](../../../raw/current/scripts/prefabs/goldnugget.lua#L114-L162)
- **边界：两金块堆叠都是默认 20**：stackable 未设 maxsize（对比四矿物的 40），沿用 stackable 组件默认 `STACK_SIZE_MEDITEM`。[stackable 默认](../../../raw/current/scripts/components/stackable.lua#L28)
- **边界：goldnugget 挂 tradable 但猪王不收自己**：`goldvalue` 未设（0），猪王 AcceptTest 要求 `goldvalue>0`；lucky 因 `goldvalue=1` 可交给猪王，按值抛 1 枚普通金块（幸运金→普通金兑换口）。[AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605)、[抛金](../../../raw/current/scripts/prefabs/pigking.lua#L74-L77)
- **边界：quakedebris 的物品在地震落石附近会被直接移除**：不是掉落保护而是破坏标记；flint/nitre/rocks/marble/goldnugget 都带，lucky 不带。[quaker](../../../raw/current/scripts/components/quaker.lua#L195-L230)
- **边界：rocks 的三动画变体跨存档保持**：f1/f2/f3 在 OnSave 存 `data.anim`、OnLoad 恢复；生成新实体则随机。改 rocks 外观要兼容该字段。[保存恢复](../../../raw/current/scripts/prefabs/inv_rocks.lua#L8-L17)、[随机取](../../../raw/current/scripts/prefabs/inv_rocks.lua#L44-L45)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核验 flint／nitre／inv_rocks／inv_marble／goldnugget 五文件全文（六个注册名、组件接线、shine／knockback 机制、动画变体存档）、世界态产出（rocks.lua loot 表与注册、marbletree loot）、猪王 cointoss／MINIGAME_ITEM／AcceptTest、lucky 的消费者与产出（knight／chessjunk／kitcoonden／boatrace_start／yotc_raceprizemanager／yotb_stager／oceanfishdef／mermking／bundle／worm／worm_boss／shadowthrall_mouth、YOT 配方族）、quakedebris／minigameitem／knockbackdelayinteraction 标签的实际消费方、CHEMICAL 燃料消费者、tuning 实际取值、简中名称键。未覆盖：marble 除 marbletree 外的生成点、flint 的 `renewable` 标签消费链、ELEMENTAL 食用者生物清单、YOT 活动配方科技门内部实现、quagmire 模式 inv_rocks postinit 内容。
- 游戏运行：未运行。堆叠／燃料／修补数值为 tuning 源码推导，非运行测量；"约 60 条"配方为 recipes.lua L1098-L1190 区段行数统计，非精确计数承诺。
- 结构检查：validate_page.py 与 validate_source_links.py 通过。
- 修订：2026-11-08，主代理独立检视回源码复核后修订五处：①rock_moon loot 产物名更正为"月岩（`moonrocknugget`）"，官方简中名核验 chinese_s.po msgctxt `MOONROCKNUGGET`（#L345669-L345672）；②"卡位车"更正为胡萝卜鼠赛跑（YOTC=`year_of_the_carrat`，constants.lua#L850；CARRAT=胡萝卜鼠，po #L340635-L340637）；③"蠕虫宝箱"改写为蠕虫 Boss 幸运掉落（worm_boss.lua 为巨大洞穴蠕虫实体）；④oceanfishdef 两处锚点标注改准——#L1089-L1094 为 `SpecialEventSetup` 的 loot／heavy_loot／oncooked_fn 接线，#L149-L153 为烤鱼产物函数 `MEDIUM_YOT_ONCOOKED_FN` 定义；⑤hauntable 行计数改准（flint／nitre／rocks／marble 四矿物＋goldnugget 共 5 注册）；⑥sources 补记正文实际核验的 worm.lua／worm_boss.lua／shadowthrall_mouth.lua／brains/molebrain.lua。
- 外部复检修订：2026-11-08，按外部复检报告（docs/reports/w05/prefab-inventory-review-b01-20261108.md R13，主代理回源码验证成立）修订两处：①goldnugget 行"YOT 活动期外作为猪王小游戏投掷物"收窄为"非 YOTP（猪年）活动期"——`MINIGAME_ITEM` 仅在 `IsSpecialEventActive(SPECIAL_EVENTS.YOTP)` 时切为 lucky_goldnugget（pigking.lua#L540），其他 YOT 活动期仍是普通金块；②补记幸运金兑换的玩家门禁：`AcceptTest` 要求 `goldvalue>0` 且 merm 角色递非 pig_token 直接拒绝（pigking.lua#L598-L606），nav 表 AcceptTest 锚同步扩到含 merm 分支的完整函数。

返回 [[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
