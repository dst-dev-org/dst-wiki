---
title: "猫尾（coontail）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "浣猫掉落的 cattoy 物品：0.33 独立判定、默认 20 叠、六条配方，以及给浣猫当玩具换忠诚与被叼进猫背包两种消耗。"
aliases: ["coontail", "猫尾", "Cat Tail", "猫尾巴"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/coontail.lua", "prefabs/catcoon.lua", "prefabs/ticoon.lua", "prefabs/catcoonden.lua", "components/lootdropper.lua", "components/stackable.lua", "components/trader.lua", "standardcomponents.lua", "brains/catcoonbrain.lua", "actions.lua", "componentactions.lua", "recipes.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 猫尾（coontail）

## 快速判断

- 职责与适用：`prefabs/coontail.lua` 顶层 `fn` 直接注册的单一物品（无变体、无皮肤），工厂唯一显式 `AddTag` 是 `cattoy`（`inspectable` 组件会另加同名标签）；主模拟只挂 `inspectable`／`inventoryitem`／`stackable`／`hauntable`，公共端挂 `floater`；没有 `fuel`／`edible`／`tradable`／`cookable`。[注册](../../../raw/current/scripts/prefabs/coontail.lua#L38)、[主模拟段](../../../raw/current/scripts/prefabs/coontail.lua#L29-L33)、[inspectable 标签](../../../raw/current/scripts/components/inspectable.lua#L3)
- 优先使用：查掉率与其它来源看[来源：谁掉猫尾](#来源谁掉猫尾)；查它能做什么看[配方与消耗](#配方与消耗)；只关心堆叠与浮水看[注册与构造](#注册与构造)。
- 重要副作用：浣猫掉它走**独立判定**——`meat` 1.00 必掉、`coontail` 0.33 单独掷一次，同一次死亡可以两件都出；判定前还会过击杀者的幸运公式。[掉表](../../../raw/current/scripts/prefabs/catcoon.lua#L72-L76)、[逐条判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[幸运](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)
- 修改边界：浣猫本体行为（窝、送礼）归猫侧页面，本页只写"谁掉"与"谁消耗"（含被抓玩具这一支）；鞭子等成品页不重复本页配方，见 [[reference/prefabs/whip]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `coontail` | 无包装工厂：`fn()` 直接构造；公共端（Transform／AnimState／Network／物理／动画／标签／floater）→ `SetPristine` → 非主模拟返回 → 主模拟挂件 | 单注册、无皮肤、无 placer。[工厂](../../../raw/current/scripts/prefabs/coontail.lua#L6-L36) |

- 公共端：bank／build 都是 `coontail`、播 `idle`；`MakeInventoryPhysics`；标签 `cattoy`；`MakeInventoryFloatable(inst)` 用默认尺寸 `small`。[外形与标签](../../../raw/current/scripts/prefabs/coontail.lua#L13-L21)、[floater 默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1774)
- 主模拟顺序：`inspectable` → `inventoryitem` → `stackable`（**没有**显式设 `maxsize`）→ `MakeHauntableLaunch`（只弹飞，没有 `MakeSmallBurnable`，不能烧）。[挂件](../../../raw/current/scripts/prefabs/coontail.lua#L29-L33)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **堆叠 20**：`stackable` 构造默认取 `TUNING.STACK_SIZE_MEDITEM`＝20，本文件未覆盖；图鉴数据同为 20。[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L29)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L354)

### 名称核验

`STRINGS.NAMES.COONTAIL`＝**猫尾**（Cat Tail）；本文件不改 prefab 名、不用 nameoverride，注册名与显示名一致（S1）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L341684-L341687)、[键](../../../raw/current/scripts/strings.lua#L2356)

## 来源：谁掉猫尾

| 宿主 | 掉落表与判定 | 条件／证据 |
|---|---|---|
| 浣猫 `catcoon` | `meat` 1.00＋`coontail` 0.33；`SetChanceLootTable('catcoon')`，两条各自独立掷 | 击杀浣猫；1.00 恒掉，0.33 单独 `math.random() <= 0.33`。[表](../../../raw/current/scripts/prefabs/catcoon.lua#L72-L76)、[接线](../../../raw/current/scripts/prefabs/catcoon.lua#L403-L404)、[逐条判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L227) |
| 大虎 `ticoon` | `meat` 0.50＋`coontail` 0.05；同样独立判定 | 大虎有 `health` 与 `lootdropper`，可被击杀；它由 `ticoon_builder` 生成，配方为 1 `lucky_goldnugget`、`TECH.CATCOONOFFERING_THREE`。[表](../../../raw/current/scripts/prefabs/ticoon.lua#L18-L22)、[接线](../../../raw/current/scripts/prefabs/ticoon.lua#L280-L281)、[builder](../../../raw/current/scripts/prefabs/ticoon.lua#L333-L347)、[配方](../../../raw/current/scripts/recipes.lua#L1145) |
| 浣猫窝 `catcoonden` | 2 log＋2 twigs，**没有**猫尾 | 锤击窝时按表掉落。[表](../../../raw/current/scripts/prefabs/catcoonden.lua#L18-L25) |

两个宿主都是"击杀结算时一次判定"，不按天刷新；本页核对的 `scripts` 检索中，除上述两张掉落表外没有 `SpawnPrefab("coontail")`。

## 配方与消耗

六条配方（材料逐条核 `recipes.lua`）：

| 产物 | 材料（含猫尾） | 科技／附加条件 | 证据 |
|---|---|---|---|
| 三尾猫鞭 `whip` | coontail 3＋tentaclespots 1 | `TECH.SCIENCE_TWO` | [L479](../../../raw/current/scripts/recipes.lua#L479) |
| 猫帽 `catcoonhat` | coontail 1＋silk 4 | `TECH.SCIENCE_TWO` | [L536](../../../raw/current/scripts/recipes.lua#L536) |
| 餐桌灯 `decor_lamp` | twigs 2＋coontail 2＋lightbulb 2 | `TECH.CARPENTRY_TWO`，`station_tag="carpentry_station"` | [L1035](../../../raw/current/scripts/recipes.lua#L1035) |
| 甩尾弹弓皮筋 `slingshot_band_tentacle` | coontail 1＋tentaclespots 1 | `TECH.NONE`，`builder_skill="walter_slingshot_bands"`，`nounlock` | [L236](../../../raw/current/scripts/recipes.lua#L236) |
| 再消化电路 `wx78module_digestion` | scandata 2＋coontail 1 | `TECH.ROBOTMODULECRAFT_ONE`，`builder_tag="upgrademoduleowner"` | [L778](../../../raw/current/scripts/recipes.lua#L778) |
| 浣猫崽 `critter_kitten_builder` | coontail 1＋fishsticks 1 | `TECH.ORPHANAGE_ONE`，`nounlock`，`actionstr="ORPHANAGE"` | [L820](../../../raw/current/scripts/recipes.lua#L820) |

- **给浣猫当玩具**：浣猫 `trader` 的接受判定认 `cattoy` 标签（猫尾正好有），交出后 `item:Remove()` 无条件执行；只有"猫当前战斗目标≠给食者"且给食者有 `leader` 时才 `AddLoyaltyTime`。参数 `CATCOON_LOYALTY_PER_ITEM`＝`total_day_time*0.1`＝48 秒（基准 480 秒/日），**实得时间还要乘当前 leader 的 `loyaltyeffectiveness`，并夹到剩余不超过 `maxfollowtime`＝480 秒**：原剩余 470 秒、倍率 1 时只增加 10 秒。[接受判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[交出](../../../raw/current/scripts/prefabs/catcoon.lua#L281-L302)、[常量](../../../raw/current/scripts/tuning.lua#L2971-L2972)、[加忠诚](../../../raw/current/scripts/components/follower.lua#L365-L379)
- **被浣猫叼走**：地上的猫尾带 `cattoy`，浣猫 brain 会把它当玩物；因为猫尾没有 `cattoy` 组件，`CATPLAYGROUND` 落到"可拾取物品"分支，按 `CATCOON_PICKUP_ITEM_CHANCE`＝0.67 塞进浣猫自己的背包（4 格）。[寻找玩物](../../../raw/current/scripts/brains/catcoonbrain.lua#L34-L60)、[动作分支](../../../raw/current/scripts/actions.lua#L3895-L3902)、[常量](../../../raw/current/scripts/tuning.lua#L2985)、[背包格数](../../../raw/current/scripts/prefabs/catcoon.lua#L440-L441)
- 猫尾**不是**燃料、不是食物（无 `fuel`／`edible`／`cookable`），也不能交易（无 `tradable`）。[挂件段](../../../raw/current/scripts/prefabs/coontail.lua#L29-L33)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 默认 20 叠，未设 `maxsize` | 主模拟 | [[reference/components/stackable]]；[挂载](../../../raw/current/scripts/prefabs/coontail.lua#L31) |
| 组件：`floater` | `MakeInventoryFloatable(inst)`，默认 `small` | 公共端 | [[reference/components/floater]]；[调用](../../../raw/current/scripts/prefabs/coontail.lua#L21) |
| 组件：`hauntable` | `MakeHauntableLaunch`：作祟只弹飞 | 主模拟 | [[reference/components/hauntable]]；[调用](../../../raw/current/scripts/prefabs/coontail.lua#L33) |
| 标签：`cattoy` | 浣猫 `trader` 的接受判定与 brain 的玩物筛选都读它 | 世界态物品 | [接受判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[brain](../../../raw/current/scripts/brains/catcoonbrain.lua#L24) |
| 关联 Prefab：`catcoon`／`ticoon` | 两个掉落宿主 | 击杀结算 | [浣猫表](../../../raw/current/scripts/prefabs/catcoon.lua#L72-L76)、[大虎表](../../../raw/current/scripts/prefabs/ticoon.lua#L18-L22) |
| 关联 Prefab：`whip`／`catcoonhat` 等 | 六条配方的产物 | 制作时消耗 | [配方表](#配方与消耗) |

## 典型接入

### 调高浣猫的猫尾掉率

- 前提与选择：服务端 `modmain.lua`。浣猫用 `SetChanceLootTable('catcoon')` 读全局 `LootTables.catcoon`，改表项即可；放在 `AddPrefabPostInit("catcoon", ...)` 里可保证 `SetSharedLootTable` 已登记该表。
- 生效与撤销：`GenerateLoot` 每次掉落时现读表，改动对已存在的浣猫同样生效；只影响之后结算的掉落，卸载模组后新进程恢复 0.33。
- 原版依据：`SetSharedLootTable` 只登记表，`GenerateLoot` 按表项逐条读 `chance`。[登记](../../../raw/current/scripts/components/lootdropper.lua#L21-L28)、[读取](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)

```lua
-- modmain.lua：猫尾掉率 0.33 -> 0.5（共享表，影响所有浣猫）
AddPrefabPostInit("catcoon", function(inst)
    for _, entry in ipairs(GLOBAL.LootTables.catcoon or {}) do
        if entry[1] == "coontail" then
            entry[2] = 0.5
        end
    end
end)
```

- 调整与验证：`chance` 取 0–1，`>= 1` 走恒掉分支；改的是共享表，所有浣猫一致。未运行，只核对了表结构与读取顺序。

## 常见陷阱与限制

- **误区：`meat` 1.00 表示"只掉肉"。** 1.00 只说明肉必掉，`coontail` 0.33 是同表另一条独立判定，可以两件都掉。[表](../../../raw/current/scripts/prefabs/catcoon.lua#L72-L76)
- **边界：0.33 不是最终概率。** 有幸运加成的击杀者会先过 `GetEntityLuckChance`（`LuckFormulas.LootDropperChance`），再与 `math.random()` 比较。[入口](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)
- **边界：给猫吃是交易，不是喂食。** 浣猫接受 `cattoy` 后无条件 `item:Remove()`；正被猫敌视时只清掉它的战斗目标、不加忠诚，整件仍被吞；即使加忠诚，也受 480 秒上限与倍率约束。[交出](../../../raw/current/scripts/prefabs/catcoon.lua#L281-L302)、[上限](../../../raw/current/scripts/components/follower.lua#L365-L379)
- **边界：地上的猫尾会招猫。** 浣猫会把 `cattoy` 当玩物，0.67 概率叼进自己背包；摆在地上做装饰时可能被收走。[动作](../../../raw/current/scripts/actions.lua#L3895-L3902)
- **未核验**：未运行游戏。0.33／0.05 的实际掉落频率、幸运加成后的期望、大虎在活动外的实际可获得性均未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `coontail.lua` 全文（单注册、无 fuel／edible／tradable）、`catcoon.lua` 掉落表与 `trader` 段、`ticoon.lua` 掉落表与 builder、`catcoonden.lua` 窝表；`lootdropper` 的 `SetChanceLootTable`／`GenerateLoot`／幸运入口、`stackable` 默认值、`standardcomponents` 的 floater 与 hauntable；`recipes.lua` 六条配方；`tuning` 堆叠与忠诚常量；`chinese_s.po` 名称；`scrapbookdata` 交叉核对。
- 游戏运行：未运行。天数按 `total_day_time`＝480 秒换算，概率与消耗均为源码推导。
- 复审修订：2026-11-10，R1 条目：P2-5（忠诚条件与反例）、P3-1（cattoy 标签口径）、P3-9（修改边界与正文一致）。
- 复审修订：2026-11-10，R10（忠诚实得时间与上限）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 消费者：[[reference/prefabs/whip]]、[[reference/prefabs/hats-environment]]、[[reference/prefabs/silk]]；组件：[[reference/components/lootdropper]]、[[reference/components/stackable]]、[[reference/components/trader]]、[[reference/components/hauntable]]、[[reference/components/floater]]

返回 [[reference/prefabs/index|Prefab 索引]]。
