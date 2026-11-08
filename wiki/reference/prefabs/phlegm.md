---
title: "脓鼻涕（phlegm）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "砂砾兽（spat）专属掉落物：GENERIC 食物，饥饿 12.5、理智 −15、生命 0，40 叠、金值默认为 0、没有鲜度也不可燃；全脚本只有砂砾兽的随机掉落表会产出它，另一个去向是冬宴烤箱的南瓜派配方。"
aliases: ["phlegm", "脓鼻涕", "Phlegm", "砂砾兽掉落", "冬宴南瓜派材料"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/phlegm.lua", "prefabs/spat.lua", "brains/spatbrain.lua", "prefabs/wintersfeastoven.lua", "recipes.lua", "cooking.lua", "components/edible.lua", "components/eater.lua", "components/tradable.lua", "components/stackable.lua", "componentactions.lua", "standardcomponents.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 脓鼻涕（phlegm）

## 快速判断

- 职责与适用：单一注册 `phlegm`，物品态食物，原版里是砂砾兽（spat）身上掉的战利品——吃下去净亏：回 12.5 饥饿、扣 15 理智、只回 0 生命。简中名 **脓鼻涕**（`NAMES.PHLEGM`，英文同名 "Phlegm"）。[注册](../../../raw/current/scripts/prefabs/phlegm.lua#L46-L46)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L346840-L346842)
- 优先使用：数值看[数值](#数值)；只想确认"从哪来、能干嘛"看[生成与用途](#生成与用途)；改可交易性看[关联机制导航](#关联机制导航)的 `tradable` 行。
- 重要副作用：理智是负值，`edible` 会在赋值时自动给它加 `badfood` 标签——非 PVP 服里因此喂不了其他玩家：`FEEDPLAYER` 的强胃分支同时要求物品带 `monstermeat`，而它是 GENERIC、没有这个标签；它**没有** `perishable`，也没有 `burnable`，所以既不会腐烂也点不着。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[构造](../../../raw/current/scripts/prefabs/phlegm.lua#L27-L43)
- 修改边界：本页只负责 `phlegm` 自身；砂砾兽本体、它的"鼻涕炸弹"武器与 `spat_bomb` 投射物归 `prefabs/spat.lua`（Wiki 待建），冬宴烤箱归 `prefabs/wintersfeastoven.lua`。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `phlegm` | `fn()`：`CreateEntity` → bank／build `phlegm`、播 `idle` → `MakeInventoryFloatable(inst, "med", nil, 0.75)` → `SetPristine`；**非主模拟在此返回** | 物品态：`inspectable`、`inventoryitem`、`stackable`、`edible`、`tradable`，可作祟；没有鲜度、可燃、诱饵或烤制组件。[工厂](../../../raw/current/scripts/prefabs/phlegm.lua#L6-L44)、[主模拟分界](../../../raw/current/scripts/prefabs/phlegm.lua#L21-L27) |

注意它**没有** `MakeSmallBurnable`／`MakeSmallPropagator`（对比同批的饼干屑与沃比零食都带了这两行），所以它不是可燃物；作祟只有 `MakeHauntableLaunch`，没有附带的点燃或粉碎分支。[构造](../../../raw/current/scripts/prefabs/phlegm.lua#L27-L43)、[Launch helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

## 数值

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒（本页没有以天计的计时，列出以备换算）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 值 | 定义处／结算处 |
|---|---|---|
| 饥饿 | `CALORIES_SMALL`＝12.5 | [赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L35-L35)、[常量](../../../raw/current/scripts/tuning.lua#L2059-L2059)、[结算](../../../raw/current/scripts/components/eater.lua#L224-L224) |
| 生命 | **显式写 0** | [赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L34-L34)、[结算](../../../raw/current/scripts/components/eater.lua#L213-L213) |
| 理智 | `-SANITY_MED`＝−15 | [赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L36-L36)、[常量](../../../raw/current/scripts/tuning.lua#L2035-L2035)、[结算](../../../raw/current/scripts/components/eater.lua#L233-L235) |
| 食物类型 | `FOODTYPE.GENERIC`，同步加 `edible_GENERIC` 标签 | [赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L37-L37)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | [赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 交易值 | **未赋值** → `tradable` 构造默认 0（不能卖金块） | [挂件](../../../raw/current/scripts/prefabs/phlegm.lua#L39-L39)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L3) |
| 鲜度 | 无 `perishable` 组件，永不腐坏 | [构造段](../../../raw/current/scripts/prefabs/phlegm.lua#L27-L43) |

采集册记 `hungervalue=12.5`、`healthvalue=0`、`sanityvalue=-15`、`stacksize=40`、`foodtype="GENERIC"`，与源码逐项一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1109-L1109)

- **负理智会自己挂上 `badfood`**：`edible` 的赋值回调在 `healthvalue < 0` 或 `sanityvalue < 0` 时 `AddTag("badfood")`，所以这块东西一构造完就带这个标签，而 prefab 里没有写它。[回调](../../../raw/current/scripts/components/edible.lua#L1-L7)、[赋值](../../../raw/current/scripts/prefabs/phlegm.lua#L34-L36)
- **负值只在这道门之后结算**：`eater` 的生命与理智分支都以 `值 >= 0 or DoFoodEffects(food)` 为条件；`phlegm` 既没有 `monstermeat` 也没有 `rawmeat` 标签，普通食客的 `DoFoodEffects` 返回真，所以对普通食客就是实打实的 −15（只有食物亲和一类例外才会跳过）。[门槛](../../../raw/current/scripts/components/eater.lua#L166-L170)、[理智分支](../../../raw/current/scripts/components/eater.lua#L233-L235)

## 生成与用途

| 环节 | 内容 | 证据 |
|---|---|---|
| 生成（唯一来源） | 砂砾兽的共享掉落表 `'spat'`：`{'phlegm', 1.00}` 一条、`{'phlegm', 0.50}` 一条；砂砾兽 `lootdropper:SetChanceLootTable('spat')` 后在死亡时按此表产出 | [掉落表](../../../raw/current/scripts/prefabs/spat.lua#L35-L46)、[接线](../../../raw/current/scripts/prefabs/spat.lua#L251-L251) |
| 依赖声明 | `prefabs/spat.lua` 的 `local prefabs` 里列了 `phlegm`（L15），那是预加载声明 | [声明](../../../raw/current/scripts/prefabs/spat.lua#L13-L16) |
| 检索范围（否定命题） | 本页在 `scripts/` 下以 `phlegm` 全文检索（2026-11-09，build 26474873）：命中只有清单／采集册／皮肤表、`spat.lua` 的依赖表与掉落表、`recipes.lua` 的冬宴配方，以及 `spatbrain` 的一处注释——**没有**任何 `SpawnPrefab("phlegm")`，也没有其它掉落表引用它 | [依赖表](../../../raw/current/scripts/prefabs/spat.lua#L13-L16)、[掉落表](../../../raw/current/scripts/prefabs/spat.lua#L35-L46) |
| 去向：冬宴烤箱 | `Recipe2("wintercooking_pumpkinpie", {Ingredient("wintersfeastfuel", 1), Ingredient("ash", 1), Ingredient("phlegm", 1)}, TECH.WINTERSFEASTCOOKING_ONE, {nounlock = true, manufactured = true, actionstr = "COOK", …})`：1 份脓鼻涕＋1 份冬宴燃料＋1 份灰；烤箱把 `wintercooking_<dish>` 映射回真正的菜 | [配方](../../../raw/current/scripts/recipes.lua#L1442-L1442)、[映射](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L57-L63)、[上菜](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L234-L237) |
| 不作锅料 | 进锅资格由 `IsCookingIngredient(prefabname)` 判定：它只查 `ingredients[aliases[prefabname] or prefabname]`，必须先被 `AddIngredientValues` 注册；本页在该文件检索不到 `phlegm` 的注册行（检索范围：`scripts/cooking.lua`，2026-11-09），所以它不是锅食材 | [判定](../../../raw/current/scripts/cooking.lua#L185-L187) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 写 GENERIC 与 12.5／0／−15；负理智自动加 `badfood` | 仅主模拟 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/phlegm.lua#L33-L37) |
| 组件：`eater`（消费端） | 按 `edible_GENERIC` 匹配食性，三维分别结算 | 任意进食者 | [[reference/components/eater]]；[匹配](../../../raw/current/scripts/components/eater.lua#L288-L302)、[结算](../../../raw/current/scripts/components/eater.lua#L213-L234) |
| 组件：`stackable` | `maxsize`＝40 | 两端由 Replica 同步 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/phlegm.lua#L30-L31) |
| 组件：`tradable` | 挂上但没写 `goldvalue`，保持默认 0 | 主模拟 | [[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/phlegm.lua#L39-L39) |
| 组件：`hauntable` | `MakeHauntableLaunch` → 作祟时把物品抛飞 | 主模拟 | [[reference/components/hauntable]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 关联 Prefab：`spat` | 唯一产出者；死亡按 `'spat'` 掉落表掷 1.00／0.50 两条 | 服务端；由 `lootdropper` 决定 | `prefabs/spat.lua`（Wiki 待建）；[接线](../../../raw/current/scripts/prefabs/spat.lua#L251-L251) |
| 关联 Prefab：`spat_bomb`／Snotbomb | **不是** `phlegm`：砂砾兽手上那件"鼻涕炸弹"是 `spat.lua` 里 `CreateEntity()` 造的同名武器（`persists = false`，投射物 `spat_bomb`），行为树里 `EquipPhlegm` 装的也是它 | 服务端非联网实体 | [Snotbomb](../../../raw/current/scripts/prefabs/spat.lua#L90-L108)、[行为树](../../../raw/current/scripts/brains/spatbrain.lua#L57-L86) |
| 资源：`phlegm.zip` | bank／build 都是 `phlegm`，动画 `idle`；另外 `spat_phlegm.zip` 是砂砾兽自己的素材 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/phlegm.lua#L1-L4)、[砂砾兽素材](../../../raw/current/scripts/prefabs/spat.lua#L4-L8) |

## 常见陷阱与限制

- **误区：砂砾兽吐出来、玩家捡到的就是 `phlegm`。** 它的远程武器是 `spat.lua` 里现造的 Snotbomb 实体（`weapon`＋`equippable`＋`nosteal`，投射物 `spat_bomb`），与本 prefab 无关；`phlegm` 只从死亡掉落表出。[Snotbomb](../../../raw/current/scripts/prefabs/spat.lua#L90-L108)、[掉落表](../../../raw/current/scripts/prefabs/spat.lua#L35-L46)
- **误区：它会腐烂／能烧掉。** 构造段既没有 `perishable` 也没有 `MakeSmallBurnable`，两件事都不成立。[构造](../../../raw/current/scripts/prefabs/phlegm.lua#L27-L43)
- **边界：`tradable` 挂了不等于值钱。** 没写 `goldvalue` 时保持组件默认 0，和猪王换不到东西。[组件默认](../../../raw/current/scripts/components/tradable.lua#L1-L3)
- **边界：吃它是净亏。** 12.5 饥饿换 −15 理智，生命回复只有 0；`badfood` 还会影响玩家之间的喂食候选。[数值](../../../raw/current/scripts/prefabs/phlegm.lua#L33-L37)、[喂食判定](../../../raw/current/scripts/componentactions.lua#L1164-L1171)
- **未核验**：未运行游戏；砂砾兽掉落表的实际命中分布、冬宴烤箱的解锁与上菜时序均未实测。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/phlegm.lua` 全文（46 行）；核对 `components/edible.lua` 的负值回调与标签同步、`components/eater.lua` 的三维结算与 `TestFood`、`components/tradable.lua` 默认金值、`componentactions.lua` 的喂食候选、`standardcomponents.lua` 的 `MakeHauntableLaunch`；来源侧核对 `prefabs/spat.lua` 的 `prefabs` 声明、`SetSharedLootTable('spat')` 与 `SetChanceLootTable` 接线、以及 Snotbomb 的构造；`brains/spatbrain.lua` 的 `CanPhlegmNow`／`EquipPhlegm`；去向核对 `recipes.lua` 的 `wintercooking_pumpkinpie` 与 `prefabs/wintersfeastoven.lua` 的 `wintercooking_` 映射；`tuning.lua` 常量与 `scrapbookdata.lua` 交叉一致。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.PHLEGM`＝脓鼻涕。
- 未覆盖：砂砾兽的战斗与 `pinnable` 联动、冬宴烤箱的配方解锁条件、以及 `phlegm` 是否被模组或未检索到的脚本另作他用。
- 游戏运行：未运行；数值均为源码推导。
- 修订：2026-11-09，初检 P1（`FEEDPLAYER` 的强胃分支要 `monstermeat`，对 GENERIC 的 phlegm 不成立）、P2（否定命题改引 `IsCookingIngredient` 与检索范围；理智结算锚点改 L233-L235）、P3（负理智的结算门槛）——已按源代码更正。
- 结构检查：见交付回执。

## 相关链接

- 可食与结算：[[reference/components/edible]]、[[reference/components/eater]]、[[reference/components/stackable]]
- 交易与作祟：[[reference/components/tradable]]、[[reference/components/hauntable]]
- 同批小件：[[reference/prefabs/dustmeringue]]、[[reference/prefabs/woby-treat]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
