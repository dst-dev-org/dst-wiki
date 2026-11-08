---
title: "龙虾与月光龙虾（wobster_sheller、wobster_sheller_land、wobster_moonglass、wobster_moonglass_land、wobster_sheller_dead、wobster_sheller_dead_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "六个注册的海陆两态：水体龙虾／月光龙虾的世界态与海钓捕获、出水投射落地后的陆态活体、宰杀与熟制两件物品，以及月光陆态的不可烤、掉月光玻璃等差异。"
aliases: ["wobster_sheller", "wobster_sheller_land", "wobster_moonglass", "wobster_moonglass_land", "wobster_sheller_dead", "wobster_sheller_dead_cooked", "龙虾", "月光龙虾", "死龙虾", "美味的龙虾", "抓龙虾", "龙虾窝产出"]
tags: [prefab, creature, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wobster.lua", "prefabs/wobster_den.lua", "brains/wobsterbrain.lua", "brains/wobsterlandbrain.lua", "stategraphs/SGwobster.lua", "stategraphs/SGwobsterland.lua", "components/oceanfishable.lua", "components/oceanfishingrod.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "components/weighable.lua", "components/perishable.lua", "components/cookable.lua", "components/lootdropper.lua", "components/tradable.lua", "components/halloweenmoonmutable.lua", "components/trap.lua", "standardcomponents.lua", "componentactions.lua", "actions.lua", "tuning.lua", "constants.lua", "containers.lua", "prefabs/trap.lua", "prefabs/wurt.lua", "strings.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 龙虾与月光龙虾（wobster_sheller、wobster_sheller_land、wobster_moonglass、wobster_moonglass_land、wobster_sheller_dead、wobster_sheller_dead_cooked）

## 快速判断

- 职责与适用：`prefabs/wobster.lua` 一次注册六个 prefab，分成三对——水体态 `wobster_sheller`／`wobster_moonglass`（只能海钓，不可拾取）、陆态活体 `wobster_sheller_land`／`wobster_moonglass_land`（从水里钓上来或自己跳上岸）、物品态 `wobster_sheller_dead`／`wobster_sheller_dead_cooked`。[注册](../../../raw/current/scripts/prefabs/wobster.lua#L534-L539)
- 优先使用：判断"手里这条是哪种形态"看[注册与构造](#注册与构造)；钓与上岸看[捕获与出水投射](#捕获与出水投射)；数值、鲜度、宰杀与熟制看[数值与熟制](#数值与熟制)。
- 重要副作用：陆态活体 `inventoryitem.canbepickedup` **常态为 false**，但落地进 `stunned`（或被震晕）时会在 3–7 秒窗口里置 true，**这期间玩家能徒手捡起活龙虾**；普通陆态带 `cookable`，可以直接烤成熟龙虾，月光陆态**没有** `cookable`、宰杀只掉 `moonglass`。[陆态挂件](../../../raw/current/scripts/prefabs/wobster.lua#L361-L379)、[震晕窗口](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L53-L72)、[月光陆态](../../../raw/current/scripts/prefabs/wobster.lua#L427-L433)
- 修改边界：窝与刷新预算归 [[reference/prefabs/wobster-den]]，竿与浮子归 [[reference/prefabs/fishing-tools]]、[[reference/components/oceanfishable]]（接口不展开），月光玻璃归 [[reference/prefabs/moonglass-materials]]（S8）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wobster_sheller` | `base_water_wobster("lobster_sheller", WOBSTER_FISH_DEF)`：bank `lobster_water`、build `lobster_sheller` | 水体态：游泳碰撞掩码、水下排序，无 `health`／`inventoryitem`／`lootdropper`；`fish_def` 掉 `wobster_sheller_dead`、重量 153.67–307.34。[工厂](../../../raw/current/scripts/prefabs/wobster.lua#L221-L232) |
| `wobster_moonglass` | `moonglass_water()` → 同一工厂 + `lunar_aligned` | 掉 `moonglass`、重量 112.06–224.12。[工厂](../../../raw/current/scripts/prefabs/wobster.lua#L234-L247) |
| `wobster_sheller_land` | `base_land_wobster("lobster_sheller", "wobster_sheller", WOBSTER_FISH_DEF, false, "wobster_sheller_dead_cooked")` | 有 `cookable`（`product = wobster_sheller_dead_cooked`）、死亡不淡出；事后挂 `halloweenmoonmutable`。[陆地](../../../raw/current/scripts/prefabs/wobster.lua#L414-L425) |
| `wobster_moonglass_land` | `base_land_wobster("lobster_moonglass", "wobster_moonglass", MOONGLASS_WOBSTER_FISH_DEF, true)` | 无 `cookable`、`fadeout = true`；外层补 `lunar_aligned`。[陆地](../../../raw/current/scripts/prefabs/wobster.lua#L427-L433) |
| `wobster_sheller_dead` | `lobster_dead_fn()`：bank `lobster`、build `lobster_sheller`、播 `idle_dead` | 物品态：`fishmeat`／`catfood`／`cookable` 标签、3 天、10 叠、烤成 `wobster_sheller_dead_cooked`。[工厂](../../../raw/current/scripts/prefabs/wobster.lua#L435-L486) |
| `wobster_sheller_dead_cooked` | `lobster_dead_cooked_fn()`：播 `idle_cooked` | 6 天、10 叠、无 `cookable`；不再参与任何烤制。[工厂](../../../raw/current/scripts/prefabs/wobster.lua#L488-L532) |

水体态（两端）：四向朝向、`COLLISION.CHARACTERS`、胶囊 0.5×1，标签 `ediblefish_meat`／`ignorewalkableplatforms`／`NOBLOCK`／`NOCLICK`／`notarget`／`oceanfishable`／`oceanfishable_creature`／`oceanfishinghookable`／`swimming`；月光再加 `lunar_aligned`，`scrapbook_proxy` 指向对应陆地态。[水体构造](../../../raw/current/scripts/prefabs/wobster.lua#L153-L191)

陆态活体（两端）：物理碰撞掩码按 `TheWorld.has_ocean` 在 `GROUND`／`WORLD` 之间切；标签 `animal`／`canbetrapped`／`prey`／`smallcreature`／`whackable`／`smalloceancreature`／`stunnedbybomb`，有 `cook_product` 时加 `cookable`，恒加 `weighable_fish`；`MakeSmallPerishableCreaturePristine` 再加 `show_spoilage`；最后 `SetPrefabNameOverride(nameoverride)`。[陆态标签](../../../raw/current/scripts/prefabs/wobster.lua#L299-L349)（S3）

陆态主模拟挂件：`locomotor`（`allowocean`）→ `inspectable`（`nameoverride`）→ `inventoryitem`（`nobounce`、`canbepickedup = false`、`canbepickedupalive = true`）→ `murderable` → `lootdropper`（`SetLoot(fish_def.loot)`、`forcewortoxsouls`）→ `weighable` → 有熟品时 `cookable` → `tradable`（1）→ `sleeper` → `health`（25）→ `combat` → 可烧／可冻 → 作祟 → 监听 `on_landed`／`on_loot_dropped` → `SGwobsterland`＋`wobsterlandbrain` → `MakeSmallPerishableCreature(inst, TUNING.WOBSTER.SURVIVE_TIME)`。[挂件](../../../raw/current/scripts/prefabs/wobster.lua#L354-L411)

- **`canbepickedup` 的两态**：玩家侧 `PICKUP` 候选读 `inventoryitem:CanBePickedUp(doer)`，只看 `canbepickedup`。它常态是 false，但 `stunned` 的 onenter 置 true、onexit 置回 false，超时是 `GetRandomWithVariance(5, 2)`＝3–7 秒，所以**玩家捡起活龙虾的唯一窗口就是震晕期**（落地进 `stunned`，或吃到 `stunbomb` 事件进同一状态）；`canbepickedupalive` 则在服务端 `PICKUP.fn` 里对**非玩家**执行者放行（仆从、眼植物那一类）。[候选](../../../raw/current/scripts/componentactions.lua#L470-L480)、[可捡判定](../../../raw/current/scripts/components/inventoryitem_replica.lua#L88-L97)、[震晕状态](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L50-L78)、[震爆入口](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L36-L38)、[非玩家放行](../../../raw/current/scripts/actions.lua#L894-L902)
- **陷阱能抓**：陆态带 `canbetrapped`，`prefabs/trap.lua` 把 `targettag` 从组件默认的 `smallcreature` 改成 `canbetrapped`，触发时还要目标 `inventoryitem.trappable`（默认 true）。[标签](../../../raw/current/scripts/prefabs/wobster.lua#L325-L331)、[陷阱目标标签](../../../raw/current/scripts/prefabs/trap.lua#L79)、[可抓捕判定](../../../raw/current/scripts/components/trap.lua#L184-L188)
- **`smalloceancreature` 的消费者是鱼箱**：`containers.lua` 的 `fish_box` 只接受带该标签的物品。[鱼箱过滤](../../../raw/current/scripts/containers.lua#L1309-L1311)
- **`sleeper` 只有组件与状态**：`SGwobsterland` 装了 `AddSleepExStates`，但 `wobsterlandbrain` 里没有入睡节点，本页没找到主动入睡入口。[睡眠状态](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L185-L189)、[行为树](../../../raw/current/scripts/brains/wobsterlandbrain.lua#L64-L96)

### 名称核验

`NAMES.WOBSTER_SHELLER`＝**龙虾**、`NAMES.WOBSTER_MOONGLASS`＝**月光龙虾**、`NAMES.WOBSTER_SHELLER_DEAD`＝**死龙虾**、`NAMES.WOBSTER_SHELLER_DEAD_COOKED`＝**美味的龙虾**；两个 `_land` 注册都不靠自己的名字键，显示名由 `nameoverride` 取水体名，仍是"龙虾""月光龙虾"：`NAMES.WOBSTER_SHELLER_LAND` 在 `strings.lua` 里存在但本 prefab 不读，`NAMES.WOBSTER_MOONGLASS_LAND` 根本不存在。[SHELLER](../../../raw/current/scripts/languages/chinese_s.po#L352339-L352342)、[MOONGLASS](../../../raw/current/scripts/languages/chinese_s.po#L352334-L352337)、[DEAD](../../../raw/current/scripts/languages/chinese_s.po#L352344-L352347)、[COOKED](../../../raw/current/scripts/languages/chinese_s.po#L352349-L352352)、[nameoverride](../../../raw/current/scripts/prefabs/wobster.lua#L341)、[陆地名键](../../../raw/current/scripts/strings.lua#L3602)（S7）

## 捕获与出水投射

海钓链（不是池塘钓，S2）：`oceanfishingrod` 抛出的浮子带 `fishinghook` 标签，`wobsterbrain` 在 5 格内找到它、按 `fish_def.lures`（`OCEANFISH_LURE_PREFERENCE.WOBSTER`：hook 0.25、special／rot／berry／spoon／spinnerbait／insect 都是 1.00、seed 0.25）累积兴趣并咬钩；上钩后由 `oceanfishable` 的体力与挣扎推进。[寻饵](../../../raw/current/scripts/brains/wobsterbrain.lua#L69-L120)、[偏好表](../../../raw/current/scripts/tuning.lua#L322-L332)、[体力设置](../../../raw/current/scripts/prefabs/wobster.lua#L206-L211)

- 上钩时 `set_on_rod` 加 `partiallyhooked`／`scarytooceanprey`；`OnReelingIn` 清 `partiallyhooked` 并 `ResetStruggling`。[上下竿](../../../raw/current/scripts/prefabs/wobster.lua#L127-L143)
- 收线成功：`OceanFishingRod:CatchFish` 调 `oceanfishable:MakeProjectile()`（龙虾的 `makeprojectilefn` 就是 `on_make_projectile`，返回**同一个实体**并给它加 `complexprojectile`），把 `weighable` 的 owner 记成钓手，再抛向钓手面前。[CatchFish](../../../raw/current/scripts/components/oceanfishingrod.lua#L337-L361)、[发射](../../../raw/current/scripts/prefabs/wobster.lua#L111-L125)、[捕获距离](../../../raw/current/scripts/tuning.lua#L244)
- 落点分流（S4）：命中点可通行 → 生成 `fish_def.prefab.."_land"`、`SetLanded(true, false)`、`CopyWeighable` 后删掉投射体；不可通行（水面）→ 移除 `complexprojectile`、恢复游泳碰撞与水下渲染、回 `idle`（`jump_pst`）、`RestartBrain("wobster_out_of_water")`、**在同一实体上**清 owner、溅水。[落点](../../../raw/current/scripts/prefabs/wobster.lua#L79-L109)
- 落地后的陆态自己会动：`on_ground_wobster_landed` 先看是否被持有；可通行 → `stunned`；带海洋且落水 → 先生成水体态、`CopyWeighable`（连同 owner 一起复制），再对**即将删除的陆态实例**清 owner，新水体态仍保留 owner；否则 `SinkEntity`。[陆态落地](../../../raw/current/scripts/prefabs/wobster.lua#L253-L276)
- 陆态活体被留在水边（钓上来后自己爬，或被玩家趁震晕窗口搬过去）时，行为树会找 1.5 格内可游泳点并推 `onhop`，状态机跳完就调 `_enter_water()` 换回水体态。[跳水](../../../raw/current/scripts/prefabs/wobster.lua#L284-L297)、[行为树](../../../raw/current/scripts/brains/wobsterlandbrain.lua#L43-L62)、[状态机](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L84-L124)
- 普通陆态的 `halloweenmoonmutable` 用 `SetPrefabMutated("wobster_moonglass_land")`：命中月亮药水时**生成替换实体**，与龙虾窝的就地改写不同（S6）。[接线](../../../raw/current/scripts/prefabs/wobster.lua#L421-L422)、[转换实现](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L45-L68)

## 数值与熟制

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度天数一律 ×480 秒（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 生命 | 饥饿／生命／理智（可食） | 鲜度 | 堆叠 | 交易值 | 关键产物 |
|---|---|---|---|---|---|---|
| `wobster_sheller` / `wobster_moonglass` | 无 `health` | — | — | — | — | 自身没有 `lootdropper`；`fish_def.loot` 是 `wobster_sheller_dead`／`moonglass`，供陆地态死亡与炮台、海獭等外部系统取用 |
| `wobster_sheller_land` / `wobster_moonglass_land` | 25 | —（活体不可食） | `WOBSTER.SURVIVE_TIME`＝4 天＝1920 秒，拿起（进背包或手上）即计时、落地停表 | — | 1 | 宰杀／死亡 → 死龙虾／月光玻璃；普通陆态可烤成熟龙虾 |
| `wobster_sheller_dead` | — | 12.5／1／0 | `PERISH_SUPERFAST`＝3 天＝1440 秒 → `spoiled_fish` | 10（`STACK_SIZE_LARGEITEM`） | 1 | 烤制 → `wobster_sheller_dead_cooked` |
| `wobster_sheller_dead_cooked` | — | 12.5／1／0 | `PERISH_FAST`＝6 天＝2880 秒 → `spoiled_fish` | 10 | 1 | 无 |

数值来源：[生命](../../../raw/current/scripts/tuning.lua#L5463-L5471)、[死龙虾](../../../raw/current/scripts/prefabs/wobster.lua#L463-L483)、[熟龙虾](../../../raw/current/scripts/prefabs/wobster.lua#L512-L529)、[堆叠](../../../raw/current/scripts/tuning.lua#L81)、[交易值](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1855-L1858)。采集册与源码一致：陆态 `perishable=1920`、`health=25`，死龙虾 1440、熟龙虾 2880、两者叠 10。

- **活体的 4 天从"拿起"开始算（S9）**：`MakeSmallPerishableCreature` 构造时 `StopPerishing`，`SetOnPutInInventoryFn` 里 `StartPerishing`——进背包或**拿在手上**都算，落地 `SetOnDroppedFn` 再 `StopPerishing`；所以趁震晕窗口捡起来的那一刻就开始腐坏。到点按 `onperish` 把战利品交给所在容器。[计时](../../../raw/current/scripts/standardcomponents.lua#L972-L994)、[接线](../../../raw/current/scripts/prefabs/wobster.lua#L409)
- **宰杀与死亡掉同一件**：`murderable` 给 `MURDER` 候选、`GenerateLoot` 取 `fish_def.loot`；被玩家打死则走 `SGwobsterland` 的 `death` 状态——月光陆态（`_fades_out`）死亡瞬间 `DropLoot`，普通陆态在动画结束时 `SpawnPrefab(fish_def.loot[1])` 后自删。[动作](../../../raw/current/scripts/actions.lua#L3166-L3197)、[死亡状态](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L146-L182)
- **活体直接烤**：普通陆态 `cookable.product = wobster_sheller_dead_cooked`，`SetOnCookedFn` 播受击音；月光陆态没传 `cook_product`，整段 `cookable` 不挂。[烤制](../../../raw/current/scripts/prefabs/wobster.lua#L375-L379)
- **熟龙虾不能再烤**：熟品没有 `cookable` 组件与标签，只有 `fishmeat`／`catfood`；两件物品都**不带** `fish` 标签，沃特的鱼保鲜与鱼人一系判定不作用于它们（S3）。[标签](../../../raw/current/scripts/prefabs/wobster.lua#L450-L454)、[鱼保鲜](../../../raw/current/scripts/prefabs/wurt.lua#L370-L372)

### 重量与归属

- 分布是**三次方偏小**：`math.random()` 取 0–1 后三次方再 `Lerp(weight_min, weight_max, …)`，多数个体明显小于区间中点（与池塘鱼的 `(p³+1)/2` 中段集中、海鱼的线性分布不同，三套不能互套）。（S5）[SetupWeighable](../../../raw/current/scripts/prefabs/wobster.lua#L145-L151)、[取重与截断](../../../raw/current/scripts/components/weighable.lua#L58-L60)
- `CopyWeighable` 走 `OnSave`／`OnLoad`，**连 `owner_userid`／`owner_name`／`prefab_override_owner` 一起复制**：陆态换水体时先复制、再对**旧实例**清 owner，新水体仍带着原 owner；只有"投射体落回水面"这条是在同一实体上真的清 owner。作为掉落物生成时 `on_loot_dropped` 再写入 `prefab_override_owner`。[复制](../../../raw/current/scripts/components/weighable.lua#L39-L43)、[监听](../../../raw/current/scripts/prefabs/wobster.lua#L278-L282)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`oceanfishable` | 水体态：`makeprojectilefn`／`onreelinginfn`／`onsetrodfn`、体力与挣扎、`catch_distance`＝2.5 | 仅水体态、主模拟 | [[reference/components/oceanfishable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L206-L211) |
| 组件：`inventoryitem` | 陆态：`nobounce`、`canbepickedup = false`（`stunned` 期置 true）、`canbepickedupalive = true` | 仅陆地态 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L361-L364) |
| 组件：`weighable` | 两种形态都带；`type = TROPHYSCALE_TYPES.FISH` | 主模拟；水体与陆态各自 Initialize | [[reference/components/weighable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L197-L198) |
| 组件：`perishable` | 陆态 4 天（拿起后计时、落地停表）；死龙虾 3 天、熟龙虾 6 天 | 物品态构造即走表；陆态靠 helper 的进／出背包回调 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L463-L466) |
| 组件：`cookable` | 陆态 → 熟龙虾；死龙虾 → 熟龙虾 | 仅普通陆态与死龙虾 | [[reference/components/cookable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L479-L480) |
| 组件：`health`／`combat` | 25 点生命、`murdersound`、`nofadeout` 按变体 | 仅陆地态 | [接线](../../../raw/current/scripts/prefabs/wobster.lua#L388-L394) |
| 组件：`lootdropper` | `SetLoot(fish_def.loot)`、`forcewortoxsouls` | 仅陆地态；水面态没有掉落表 | [[reference/components/lootdropper]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L368-L370) |
| 组件：`sleeper`／`tradable` | 睡眠状态；金值 1 | 仅陆地态 | [[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L381-L384) |
| 组件：`halloweenmoonmutable` | 普通陆态 → 月光陆态（生成替换实体） | 仅服务端、仅 `wobster_sheller_land` | [[reference/components/halloweenmoonmutable]]；[接线](../../../raw/current/scripts/prefabs/wobster.lua#L421-L422) |
| SG／Brain：`SGwobster`／`wobsterbrain`、`SGwobsterland`／`wobsterlandbrain` | 水体：寻饵、挣扎、回家；陆地：跳水逃跑 | 分别挂在两种形态 | [水体](../../../raw/current/scripts/prefabs/wobster.lua#L215-L216)、[陆地](../../../raw/current/scripts/prefabs/wobster.lua#L406-L407) |
| 关联 Prefab：`wobster_den`／`moonglass_wobster_den` | 陆态依赖表声明（`land_prefabs`／`moonglass_land_prefabs`）；孩子由窝刷出 | 声明不等于生成 | [[reference/prefabs/wobster-den]]；[依赖](../../../raw/current/scripts/prefabs/wobster.lua#L46-L57) |
| 关联 Prefab：`moonglass`／`spoiled_fish` | 月光陆态掉落、两种物品态的腐坏产物 | 由 `lootdropper`／`perishable` 产生 | [[reference/prefabs/moonglass-materials]]；[依赖](../../../raw/current/scripts/prefabs/wobster.lua#L59-L68) |
| 资源：`lobster.zip`／`lobster_water.zip` 等 | 水体与陆／物品两套 bank／build，另有 `INV_IMAGE` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/wobster.lua#L4-L32) |

## 常见陷阱与限制

- **误区：活龙虾要么永远捡不起来、要么随时能捡。** 两种说法都不对：常态 `canbepickedup = false`，玩家侧 `PICKUP` 候选落空；落地命中可通行点会进 `stunned`（吃到震爆也进同一状态），onenter 把 `canbepickedup` 置 true 并持续 3–7 秒，**这期间玩家可以捡**，onexit 再改回 false；`canbepickedupalive` 只是非玩家执行者那条常规路径。[候选](../../../raw/current/scripts/componentactions.lua#L470-L480)、[震晕窗口](../../../raw/current/scripts/stategraphs/SGwobsterland.lua#L50-L78)、[服务端放行](../../../raw/current/scripts/actions.lua#L894-L902)
- **边界：月光陆态不能烤、也掉不出死龙虾。** 它的 `fish_def.loot` 是 `{"moonglass"}`，且没有 `cook_product`；想吃龙虾只能抓普通的。[数据](../../../raw/current/scripts/prefabs/wobster.lua#L234-L241)、[陆地工厂](../../../raw/current/scripts/prefabs/wobster.lua#L427-L433)
- **边界：投射落地与"持有中落地"是两条路。** 钓起后飞过去命中才走 `on_projectile_landed`；已经拿在手上（仆从携带，或玩家趁震晕窗口捡起后再放下）掉在地上走 `on_landed`，据此进 `stunned` 或跳回水里。[投射落地](../../../raw/current/scripts/prefabs/wobster.lua#L79-L109)、[物品落地](../../../raw/current/scripts/prefabs/wobster.lua#L253-L276)
- **边界：`fishmeat` 不等于 `fish`。** 死龙虾／熟龙虾只有 `fishmeat`＋`catfood`，没有 `fish`，沃特保鲜与鱼人一系判定都不认它们。[标签](../../../raw/current/scripts/prefabs/wobster.lua#L450-L454)
- **边界：落水会换实体。** 陆态落进海里不是"泡着"，而是 `SpawnPrefab(fish_def.prefab)` 换成水体态、搬重量、删原实体；模组缓存实体引用会失效。[入水](../../../raw/current/scripts/prefabs/wobster.lua#L284-L297)、[落水](../../../raw/current/scripts/prefabs/wobster.lua#L261-L272)
- **未核验**：未运行游戏。"活龙虾放进鱼箱"按源码**推导可行**——`fish_box` 只收 `smalloceancreature`，活体正好带这个标签，玩家可趁 3–7 秒震晕窗口捡起后放进鱼箱，未实测；`sleeper` 的主动入睡入口也没找到。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/wobster.lua` 全文（六个工厂、两组 `fish_def`、水体与陆地两套挂件、投射与入水、月光转换）；`brains/wobsterbrain.lua` 与 `brains/wobsterlandbrain.lua` 全文、`SGwobster`／`SGwobsterland` 的相关状态（`death`、`hop`、`hop_pst`、`launched_out_of_water`）；`components/oceanfishable.lua` 与 `oceanfishingrod.lua:CatchFish`；`inventoryitem`（`SetLanded`、`CanBePickedUp`）、`weighable`、`perishable`、`standardcomponents` 的 `MakeSmallPerishableCreature`、`lootdropper`、`trap` 的目标标签、`containers.lua` 的 `fish_box` 过滤；`tuning.lua` 的 `WOBSTER`、`OCEANFISH_LURE_PREFERENCE`、`FISHING_CATCH_DIST`、`STACK_SIZE_LARGEITEM`；`chinese_s.po` 与 `strings.lua` 名称；`scrapbookdata.lua` 交叉核对。
  检索范围：`wobster` 相关检索覆盖 `prefabs/wobster.lua`、`wobster_den.lua`、两套 SG／Brain、`oceanfishable`／`oceanfishingrod`／`inventoryitem`／`trap`；未覆盖：其它角色技能对 `smalloceancreature` 的读取、模组注入与调试命令。
  名称核验：六个注册的显示名与 `nameoverride` 结论见上文。
- 游戏运行：未运行。天数、秒数、重量区间与倍率均为源码推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 窝与刷新：[[reference/prefabs/wobster-den]]
- 海钓链：[[reference/prefabs/fishing-tools]]、[[reference/components/oceanfishable]]
- 物品组件：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/cookable]]、[[reference/components/weighable]]
- 关联产物：[[reference/prefabs/moonglass-materials]]

返回 [[reference/prefabs/index|Prefab 索引]]。
