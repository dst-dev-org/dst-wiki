---
title: "治疗药膏与绷带（healingsalve、bandage 族）"
created: "2026-11-10"
updated: "2026-11-11"
type: prefab
summary: "治疗药膏、酸蚀药膏、避火膏、蜂蜜药膏与扑腾绷带五个治疗物品注册及其两个 buff：healer 链使用入口、healerbuffs 标签语义（可对 cannotheal 目标出动作但跳过回血）、治疗量与执行者倍率、配方与获取、buff 挂接与存档恢复。"
aliases: ["healingsalve", "healingsalve_acid", "healingsalve_fumarole", "healingsalve_acidbuff", "healingsalve_fumarolebuff", "bandage", "bandage_butterflywings", "治疗药膏", "黏糊糊的药膏", "避火膏", "蜂蜜药膏", "扑腾绷带", "治疗药膏与绷带", "药膏回血"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/healingsalve.lua", "prefabs/bandage.lua", "prefabs/bandage_butterflywings.lua", "prefablist.lua", "components/healer.lua", "components/debuff.lua", "components/debuffable.lua", "components/health.lua", "components/health_replica.lua", "components/acidlevel.lua", "components/temperature.lua", "components/stackable.lua", "entityscript.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "recipes.lua", "recipes_filter.lua", "prefabs/wanda.lua", "prefabs/skilltree_walter.lua", "prefabs/treasurechest.lua", "scenarios/chest_cavesupplies.lua", "map/static_layouts/chessy_3.lua", "map/static_layouts/cave_base_1.lua", "map/layouts.lua", "map/levels/forest.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 治疗药膏与绷带（healingsalve、bandage 族）

## 快速判断

- 职责与适用：五个直接回血的**库存物品**注册——治疗药膏（healingsalve）、酸蚀药膏（healingsalve_acid）、避火膏（healingsalve_fumarole）出自 `healingsalve.lua` 的同一包装工厂，蜂蜜药膏（bandage）与扑腾绷带（bandage_butterflywings）各自独立文件；同文件还注册两个非联网 buff 实体（healingsalve_acidbuff、healingsalve_fumarolebuff），本页作关联注册说明。治疗动作的组件语义归 [[reference/components/healer]]，本页管**这些物品实际怎么接入**。
- 优先使用：查单件数值与配方看[注册与构造](#注册与构造)和[获得与消耗](#获得与消耗)；想复刻「用一次挂一个 buff」的药膏看 [healer 链共用规则](#healer-链共用规则族级主章节)（本批犁地草膏页及后续同链页从该节接入）。
- 重要副作用：healingsalve_acid／fumarole 带 `healerbuffs` 标签——对**不可常规治疗**的目标（如 Wanda、墙类）也会给出治疗动作，此时跳过回血但仍挂 buff 并消耗物品；酸膏 buff 给酸雨免疫，火膏 buff 给火焰伤害 ×0 与夏季隔热。[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36)、[healingsalve.lua:84](../../../raw/current/scripts/prefabs/healingsalve.lua#L84)
- 修改边界：不展开 `healer`／`maxhealer` 组件接口语义与 health 结算（归组件页与 [[reference/components/health]]）；不展开酸雨机制本体（[[reference/components/acidlevel]]）；buff 存档只核验本 build 的挂接与恢复路径，未实测读档。

## 注册与构造

[healingsalve.lua 注册行](../../../raw/current/scripts/prefabs/healingsalve.lua#L268-L272)、[bandage](../../../raw/current/scripts/prefabs/bandage.lua#L42)、[bandage_butterflywings](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L89)；[prefablist 三个文件名登记](../../../raw/current/scripts/prefablist.lua#L94-L95)、[healingsalve](../../../raw/current/scripts/prefablist.lua#L547)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `healingsalve`（治疗药膏） | `MakeHealingSalve("healingsalve", nil, nil, nil, assets)`：bank/build `spider_gland_salve` → 主模拟挂 `inspectable`／`inventoryitem`／`stackable`(40)／`healer`(20) | 无标签、无 OnHealFn。[工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L21-L69)、[治疗量](../../../raw/current/scripts/prefabs/healingsalve.lua#L56-L57)、[工厂实参](../../../raw/current/scripts/prefabs/healingsalve.lua#L268) |
| `healingsalve_acid`（黏糊糊的药膏） | 同工厂＋`ACID_DATA`（bank/build `healingsalve_acid`）→ common `AddTag("healerbuffs")` → 主模拟 `SetOnHealFn(acid_OnHealFn)`：`AddDebuff("healingsalve_acidbuff","healingsalve_acidbuff")` | `prefabs` 数组声明 acidbuff（依赖声明，非生成）。[标签](../../../raw/current/scripts/prefabs/healingsalve.lua#L83-L85)、[OnHealFn](../../../raw/current/scripts/prefabs/healingsalve.lua#L79-L81)、[数据](../../../raw/current/scripts/prefabs/healingsalve.lua#L73-L77) |
| `healingsalve_fumarole`（避火膏） | 同工厂＋`FUMAROLE_DATA`（bank/build `healingsalve_fumarole`）→ common `AddTag("healerbuffs")` → 主模拟 `SetOnHealFn(fumarole_OnHealFn)`：挂 `healingsalve_fumarolebuff` | [标签](../../../raw/current/scripts/prefabs/healingsalve.lua#L103-L105)、[OnHealFn](../../../raw/current/scripts/prefabs/healingsalve.lua#L99-L101)、[工厂实参](../../../raw/current/scripts/prefabs/healingsalve.lua#L270) |
| `healingsalve_acidbuff` | `MakeHealingSalveBuff` 独立工厂：非联网、`Hide`、`CLASSIFIED`、`persists=false`、`debuff`(`keepondespawn=true`)；attach 时 `AddTag("acidrainimmune")` | 服务端实体，客户端生成即删。[工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L139-L204)、[实体](../../../raw/current/scripts/prefabs/healingsalve.lua#L171-L201)、[效果](../../../raw/current/scripts/prefabs/healingsalve.lua#L209-L211) |
| `healingsalve_fumarolebuff` | 同工厂；attach 时火焰伤害乘数置 0＋夏季隔热 +60（`INSULATION_SMALL`） | [效果](../../../raw/current/scripts/prefabs/healingsalve.lua#L233-L240) |
| `bandage`（蜂蜜药膏） | `fn`：bank/build `bandage` → 主模拟 `stackable`(40)／`inventoryitem`／`healer`(30, `HEALING_MEDLARGE`) | 无标签、无回调。[工厂](../../../raw/current/scripts/prefabs/bandage.lua#L6-L40) |
| `bandage_butterflywings`（扑腾绷带） | `fn`：bank/build `bandage_butterfly` → 模块级缓存翅量 → 主模拟 `healer`(8×翅量)＋`SetOnHealFn`（目标理智 +10，仅当**执行者**有 `walter_camp_firstaid`） | 治疗量随配方推导，见[常见陷阱](#常见陷阱与限制)。[治疗量](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L80-L82)、[OnHealFn](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L8-L18) |

- 两处共同点：五件都 `MakeInventoryFloatable` 并在服务端 `MakeHauntableLaunch`——本 build 该 helper 只注册「作祟时抛飞」回调，不含打碎或点燃分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- 数值常量：`HEALING_MEDSMALL/MED/MEDLARGE`＝8/20/30，`SANITY_SMALL`＝10，`STACK_SIZE_SMALLITEM`＝40。[治疗档](../../../raw/current/scripts/tuning.lua#L2022-L2030)、[理智](../../../raw/current/scripts/tuning.lua#L2034)、[堆叠](../../../raw/current/scripts/tuning.lua#L83)
- 名称核验：五个注册名均有简中 `STRINGS.NAMES` 实显——蜂蜜药膏／扑腾绷带／治疗药膏／黏糊糊的药膏／避火膏；两个 buff 实体无 NAMES 条目（玩家不可见，仅到期台词）。[蜂蜜药膏](../../../raw/current/scripts/languages/chinese_s.po#L339035-L339037)、[扑腾绷带](../../../raw/current/scripts/languages/chinese_s.po#L339040-L339042)、[治疗药膏](../../../raw/current/scripts/languages/chinese_s.po#L343745-L343747)、[酸蚀](../../../raw/current/scripts/languages/chinese_s.po#L343750-L343752)、[避火膏](../../../raw/current/scripts/languages/chinese_s.po#L343755-L343757)

## healer 链共用规则（族级主章节）

组件接口（`SetHealthAmount`／`SetCanHealFn`／`SetOnHealFn`／`Heal` 全签名与返回值）归 [[reference/components/healer#关键接口]]，本节只写**物品侧接入约定**。以下四条适用于所有挂 `healer` 的治疗物品（含本批 [[reference/prefabs/tillweedsalve]] 与后续 mosquitomermsalve、spidergland、mosquitomermsalve、spider_healer_item 等同链页）：

1. **使用入口与候选门禁。** 治疗走 `ACTIONS.HEAL`（`mount_valid`，带 Brightshade 类补偿距离）；执行时目标＝点选对象或自己，要求有 `health` 且未死亡、非幽灵，先试 `healer:Heal` 再试 `maxhealer:Heal`。[动作定义](../../../raw/current/scripts/actions.lua#L438)、[补偿距离](../../../raw/current/scripts/actions.lua#L246-L258)、[执行函数](../../../raw/current/scripts/actions.lua#L3217-L3226)。候选动作（对场景目标用物品、对背包内物品右键自用两条路径）要求目标 `replica.health:CanHeal()`（即无 `cannotheal` 标签）**或物品带 `healerbuffs`**；`CanHeal` 只查标签，不检查是否缺血。骑乘门禁只在场景目标路径：骑乘中只能对自己持有的库存物品目标出动作，背包右键自用不受此限。[场景候选](../../../raw/current/scripts/componentactions.lua#L1346-L1362)、[背包候选](../../../raw/current/scripts/componentactions.lua#L2737-L2741)、[CanHeal 语义](../../../raw/current/scripts/components/health_replica.lua#L192-L202)。`cannotheal` 来源是 health 组件 `canheal` 字段经 netvar 同步成标签；Wanda 出厂即禁疗，墙类设 `canheal=false`。[同步](../../../raw/current/scripts/components/health.lua#L48-L49)、[Wanda](../../../raw/current/scripts/prefabs/wanda.lua#L234)、[墙例](../../../raw/current/scripts/prefabs/walls.lua#L345)
2. **治疗量与执行者倍率。** `SetHealthAmount` 的点数乘**执行者** `efficientuser` 的 `ACTIONS.HEAL` 倍率（无组件按 1）；当前 build 只有沃尔特技能 `walter_camp_firstaid` 设置该倍率 ×1.5（`fasthealer`）。倍率作用于每次用药，物品照常消耗一份。[乘法](../../../raw/current/scripts/components/healer.lua#L36)、[技能接线](../../../raw/current/scripts/prefabs/skilltree_walter.lua#L409-L414)、[常量](../../../raw/current/scripts/tuning.lua#L7369)
3. **healerbuffs 型药膏（本族 acid/fumarole 的模式）。** 三件事配合：common 期（pristine 前）`AddTag("healerbuffs")`；主模拟 `SetOnHealFn` 里 `target:AddDebuff(...)`；文件尾部另注册非联网 buff 实体（`debuff` 组件三回调＋`keepondespawn`）。目标 `canheal=false` 时 `Heal` 跳过 `DoDelta` 但**照跑 onhealfn**——所以这类药膏对 Wanda／墙不回血但能挂 buff，源码注释明言设计意图是「不按时间回血」。[标签位](../../../raw/current/scripts/prefabs/healingsalve.lua#L84)、[跳过回血仍跑回调](../../../raw/current/scripts/components/healer.lua#L38-L44)、[注释](../../../raw/current/scripts/prefabs/healingsalve.lua#L208)
4. **本页例外接入点。** 同链页只需继承上面三条，在各自页写三处差异：治疗量数值、是否加 `healerbuffs`（加标签才有「对禁疗目标也出动作」）、`SetOnHealFn`／`SetCanHealFn` 里的门禁与副作用；持续回复（overtime）不走本节，参见 [[reference/prefabs/tillweedsalve]] 的 regen buff。

### buff 的挂接、时长与存档

- 挂接：`AddDebuff` 自动补 `debuffable` 组件，仅在 `DebuffsEnabled()` 且目标未死非幽灵时生成 buff 实体并 `AttachTo`；同名已有则走 `Extend`。[AddDebuff](../../../raw/current/scripts/entityscript.lua#L2140-L2154)、[注册](../../../raw/current/scripts/components/debuffable.lua#L72-L93)、[AttachTo](../../../raw/current/scripts/components/debuff.lua#L35-L41)
- 时长：两个 buff 的到期任务都用 `TUNING.HEALINGSALVE_ACIDBUFF_DURATION`（`total_day_time/2`＝240 秒），初始与 Extend 同一常量——工厂收下的 `data.duration`（fumarole 为 `HEALINGSALVE_FUMAROLEBUFF_DURATION`＝120 秒）赋给了局部变量 `duration` 但**从未被读取**，故 fumarolebuff 实际也是 240 秒。[任务](../../../raw/current/scripts/prefabs/healingsalve.lua#L165)、[未用局部变量](../../../raw/current/scripts/prefabs/healingsalve.lua#L140)、[常量](../../../raw/current/scripts/tuning.lua#L7759)、[未消费常量](../../../raw/current/scripts/tuning.lua#L9515)
- 效果：acidbuff 仅 `AddTag("acidrainimmune")`，detach 时移除并播 `ANNOUNCE_HEALINGSALVE_ACIDBUFF_DONE`；该标签在 `IsAcidSizzling` 与酸雨伤害结算处短路。fumarolebuff 给 `health.externalfiredamagemultipliers` 置 0 乘数（火焰伤害 ×0，进 `GetFireDamageScale`）＋夏季隔热修正 +60；detach 时撤回并播台词（需 `talker`）。[acid 效果](../../../raw/current/scripts/prefabs/healingsalve.lua#L209-L221)、[标签消费](../../../raw/current/scripts/entityscript.lua#L812-L814)、[伤害结算](../../../raw/current/scripts/components/acidlevel.lua#L87-L89)、[fumarole 效果](../../../raw/current/scripts/prefabs/healingsalve.lua#L233-L257)、[火伤乘数](../../../raw/current/scripts/components/health.lua#L190-L191)、[隔热](../../../raw/current/scripts/components/temperature.lua#L231-L234)、[常量](../../../raw/current/scripts/tuning.lua#L2260)
- 存档：buff 实体自身不入档（`persists=false`）；`debuffable` 把每个 debuff 实体的 `GetSaveRecord` 嵌进宿主存档，读档经 `SpawnSaveRecord` 重生成并重新 attach；本族 buff 自定义 `OnSave`／`OnLoad` 只存到期任务剩余时间（恢复时取消满时长任务再按剩余重建）。[debuffable 存取](../../../raw/current/scripts/components/debuffable.lua#L126-L137)、[L139-L151](../../../raw/current/scripts/components/debuffable.lua#L139-L150)、[buff 存取](../../../raw/current/scripts/prefabs/healingsalve.lua#L119-L137)、[载入顺序 OnPreLoad→组件 OnLoad→实体 OnLoad](../../../raw/current/scripts/entityscript.lua#L1988-L2013)

## 获得与消耗

本族五配方均在 `recipes.lua`；五件物品全部登记在合成栏「恢复」（RESTORATION）过滤分类，避火膏另入「防暑」（SUMMER）。[配方过滤](../../../raw/current/scripts/recipes_filter.lua#L1144-L1160)、[SUMMER](../../../raw/current/scripts/recipes_filter.lua#L1514-L1528)

| 物品 | 配方（材料） | 科技 | 证据 |
|---|---|---|---|
| 治疗药膏 | 灰 2＋石 1＋蜘蛛腺体 1 | SCIENCE_ONE | [配方](../../../raw/current/scripts/recipes.lua#L414) |
| 酸蚀药膏 | 治疗药膏 1＋硝石 1＋大理石 1＋鼻涕虫黏液 1 | SCIENCE_TWO | [配方](../../../raw/current/scripts/recipes.lua#L417)；源码注释逐字保留 `slurtleslime` 食材是为了「安抚 Snurtle/Slurtle 的副作用」（原文 passifying），机制归 slurtle 页，本页不展开。[NOTES](../../../raw/current/scripts/recipes.lua#L415-L416) |
| 蜂蜜药膏 | 纸 1＋蜂蜜 2 | SCIENCE_TWO | [配方](../../../raw/current/scripts/recipes.lua#L419) |
| 扑腾绷带 | 蝴蝶翅膀 3 | TECH.NONE＋`builder_skill="walter_camp_firstaid"` | [配方](../../../raw/current/scripts/recipes.lua#L249) |
| 避火膏 | 灰 2＋燧石 1＋火腺 1 | SCIENCE_TWO | [配方](../../../raw/current/scripts/recipes.lua#L1084) |

- 酸蚀药膏消耗治疗药膏本体——这是本族唯一的物品间消耗关系。全库检索七注册名无任何 `SpawnPrefab` 生成点、无掉落表登记（检索范围见[验证记录](#验证记录)）。
- 世界生成例外一：森林随机布景六选四（Chessy_1..6）中仅 Chessy_3 布局内含 1 个蜂蜜药膏。[布局内 bandage](../../../raw/current/scripts/map/static_layouts/chessy_3.lua#L120)、[布景注册](../../../raw/current/scripts/map/layouts.lua#L854)、[随机布景列表](../../../raw/current/scripts/map/levels/forest.lua#L14-L25)
- 世界生成例外二：场景脚本 `chest_cavesupplies`（洞穴补给箱，内含治疗药膏 2–6 个）挂在 CaveBase／MushBase／SinkBase 静态布景上，但这三个布景名在本 build 的地图任务／房间／等级数据中未检索到引用——原版常规世界生成不可达，仅 mod 或场景复用时可出现。[场景内容](../../../raw/current/scripts/scenarios/chest_cavesupplies.lua#L49-L50)、[布景挂接](../../../raw/current/scripts/map/static_layouts/cave_base_1.lua#L125)、[布景注册](../../../raw/current/scripts/map/layouts.lua#L724-L726)
- 检索陷阱：`treasurechest.lua` 里的 `"healingsalve"` 出现在 `terrarium_scrapbook_adddeps`（图鉴展示依赖），**不是**掉落表。[上下文](../../../raw/current/scripts/prefabs/treasurechest.lua#L578-L592)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`healer` | 五件主体：治疗量、acid/fumarole 的 OnHealFn、绷带的理智回调 | 仅主模拟；语义归组件页 | [[reference/components/healer#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/healingsalve.lua#L56-L57) |
| 组件：`stackable` | 三处各设 `STACK_SIZE_SMALLITEM`=40；用药时拆一件消耗 | 主模拟 | [[reference/components/stackable#关键接口]]；[工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L53-L54) |
| 组件：`debuff`／`debuffable` | buff 三回调、`keepondespawn`、嵌套存档 | buff 仅服务端、非联网 | [[reference/components/debuff#关键接口]]、[[reference/components/debuffable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/healingsalve.lua#L189-L193) |
| 数据：`TUNING.HEALINGSALVE_*` | buff 时长常量（注意 fumarole 值未被消费） | 全局 | [酸蚀](../../../raw/current/scripts/tuning.lua#L7759)、[避火（未用）](../../../raw/current/scripts/tuning.lua#L9515) |
| 关联 Prefab：`healingsalve_acidbuff`／`healingsalve_fumarolebuff` | 本页关联注册；`prefabs` 数组声明＋OnHealFn `AddDebuff` | acid/fumarole 用药即挂 | [声明](../../../raw/current/scripts/prefabs/healingsalve.lua#L9-L11)、[L17-L19](../../../raw/current/scripts/prefabs/healingsalve.lua#L17-L19) |
| 资源：`spider_gland_salve.zip`／`healingsalve_acid.zip`／`healingsalve_fumarole.zip`／`bandage.zip`／`bandage_butterfly.zip` | 五件动画 | 只核验声明 | [assets](../../../raw/current/scripts/prefabs/healingsalve.lua#L1-L15)、[bandage](../../../raw/current/scripts/prefabs/bandage.lua#L1-L4)、[蝴蝶](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L1-L4) |

- 配方材料侧页面（蜘蛛腺体、蝴蝶翅膀、火腺、灰、鼻涕虫黏液、蜂刺、烂食物）只作材料引用，本体与获取不在本页范围；互链以 [[reference/prefabs/mitegland]]（火腺）为已落地示例，其余待对应批页落地后由主代理回补。

## 典型接入

### 复刻「用一次挂一个 buff」的药膏

- 前提与选择：新物品文件或 postinit 挂 `healer`；若希望对禁疗目标也生效，必须三件事齐备（见[共用规则第 3 条](#healer-链共用规则族级主章节)）——只加标签不写 OnHealFn 会变成「对 Wanda 白耗药」，只写 OnHealFn 不加标签则禁疗目标根本出不来动作。
- 生效与撤销：buff 实体独立注册（非 `prefabs` 数组生成）、`debuff` 组件接三回调、到期任务与存档剩余时间按本族写法；`keepondespawn=true` 与否决定宿主消失时 buff 是否随行（`RemoveOnDespawn` 只清未标记的）。
- 原版依据：[工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L21-L69)、[buff 工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L139-L204)、[注册行](../../../raw/current/scripts/prefabs/healingsalve.lua#L268-L272)；完整造物工程归 cookbook。

## 常见陷阱与限制

- **避火膏 buff 时长改常量不生效**：`TUNING.HEALINGSALVE_FUMAROLEBUFF_DURATION`（120 秒）在本 build 未被读取，实际时长走 `HEALINGSALVE_ACIDBUFF_DURATION`（240 秒）；调优须改后一个常量或重写工厂取值。[任务](../../../raw/current/scripts/prefabs/healingsalve.lua#L165)、[未用局部变量](../../../raw/current/scripts/prefabs/healingsalve.lua#L140)
- **扑腾绷带治疗量不是写死的**：首件实例生成时从 `AllRecipes.bandage_butterflywings` 累计纯蝴蝶翅膀食材数（非纯翅膀配方回退 `DEFAULT_COST=3`），当前 3 翅 → 8×3=24；模块级缓存后配方改动不影响已缓存值。[缓存](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L70-L72)、[推导](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L27-L43)
- **理智加成只看执行者**：OnHealFn 要求**使用药膏的人**已激活 `walter_camp_firstaid` 且目标有 `sanity` 组件；非沃尔特使用（postinit 放开配方或他人代用）只回血不给理智。[回调](../../../raw/current/scripts/prefabs/bandage_butterflywings.lua#L8-L18)
- **把 treasurechest 当获得来源**：该表的 `"healingsalve"` 是图鉴依赖数组元素，不是掉落。[上下文](../../../raw/current/scripts/prefabs/treasurechest.lua#L578-L592)
- **满血用药照样消耗**：`Heal` 不比较使用前后生命，动作候选也不检查缺血；这是组件级行为，本页沿用 [[reference/components/healer#常见陷阱与限制]] 的结论与门禁建议。
- **作祟只抛飞**：五件的 `MakeHauntableLaunch` 无打碎／点燃分支，不要按 helper 名推断会碎或着火。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核对 `healingsalve.lua` 全文件（双工厂、ACID/FUMAROLE 数据、buff 实体、表尾注册行）、`bandage.lua`／`bandage_butterflywings.lua` 全文件、`healer.lua` 全文件、`debuff.lua`／`debuffable.lua` 的 AttachTo 与存取段、`health.lua` canheal 字段与火伤乘数、`health_replica.lua` CanHeal、`actions.lua` HEAL 定义与执行函数、`componentactions.lua` healer／maxhealer 四处候选、`entityscript.lua` AddDebuff／IsAcidSizzling／载入顺序段、`acidlevel.lua` 酸雨伤害段、`temperature.lua` 隔热修正、`tuning.lua` 相关常量、`recipes.lua` 五配方及 NOTES、`recipes_filter.lua` 分类、`wanda.lua`／`walls.lua` 禁疗例、`skilltree_walter.lua` 倍率接线、`standardcomponents.lua` 作祟 helper、`treasurechest.lua` scrapbook 段、场景与地图四文件（chest_cavesupplies、cave_base_1、chessy_3、forest）、`prefablist.lua` 登记。
  生成点否定断言范围：对七注册名（五物品＋两 buff）做全 `scripts` 目录 `SpawnPrefab("healingsalve…`／`bandage…`／`lifeinjector`／`tillweedsalve` 模式与全名字符串检索（排除本族文件、语言、配方、图鉴、登记表后仅余 debugcommands、scenario 情景与 Chessy_3 布局）；动态拼名未检索，结论限于字面注册名。
  名称核验：五个物品名按 `languages/chinese_s.po` 的 `STRINGS.NAMES.*` msgctxt 实显核对（见[注册与构造](#注册与构造)）；buff 注册名无 po 条目，以到期台词汇总 `ANNOUNCE_HEALINGSALVE_*BUFF_DONE`（[po:6575](../../../raw/current/scripts/languages/chinese_s.po#L6575-L6582)）佐证其非物品属性。
  未覆盖：未审校全部角色禁疗位与皮肤；未实测读档、骑乘用药与作祟表现；CaveBase 三布景不可达结论基于布景名的字面检索。
- 游戏运行：未运行。治疗量、buff 时长与隔热值均由常量与工厂推导；240 秒 fumarole 时长结论依赖「`duration` 局部变量无其他读取点」的全文件检索（`healingsalve.lua` 内 `duration` 仅 L140 一处赋值）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 待主代理验收时统一运行（见交付回报）。

## 相关链接

- 组件语义：[[reference/components/healer]]、[[reference/components/maxhealer]]、[[reference/components/health]]、[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/acidlevel]]、[[reference/components/temperature]]
- 同链物品页：[[reference/prefabs/tillweedsalve]]、[[reference/prefabs/lifeinjector]]
- 材料页（已落地）：[[reference/prefabs/mitegland]]

返回 [[reference/prefabs/index|Prefab 索引]]。
