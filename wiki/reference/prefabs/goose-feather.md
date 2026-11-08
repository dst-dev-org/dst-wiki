---
title: "麋鹿鹅羽毛（goose_feather）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册的麋鹿鹅羽毛：麋鹿鹅与小鹅的必掉＋概率掉落、克劳斯袋与弃船宝箱两条支线，20 叠、7.5 秒燃料、3 秒燃烧，主要消费是羽毛扇与龙卷风法杖等 6 条配方。"
aliases: ["goose_feather", "麋鹿鹅羽毛", "鹅毛", "大鹅羽毛", "麋鹿鹅掉落", "羽毛扇材料", "龙卷风法杖材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/goose_feather.lua", "prefabs/moose.lua", "prefabs/mossling.lua", "components/lootdropper.lua", "components/klaussackloot.lua", "scenarios/chest_abandonedboat.lua", "scenarios/chestfunctions.lua", "components/tradable.lua", "components/fueled.lua", "components/snowmandecoratable.lua", "standardcomponents.lua", "prefabs/pigking.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 麋鹿鹅羽毛（goose_feather）

## 快速判断

- 职责与适用：`prefabs/goose_feather.lua` 只注册一个物品 `goose_feather`（麋鹿鹅羽毛），没有任何变体或 `_fall`／`_inv` 兄弟注册。[注册](../../../raw/current/scripts/prefabs/goose_feather.lua#L49-L49)
- 优先使用：查掉落看[来源](#来源麋鹿鹅与两条支线)；查"几根做一件"看[消费者](#消费者6-条配方)；查堆叠／燃料／燃烧时间看[注册与构造](#注册与构造)。
- 重要副作用：**没有 `tradable` 组件**（不是"金值 0"，是根本没挂），所以猪王完全不接受；带 `cattoy`／`birdfeather` 两个玩法标签，可喂浣猫、可作猫年神龛供奉；燃料值 7.5 秒但燃烧只需 3 秒。[组件链](../../../raw/current/scripts/prefabs/goose_feather.lua#L31-L44)、[标签](../../../raw/current/scripts/prefabs/goose_feather.lua#L22-L23)
- 修改边界：麋鹿鹅本体（刷新、巢、小鹅、狂暴）归 `moose.lua`／`mossling.lua`，本页只写"谁产出它"的那几行；羽毛扇见 [[reference/prefabs/cooling-fans]]，龙卷风法杖见 [[reference/prefabs/staffs-more]]（S2）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `goose_feather` | `fn()`：实体→`MakeInventoryPhysics`→bank／build `goose_feather`、播 `idle`→`MakeInventoryFloatable(inst, "med", 0.05, 0.68)`→打标签→`SetPristine()`；主模拟挂组件 | 单一注册，无分支、无外包工厂。[实体与动画](../../../raw/current/scripts/prefabs/goose_feather.lua#L6-L25) |

主模拟组件顺序：`inspectable`→`inventoryitem`→`stackable`（`STACK_SIZE_MEDITEM`）→`fuel`（`TINY_FUEL`）→`snowmandecor`→`MakeSmallBurnable(TINY_BURNTIME)`→`MakeSmallPropagator`→`MakeHauntableLaunch`。[组件链](../../../raw/current/scripts/prefabs/goose_feather.lua#L31-L46)

| 属性 | 取值 | 定义处 | 结算处与条件 |
|---|---|---|---|
| 堆叠 | 20 | `maxsize = TUNING.STACK_SIZE_MEDITEM`。[定义](../../../raw/current/scripts/prefabs/goose_feather.lua#L34-L35)、[常量](../../../raw/current/scripts/tuning.lua#L82-L82) | 与四色鸟羽的 40 叠不同 |
| 燃料 | 7.5 秒 | `TINY_FUEL = seg_time*.25`，`seg_time`＝30 秒。[定义](../../../raw/current/scripts/prefabs/goose_feather.lua#L37-L38)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1673) | `fuelvalue × bonusmult × mult × wetmult × masterymult`；湿件再乘 0.75。[结算](../../../raw/current/scripts/components/fueled.lua#L197-L212)、[湿惩罚](../../../raw/current/scripts/tuning.lua#L1671-L1671) |
| 燃烧 | 3 秒 | `TINY_BURNTIME = seg_time*.1`。[接线](../../../raw/current/scripts/prefabs/goose_feather.lua#L42-L42)、[常量](../../../raw/current/scripts/tuning.lua#L1680-L1680) | 入 `burnable:SetBurnTime`；同为羽毛却比四色鸟羽短 4.5 秒。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L171) |
| 漂浮 | `med`／偏移 0.05／缩放 0.68 | [接线](../../../raw/current/scripts/prefabs/goose_feather.lua#L20-L20) | `floater:SetSize`／`SetVerticalOffset`／`SetScale`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1782) |

名称核验（S1）：`NAMES.GOOSE_FEATHER`＝麋鹿鹅羽毛，prefab 没有 `SetPrefabName`，注册名与显示名一致。[po](../../../raw/current/scripts/languages/chinese_s.po#L343384-L343387)

## 来源：麋鹿鹅与两条支线

麋鹿鹅与小鹅都用 `SetChanceLootTable`——表里 `1.00` 的项必掉，`0.33` 的项各自独立判定，且 `GetChance` 会按击杀者幸运放大低于 1 的概率。[结算](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[表分派](../../../raw/current/scripts/components/lootdropper.lua#L213-L221)

| 来源 | 接线 | 实际结果 |
|---|---|---|
| 麋鹿鹅 `moose` | `SetChanceLootTable('moose')`：`goose_feather` 三行 1.00＋两行 0.33 | **必掉 3 根**，另 2 根各 33%，期望 3＋2×0.33＝**3.66 根**（推导）；同表另有肉 6、鸡腿 2、素描 1。[表](../../../raw/current/scripts/prefabs/moose.lua#L26-L42)、[接线](../../../raw/current/scripts/prefabs/moose.lua#L227-L228) |
| 小鹅 `mossling` | `SetChanceLootTable('mossling')`：两行 1.00＋一行 0.33 | **必掉 2 根**，再 33% 一根，期望 2.33 根。[表](../../../raw/current/scripts/prefabs/mossling.lua#L22-L29)、[接线](../../../raw/current/scripts/prefabs/mossling.lua#L163-L164) |
| 克劳斯袋 `klaussack` | `giant_loot3` **9 项**里抽 2 项（`i1`、`i2` 互不重复，撞号时映射到第 9 项） | 羽毛是 9 项之一，命中概率 2/9≈**22.2%**（两次抽取各占一个名额，故不是 1/9）。[表](../../../raw/current/scripts/components/klaussackloot.lua#L25-L33)、[抽取](../../../raw/current/scripts/components/klaussackloot.lua#L115-L123) |
| 弃船宝箱 `chest_abandonedboat` | 条目 `item = {"goose_feather", "malbatross_feather"}`、`count` 2–5、`chance` 0.33 | 先 33% 判定是否生成；生成时**整堆 2–5 个同色**——类型在 `count` 循环之前只掷一次（两色均匀 1/2），循环里每个都用同一个 `itemToSpawn`。[条目](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L30-L32)、[整堆同色](../../../raw/current/scripts/scenarios/chestfunctions.lua#L22-L36) |

## 消费者：6 条配方

| 配方（注册名） | 用量 | 科技／门槛 | 证据 |
|---|---|---|---|
| `featherfan`（羽毛扇） | **5**（＋芦苇 2、绳索 2） | `TECH.SCIENCE_TWO` | [L526](../../../raw/current/scripts/recipes.lua#L526-L526) |
| `staff_tornado`（龙卷风法杖） | **10**（＋闪电羊角 1、齿轮 1） | `TECH.SCIENCE_TWO` | [L490](../../../raw/current/scripts/recipes.lua#L490-L490) |
| `book_rain`（降雨书） | 2（＋纸莎草 2） | `builder_tag = bookbuilder` | [L289](../../../raw/current/scripts/recipes.lua#L289-L289) |
| `battlesong_instant_revive`（复活战歌） | 3（＋纸莎草 1、羽毛笔 1） | 技能 `wathgrithr_songs_revivewarrior` | [L189](../../../raw/current/scripts/recipes.lua#L189-L189) |
| `oceanfishingbobber_goose`（海钓浮子） | 1 | `TECH.FISHING_ONE` | [L651](../../../raw/current/scripts/recipes.lua#L651-L651) |
| `wx78module_spin`（WX 旋转模块） | 1（＋废料数据 6） | `TECH.ROBOTMODULECRAFT_ONE` | [L781](../../../raw/current/scripts/recipes.lua#L781-L781) |

两条大消耗链的效果与上限不在本页：羽毛扇的扇风结算见 [[reference/prefabs/cooling-fans#扇风结算]]，龙卷风法杖的召唤与伤害见 [[reference/prefabs/staffs-more#配方与获取]]（S2）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable`／`fuel`／`burnable`／`hauntable` | 20 叠；燃料 7.5 秒、燃烧 3 秒、`MakeSmallPropagator` 受热点燃；`MakeHauntableLaunch` 只抛飞、无点火分支 | 主模拟 | [接线](../../../raw/current/scripts/prefabs/goose_feather.lua#L34-L44)、[hauntable](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 组件：`snowmandecor` | 标记组件＋`ITEM_DATA` 里 `goose_feather` 一行（`snowman_decor` 动画） | 主模拟；按 `hash(prefab)` 查表扣件 | [接线](../../../raw/current/scripts/prefabs/goose_feather.lua#L40-L40)、[数据](../../../raw/current/scripts/components/snowmandecoratable.lua#L26-L26) |
| 标签：`cattoy`／`birdfeather` | 浣猫玩具、猫年神龛供奉 | 两端均有 | 消费端与四色鸟羽同一处，见 [[reference/prefabs/bird-feathers#关联机制导航]] |
| 关联 Prefab：`featherfan`／`staff_tornado`／`book_rain`／`battlesong_instant_revive`／`oceanfishingbobber_goose`／`wx78module_spin` | 6 条配方消费本物品 | 见上表 | [[reference/prefabs/cooling-fans]]、[[reference/prefabs/staffs-more]] |
| 资源：`anim/goose_feather.zip` | 单条 animation 包，bank＝build＝`goose_feather` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/goose_feather.lua#L1-L4) |

## 典型接入

### 让麋鹿鹅羽毛可以卖给猪王

- 前提与选择：服务端 `modmain.lua` postinit。与四色鸟羽不同，本 prefab **根本没有 `tradable` 组件**，要先 `AddComponent` 再写 `goldvalue`（直接赋值会因 `components.tradable == nil` 报错）；只影响之后生成的实体，移除模组即回到无组件状态。[原组件链](../../../raw/current/scripts/prefabs/goose_feather.lua#L31-L44)
- 原版依据：`pigking` 的接受判定读 `item.components.tradable.goldvalue > 0`，没有组件直接不满足；组件默认金值 0。[判定](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605)、[默认值](../../../raw/current/scripts/components/tradable.lua#L3-L3)

```lua
-- modmain.lua：goose_feather 原版没有 tradable，先补组件再给金值
AddPrefabPostInit("goose_feather", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.tradable == nil then
        inst:AddComponent("tradable")
    end
    inst.components.tradable.goldvalue = 1
end)
```

- 调整与验证：金值同时决定猪王回给的金块数；本例未运行，只核对组件创建顺序与猪王判定，实机需递一根确认收下。

## 常见陷阱与限制

- **误区：麋鹿鹅羽毛能卖。** 它没有 `tradable` 组件（不是金值 0），猪王判定直接落空。[组件链](../../../raw/current/scripts/prefabs/goose_feather.lua#L31-L44)、[判定](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605)
- **误区：一杀必掉 3 根就完了。** 表里还有两条 0.33，各自独立判定，运气好是 5 根；这两条同样受击杀者幸运影响（`GetChance` → `GetEntityLuckChance`）。[表](../../../raw/current/scripts/prefabs/moose.lua#L36-L40)、[幸运](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)
- **边界：燃烧时间只有 3 秒。** 燃料值 7.5 秒与燃烧时间 3 秒是两套数值（`TINY_FUEL` vs `TINY_BURNTIME`），不要混用。[燃料](../../../raw/current/scripts/tuning.lua#L1673-L1673)、[燃烧](../../../raw/current/scripts/tuning.lua#L1680-L1680)
- **边界：堆叠 20 而不是 40。** 同为羽毛，四色鸟羽是 `STACK_SIZE_SMALLITEM`＝40，本物品是 `STACK_SIZE_MEDITEM`＝20。[定义](../../../raw/current/scripts/prefabs/goose_feather.lua#L35-L35)、[对照](../../../raw/current/scripts/tuning.lua#L82-L83)
- **未核验**：未运行游戏。克劳斯袋与弃船宝箱的概率按表推导（2/9 与 0.33×整堆同色），未在存档里验证实际出现频率；幸运倍率的具体曲线未展开。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/goose_feather.lua` 全文；`prefabs/moose.lua` 的 `SetSharedLootTable('moose')` 与接线；`prefabs/mossling.lua` 的对应表；`components/klaussackloot.lua` 的 `giant_loot3` 与两次抽取；`scenarios/chest_abandonedboat.lua` 与 `chestfunctions.lua` 的"整堆同色"（类型只掷一次）；`components/lootdropper.lua` 的 `GetChance`／`GetEntityLuckChance` 路径与 `chanceloottable` 分派；`components/fueled.lua` 的 `TakeFuelItem`；`standardcomponents.lua` 的 `MakeSmallBurnable`／`MakeHauntableLaunch`／`MakeInventoryFloatable`；`components/snowmandecoratable.lua` 的条目；`tuning.lua` 的四个常量；`recipes.lua` 的 6 行；`chinese_s.po` 名称。
  检索范围：`goose_feather` 定向检索覆盖 `recipes.lua`、`moose.lua`、`mossling.lua`、`klaussackloot.lua`、`chest_abandonedboat.lua`、`snowmandecoratable.lua`、`prefablist.lua`；未覆盖：麋鹿鹅本体的巢与刷新、WX 模块的实际效果。
- 游戏运行：未运行。期望根数与时长为源码推导，基准 `seg_time`＝30 秒、`total_day_time`＝480 秒。
- 复审修订：2026-11-10，R2 条目：P1-1（克劳斯袋 9 项、2/9）、P1-2（宝箱整堆同色）、P3-6（补 `prefabs/pigking.lua`）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 羽毛扇：[[reference/prefabs/cooling-fans]]；龙卷风法杖：[[reference/prefabs/staffs-more]]
- 同族羽毛与宿主：[[reference/prefabs/bird-feathers]]、[[reference/prefabs/malbatross-feather]]；麋鹿鹅本体 Wiki 待建（`prefabs/moose.lua`）

返回 [[reference/prefabs/index|Prefab 索引]]。
