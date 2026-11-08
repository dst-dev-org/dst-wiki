---
title: "淡水鱼与活鳗鱼（pondfish、pondeel）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "池塘钓上来的两条活鱼：pondfish／pondeel 共用工厂下的标签、三维、3 天鲜度、宰杀与烤制产物（fishmeat_small、eel）、重量分布、鱼人建造配方，以及它们与旧鱼物品、海鱼链的分界。"
aliases: ["pondfish", "pondeel", "淡水鱼", "活鳗鱼", "池塘活鱼", "钓上来的活鱼"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pondfish.lua", "prefabs/pond.lua", "prefabs/oasislake.lua", "prefabs/fishingrod.lua", "prefabs/fish.lua", "prefabs/oceanfish.lua", "prefabs/merm.lua", "prefabs/mermking.lua", "prefabs/catcoon.lua", "prefabs/wurt.lua", "prefabs/pigking.lua", "components/fishable.lua", "components/fishingrod.lua", "components/perishable.lua", "components/weighable.lua", "components/lootdropper.lua", "components/inventoryitem.lua", "components/murderable.lua", "componentactions.lua", "actions.lua", "cooking.lua", "recipes.lua", "tuning.lua", "constants.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 淡水鱼与活鳗鱼（pondfish、pondeel）

## 快速判断

- 职责与适用：`prefabs/pondfish.lua` 用一个 `commonfn` 注册两件**活体物品**：地表池塘（`pond`、`pond_mos`）钓上的 `pondfish`（淡水鱼），洞穴池塘（`pond_cave`）钓上的 `pondeel`（活鳗鱼）。两件都带 `inventoryitem`，能拿在手里、能宰杀，也会自己腐坏。[注册](../../../raw/current/scripts/prefabs/pondfish.lua#L195-L196)
- 优先使用：改数值／看产物查[数值与消费端](#数值与消费端)；判断"钓上来的鱼算不算鱼"查[获得](#获得)与[与旧鱼、海鱼的分界](#与旧鱼海鱼的分界)。
- 重要副作用：宰杀与腐坏都**绕过**旧鱼物品 `fish`——`pondfish` 两条结局都是 `fishmeat_small`／`fishmeat_small_cooked`，只有 `pondeel` 才产出 `eel`；两件都**没有** `stackable`（一条一格）；`StartPerishing` 在构造时就调用，放在地上也走表。[数据](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L190)、[计时](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L114)
- 修改边界：只写这两件活体；`fishmeat*` 与 `eel` 的数值分别归 [[reference/prefabs/fishmeat]]、[[reference/prefabs/pond-fish]]，竿与抛收线归 [[reference/prefabs/fishing-tools]]，海鱼 `oceanfish_*` 归本批 `ocean-fish` 页（S8）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pondfish` | `pondfishfn()` → `commonfn("fish", "fish", "fish01", pondfish_data)`：bank／build 都是 `fish`，`char_anim_build` 存 `fish01` | 数据表：重量 40.89–55.28、`perish_product` 与 `loot` 都是 `fishmeat_small`、`cookable_product = fishmeat_small_cooked`、`HEALING_TINY`／`CALORIES_SMALL`、`GOLD_VALUES.MEAT`、影子 `SHADOW_MEDIUM`。[数据](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L176) |
| `pondeel` | `pondeelfn()` → `commonfn("eel", "eel", "eel01", pondeel_data)` | 重量 165.16–212.12、`perish_product` 与 `loot` 是 `eel`、`cookable_product = eel_cooked`、`HEALING_SMALL`／`CALORIES_TINY`、`GOLD_VALUES.RAREMEAT`、影子 `SHADOW_LONG`。[数据](../../../raw/current/scripts/prefabs/pondfish.lua#L178-L193) |

`commonfn` 的顺序（共用段 → `SetPristine` → 主模拟）：`AddTransform`／`AddAnimState`／按 `dynamic_shadow` 加影子／`AddNetwork` → `MakeInventoryPhysics` → bank／build／播 `idle` → 标签 → `SetPristine` → `inst.build = char_anim_build`（供 SGwilson 换手持图样，值由钓竿的 `fishingcatch` 事件带出）→ 主模拟挂件。[构造](../../../raw/current/scripts/prefabs/pondfish.lua#L67-L156)、[build 赋值](../../../raw/current/scripts/prefabs/pondfish.lua#L106)

标签（pristine，两端生效）：`fish`、`pondfish`、`meat`、`catfood`、`smallcreature`；数据表给了 `weight_min`／`weight_max` 时再加 `weighable_fish`——这条标签来自 `weighable` 的 `type` setter，提前写进 pristine 只为省一次客户端同步。[标签](../../../raw/current/scripts/prefabs/pondfish.lua#L89-L98)、[type setter](../../../raw/current/scripts/components/weighable.lua#L1-L8)（S3）

主模拟挂件顺序：`bait` → `perishable`（`SetPerishTime`＋`StartPerishing`＋`onperishreplacement`＋`ignorewentness`）→ `cookable`（只写 `product`）→ `inspectable` → `inventoryitem`（`SetOnDroppedFn`／`SetOnPutInInventoryFn`／`SetSinks(true)`）→ `murderable` → `lootdropper`（`SetLoot`）→ `edible` → `MakeHauntableLaunchAndPerish` → `tradable` → 有重量时 `weighable` → `on_loot_dropped` 监听 → 启动 `flop_task`。[挂件](../../../raw/current/scripts/prefabs/pondfish.lua#L108-L153)

- **没有 `stackable`**：两件都是"一条一格"，与 [[reference/prefabs/pond-fish]] 的 40 叠不同；宰杀时叠数因此恒为 1。[挂件段](../../../raw/current/scripts/prefabs/pondfish.lua#L108-L149)、[MURDER 取叠数](../../../raw/current/scripts/actions.lua#L3179-L3189)
- **拍水动画**：`ondropped` 先播一次 `idle`、排 0–3 秒后的 `flop_task`；`flop` 先检查 `inventoryitem.canbepickedup`，为假时直接取消任务（避免在容器或特殊位置里乱跳），否则随机推 1–2 次 `idle` 再排 2–6 秒（`math.random()*2 + num*2`，`num` 取 1–2）；`onpickup` 取消任务。[拍水](../../../raw/current/scripts/prefabs/pondfish.lua#L28-L65)
- **入水下沉**：`inventoryitem:SetSinks(true)`，落水不留漂浮体（两件都没有 `floater` 组件）。[下沉](../../../raw/current/scripts/prefabs/pondfish.lua#L124)

### 名称核验

`NAMES.PONDFISH`＝**淡水鱼**（Freshwater Fish），`NAMES.PONDEEL`＝**活鳗鱼**（Live Eel），键名与注册名同名、各命中一次。[PONDFISH](../../../raw/current/scripts/languages/chinese_s.po#L347154-L347157)、[PONDEEL](../../../raw/current/scripts/languages/chinese_s.po#L347149-L347152)（S7）采集册把 `pondfish` 记成 `type="food"`、`perishable=1440`、`hungervalue=12.5`、`healthvalue=1`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1156)

## 获得

两条链都是"池塘挂 `fishable`、竿挂 `fishingrod`"，与海钓的 `oceanfishingrod`／`oceanfishable` 无关（S2）。竿侧细节归 [[reference/prefabs/fishing-tools]]，这里只写谁挂什么。

| 宿主 | 挂的鱼 | 证据 |
|---|---|---|
| `pond`（青蛙池塘）、`pond_mos`（蚊子池塘） | `fishable:AddFish("pondfish")` | [青蛙池塘](../../../raw/current/scripts/prefabs/pond.lua#L363)、[蚊子池塘](../../../raw/current/scripts/prefabs/pond.lua#L329) |
| `pond_cave`（洞穴池塘） | `fishable:AddFish("pondeel")` | [洞穴池塘](../../../raw/current/scripts/prefabs/pond.lua#L480) |
| `oasislake`（绿洲湖） | `SetGetFishFn(GetFish)`：先按 `OASISLAKE_WETPOUCH_CHANCE`＝0.6 掷幸运，命中出 `wetpouch`，否则出 `pondfish` | [取鱼函数](../../../raw/current/scripts/prefabs/oasislake.lua#L195-L197)、[接线](../../../raw/current/scripts/prefabs/oasislake.lua#L246-L249) |

- 池塘的 `fishable` 用组件默认值：`maxfish = 10`、`fishleft = 10`，`SetRespawnTime(TUNING.FISH_RESPAWN_TIME)`＝`day_time/3`＝100 秒补 1 条；绿洲湖另设 `maxfish = OASISLAKE_MAX_FISH`＝15 与自己的刷新时间。[组件默认](../../../raw/current/scripts/components/fishable.lua#L9-L18)、[池塘设置](../../../raw/current/scripts/prefabs/pond.lua#L264-L265)、[常量](../../../raw/current/scripts/tuning.lua#L1141)
- 中鱼流程：`ACTIONS.FISH` 要求目标带 `fishable` 标签、竿上还没有已钓起的鱼，且该目标不是当前已抛竿的目标（是的话给的是 `REEL`）→ `StartFishing` 记 target／fisherman；`WaitForFish` 按池塘剩余鱼量线性插值咬钩等待（`SetWaitTimes(4, 40)`）→ 咬钩后 `ACTIONS.REEL` 走 `Hook` → `HookFish` **真的 Spawn** 一条活鱼、`AddChild` 挂到池塘下、`entity:Hide()`、`persists = false`、停物理，并把 `weighable` 的 owner 记成钓手 → 再 `REEL` 走 `Reel`：`RemoveFish` 解除父子、把鱼挪到钓手面前、抛 `fishingcatch{build}` → 最后 `Collect` 让鱼显示、恢复物理与 `persists = true`，竿消耗 1 点耐久。[候选](../../../raw/current/scripts/componentactions.lua#L2339-L2347)、[HookFish](../../../raw/current/scripts/components/fishable.lua#L58-L80)、[Reel](../../../raw/current/scripts/components/fishingrod.lua#L166-L185)、[Collect](../../../raw/current/scripts/components/fishingrod.lua#L187-L201)、[耐久](../../../raw/current/scripts/prefabs/fishingrod.lua#L32-L36)
- 其它来源：鱼人（`merm`）的死亡掉落表里有 `pondfish`；守卫变体用另一张 `merm_guard_loot`，同样固定含 `pondfish`；天然鱼人屋（`mermhouse`，漏雨的小屋）的固定 `loot` 是 `boards`／`rocks`／`pondfish`，锤掉会出鱼；**出活鱼还是熟鱼块取决于烧到哪一步**：还在燃烧、尚未烧毁时锤，`onhammered` 先 `burnable:Extinguish()` 再 `DropLoot()`，火已被自己熄掉，所以照普通表出**活 `pondfish`**；已经烧毁（带 `burnt` 标签）再锤，`lootdropper` 才用 `BURNED_LOOT_OVERRIDES` 把它换成 `fishmeat_small_cooked`——这是**掉落物被火改写**，不是鱼自己烤。鱼人屋这条链的数值见 [[reference/prefabs/fishmeat#获得与消费]]。[锤击](../../../raw/current/scripts/prefabs/mermhouse.lua#L99-L104)、[鱼人表](../../../raw/current/scripts/prefabs/merm.lua#L69-L73)、[守卫表](../../../raw/current/scripts/prefabs/merm.lua#L75-L79)、[守卫装载](../../../raw/current/scripts/prefabs/merm.lua#L1325)、[小屋战利品](../../../raw/current/scripts/prefabs/mermhouse.lua#L21-L26)、[战利品装载](../../../raw/current/scripts/prefabs/mermhouse.lua#L330)、[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)

## 数值与消费端

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度天数一律 ×480 秒（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 重量 | 交易值 | 宰杀／腐坏产物 | 烤制产物 |
|---|---|---|---|---|---|---|
| `pondfish` | 12.5／1／0 | `PERISH_SUPERFAST`＝3 天＝1440 秒 | 40.89–55.28 | 1（`GOLD_VALUES.MEAT`） | `fishmeat_small`／`fishmeat_small` | `fishmeat_small_cooked` |
| `pondeel` | 9.375／3／0 | 3 天＝1440 秒 | 165.16–212.12 | 5（`GOLD_VALUES.RAREMEAT`） | `eel`／`eel` | `eel_cooked` |

数值来源：[常量](../../../raw/current/scripts/tuning.lua#L2021-L2022)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2059)、[鲜度](../../../raw/current/scripts/tuning.lua#L2045)、[交易值](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[数据表](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L190)。理智是源码写死的 0（`edible.sanityvalue = 0`），不是组件默认值巧合。[三维](../../../raw/current/scripts/prefabs/pondfish.lua#L131-L136)

- **鲜度（S9）**：两件都是 `PERISH_SUPERFAST`＝3 天，`StartPerishing` **在构造时**就调用，没有 `MakeSmallPerishableCreature` 的"进容器才走表"逻辑：放在地上按 `PERISH_GROUND_MULT`＝1.5 倍速腐坏，`ignorewentness = true` 让潮湿的 ×1.3 不生效。[计时](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L114)、[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97)、[潮湿倍率](../../../raw/current/scripts/components/perishable.lua#L107-L109)
- **腐坏产物不是 `spoiled_fish`**：`onperishreplacement` 分别是 `fishmeat_small` 与 `eel`，到点直接替换成这两个 prefab。[腐坏替换](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L113)、[依赖表](../../../raw/current/scripts/prefabs/pondfish.lua#L16-L21)
- **宰杀**：`murderable` 让物品获得 `MURDER` 候选（`health` 分支在 `CanMurder` 时也会给）；`ACTIONS.MURDER.fn` 先 `RemoveFromOwner`，再按叠数逐次 `GenerateLoot` 把产物塞进执行者背包，装不下才掉在原地。[候选](../../../raw/current/scripts/componentactions.lua#L2820-L2822)、[动作](../../../raw/current/scripts/actions.lua#L3166-L3197)
- **烤制**：`cookable.product` 直接指向熟鱼排／熟鳗鱼——活鱼上锅**跳过** `fish`／`eel` 这一环。两件都带 `smallcreature` 标签，而 `Cook()` 的鲜度折算整段被 `not inst:HasTag("smallcreature")` 挡掉，所以**产物按满鲜生成、不继承剩余鲜度**；这与旧鱼 `fish`／`eel` 烤制时折半（`1-(1-p)*0.5`）的规则不同，和 `oceanfish_*` 的 `_inv` 同口径。[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L116-L117)、[折算门禁](../../../raw/current/scripts/components/cookable.lua#L29-L34)
- **炖锅食材值**：`pondeel` 记 `meat = 0.5`、`fish = 1` 且**可** `cancook`；`pondfish` 记 `meat = 0.5`、`fish = 0.5` 且**不** `cancook`（第三个参数为 `false`，不派生 `_cooked` 食材条目）。[登记](../../../raw/current/scripts/cooking.lua#L114-L115)、[派生](../../../raw/current/scripts/cooking.lua#L46-L70)
- **交易**：`pondfish` 是 1、`pondeel` 是 5，都过猪王 `goldvalue > 0` 的门槛；但 `AcceptTest` 先拒带 `merm` 标签的交付者，沃特交付不换金。[赋值](../../../raw/current/scripts/prefabs/pondfish.lua#L140-L141)、[接收判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **鱼人**：给鱼人喂带 `fish` 标签的物品会触发欢呼；招募判定也认 `fish` 标签；当着鱼人的面**宰杀**带 `fish` 标签的东西会让跟随的鱼人停止跟随并"不满"（背包里腐坏那种 `negligent` 情形不算）。[欢呼](../../../raw/current/scripts/prefabs/merm.lua#L342-L344)、[招募](../../../raw/current/scripts/prefabs/merm.lua#L421-L424)、[宰杀](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L982-L998) 鱼人王同样按 `fish` 标签接收，吃鱼时走另一条分支而不是咀嚼。[接收](../../../raw/current/scripts/prefabs/mermking.lua#L204-L211)、[吃](../../../raw/current/scripts/prefabs/mermking.lua#L237-L239)
- **其它 `fish` 消费端**：拖网（`ocean_trawler`）放下时让 `fish` 物品按 `OCEAN_TRAWLER_LOWERED_PERISH_RATE` 腐坏；`unagi` 的配方测试直接认 `pondeel`。[拖网保鲜](../../../raw/current/scripts/prefabs/ocean_trawler.lua#L90-L96)、[unagi](../../../raw/current/scripts/preparedfoods.lua#L339-L341)
- **浣猫**：`catcoon` 只收带 `cattoy`／`catfood`／`cattoyairborne` 的物品；两件活鱼都有 `catfood`，因此能喂给猫，猫再按好友礼物表回礼——tier 5 里就有 `pondfish`。[判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[礼物表](../../../raw/current/scripts/prefabs/catcoon.lua#L159-L169)
- **沃特保鲜**：`fish` 标签让鱼在沃特身上按 `WURT_FISH_PRESERVER_RATE`＝1/4 速率腐坏。[速率函数](../../../raw/current/scripts/prefabs/wurt.lua#L370-L372)
- **鱼人建造配方**（`builder_tag = "merm_builder"`，即沃特）：`mermhouse_crafted` 吃 2 条 `pondfish`，`mermhat` 吃 1 条；拆 `mermhead` 返 1 条。[鱼人屋](../../../raw/current/scripts/recipes.lua#L107)、[鱼人帽](../../../raw/current/scripts/recipes.lua#L111)、[拆解](../../../raw/current/scripts/recipes.lua#L1601)

### 重量与归属

- 分布函数 `CalcNewSize()`：`p = 2*math.random()-1`，返回 `(p³+1)/2`，再 `Lerp(weight_min, weight_max, …)`。取值在 0–1 上**向中段集中**：多数个体接近区间中点、两端稀少；这与龙虾的 `r³` 偏小、海鱼的线性分布都不同，三套不能互套。（S5）[分布](../../../raw/current/scripts/prefabs/pondfish.lua#L23-L26)、[取重](../../../raw/current/scripts/prefabs/pondfish.lua#L144-L149)、[存重取两位](../../../raw/current/scripts/components/weighable.lua#L58-L60)
- 钓上钩时 `HookFish` 就把 `weighable:SetPlayerAsOwner(fisherman)` 记成钓手；作为**掉落物**生成时，`on_loot_dropped` 再把 `prefab_override_owner` 记成掉落者 prefab，供奖杯与展示判定。[Setter](../../../raw/current/scripts/components/weighable.lua#L62-L72)、[监听](../../../raw/current/scripts/prefabs/pondfish.lua#L54-L58)、[事件源](../../../raw/current/scripts/components/lootdropper.lua#L401)

## 与旧鱼、海鱼的分界

- **和 `fish`／`fish_cooked` 不是一回事**：那两件旧鱼物品在快照里没有活体来源，且都不带 `fish` 标签（它们带 `pondfish`）；本页两件也不掉 `fish`。[旧鱼标签](../../../raw/current/scripts/prefabs/fish.lua#L35-L37)、[旧鱼生品](../../../raw/current/scripts/prefabs/fish.lua#L113-L137)
- **`eel` 的活体来源只有 `pondeel`**：宰杀与腐坏两条都指向 `eel`，数值、干制与交易链见 [[reference/prefabs/pond-fish]]。[数据](../../../raw/current/scripts/prefabs/pondfish.lua#L178-L190)
- **海鱼是另一条链**：`oceanfish_*` 走 `oceanfishingrod`＋`oceanfishable`，上钩、挣扎与出水投射都是另一套；本批海鱼页只覆盖 `oceanfish_*`，两套不能混写（S2、S4）。[海鱼标签](../../../raw/current/scripts/prefabs/oceanfish.lua#L449-L457)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`inventoryitem` | 可拾取、可入包；`SetSinks(true)`；掉落／入包回调驱动拍水 | 主模拟挂件；`canbepickedup` 未改（始终可捡） | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L121-L124) |
| 组件：`perishable` | 3 天、构造即走表；替换成 `fishmeat_small`／`eel` | 两件相同；地面 ×1.5、忽略潮湿 ×1.3 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L114) |
| 组件：`cookable` | 只设 `product`：活鱼上锅直接出熟鱼排／熟鳗鱼 | 仅主模拟；门禁见组件页 | [[reference/components/cookable]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L116-L117) |
| 组件：`edible` | `ismeat = true`、`FOODTYPE.MEAT`、三项数值 | 仅主模拟；理智写死 0 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L131-L136) |
| 组件：`murderable` | 加 `murderable` 标签，给出 `MURDER` 候选 | 主模拟；动作还需 `inventoryitem` | [[reference/components/murderable]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L126) |
| 组件：`lootdropper` | `SetLoot` 固定表（1 件）；`on_loot_dropped` 回报 dropper | 火中掉落会被覆盖表改写 | [[reference/components/lootdropper]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L128-L129) |
| 组件：`weighable` | `type = TROPHYSCALE_TYPES.FISH`、`Initialize`＋`SetWeight`；`prefab_override_owner` | 仅当数据表给了重量区间 | [[reference/components/weighable]]；[接线](../../../raw/current/scripts/prefabs/pondfish.lua#L144-L149) |
| 组件：`bait`／`tradable` | 可当陷阱饵；金值 1／5 | 主模拟 | [[reference/components/bait]]、[[reference/components/tradable]]；[bait 挂载](../../../raw/current/scripts/prefabs/pondfish.lua#L108)、[tradable](../../../raw/current/scripts/prefabs/pondfish.lua#L140-L141) |
| 组件：`fishable`（来源侧） | 池塘持有鱼表与刷新预算；`HookFish` 生成并隐藏活鱼 | 主模拟；`maxfish` 默认 10 | [[reference/components/fishable]]；[组件](../../../raw/current/scripts/components/fishable.lua#L9-L18) |
| 组件：`fishingrod`（来源侧） | 咬钩等待、拉钩断竿、收线交付 | 主模拟；`SetWaitTimes(4,40)`、`SetStrainTimes(0,5)` | [[reference/components/fishingrod]]；[设置](../../../raw/current/scripts/prefabs/fishingrod.lua#L73-L75) |
| 标签：`fish`／`pondfish` | `fish` 决定沃特保鲜、鱼人／鱼人王判定、杀鱼不满与拖网保鲜；`pondfish` 在快照内没有 `HasTag("pondfish")` 消费者 | 两端（pristine） | [标签](../../../raw/current/scripts/prefabs/pondfish.lua#L89-L93) |
| 关联 Prefab：`fishmeat_small`／`fishmeat_small_cooked`／`eel` | 宰杀／腐坏／烤制产物 | 分别由 `lootdropper`、`perishable`、`cookable` 生成 | [[reference/prefabs/fishmeat]]、[[reference/prefabs/pond-fish]] |
| 资源：`anim/fish.zip`／`anim/eel.zip` | 两套 bank／build；`char_anim_build` 另取 `fish01`／`eel01` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/pondfish.lua#L1-L7) |

## 常见陷阱与限制

- **误区：池塘鱼腐坏成 `spoiled_fish`。** `pondfish` 的 `onperishreplacement` 是 `fishmeat_small`、`pondeel` 的是 `eel`；`spoiled_fish` 只出现在没被使用的 `pondfish_prefabs` 依赖表里。[替换](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L113)、[未用表](../../../raw/current/scripts/prefabs/pondfish.lua#L9-L14)
- **边界：两件都不能叠。** 没有 `stackable`，一条一格；宰杀按叠数循环，这里恒为 1 次。[挂件段](../../../raw/current/scripts/prefabs/pondfish.lua#L108-L153)
- **边界：`pondfish` 进锅只算半条鱼。** `fish = 0.5` 且 `cancook = false`；`pondeel` 才是 `fish = 1` 且能派生熟食材。[登记](../../../raw/current/scripts/cooking.lua#L114-L115)
- **边界：活鱼在地面也会腐坏。** 构造即 `StartPerishing`，与蝴蝶等"进容器才计时"的小生物相反；地面还有 ×1.5 倍率。[计时](../../../raw/current/scripts/prefabs/pondfish.lua#L110-L114)、[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97)
- **边界：两个 `Prefab()` 共用同一份依赖表。** 文件末尾两行都传 `pondeel_prefabs`，`pondfish_prefabs`（含 `spoiled_fish`）声明后未被使用——源码瑕疵，别照抄成"pondfish 依赖 spoiled_fish"。[导出](../../../raw/current/scripts/prefabs/pondfish.lua#L195-L196)
- **边界：`on_loot_dropped` 不覆盖宰杀路径。** 只有走 `LootDropper:DropLoot`／`SpawnLootPrefab` 的掉落才推这个事件；`ACTIONS.MURDER.fn` 直接 `GenerateLoot`，不会写 `prefab_override_owner`。[事件源](../../../raw/current/scripts/components/lootdropper.lua#L401)、[宰杀](../../../raw/current/scripts/actions.lua#L3185-L3196)
- **未核验**：未运行游戏；绿洲湖 `TryLuckRoll` 的幸运修正、鱼群刷新在存档读入后的余量、模组改写 `fishable` 表时的表现都未实测。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/pondfish.lua` 全文（两个数据表、`commonfn` 全部挂件、拍水任务、`on_loot_dropped`）；核对 `prefabs/pond.lua` 三处 `AddFish` 与 `fishable` 设置、`prefabs/oasislake.lua` 的 `GetFish` 与 `maxfish`；`components/fishable.lua` 的 `HookFish`／`RemoveFish`／`RefreshFish`、`components/fishingrod.lua` 的 `StartFishing`／`Hook`／`Reel`／`Collect`、`prefabs/fishingrod.lua` 的等待／断竿／耐久；`componentactions.lua` 的 FISH／REEL／MURDER 候选与 `actions.lua` 的 `MURDER.fn`；`cooking.lua` 的登记与派生、`recipes.lua` 三条配方；`merm`／`mermking`／`catcoon`／`wurt`／`pigking` 的 `fish`／`catfood` 消费；`tuning.lua` 的鲜度、热量、金值、`FISH_RESPAWN_TIME`、`BURNED_LOOT_OVERRIDES`；`weighable`／`perishable`／`lootdropper` 相关分支；`chinese_s.po` 名称与 `scrapbookdata.lua` 记录。
  检索范围：按名字扫全 `scripts`，`pondfish` 43 处命中（配方、鱼人／鱼人王／鱼人屋／浣猫的依赖表、`prefablist`、`skins_defs_data`、采集册、`pond`／`oasislake` 的鱼表），`pondeel` 14 处（`pondfish.lua`、`pond.lua`、`cooking.lua`、`preparedfoods.lua`、`prefablist` 与采集册）；`fish` 标签的消费者另按 `HasTag("fish")` 回查，不止鱼人一系。未覆盖：模组注入、调试命令、`preparedfoods` 的完整配方表。
  名称核验：`NAMES.PONDFISH`＝淡水鱼、`NAMES.PONDEEL`＝活鳗鱼。
- 游戏运行：未运行。天数、秒数、重量区间与概率均为源码推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 旧鱼与鱼肉：[[reference/prefabs/pond-fish]]、[[reference/prefabs/fishmeat]]
- 竿与池塘：[[reference/prefabs/fishing-tools]]、[[reference/components/fishingrod]]、[[reference/components/fishable]]
- 物品组件：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/cookable]]、[[reference/components/weighable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
