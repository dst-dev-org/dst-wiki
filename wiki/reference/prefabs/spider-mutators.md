---
title: "蜘蛛变身涂鸦（mutator_*，7 注册对照）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "韦伯技能体系一次循环工厂注册的 7 件蜘蛛变异物品（战士／悬蛛／洞穴／喷吐／破碎／护士／海黾变身涂鸦）：同一段 MakeMutatorFn 产出，挂 spidermutator 组件指向 spider_<目标> prefab；给蜘蛛喂食或对蜘蛛使用即变身，配方名义上要 SPIDERCRAFT_ONE 科技，实际靠驯服对应蜘蛛解锁蓝图。"
aliases: ["mutator_warrior", "mutator_dropper", "mutator_hider", "mutator_spitter", "mutator_moon", "mutator_healer", "mutator_water", "变身涂鸦", "Switcherdoodle", "蜘蛛变异剂", "蜘蛛变异"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/spider_mutators.lua", "components/spidermutator.lua", "components/stackable.lua", "prefabs/spider.lua", "stategraphs/SGspider.lua", "stategraphs/SGspider_water.lua", "actions.lua", "componentactions.lua", "components/builder.lua", "recipes.lua", "constants.lua", "prefabs/webber.lua", "standardcomponents.lua", "tuning.lua", "widgets/recipepopup.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜘蛛变身涂鸦（mutator_*，7 注册对照）

## 快速判断

- 职责与适用：7 件可拾取变异物品（inventoryitem），由 `spider_mutators.lua` 一个循环工厂批量注册；每件挂 `spidermutator` 组件并把目标设为 `spider_<变体名>`，用于把蜘蛛个体替换成对应变种。[目标表 L9-L18](../../../raw/current/scripts/prefabs/spider_mutators.lua#L9-L18)、[组件接线 L67-L68](../../../raw/current/scripts/prefabs/spider_mutators.lua#L67-L68)
- 优先使用：组件语义读 [[reference/components/spidermutator]]；查"变异蜘蛛／变身涂鸦／mutator"到本页；变异烟表现归 [[reference/prefabs/spider_heal_fx]]。
- 重要副作用：同时是 `edible` 肉类食物（负血量）与 `fuel` 燃料；对蜘蛛以外的东西没有变身效果（变身接线只在蜘蛛实体上）。
- 修改边界：本页写 7 注册的共用构造、触发链与配方解锁；蜘蛛各变体自身的行为与属性页 Wiki 待建（掉落池复用见 [[reference/prefabs/silk]]）。

## 注册与构造

| 注册名／变体 | 动画 | 变异目标 | 简中名 |
|---|---|---|---|
| `mutator_warrior` | bank `spider_mutator_all`／build `spider_mutators`，anim `warrior` | `spider_warrior` | 战士变身涂鸦 |
| `mutator_dropper` | 同上，anim `dropper` | `spider_dropper` | 悬蛛变身涂鸦 |
| `mutator_hider` | 同上，anim `hider` | `spider_hider` | 洞穴变身涂鸦 |
| `mutator_spitter` | 同上，anim `spitter` | `spider_spitter` | 喷吐变身涂鸦 |
| `mutator_moon` | 同上，anim `moon` | `spider_moon` | 破碎变身涂鸦 |
| `mutator_healer` | 同上，anim `healer` | `spider_healer` | 护士变身涂鸦 |
| `mutator_water` | **bank/build 独立**：`spider_water_mutator`，anim `water` | `spider_water` | 海黾变身涂鸦 |

- 工厂：7 个 `Prefab("mutator_<target>", MakeMutatorFn(target, extra_data))` 由同一循环生成，assets／prefabs 依赖表共享（依赖含全部 7 个 `spider_<target>`）；只有 `water` 在 `mutator_extra_data` 里覆写 bank/build。[循环 L84-L100](../../../raw/current/scripts/prefabs/spider_mutators.lua#L84-L100)、[extra data L20-L26](../../../raw/current/scripts/prefabs/spider_mutators.lua#L20-L26)
- 共同初始化（[MakeMutatorFn L28-L78](../../../raw/current/scripts/prefabs/spider_mutators.lua#L28-L78)）：网络段加 `spidermutator`＋`monstermeat` 标签、`scrapbook_anim`／`scrapbook_deps`；master 段 `inspectable`／`inventoryitem`／`stackable`（默认上限 `STACK_SIZE_MEDITEM`＝20，[stackable.lua L28](../../../raw/current/scripts/components/stackable.lua#L28)、[tuning L82](../../../raw/current/scripts/tuning.lua#L82)）→ `edible`（foodtype MEAT、secondary MONSTER，health −`HEALING_SMALL`＝3、hunger `CALORIES_SMALL`＝12.5、sanity −`SANITY_SMALL`＝10，[tuning L2022](../../../raw/current/scripts/tuning.lua#L2022)、[L2034](../../../raw/current/scripts/tuning.lua#L2034)、[L2059](../../../raw/current/scripts/tuning.lua#L2059)）→ `spidermutator:SetMutationTarget("spider_"..target)` → `MakeHauntableLaunch` → `fuel`（`SMALL_FUEL`＝`seg_time×0.5`＝15 秒当量，[tuning L1674](../../../raw/current/scripts/tuning.lua#L1674)）→ `MakeSmallBurnable(SMALL_BURNTIME)`＝7.5 秒（[tuning L1681](../../../raw/current/scripts/tuning.lua#L1681)）。
- 名称键：`STRINGS.NAMES.MUTATOR_WARRIOR` 等 7 键。[po L346095-L346127](../../../raw/current/scripts/languages/chinese_s.po#L346095-L346127)
- 7 个目标 spider prefab 与 spider.lua 中 7 个 create 函数一一对应（[注册清单 L1087-L1094](../../../raw/current/scripts/prefabs/spider.lua#L1087-L1094)）；8 种蜘蛛的公共掉落池同样适用于变异得到的个体。

## 变身触发链

三条触发路径都收敛到 [[reference/components/spidermutator]] 的 `Mutate`，且都要求 `CanMutate`（目标 prefab 与当前蜘蛛 prefab 不同，[spidermutator.lua L9-L11](../../../raw/current/scripts/components/spidermutator.lua#L9-L11)）：

| 路径 | 流程 | 证据 |
|---|---|---|
| 对蜘蛛直接使用 | 持涂鸦对活蜘蛛（`spider` 标签、未死亡）出现 `MUTATE_SPIDER` 动作 → `Mutate(target, false, doer)`；同型拒绝（`"SAMETYPE"`） | [候选](../../../raw/current/scripts/componentactions.lua#L1803-L1806)、[动作定义](../../../raw/current/scripts/actions.lua#L619)、[执行 L5641-L5646](../../../raw/current/scripts/actions.lua#L5641-L5646) |
| 喂食（EAT 动作拦截） | 蜘蛛对涂鸦执行 EAT 时，两套蜘蛛 SG 的动作处理器直接判 `spidermutator` 组件并 `Mutate(inst, true)`（`skip_event=true`）后转入 `mutate` 状态 | [SGspider L5-L12](../../../raw/current/scripts/stategraphs/SGspider.lua#L5-L12)、[SGspider_water L8-L15](../../../raw/current/scripts/stategraphs/SGspider_water.lua#L8-L15) |
| 进食监听（备用路径） | 蜘蛛经 eater 吃下涂鸦时 `oneat` → `OnEat` → `Mutate(inst)`（推 `mutate` 事件）→ SG 的 `EventHandler("mutate")` 转入同一 `mutate` 状态 | [OnEat](../../../raw/current/scripts/prefabs/spider.lua#L366-L369)、[监听 L732](../../../raw/current/scripts/prefabs/spider.lua#L732)、[SG 事件](../../../raw/current/scripts/stategraphs/SGspider.lua#L106-L108) |

- 自由蜘蛛的变身结果：SG `mutate` 状态先落变异烟 `spider_mutate_fx`，0.25 秒后丢掉随身物品、在原地生成 `mutation_target` 新蜘蛛，继承领队（`follower` 领队或 `mutator_giver`）与战斗目标（[SGspider L594-L640](../../../raw/current/scripts/stategraphs/SGspider.lua#L594-L640)）；表现细节归 [[reference/prefabs/spider_heal_fx]]。
- 被收容蜘蛛（韦伯抓在身上／容器里）：不走 SG，`Mutate` 直接在同格换新蜘蛛并把给予者设为领队（[spidermutator.lua L14-L25](../../../raw/current/scripts/components/spidermutator.lua#L14-L25)）。
- 玩家食用只有三围效果（edible 结算），没有变身逻辑——变身接线只挂在蜘蛛工厂（`oneat`）与蜘蛛 SG 上。

## 获得：配方与解锁

| 配方 | 材料 | 证据 |
|---|---|---|
| `mutator_warrior` | 怪物肉 2＋蛛丝 1＋猪皮 1 | [recipes L329](../../../raw/current/scripts/recipes.lua#L329) |
| `mutator_dropper` | 怪物肉 1＋蛛丝 1＋兔人尾巴 1 | [recipes L330](../../../raw/current/scripts/recipes.lua#L330) |
| `mutator_hider` | 怪物肉 1＋蛛丝 2＋切石 2 | [recipes L331](../../../raw/current/scripts/recipes.lua#L331) |
| `mutator_spitter` | 怪物肉 1＋蛛丝 2＋硝石 4 | [recipes L332](../../../raw/current/scripts/recipes.lua#L332) |
| `mutator_moon` | 怪物肉 2＋蛛丝 3＋月璃 2 | [recipes L333](../../../raw/current/scripts/recipes.lua#L333) |
| `mutator_healer` | 怪物肉 2＋蛛丝 2＋蜂蜜 2 | [recipes L334](../../../raw/current/scripts/recipes.lua#L334) |
| `mutator_water` | 怪物肉 2＋蛛丝 2＋无花果 2 | [recipes L335](../../../raw/current/scripts/recipes.lua#L335) |

- 门禁两层：所有 7 条配方带 `builder_tag="spiderwhisperer"`（韦伯基础标签，[webber.lua L37-L38](../../../raw/current/scripts/prefabs/webber.lua#L37-L38)）与名义科技 `TECH.SPIDERCRAFT_ONE`（＝`{SPIDERCRAFT = 1}`，[constants L1326](../../../raw/current/scripts/constants.lua#L1326)）；快照内没有向玩家授予 SPIDERCRAFT 科技的原版途径（全库仅常量、配方与科技表引用它），制作菜单把该科技需求显示为"需要蜘蛛友好"（[recipepopup L388](../../../raw/current/scripts/widgets/recipepopup.lua#L388)、制作页签 `owner_tag="spiderwhisperer"`：[constants L1574](../../../raw/current/scripts/constants.lua#L1574)）。
- **实际解锁途径＝驯服对应蜘蛛**：每只蜘蛛工厂里写死 `inst.recipe = "mutator_<自己的变体>"`（战士 [L808](../../../raw/current/scripts/prefabs/spider.lua#L808)、洞穴 [L837](../../../raw/current/scripts/prefabs/spider.lua#L837)、喷吐 [L873](../../../raw/current/scripts/prefabs/spider.lua#L873)、悬蛛 [L903](../../../raw/current/scripts/prefabs/spider.lua#L903)、破碎 [L957](../../../raw/current/scripts/prefabs/spider.lua#L957)、护士 [L992](../../../raw/current/scripts/prefabs/spider.lua#L992)、海黾 [L1081](../../../raw/current/scripts/prefabs/spider.lua#L1081)）；开始牵引（驯服成功）时 `OnStartLeashing` 检查领队 `CanLearn`（builder_tag 匹配即通过）后 `UnlockRecipe` 永久解锁蓝图（[spider.lua L344-L352](../../../raw/current/scripts/prefabs/spider.lua#L344-L352)、[监听 L728](../../../raw/current/scripts/prefabs/spider.lua#L728)、[CanLearn](../../../raw/current/scripts/components/builder.lua#L954-L966)、[UnlockRecipe](../../../raw/current/scripts/components/builder.lua#L463)）。已知蓝图后 `KnowsRecipe` 直接放行，不再受科技距离限制（[builder.lua L883](../../../raw/current/scripts/components/builder.lua#L883)）。
- 即：7 种涂鸦各要**先驯服过一次对应种类的蜘蛛**；普通蜘蛛没有对应涂鸦配方。

## 常见陷阱与限制

- **边界：同型不能重复变异**：`CanMutate` 只比较 prefab 名，对已是目标变体的蜘蛛使用/喂食会被拒绝（动作层返回 `"SAMETYPE"`）。[spidermutator.lua L9-L11](../../../raw/current/scripts/components/spidermutator.lua#L9-L11)
- **边界：玩家吃不变身**：变身接线只在蜘蛛实体（`oneat` 监听）与蜘蛛 SG 的 EAT 拦截上；玩家食用走通用 `edible` 结算，只有 −3 血／+12.5 饱食／−10 理智。[L60-L65](../../../raw/current/scripts/prefabs/spider_mutators.lua#L60-L65)
- **边界：变异器也是燃料**：`fuel`（15 秒当量）＋`SMALL_BURNTIME`（7.5 秒烧毁），可喂火堆也可被点燃。[L72-L75](../../../raw/current/scripts/prefabs/spider_mutators.lua#L72-L75)
- **边界：作祟只弹飞**：`MakeHauntableLaunch` 无点燃／粉碎分支。[L70](../../../raw/current/scripts/prefabs/spider_mutators.lua#L70)、[helper L1053-L1070](../../../raw/current/scripts/standardcomponents.lua#L1053-L1070)
- **边界：`mutate` 事件有原版监听者**：`SGspider` 的 `EventHandler("mutate")` 会转入变身状态——`skip_event=true` 的 EAT 拦截路径直接进状态、不再推事件，两条路径不要混为一谈。[SGspider L106-L108](../../../raw/current/scripts/stategraphs/SGspider.lua#L106-L108)
- **边界：水蜘蛛动画独立**：只有 `mutator_water` 用独立 bank/build（`spider_water_mutator`），其余 6 件共用一套动画的不同 anim 名。[L20-L26](../../../raw/current/scripts/prefabs/spider_mutators.lua#L20-L26)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`spidermutator` | 7 件各设一个 `spider_<target>`；执行即换实体并消耗自身 | 服务端 | [[reference/components/spidermutator]]；[L67-L68](../../../raw/current/scripts/prefabs/spider_mutators.lua#L67-L68) |
| 组件：`edible` | MEAT／MONSTER 双食物类型，负血量小食 | 服务端 | [[reference/components/edible]]；[L60-L65](../../../raw/current/scripts/prefabs/spider_mutators.lua#L60-L65) |
| 组件：`fuel`／`burnable` | SMALL_FUEL 燃料＋可燃可传火 | 服务端 | [[reference/components/fuel]]；[L72-L75](../../../raw/current/scripts/prefabs/spider_mutators.lua#L72-L75) |
| 组件：`builder`（解锁） | 驯服对应蜘蛛 → `UnlockRecipe` 永久解锁 | 服务端；领队需 spiderwhisperer | [[reference/components/builder]]；[spider.lua L344-L352](../../../raw/current/scripts/prefabs/spider.lua#L344-L352) |
| SG／状态：`mutate` | 变身执行状态（烟→0.25 秒→换实体） | 服务端；SGspider／SGspider_water 各有入口 | [SGspider L594-L640](../../../raw/current/scripts/stategraphs/SGspider.lua#L594-L640) |
| 关联 Prefab：`spider_*` 7 变体 | 变异目标与涂鸦依赖；驯服即解锁来源 | 蜘蛛本体页 Wiki 待建 | [注册 L1087-L1094](../../../raw/current/scripts/prefabs/spider.lua#L1087-L1094) |
| 关联 Prefab：`spider_mutate_fx` | 变身烟表现 | 服务端生成 | [[reference/prefabs/spider_heal_fx]] |
| 资源：`anim/spider_mutators.zip`／`anim/spider_water_mutator.zip` | 6 件共用前一资源，water 独立 | assets 声明 | [L3-L7](../../../raw/current/scripts/prefabs/spider_mutators.lua#L3-L7) |

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `spider_mutators.lua` 全文（99 行）、`components/spidermutator.lua` 全文、蜘蛛 8 注册与 7 条 `inst.recipe`、`OnStartLeashing` 解锁链、`SGspider`（EAT 拦截、mutate 事件、mutate 状态）与 `SGspider_water`（EAT 拦截）、`MUTATE_SPIDER` 候选与执行、`builder` 的 `CanLearn`／`KnowsRecipe`／`UnlockRecipe`、7 条配方行、SPIDERCRAFT 科技/页签常量与 recipepopup 显示串、韦伯标签、燃烧／作祟 helper、`chinese_s.po` 7 名。
- 检索范围：SPIDERCRAFT 全库大小写不敏感检索（constants／recipes／techtree／tuning／strings／widgets 命中），未发现原版授予该科技的机制，已如实记录为"名义科技"。
- 游戏运行：未运行；驯服解锁与变身结果均为源码审阅。
- 机械校验（页面、导航、严格链接、索引一致性）待验收时运行。

相关：[[reference/components/spidermutator]]、[[reference/components/builder]]、[[reference/prefabs/spider_heal_fx]]、[[reference/prefabs/silk]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
