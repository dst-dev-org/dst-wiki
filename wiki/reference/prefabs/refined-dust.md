---
title: "尘土块（refined_dust）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "档案馆限量产出的元素类材料：配方是盐晶（saltrock）＋石头＋硝石，但 TECH.LOST 只能靠『蒸馏的知识』教出来；成品用于铥矿捕虫网、琥珀美食与蚁狮进贡，本身可食、可当陷阱饵。"
aliases: ["refined_dust", "尘土块", "Collected Dust", "精炼粉尘", "refined_dust_blueprint"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/refined_dust.lua", "prefabs/dustmoth.lua", "stategraphs/SGdustmoth.lua", "recipes.lua", "recipes_filter.lua", "preparednonfoods.lua", "cooking.lua", "constants.lua", "components/edible.lua", "components/tradable.lua", "components/stackable.lua", "components/inventoryitem.lua", "components/bait.lua", "components/builder.lua", "components/trap.lua", "prefabs/antlion.lua", "prefabs/blueprint.lua", "prefabs/archive_lockbox.lua", "prefabs/cookpot.lua", "map/archive_worldgen.lua", "map/maze_layouts.lua", "map/static_layouts/rooms/archive_start/archive_start.lua", "scenarios/archive_cookpot.lua", "scenarios/chestfunctions.lua", "containers.lua", "actions.lua", "standardcomponents.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 尘土块（refined_dust）

## 快速判断

- 职责与适用：单一注册 `refined_dust`，物品态材料，简中名**尘土块**（英文 "Collected Dust"）。它同时是元素类食材（`FOODTYPE.ELEMENTAL`）与铥矿捕虫网、琥珀美食的必需材料，也是蚁狮收的贡品之一。[注册](../../../raw/current/scripts/prefabs/refined_dust.lua#L48)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L348570-L348572)
- 优先使用：查怎么拿到 → [获取与配方准入](#获取与配方准入)；查谁消耗它 → [用途与消费端](#用途与消费端)；查堆叠／可食／进贡数值 → [关联机制导航](#关联机制导航)。
- 重要副作用：**它能吃**（饥饿 1、生命 10、理智 0），拿唯一的建造材料当零食会亏；`tradable.rocktribute = 1` 让它能进贡蚁狮，`bait` 组件让它能塞进陷阱当饵。[可食](../../../raw/current/scripts/prefabs/refined_dust.lua#L26-L28)、[进贡值](../../../raw/current/scripts/prefabs/refined_dust.lua#L30-L31)、[饵](../../../raw/current/scripts/prefabs/refined_dust.lua#L41)
- 修改边界：本页管单注册构造、配方门禁与五个消费端；『蒸馏的知识』（`archive_lockbox`）的乐师谜题内部、以及同批产物 `archive_resonator_item`／`turfcraftingstation` 不在本页。锅中配方与产物归 [[reference/prefabs/dustmeringue]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `refined_dust` | `Prefab("refined_dust", fn, assets)`：公共端建实体＋transform／anim／sound／network、`MakeInventoryPhysics`、bank／build／`idle` → `SetPristine` → 仅主模拟挂组件 | 只有物品态，没有世界态或投射态；主模拟挂载顺序 `edible` → `tradable` → `stackable` → `inspectable` → `inventoryitem` → `bait` → 作祟接线。[工厂](../../../raw/current/scripts/prefabs/refined_dust.lua#L6-L46) |

- 公共端只声明一个动画包 `anim/refined_dust.zip`，bank 与 build 都叫 `refined_dust`，播 `idle`；没有 `pickupsound` 覆写。[资源](../../../raw/current/scripts/prefabs/refined_dust.lua#L1-L4)、[动画](../../../raw/current/scripts/prefabs/refined_dust.lua#L16-L18)
- `inventoryitem:SetSinks(true)`：掉进海里会下沉，本文件**没有** `floater`，与 `scandata`／`wagpunk_bits` 的浮水接线相反。[下沉](../../../raw/current/scripts/prefabs/refined_dust.lua#L38-L39)

## 获取与配方准入

- 唯一配方：盐晶（`saltrock`）×1 ＋ 石头 ×2 ＋ 硝石 ×1，`TECH.LOST`，产物即 `refined_dust`。[配方](../../../raw/current/scripts/recipes.lua#L472)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L348930-L348932)
- `TECH.LOST` 在 `constants.lua` 里是魔法／科学／远古各 **10** 级；任何科技站都给不出这个等级，所以不学会就做不了。`Builder:KnowsRecipe` 先查已学列表，命中直接放行，否则逐项比对 `recipe.level` 与身上可用科技。[等级表](../../../raw/current/scripts/constants.lua#L1324)、[已学优先](../../../raw/current/scripts/components/builder.lua#L883-L887)、[科技比对](../../../raw/current/scripts/components/builder.lua#L896-L907)
- 学配方的入口只有『蒸馏的知识』（`archive_lockbox`）：它的 `product_orchestrina` 由世界生成在三件产物间轮转分配，取到 `refined_dust` 的那台激活后对 20 世界单位（约 5 格）内每个**还不会**该配方的玩家 `SpawnPrefab("refined_dust_blueprint")` 直接塞进背包。[产物分配](../../../raw/current/scripts/map/archive_worldgen.lua#L19-L28)、[教配方](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L47-L80)
- 这张蓝图是**稀有蓝图**：只要配方 `level` 里有 ≥10 的项就置 `is_rare`，显示名走 `BLUEPRINT_RARE` 模板。[稀有判定](../../../raw/current/scripts/prefabs/blueprint.lua#L212-L214)、[命名](../../../raw/current/scripts/prefabs/blueprint.lua#L227-L230)
- 不学配方也能捡到成品，两条都在档案馆：
  - 起始房间的 `archive_cookpot` 挂 scenario `"archive_cookpot"`，锅里固定放 1 个（`chance = 1`、`count` 缺省 1）。[布局](../../../raw/current/scripts/map/static_layouts/rooms/archive_start/archive_start.lua#L114-L124)、[场景](../../../raw/current/scripts/scenarios/archive_cookpot.lua#L4-L14)、[发放入锅](../../../raw/current/scripts/scenarios/chestfunctions.lua#L14-L36)
  - 迷宫房间按 20% 概率生成 `archive_cookpot` 并带 `data.additems = {"refined_dust"}`，载入后由 `onloadpostpass` 放进锅。[概率](../../../raw/current/scripts/map/maze_layouts.lua#L26-L34)、[放物](../../../raw/current/scripts/prefabs/cookpot.lua#L203-L209)
- 学会后它挂在**精炼**页签（`CRAFTING_FILTERS.REFINE`，列表末尾就是 `refined_dust`）；配方本身不需要站台，随身可做。[页签](../../../raw/current/scripts/recipes_filter.lua#L997-L1017)

## 用途与消费端

| 消费端 | 用法 | 需要 | 证据 |
|---|---|---|---|
| 铥矿捕虫网 `thulecitebugnet` | 铥矿 ×2 ＋ 尘土块 ×1，`TECH.LOST`——蓝图靠喂尘蛾「琥珀美食」（`dustmeringue`）后它打喷嚏掉，蒸馏的知识不教这条 | 1 | [配方](../../../raw/current/scripts/recipes.lua#L400)、[喂食](../../../raw/current/scripts/prefabs/dustmoth.lua#L49-L56)、[掉落](../../../raw/current/scripts/prefabs/dustmoth.lua#L137-L171)、[喷嚏](../../../raw/current/scripts/stategraphs/SGdustmoth.lua#L117-L123) |
| 琥珀美食 `dustmeringue` | 锅中 `test` 只要求 `names.refined_dust` 非空，是唯一必需命名食材 | 1（锅要 4 格全满） | [配方](../../../raw/current/scripts/preparednonfoods.lua#L29-L33)；机制见 [[reference/prefabs/dustmeringue]] |
| 烹饪食材值 | `AddIngredientValues({"refined_dust"}, {decoration=2})`：只有装饰分，不带蔬菜／肉／蛋等任何类别 | — | [食材表](../../../raw/current/scripts/cooking.lua#L159) |
| 蚁狮进贡 | `rocktribute = 1` 只是**贡品资格**：蚁狮有待发奖励或已进入索战条件时 `AcceptTest` 直接拒绝，交易前还先过 `AbleToAccept`（死亡／睡眠／忙碌）；普通进贡给 **160 秒增量**，累计与计时器都封顶 6 天 | 1 | [受理](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[结算](../../../raw/current/scripts/prefabs/antlion.lua#L141-L151)、[交易入口](../../../raw/current/scripts/components/trader.lua#L104-L134) |
| 陷阱饵 | `bait` 组件 → `BAIT` 动作 → `Trap:SetBait` 把饵挂到陷阱上 | 1 | [动作](../../../raw/current/scripts/actions.lua#L1600-L1605)、[接线](../../../raw/current/scripts/components/trap.lua#L412-L425) |

- 锅只收**注册过的烹饪食材**：`params.cookpot.itemtestfn` 判 `cooking.IsCookingIngredient(item.prefab)`，而 `refined_dust` 正是被上面那行 `AddIngredientValues` 注册进 `ingredients` 表的。[锅过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[注册表](../../../raw/current/scripts/cooking.lua#L46-L48)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | `foodtype = FOODTYPE.ELEMENTAL`、`hungervalue = 1`；生命／理智未赋值，沿用默认 10／0，并加 `edible_ELEMENTAL` 标签 | 仅主模拟；进食时结算 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/refined_dust.lua#L26-L28)、[默认值](../../../raw/current/scripts/components/edible.lua#L21-L24)、[标签](../../../raw/current/scripts/components/edible.lua#L11-L15) |
| 组件：`tradable` | `rocktribute = 1`；普通进贡按 1 × 1/3 天 = **160 秒增量**（480/3 推导），`maxragetime` 与计时器都封顶 `ANTLION_RAGE_TIME_MAX`＝6 天 | 蚁狮受理且状态允许时结算 | [[reference/components/tradable]]；[倍率](../../../raw/current/scripts/tuning.lua#L3321-L3323)、[结算](../../../raw/current/scripts/prefabs/antlion.lua#L141-L151) |
| 组件：`stackable` | `maxsize = TUNING.STACK_SIZE_SMALLITEM` = 40；满堆判定 `stacksize >= maxsize` | 仅主模拟 | [[reference/components/stackable]]；[定义](../../../raw/current/scripts/tuning.lua#L83)、[接线](../../../raw/current/scripts/prefabs/refined_dust.lua#L33-L34)、[判定](../../../raw/current/scripts/components/stackable.lua#L63-L65) |
| 组件：`inventoryitem` | `SetSinks(true)`，掉海下沉；无 `floater` | 仅主模拟 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/refined_dust.lua#L38-L39) |
| 组件：`bait` | 无参数构造；决定它能否被当饵挂上陷阱 | 仅主模拟 | [[reference/components/bait]]；[接线](../../../raw/current/scripts/prefabs/refined_dust.lua#L41) |
| 组件：`hauntable` | `MakeHauntableLaunchAndSmash`：`smash` 分支整段被注释（`#HAUNTFIX`），实际只弹飞 | 仅主模拟；作祟时 | [[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/refined_dust.lua#L43)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1084-L1100) |
| API／数据：配方 `refined_dust` | `TECH.LOST` 配方＋`refined_dust_blueprint` 教学；学会后进 REFINE 页签 | 建造端、玩家已学 | [配方](../../../raw/current/scripts/recipes.lua#L472)、[教学](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L73-L80) |
| 关联 Prefab：`dustmeringue`／`thulecitebugnet` | 两个下游产物；捕虫网那条同为 `TECH.LOST` | 配方判定 | [[reference/prefabs/dustmeringue]]、[捕虫网](../../../raw/current/scripts/recipes.lua#L400)（Wiki 待建） |
| 资源：`anim/refined_dust.zip` | bank＝build＝`refined_dust`，动画 `idle`；无额外音效资源 | 公共端 | [assets](../../../raw/current/scripts/prefabs/refined_dust.lua#L1-L4) |

## 典型接入

### 让自定义祭品具备蚁狮贡品资格

- 前提与选择：服务端 `modmain.lua` 里对目标 prefab 做 `AddPrefabPostInit`；示例只做「让物品具备贡品资格」（有 `tradable` 且 `rocktribute > 0`），不改变蚁狮的接受时机——它有待发奖励或已进入索战时仍会拒绝，交易前还要过 `AbleToAccept` 的死亡／睡眠／忙碌判定。
- 生效与撤销：只对**已挂 `tradable`** 的对象生效；改的是组件字段，重载存档后按新值生效，撤销即把字段改回 `nil`。
- 原版依据：`AcceptTest`＝蚁狮无待发奖励、未进索战且物品 `rocktribute > 0`；通过后 `OnGivenItem` 按 `rocktribute × TUNING.ANTLION_TRIBUTE_TO_RAGE_TIME` 加增量，`maxragetime` 与计时器都取 `min(…, ANTLION_RAGE_TIME_MAX)`。[受理](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[结算](../../../raw/current/scripts/prefabs/antlion.lua#L141-L151)

```lua
-- modmain.lua：让自定义祭品能被蚁狮收下
AddPrefabPostInit("my_offering", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.tradable ~= nil then
        inst.components.tradable.rocktribute = 1  -- 160 秒增量，封顶 6 天
    end
end)
```

- 调整与验证：`rocktribute` 是整数倍率，1 对应 160 秒**增量**（不是固定总时长，可能被 6 天上限截断）；给 0 或不设字段会被 `AcceptTest` 拒收，蚁狮有待发奖励或已索战时也不收。示例里 `my_offering` 是占位注册名，须换成已注册实体；未运行游戏。

## 常见陷阱与限制

- **误区：把它当普通食材喂锅或喂人。** 它能吃，但只回 1 饥饿、10 生命，理智 0；锅里也只算 `decoration=2`，凑不出任何主食材类别。真正值钱的用途是铥矿捕虫网与琥珀美食。[可食](../../../raw/current/scripts/prefabs/refined_dust.lua#L26-L28)、[食材值](../../../raw/current/scripts/cooking.lua#L159)
- **边界：不学会配方，制作栏里就没有它。** `TECH.LOST` 三系都要求 10 级，靠科技站永远够不到；必须先拿『蒸馏的知识』发的稀有蓝图。[等级](../../../raw/current/scripts/constants.lua#L1324)、[已学判定](../../../raw/current/scripts/components/builder.lua#L883-L887)
- **边界：`MakeHauntableLaunchAndSmash` 现在只弹飞。** 函数名里的 smash 段被 `#HAUNTFIX` 整块注释，作祟不会把尘土块砸碎或转成别的物件。[实现](../../../raw/current/scripts/standardcomponents.lua#L1084-L1100)
- **未核验**：未运行游戏；档案馆三件产物的分配是**轮转**（按 dispenser 出现顺序取 `products[index]`）还是随地图随机，本页只核到代码里的下标循环，未验证实际存档中的分布。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/refined_dust.lua` 全文；核验 `recipes.lua` 两条相关配方（`refined_dust`、`thulecitebugnet`）、`constants.lua` 的 `TECH.LOST`、`builder.lua` 的 `KnowsRecipe`／`AddRecipe`、`blueprint.lua` 的稀有判定与命名、`archive_lockbox.lua` 的教学段与 `archive_worldgen.lua` 的产品分配、`scenarios/archive_cookpot.lua` 与 `map/maze_layouts.lua`／`prefabs/cookpot.lua` 的入锅路径、`preparednonfoods.lua` 的锅配方判定、`cooking.lua` 的食材注册、`antlion.lua` 的受理与怒气结算、`trap.lua`／`actions.lua` 的 BAIT 链、`edible.lua`／`stackable.lua` 的默认值与判定、`standardcomponents.lua` 的作祟工厂、简中 `.po` 名称键。
  未覆盖：`archive_lockbox` 的乐师谜题如何触发、`archive_resonator_item`／`turfcraftingstation` 两条同批产物、模组注入与调试命令。
- 游戏运行：未运行。160 秒怒气与 40 堆叠均为源码推导（`total_day_time` = 480 秒基准），非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。
- 外部复检修订：2026-11-10，R09 条目：蚁狮受理补「有待发奖励／已索战即拒绝」与 `AbleToAccept` 前置（L53）；160 秒改称增量并补 `ANTLION_RAGE_TIME_MAX` 封顶（L63）；接入标题、前提、依据、示例注释与自验均改「具备贡品资格」口径（L74、L76、L78、L84、L89）。
- 复审修订：2026-11-10，P2/P3 条目：①配方材料名改「盐晶（saltrock）」并加 .po 名称锚点（L6、L37）；②铥矿捕虫网蓝图来源改尘蛾打喷嚏掉落、写明蒸馏的知识不教（L50）；③标签大小写改 `edible_ELEMENTAL`（L62）；④消费端数量改五个（L24）；⑤「20 格」改 20 世界单位≈5 格（L39）。

## 相关链接

- 下游食物：[[reference/prefabs/dustmeringue]]
- 组件：[[reference/components/edible]]、[[reference/components/tradable]]、[[reference/components/stackable]]、[[reference/components/bait]]

返回 [[reference/prefabs/index|Prefab 索引]]。