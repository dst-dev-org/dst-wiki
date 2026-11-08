---
title: 骨头碎片（boneshard）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 单文件注册的可堆叠骨材：堆叠走组件默认 20、可当雪人装饰（表内 canflip）、作祟只弹飞；没有 edible／fuel／tradable／bait 接线。本页列清骨堆、骷髅、骨鱼与石化树的实掉条件，以及配方消费和"能进锅但只算 inedible 权重"的烹饪门禁。
aliases: ["boneshard", "骨头碎片", "Bone Shards", "骨片", "骨材"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/boneshard.lua", "components/stackable.lua", "components/snowmandecoratable.lua", "components/edible.lua", "standardcomponents.lua", "prefabs/houndbone.lua", "prefabs/houndmound.lua", "prefabs/shallow_grave.lua", "prefabs/skeleton.lua", "prefabs/scorched_skeleton.lua", "prefabs/dead_sea_bones.lua", "prefabs/klaus_sack.lua", "prefabs/catcoon.lua", "prefabs/tumbleweed.lua", "prefabs/tree_rock_data.lua", "prefabs/tree_rocks.lua", "prefabs/spoiledfood.lua", "cooking.lua", "preparedfoods.lua", "containers.lua", "components/cooker.lua", "components/stewer.lua", "recipes.lua", "componentactions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 骨头碎片（boneshard）

## 快速判断

- 职责与适用：单文件单注册的 `inventoryitem` 骨材——骨头碎片（`boneshard`），靠骨堆／骷髅／骨鱼等实体的掉落表获得，主要身份是配方材料。可借鉴：可堆叠材料的最小接线（组件段只有 `inspectable`／`inventoryitem`／`stackable`／`snowmandecor`／`hauntable`），以及"登记为锅料但只贡献权重、不提供食物标签"的写法。[注册](../../../raw/current/scripts/prefabs/boneshard.lua#L41)
- 优先使用：查掉落数量与概率见[获得链](#获得链)；查它能做什么见[用途与配方](#用途与配方)；确认它能不能吃／能不能进锅见[常见陷阱与限制](#常见陷阱与限制)。
- 重要副作用：没有 `edible`，所以不能吃、也不能放在篝火上烤（`cookable` 也不存在）；没有 `fuel`／`burnable`，不能当燃料、不会自燃；没有 `tradable`／`bait`，不能交易、不能进陷阱诱饵槽。
- 修改边界：本页不覆盖掉落宿主（猎犬、骷髅、石化树等）的生成与刷新机制，只列它们产出骨头碎片的掉落表条目与触发动作。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boneshard` | 单文件单注册 `Prefab("boneshard", fn, assets)`，无包装工厂。公共端：`CreateEntity` → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `bone_shards` → `PlayAnimation("idle")` → `pickupsound="rock"` → `MakeInventoryFloatable("small", 0.15, 0.8)` → `SetPristine`（非主模拟返回）。 | 主模拟只挂 5 个组件：`inspectable`、`inventoryitem`、`stackable`（不设 `maxsize`，取组件默认 20）、`snowmandecor`、`MakeHauntableLaunch`。[公共端与主模拟](../../../raw/current/scripts/prefabs/boneshard.lua#L6-L38) |

### 名称与实显

简中 `.po` 键 `NAMES.BONESHARD`＝骨头碎片（英文 `Bone Shards`，复数不体现在中文）。[BONESHARD](../../../raw/current/scripts/languages/chinese_s.po#L339830-L339832)

## 数值与参数

| 项 | 实际值 | 证据 |
|---|---|---|
| 堆叠上限 | 20（`stackable` 构造默认 `TUNING.STACK_SIZE_MEDITEM`；本文件未显式设） | [默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[tuning](../../../raw/current/scripts/tuning.lua#L81-L83) |
| 浮水 | 尺寸 `small`、垂直偏移 0.15、缩放 0.8 | [调用](../../../raw/current/scripts/prefabs/boneshard.lua#L21)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791) |
| 拾取音 | `rock` | [L19](../../../raw/current/scripts/prefabs/boneshard.lua#L19) |
| 雪人装饰 | 表内有项，`canflip=true`（可翻转）；bank／build 都是 `bone_shards` | [表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L12) |
| 作祟 | `MakeHauntableLaunch`：只弹飞，不改堆叠、不破坏 | [调用](../../../raw/current/scripts/prefabs/boneshard.lua#L36)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：stackable` | 承载 20 上限的合并栈；`SetStackSize` 会推 `stacksizechange` | 主模拟；上限未显式设，改数值要写 `maxsize` 到 `STACK_SIZE_*` | [[reference/components/stackable#关键接口]]；[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L34) |
| `组件：snowmandecor` | 允许被装饰到雪人上；具体渲染数据按注册名 hash 查 `ITEM_DATA` | 主模拟挂组件；客户端拖动时 `GetItemData(hash(prefab))` 命中才画得出 | Wiki 待建；[挂载](../../../raw/current/scripts/prefabs/boneshard.lua#L34)、[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L12)、[取表](../../../raw/current/scripts/components/snowmandecoratable.lua#L147-L149) |
| `组件：hauntable` | `MakeHauntableLaunch`（无参：必弹飞、默认冷却与 `HAUNT_TINY`） | 主模拟；弹飞会把 `inventoryitem.is_landed` 置 false | [[reference/components/hauntable#按效果选工厂]]；[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| `API／数据：cooking.ingredients` | `AddIngredientValues({"boneshard"}, {inedible=1})` 把骨头碎片登记成锅料，只带 `inedible` 权重 1 | 模块加载期注册；`IsCookingIngredient` 据此放行进锅 | [[reference/components/edible#关键接口]]；[登记](../../../raw/current/scripts/cooking.lua#L151)、[判定](../../../raw/current/scripts/cooking.lua#L185-L187)、[锅的 itemtestfn](../../../raw/current/scripts/containers.lua#L373-L375) |
| `缺失组件（否定断言）` | 无 `edible`／`cookable`／`fuel`／`burnable`／`tradable`／`bait`／`repairer` | 由 `prefabs/boneshard.lua` 全文（41 行）确认；对照同批 ash／charcoal 有 `edible` | [全文](../../../raw/current/scripts/prefabs/boneshard.lua#L6-L38) |

## 获得链

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 猎犬骨堆（`houndbone`） | 1 个必掉 | 共享表一条 1.0；若该骨堆是第 3 型，另加 50% 犬牙 | [表](../../../raw/current/scripts/prefabs/houndbone.lua#L15-L18)、[第 3 型](../../../raw/current/scripts/prefabs/houndbone.lua#L20-L29) |
| 猎犬巢（`houndmound`） | 2 个必掉 | 共享表：犬牙 3 条＋骨头碎片 2 条＋宝石各 1% | [表](../../../raw/current/scripts/prefabs/houndmound.lua#L19-L28) |
| 浅坟（`shallow_grave`，玩家骷髅表） | 2 个必掉 | `SetChanceLootTable('shallow_grave_player')` | [表](../../../raw/current/scripts/prefabs/shallow_grave.lua#L12-L16) |
| 骷髅（`skeleton`） | 2 个必掉＋10% 图鉴页 | 自然骷髅表；`skeleton_notplayer`／`_1`／`_2` 三个变体共用 `common_fn` 设的同一张 `skeleton` 表 | [表](../../../raw/current/scripts/prefabs/skeleton.lua#L13-L18)、[统一设置](../../../raw/current/scripts/prefabs/skeleton.lua#L159)、[变体注册](../../../raw/current/scripts/prefabs/skeleton.lua#L236-L240) |
| 玩家骷髅（`skeleton_player`） | 2 个必掉 | 与上表分开的第二张表 | [表](../../../raw/current/scripts/prefabs/skeleton.lua#L20-L24) |
| 烧焦骷髅（`scorched_skeleton`） | 1 个＋0.75 再 1 个；同时掉 1–4 个灰烬 | 锤碎时 `DropLoot` | [表](../../../raw/current/scripts/prefabs/scorched_skeleton.lua#L14-L21) |
| 海滩骨堆（`dead_sea_bones`） | 2 个＋0.5 再 1 个 | 锤碎时 `DropLoot` | [表](../../../raw/current/scripts/prefabs/dead_sea_bones.lua#L12-L17) |
| 克劳斯的袋子（`klaus_sack`） | 1 个 | 用错钥匙时 `LaunchAt(SpawnPrefab("boneshard"), …)` 弹出，不掉袋子 | [错钥匙分支](../../../raw/current/scripts/prefabs/klaus_sack.lua#L78) |
| 浣猫友善礼物（`catcoon`） | 1 个 | 友善时按 tier 抽，tier 3 含 `boneshard` | [table](../../../raw/current/scripts/prefabs/catcoon.lua#L140-L144)、[抽取](../../../raw/current/scripts/prefabs/catcoon.lua#L265-L273) |
| 滚草（`tumbleweed`） | 概率件（权重 0.1，约 0.1%，整株抽 3 次） | `MakeLoot` 的权重表里与其他杂物同池 | [权重](../../../raw/current/scripts/prefabs/tumbleweed.lua#L175-L183)、[抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245) |
| 石化树（`tree_rocks`） | 按区域权重，抽 5 次 | 沙漠区 `boneshard` 与 `rocks` 同为 10；月码头区 5（总 33）；洞穴区 5（总 69.5） | [权重表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L55-L58)、[抽取 5 次](../../../raw/current/scripts/prefabs/tree_rocks.lua#L126-L127) |
| 腐烂的小鱼（`spoiledfood` 的 fish_small） | 与腐烂食物二选一（各权重 1，只抽 1 件） | 锤碎尸体时 `DropLoot` | [掉落设置](../../../raw/current/scripts/prefabs/spoiledfood.lua#L247-L253) |
| 炼金转化（Wilson 炼金天赋，`transmute_boneshard`） | 1 个 | 消耗 2 个犬牙；`builder_skill=wilson_alchemy_10`，属制造而非掉落 | [配方](../../../raw/current/scripts/recipes.lua#L1491) |

## 用途与配方

- **配方**（`recipes.lua` 实际登记）：弹弓骨框 `slingshot_frame_bone`×2、复活怀表 `pocketwatch_revive`×4（另需钟表匠）、钓具台 `tacklestation`×1、肥料 `fertilizer`×2、雨帽 `rainhat`×1、雨衣 `raincoat`×2、眼球伞帽 `eyebrellahat`×4、鞍角 `saddlehorn`×2、沙漠地皮 `turf_desertdirt`×1、冬宴肉汁 `wintercooking_gravy`×1、炼金转化两向：`transmute_houndstooth` 消耗骨头碎片×2 产出犬牙（本页的消耗端）、`transmute_boneshard` 消耗犬牙×2 产出骨头碎片（本页的获得端）；拆解回收：墓碑×2、骷髅护甲×10、骷髅帽×10。[slingshot_frame_bone](../../../raw/current/scripts/recipes.lua#L237)、[pocketwatch_revive](../../../raw/current/scripts/recipes.lua#L365)、[tacklestation](../../../raw/current/scripts/recipes.lua#L385)、[fertilizer](../../../raw/current/scripts/recipes.lua#L444)、[拆解](../../../raw/current/scripts/recipes.lua#L1666-L1676)
- **烹饪的三层门禁**：`AddIngredientValues({"boneshard"}, {inedible=1})` 只登记 `inedible` 权重、不提供任何食物标签，所以三层要分开看。①**能进锅**：锅的原料过滤器只查 `IsCookingIngredient`，骨头碎片已登记，可放进 4 个原料格。②**能匹配配方**：`tags.inedible` 按件累加，写 `not tags.inedible` 的配方会失败并由潮湿黏糊（`wetgoop`，`priority=-10`）兜底；但容忍 inedible 的配方仍成立——四块骨头碎片满足蒸树枝（`beefalofeed`，`priority=-5`，只要求 `tags.inedible` 且无怪兽／肉／鱼／蛋／脂肪／奶／魔法），两土豆＋一树枝＋一块骨头碎片满足花式回旋块茎（`potatotornado`，要求 `names.twigs` 且 `inedible<=2`，`priority=10`）。反例是炸鱼排（`fishsticks`）：它要求 `names.twigs`，而树枝本身已计入 `inedible=1`，再放骨头碎片就超过 `inedible<=1`。③**能直接在火上烤**：不能，没有 `cookable`，`Cooker:CanCook` 直接拒绝。[登记](../../../raw/current/scripts/cooking.lua#L151)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[beefalofeed](../../../raw/current/scripts/preparedfoods.lua#L1034-L1049)、[potatotornado](../../../raw/current/scripts/preparedfoods.lua#L480-L490)、[fishsticks](../../../raw/current/scripts/preparedfoods.lua#L79-L83)、[wetgoop](../../../raw/current/scripts/preparedfoods.lua#L352-L362)
- **装饰与作祟**：可做雪人装饰（可翻转）；作祟只弹飞。[装饰表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L12)

## 典型接入

### 给自定义实体补一条骨头碎片掉落

- 前提与选择：服务端 `modmain.lua`；用 `lootdropper:AddChanceLoot` 追加机会掉落，不改原掉落表。若目标用 `SetLoot`／`SetChanceLootTable` 定义主表，追加项与主表并存（两者是不同字段）。
- 生效与撤销：`AddPrefabPostInit` 在 prefab 构造完成后执行，因此追加发生在原表设置之后；移除模组即恢复。

```lua
-- modmain.lua：让自定义生物死亡时额外 50% 掉 1 个骨头碎片
AddPrefabPostInit("my_creature", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.lootdropper ~= nil then
        inst.components.lootdropper:AddChanceLoot("boneshard", 0.5)
    end
end)
```

- 调整与验证：概率是 0–1 的小数，`DropLoot` 时逐条判定；`boneshard` 无 `perishable`，不需要额外处理腐坏。本例为源码推导，未运行游戏。

## 常见陷阱与限制

- **误区：骨头碎片"不能进锅"**——它已被 `AddIngredientValues` 登记为锅料，烹饪锅的容器过滤器 `params.cookpot.itemtestfn` 只查 `cooking.IsCookingIngredient(item.prefab)`，所以能放进 4 个原料格。它没有 `cookable` 组件，只是不能像食材那样放在篝火上直接烤（`Cooker:CanCook` 要求 `cookable`）。[登记](../../../raw/current/scripts/cooking.lua#L151)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[篝火烤制门禁](../../../raw/current/scripts/components/cooker.lua#L12-L17)
- **边界：`inedible=1` 是配方权重，不是"不可食"开关**——`GetIngredientValues` 把每件原料的标签累加；写 `not tags.inedible` 的配方放骨头碎片即失败，由 `wetgoop`（`priority=-10`）兜底。但容忍 inedible 的配方仍能成立：4 块骨头碎片（`inedible=4`）走 `beefalofeed`（−5），2 土豆＋1 树枝＋1 骨头碎片（`inedible=2`）走 `potatotornado`（10）。反例 `fishsticks` 要求 `names.twigs` 且 `inedible<=1`，树枝已占满 1，再放骨头碎片就超限。[标签累加](../../../raw/current/scripts/cooking.lua#L217-L227)、[beefalofeed](../../../raw/current/scripts/preparedfoods.lua#L1034-L1049)、[potatotornado](../../../raw/current/scripts/preparedfoods.lua#L480-L490)、[fishsticks](../../../raw/current/scripts/preparedfoods.lua#L79-L83)、[wetgoop](../../../raw/current/scripts/preparedfoods.lua#L352-L362)、[twigs 权重](../../../raw/current/scripts/cooking.lua#L134)
- **边界：堆叠 20 不是本文件写的**——`boneshard.lua` 只 `AddComponent("stackable")`，20 来自组件构造函数默认 `STACK_SIZE_MEDITEM`；要改请显式设 `maxsize`，别以为源码里写了 20。[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34)
- **边界：雪人装饰要"组件＋表项"两件套**——物品侧要有 `snowmandecor`，渲染数据还要在 `snowmandecoratable.ITEM_DATA[hash(注册名)]` 里有项；缺表项时客户端 `GetItemData` 取不到、拖到雪人上画不出来。骨头碎片两者齐备且 `canflip=true`。[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L12)
- **边界：作祟只弹飞**——`MakeHauntableLaunch` 不含砸碎或点燃段；不要照 `LaunchAndSmash`／`LaunchAndIgnite` 的命名推断它会破坏或起火。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **误区：有 `bait` 才可能被鼹鼠偷／进陷阱**——本注册没有 `bait` 也没有 `molebait`，两条都不成立；需要诱饵时用 ash／charcoal／gears 等带 `bait` 的物品。[全文](../../../raw/current/scripts/prefabs/boneshard.lua#L6-L38)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：`prefabs/boneshard.lua` 全文与 `stackable` 默认上限、浮水参数、`snowmandecor` 表项（含 `canflip`）、`MakeHauntableLaunch`；全部获得链宿主（`houndbone`／`houndmound`／`shallow_grave`／`skeleton`／`scorched_skeleton`／`dead_sea_bones`／`klaus_sack`／`catcoon`／`tumbleweed`／`tree_rock*`／`spoiledfood`）的掉落表与触发动作；`recipes.lua` 的制造与拆解行；`cooking.lua` 权重登记与标签累加、锅的 `itemtestfn`、`Cooker:CanCook`、`wetgoop` 兜底与 `beefalofeed`／`potatotornado`／`fishsticks` 的 `inedible` 条件；简中名称键。未覆盖：各宿主的刷新与生成节奏；`houndbone` 三种外观与掉落表选择的上游；石化树区域判定（`GetLootKey`）的完整分支；运行表现。
- 游戏运行：未运行；概率、数量与烹饪结果均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 见交付回报。

相关：[[reference/prefabs/clockwork-parts]]（同批机械零件）、[[reference/prefabs/fishmeat]]（鱼肉腐坏为变质鱼）、[[reference/prefabs/pond-fish]]（池塘鱼／鳗鱼腐坏链）、[[reference/prefabs/spoiled-food]]（锤碎变质鱼取骨片）。

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|Prefab 索引]]。
