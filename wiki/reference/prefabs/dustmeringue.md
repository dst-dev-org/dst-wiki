---
title: "琥珀美食（dustmeringue）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "只靠尘土块（refined_dust）就能煮出的元素食物：饥饿 12.5、生命与理智未赋值（生命沿用 edible 默认 10）、永不腐坏；带 dustmothfood 可喂尘蛾尝试换蓝图（无最近玩家／附近已有／已学会都可能失败）、带 molebait 会被鼹鼠和陷阱当饵，进玩家背包时会记一次烹饪书数据（整堆并入已有堆叠则不记）。"
aliases: ["dustmeringue", "琥珀美食", "Amberosia", "尘蛾食物", "元素食物"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/dustmeringue.lua", "preparednonfoods.lua", "cooking.lua", "prefabs/dustmoth.lua", "brains/dustmothbrain.lua", "stategraphs/SGdustmoth.lua", "brains/molebrain.lua", "prefabs/cave.lua", "components/edible.lua", "components/eater.lua", "components/container.lua", "components/inventoryitem.lua", "components/stewer.lua", "components/inventory.lua", "components/tradable.lua", "components/bait.lua", "components/trap.lua", "standardcomponents.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 琥珀美食（dustmeringue）

## 快速判断

- 职责与适用：单一注册 `dustmeringue`，物品态食物，原版里是**烹饪锅系的元素食物**——只用尘土块（`refined_dust`）就能煮出来[尘土块名称](../../../raw/current/scripts/languages/chinese_s.po#L348570-L348572)，专门拿去喂尘蛾（dust moth）。简中名 **琥珀美食**（`NAMES.DUSTMERINGUE`，英文 "Amberosia"）。[注册](../../../raw/current/scripts/prefabs/dustmeringue.lua#L60-L60)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L342420-L342422)
- 优先使用：看数值与"哪些字段没赋值"看[数值](#数值)；谁煮得出、喂给谁看[来源与去向](#来源与去向)；两个标签各自的判定行看[标签与消费端](#标签与消费端)。
- 重要副作用：**进入玩家背包时**给主人推 `learncookbookstats`，替该玩家记一次烹饪书条目（拾取、取出、交易）；放箱子时 `onputininventory` 的 owner 是容器实体、被喂时是被喂生物，只有玩家监听，故都不记。它同时还被 `molebait` 标记，鼹鼠会来偷、陷阱合拢时会把它当饵清掉。[烹饪书](../../../raw/current/scripts/prefabs/dustmeringue.lua#L6-L10)、[诱饵判定](../../../raw/current/scripts/components/trap.lua#L313-L337)
- 修改边界：本页只负责这个注册；尘蛾本体与巢穴修复归 `prefabs/dustmoth.lua`／`SGDustmoth`（Wiki 待建），锅的配方匹配归源码 `cooking.lua` 与 `preparednonfoods.lua`（Wiki 待建）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dustmeringue` | `fn()`：`CreateEntity` → 动画与音效节点 → bank `cook_pot_food`／build `cook_pot_food6`、播 `idle`、`OverrideSymbol("swap_food", …)` → 标签 → `MakeInventoryFloatable` → `SetPristine`；**非主模拟在此返回** | 物品态：`edible`、`tradable`、`stackable`、`inspectable`、`inventoryitem`、`bait`，可作祟，并监听 `onputininventory`；**没有** `perishable`／`cookable`。[工厂](../../../raw/current/scripts/prefabs/dustmeringue.lua#L12-L58)、[主模拟分界](../../../raw/current/scripts/prefabs/dustmeringue.lua#L33-L39) |

两个细节：物品图复用锅具 `cook_pot_food6` 再换 `swap_food` 符号，采集册用同一组 `scrapbook_overridedata`；`MakeHauntableLaunchAndSmash` 在本 build 只剩 `Launch`，smash 分支整段被注释。[符号与采集册](../../../raw/current/scripts/prefabs/dustmeringue.lua#L22-L26)、[仅 Launch](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)

## 数值

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 值 | 定义处／结算处 |
|---|---|---|
| 饥饿 | `CALORIES_SMALL`＝12.5 | [赋值](../../../raw/current/scripts/prefabs/dustmeringue.lua#L41-L41)、[常量](../../../raw/current/scripts/tuning.lua#L2059-L2059)、[结算](../../../raw/current/scripts/components/eater.lua#L224-L224) |
| 生命 | **未赋值**，沿用 `edible` 构造默认的 10 | [赋值缺失](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L42)、[默认值](../../../raw/current/scripts/components/edible.lua#L19-L25)、[结算](../../../raw/current/scripts/components/eater.lua#L213-L213) |
| 理智 | **未赋值**，沿用默认 0 | [赋值缺失](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L42)、[默认值](../../../raw/current/scripts/components/edible.lua#L23-L23) |
| 食物类型 | `FOODTYPE.ELEMENTAL`，同步加 `edible_ELEMENTAL` 标签 | [赋值](../../../raw/current/scripts/prefabs/dustmeringue.lua#L40-L40)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | [赋值](../../../raw/current/scripts/prefabs/dustmeringue.lua#L44-L45)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 交易值 | 未赋值 → `tradable` 默认 0 | [挂件](../../../raw/current/scripts/prefabs/dustmeringue.lua#L42-L42)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L3) |
| 鲜度 | 无 `perishable` 组件，永不腐坏 | [构造段](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L55) |

采集册的 `hungervalue=12.5`、`healthvalue=10`、`sanityvalue=0` 印证"生命与理智由组件默认值补上"。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L453-L453)

## 标签与消费端

| 标签 | 消费端判定行 | 实际效果与条件 |
|---|---|---|
| `dustmothfood` | ① 尘蛾的 `ShouldAcceptItem`：`item:HasTag("dustmothfood")`；② 行为树 `EATFOOD_TAGS = { "dustmothfood" }` 与 `not target:HasTag("dustmothfood")` | 只有尘蛾把它当食物；喂食还要通过"未充能＋手上没东西"两道门。[接收](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66)、[行为树](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L78) |
| `molebait` | ① 鼹鼠行为树 `TAKEBAIT_MUST_TAGS = { "molebait" }`，按 `SEE_BAIT_DIST` 找饵并发 `STEALMOLEBAIT`；② 陷阱 `BAIT_TAGS` 含 `molebait`，合拢时若有鼹鼠目标就按这条分支清理饵 | 鼹鼠会专门来偷；陷阱的"没放饵自动取饵"分支对鼹鼠也成立。[鼹鼠](../../../raw/current/scripts/brains/molebrain.lua#L56-L84)、[陷阱](../../../raw/current/scripts/components/trap.lua#L264-L267)、[判定](../../../raw/current/scripts/components/trap.lua#L320-L336) |
| `edible_ELEMENTAL` | 尘蛾的食性：`eater:SetDiet({ FOODTYPE.ELEMENTAL }, { FOODTYPE.ELEMENTAL })` | 决定尘蛾吃得下它（食物类型标签由 `edible` 自动加）。[食性](../../../raw/current/scripts/prefabs/dustmoth.lua#L256-L257)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |

## 来源与去向

| 环节 | 内容 | 证据 |
|---|---|---|
| 配方 | `preparednonfoods.dustmeringue`：`test` 只要求 `names.refined_dust` 非空，`priority = 100`、`cooktime = 2`、`perishtime = nil`、`overridebuild = "cook_pot_food6"`；但锅必须 4 格全满才开火（`stewer:CanCook` 判 `container:IsFull`），实际是 1 份尘土块＋3 份填充物。表里另写的 `health`／`hunger`／`sanity`／`oneat_desc` 只喂烹饪书（实际生命 10） | [配方](../../../raw/current/scripts/preparednonfoods.lua#L29-L43)、[展示用途的注释示例](../../../raw/current/scripts/preparednonfoods.lua#L20-L27) |
| 锅体 | `cooking.lua` 把 `preparednonfoods` 的每一条同时注册给 `cookpot`、`portablecookpot`、`archive_cookpot`，所以三种锅都能出 | [注册](../../../raw/current/scripts/cooking.lua#L210-L215) |
| 不是掉落物 | 尘蛾的共享掉落表只有 `{'smallmeat', 1.0}`，全脚本检索也没有把 `dustmeringue` 写进任何掉落表；它只由锅产出 | [掉落表](../../../raw/current/scripts/prefabs/dustmoth.lua#L13-L16) |
| 世界生成 | `prefabs/cave.lua` 的 `local prefabs` 里列了它，那是洞穴预加载声明，不代表洞穴会生成它 | [声明](../../../raw/current/scripts/prefabs/cave.lua#L120-L126) |
| 去向：喂尘蛾 | 尘蛾用 `trader` 接收，吃到 `dustmeringue` 时置 `_charged` 与 `_giveblueprint`；回到 `idle` 会转 `sneeze`，第 36 帧才调 `TryToDropBlueprint`：无最近玩家直接返回，否则查 16 格内蓝图，按 `BLUEPRINT_LOOTS`（当前只有 `thulecitebugnet`）挑玩家没学过的，挑不到才按 `BLUEPRINT_DROP_CHANCE_REPEAT` 另掷；修完巢穴 `_charged` 复位 | [接收](../../../raw/current/scripts/prefabs/dustmoth.lua#L251-L257)、[吃下](../../../raw/current/scripts/prefabs/dustmoth.lua#L49-L56)、[蓝图](../../../raw/current/scripts/prefabs/dustmoth.lua#L117-L171)、[状态](../../../raw/current/scripts/stategraphs/SGdustmoth.lua#L67-L72)、[掉蓝图帧](../../../raw/current/scripts/stategraphs/SGdustmoth.lua#L109-L124) |
| 去向：进玩家背包 | `OnPutInInventory` 给 `owner` 推 `learncookbookstats`，事件由玩家侧 `OnLearnCookbookStats` 接到 `cookbookupdater:LearnFoodStats(product)`。**只有 owner 是玩家才有人接**；整堆并入已有堆叠走 `stackable:Put`、不调 `OnPutInInventory`，这条也不记 | [推事件](../../../raw/current/scripts/prefabs/dustmeringue.lua#L6-L10)、[容器传 owner](../../../raw/current/scripts/components/container.lua#L537-L546)、[事件参数](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360)、[监听](../../../raw/current/scripts/prefabs/player_common.lua#L689-L689)、[处理](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L915-L920) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 只写 `ELEMENTAL` 与 12.5 饥饿；生命／理智留默认 | 仅主模拟 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L42) |
| 组件：`eater`（消费端） | 三维按 `GetHealth`／`GetHunger`／`sanityvalue` 结算；食性匹配走 `edible_ELEMENTAL` | 尘蛾等元素食性生物 | [[reference/components/eater]]；[结算](../../../raw/current/scripts/components/eater.lua#L213-L234)、[匹配](../../../raw/current/scripts/components/eater.lua#L288-L302) |
| 组件：`stackable`／`tradable` | 40 叠；金值默认 0 | 主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/dustmeringue.lua#L42-L45) |
| 组件：`bait` | 组件本身不判定生物，只登记"当前占用它的陷阱"；可作陷阱饵，自身被吃／被偷／被拿起时通知陷阱 | 主模拟；`bait.trap` 由陷阱写入 | [[reference/components/bait]]、[[reference/components/trap]]；[挂载](../../../raw/current/scripts/prefabs/dustmeringue.lua#L53-L53) |
| 组件：`hauntable` | `MakeHauntableLaunchAndSmash` → 只挂 `hauntable`，作祟时抛飞（smash 分支被注释） | 主模拟 | [[reference/components/hauntable]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105) |
| 关联 Prefab：`dustmoth` | 喂食对象；喂后尝试掉蓝图（可能失败：无玩家／附近已有／已学会） | 服务端；需要先未充能且手上无物 | `prefabs/dustmoth.lua`（Wiki 待建）；[接收](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66) |
| 关联 Prefab：`refined_dust` | 锅配方唯一必需的命名食材，简中名 **尘土块** | 配方 `test` 只数它的份数，但锅还要 4 格全满 | `preparednonfoods.lua`；[配方](../../../raw/current/scripts/preparednonfoods.lua#L29-L33)、[满格](../../../raw/current/scripts/components/stewer.lua#L122-L124) |
| 资源：`cook_pot_food6.zip` | build `cook_pot_food6`＋`swap_food` 符号，物品图也复用它 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/dustmeringue.lua#L1-L4)、[换符号](../../../raw/current/scripts/prefabs/dustmeringue.lua#L25-L26) |

## 常见陷阱与限制

- **误区：吃到它回 0 血／采集册数据等于实体字段。** `fn` 没写 `healthvalue`，`edible` 默认 10；配方表的 `health = 0` 只喂烹饪书，采集册也记 10——两处数值要一起改，否则书与实际对不上。[构造](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L42)、[默认值](../../../raw/current/scripts/components/edible.lua#L21-L23)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L453-L453)、[配方表](../../../raw/current/scripts/preparednonfoods.lua#L40-L42)
- **误区：它会腐烂。** 构造段没有 `perishable`，配方表还显式写 `perishtime = nil`，所以放多久都不坏。[构造](../../../raw/current/scripts/prefabs/dustmeringue.lua#L39-L55)、[配方](../../../raw/current/scripts/preparednonfoods.lua#L34-L34)
- **边界：`MakeHauntableLaunchAndSmash` 的 "Smash" 名不副实。** helper 里粉碎分支整段被注释，本 build 只 `Launch`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)
- **边界：只有进玩家背包才记烹饪书。** 触发点是 `onputininventory` 而不是 `oneaten`：拾取时 owner 是玩家会记；放箱子时 owner 是容器、被喂时是生物，都不记；**整堆并入已有堆叠**也不记——以是否触发 `OnPutInInventory` 为准，整堆并入不触发。[推事件](../../../raw/current/scripts/prefabs/dustmeringue.lua#L55-L55)、[容器传 owner](../../../raw/current/scripts/components/container.lua#L537-L546)、[事件参数](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360)、[处理](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L915-L920)、[玩家监听](../../../raw/current/scripts/prefabs/player_common.lua#L689-L689)
- **未核验**：未运行游戏；`priority = 100` 与其他锅配方的优先级冲突、以及尘蛾"充能—修巢—掉蓝图"的完整时序未逐帧核对。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/dustmeringue.lua` 全文（60 行）；核对 `preparednonfoods` 条目与三锅注册、`edible`／`eater` 默认值与结算、`tradable` 默认金值、`bait`／`trap` 取饵、`MakeHauntableLaunchAndSmash`、`stewer:CanCook` 与 `inventory` 入包；去向核对 `dustmoth` 接收／食性／`OnEat`／蓝图、`SGdustmoth` 的 `idle`→`sneeze`、`molebrain` 偷饵、`cave.lua` 声明、`learncookbookstats`；`tuning` 与 `scrapbookdata` 一致。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.DUSTMERINGUE`＝琥珀美食。
- 未覆盖：`refined_dust` 取得途径、尘蛾巢穴修复、蓝图重复概率。
- 游戏运行：未运行；数值均为源码推导。
- 修订：2026-11-09，初检 P2（`onputininventory` 的 owner 不总是玩家）——已按源代码更正，见「快速判断」「来源与去向」「常见陷阱」。
- 修订：2026-11-09，外部复检 P2（锅需 4 格；整堆并入不记；蓝图有距离与概率）——已改。
- 修订：2026-11-09，外部复检 P2（烹饪书记录改按是否触发 `OnPutInInventory` 表述）——已改。
- 结构检查：见交付回执。

## 相关链接

- 可食与结算：[[reference/components/edible]]、[[reference/components/eater]]、[[reference/components/stackable]]
- 诱饵与作祟：[[reference/components/bait]]、[[reference/components/trap]]、[[reference/components/hauntable]]
- 同批小件：[[reference/prefabs/woby-treat]]、[[reference/prefabs/crumbs]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
