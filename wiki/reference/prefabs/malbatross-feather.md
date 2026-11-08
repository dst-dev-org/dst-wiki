---
title: "邪天翁羽毛与羽毛帆布（malbatross_feather、malbatross_feathered_weave）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "两个物品注册同页：邪天翁羽毛不在邪天翁的死亡掉落表里，只由受击与撞桅两处 spawnfeather 生成（撞桅限可做工的桅杆，同一目标 3 秒只结算一次；另有弃船宝箱的小概率整堆）；羽毛帆布＝6 羽毛＋1 丝，唯一用途是 4 块一组的邪天翁帆套件。两者同为 20 叠，但标签、燃料与燃烧时间不同。"
aliases: ["malbatross_feather", "malbatross_feathered_weave", "邪天翁羽毛", "羽毛帆布", "邪天翁帆材料", "邪天翁掉落", "羽毛布"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/malbatross_feather.lua", "prefabs/malbatross_feathered_weave.lua", "prefabs/malbatross.lua", "prefabs/mast.lua", "scenarios/chest_abandonedboat.lua", "scenarios/chestfunctions.lua", "map/static_layouts/abandonedboat.lua", "components/floater.lua", "components/inventoryitem.lua", "components/fuel.lua", "components/lootdropper.lua", "components/snowmandecoratable.lua", "componentutil.lua", "standardcomponents.lua", "components/tradable.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# 邪天翁羽毛与羽毛帆布（malbatross_feather、malbatross_feathered_weave）

## 快速判断

- 职责与适用：本页覆盖两个物品注册——`malbatross_feather`（邪天翁羽毛）与 `malbatross_feathered_weave`（羽毛帆布），外加羽毛的落体载体 `malbatross_feather_fall`（不是物品）。三者在两个文件里注册，没有第三个物品兄弟。[羽毛注册](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L82-L83)、[帆布注册](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L46-L46)
- 优先使用：查"羽毛到底怎么来"看[来源](#来源不在死亡掉落表里)；查帆布能做什么看[配方链](#配方链羽毛帆布--邪天翁帆)；查两者差异看[两者差异](#两者差异逐项对照)。
- 重要副作用：羽毛**没有**出现在邪天翁的 `SetChanceLootTable('malbatross')` 里——击杀不掉毛，毛来自受击时 4 次 0.05 判定，以及撞上**可做工的桅杆**时的 3＋2×0.3（同一目标 3 秒内只结算一次；另有弃船宝箱的小概率整堆）；帆布**没有 `birdfeather`／`cattoy` 这类玩法标签**，也没有 `snowmandecor`。[掉落表](../../../raw/current/scripts/prefabs/malbatross.lua#L396-L412)、[撞桅](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L284)、[帆布组件链](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L41)
- 修改边界：邪天翁本体（俯冲、饥饿、饥饿计时、换区）归 `malbatross.lua` 本体页（Wiki 待建），本页只写产出路径；帆与桅杆归海航设施（Wiki 待建）；丝线一侧的配方口径见 [[reference/prefabs/silk#主要消费]]（S2）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `malbatross_feather` | `fn()`：实体→`MakeInventoryPhysics`→bank／build `malbatross_feather`、播 `idle`→`MakeInventoryFloatable(inst, "med", 0.05, 0.68)`→`cattoy`／`birdfeather`→`SetPristine()`；主模拟挂组件 | 物品态；组件链与麋鹿鹅羽毛逐项相同（20 叠、7.5 秒燃料、3 秒燃烧、snowmandecor、无 tradable）。[实体段](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L7-L26)、[组件段](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L32-L45) |
| `malbatross_feather_fall` | `fallfn()`：两面朝向、bank／build 同 `malbatross_feather`、播 `fall`；**只有 Transform／AnimState／SoundEmitter／Network 四个基础件** | 不是物品：无 `inventoryitem`、无标签、无物理；`animover` 时 `SpawnPrefab("malbatross_feather")` 落到同点、对**新羽毛**调 `floater:OnLandedServer()`，随后自删。[工厂](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L51-L79) |
| `malbatross_feathered_weave` | `fn()`：实体→`MakeInventoryPhysics`→bank／build `malbatross_feathered_weave`、播 `idle`→`MakeInventoryFloatable(inst, "small", 0.05, { 1.25, 1.0, 1.25 })`→`SetPristine()`；主模拟挂组件 | 独立文件；20 叠、燃料 15 秒、燃烧 7.5 秒、`inventoryitem:SetSinks(false)`；工厂**没有任何 `AddTag`**。[实体段](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L6-L26)、[组件段](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L41) |

名称核验（S1）：`NAMES.MALBATROSS_FEATHER`＝邪天翁羽毛、`NAMES.MALBATROSS_FEATHERED_WEAVE`＝羽毛帆布；两个 prefab 都没有 `SetPrefabName`／`nameoverride`。`malbatross_feather_fall` 没有名字键也不需要。[羽毛](../../../raw/current/scripts/languages/chinese_s.po#L344974-L344977)、[帆布](../../../raw/current/scripts/languages/chinese_s.po#L344979-L344982)

### 两者差异逐项对照

| 项目 | `malbatross_feather` | `malbatross_feathered_weave` | 证据 |
|---|---|---|---|
| 堆叠 | `STACK_SIZE_MEDITEM`＝20 | `STACK_SIZE_MEDITEM`＝20 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L35-L36)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L29)、[常量](../../../raw/current/scripts/tuning.lua#L82-L82) |
| 标签 | `cattoy`＋`birdfeather`（另有 `fuel` 组件自带的 `BURNABLE_fuel`） | 工厂无 `AddTag`，只有 `fuel` 组件自带的 `BURNABLE_fuel` | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L23-L24)、[fuel 定标签](../../../raw/current/scripts/components/fuel.lua#L1-L8) |
| 燃料 | `TINY_FUEL`＝7.5 秒 | `SMALL_FUEL`＝15 秒 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L38-L39)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L36-L37)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1674) |
| 燃烧 | `TINY_BURNTIME`＝3 秒 | `SMALL_BURNTIME`＝7.5 秒 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L43-L43)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L39-L39)、[常量](../../../raw/current/scripts/tuning.lua#L1680-L1681) |
| 雪人装饰 | 有（`snowmandecor`＋`ITEM_DATA` 一行） | 无 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L41-L41)、[条目](../../../raw/current/scripts/components/snowmandecoratable.lua#L30-L30) |
| 交易 | 无 `tradable` 组件 | 无 `tradable` 组件 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L32-L45)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L41) |
| 漂浮 | `med`／0.05／0.68 | `small`／0.05／`{1.25, 1.0, 1.25}` 三轴缩放 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L21-L21)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L20-L20)、[表缩放](../../../raw/current/scripts/components/floater.lua#L68-L78) |
| 下沉 | 走组件默认 `sinks = false` | 显式 `SetSinks(false)`，落到的仍是同一个默认值 | [默认](../../../raw/current/scripts/components/inventoryitem.lua#L75-L75)、[setter](../../../raw/current/scripts/components/inventoryitem.lua#L310-L311)、[接线](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L33-L34) |

## 来源：不在死亡掉落表里

邪天翁的 `lootdropper` 只挂了 `SetChanceLootTable('malbatross')`，表里是肉 ×7、喙、蓝宝石、黄宝石与素描，**没有羽毛这一行**。[接线](../../../raw/current/scripts/prefabs/malbatross.lua#L509-L510)、[表](../../../raw/current/scripts/prefabs/malbatross.lua#L396-L412)

除弃船宝箱外，羽毛都由 `spawnfeather(inst, time)` 生成：它 `SpawnPrefab("malbatross_feather_fall")`，在以本体为圆心半径 1–3 的随机点（`math.random()*2+1`）放下落体、随机朝向，并把 `inst.feathers` 累加 1；若传了 `time` 则把动画拨到对应帧。[落体生成](../../../raw/current/scripts/prefabs/malbatross.lua#L188-L206)

| 触发路径 | 判定 | 结果 |
|---|---|---|
| 被攻击 `attacked` | 每次被击中做 **4 次** `TryLuckRoll(attacker, 0.05, LuckFormulas.LootDropperChance)` | 每次判定各出一根落体；至少掉一根的概率 1−0.95⁴≈**18.55%**，期望 0.2 根／次（推导）。幸运值会抬高有效概率。[调用](../../../raw/current/scripts/prefabs/malbatross.lua#L143-L147)、[常量](../../../raw/current/scripts/tuning.lua#L9377-L9377)、[幸运](../../../raw/current/scripts/componentutil.lua#L2237-L2251) |
| 撞桅杆 | 命中该分支要连过四道门禁：目标有效且**不在 `recentlycharged`**、不是同时带 `smashable` 与 `health`（那条分支直接击杀）、有 `workable` 且 `CanBeWorked()` 且动作不是 `NET`、最后才要求带 `mast` 标签 | 命中时 3 次无条件＋2 次 `math.random() < 0.3`，期望 3＋2×0.3＝**3.6 根**（推导）；结算后写 `recentlycharged[other] = true` 并 3 秒后清除，**同一个目标 3 秒内只结算一次**。只带 `mast` 但不可做工（没有 `workable` 或 `CanBeWorked()` 为假）的目标拿不到毛，撞树等其它 `workable` 目标也不掉毛。[门禁](../../../raw/current/scripts/prefabs/malbatross.lua#L252-L263)、[结算与去重](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L290)、[清除](../../../raw/current/scripts/prefabs/malbatross.lua#L248-L249) |
| 落体落地 | `malbatross_feather_fall` 播完 `fall` 的 `animover` | 在同点生成真羽毛并对它调 `floater:OnLandedServer()`；落水时这一步会按 `OCEAN_WETNESS` 加湿、压低温度。[落体](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L70-L76)、[落地效果](../../../raw/current/scripts/components/floater.lua#L146-L158) |
| 弃船宝箱（不经邪天翁） | 条目 `item = {"goose_feather", "malbatross_feather"}`、`count` 2–5、`chance` 0.33；类型只掷一次 | 33% 出这一堆，且**整堆 2–5 个同色**（与麋鹿鹅羽毛二选一，均匀 1/2）；由 `map/static_layouts/abandonedboat.lua` 的 scenario 摆进世界。[条目](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L30-L32)、[整堆同色](../../../raw/current/scripts/scenarios/chestfunctions.lua#L22-L36)、[摆位](../../../raw/current/scripts/map/static_layouts/abandonedboat.lua#L121-L121) |

`inst.feathers` 只在 `spawnfeather` 里累加、随 `OnSave`／`OnLoad` 写回，本文件内**没有读取方**——不要把它当掉落数量上限或保底机制。[计数](../../../raw/current/scripts/prefabs/malbatross.lua#L202-L205)、[存档](../../../raw/current/scripts/prefabs/malbatross.lua#L305-L321)

## 配方链：羽毛帆布 → 邪天翁帆

| 环节 | 配方 | 说明 |
|---|---|---|
| 羽毛帆布 | `malbatross_feather` 6＋`silk` 1，`TECH.SCIENCE_TWO` | 羽毛侧最大单笔消耗；丝线侧的口径见 [[reference/prefabs/silk#主要消费]]（S2）。[L471](../../../raw/current/scripts/recipes.lua#L471-L471) |
| 邪天翁帆套件 `mast_malbatross_item` | `driftwood_log` 3＋`rope` 3＋`malbatross_feathered_weave` 4，`TECH.SEAFARING_ONE` | 帆布**唯一**的成品去向：一块帆要吃 4 张帆布，即 24 根羽毛。[L618](../../../raw/current/scripts/recipes.lua#L618-L618) |
| 部署 | `mast_malbatross_item` 的 `setondeploy(inst, "mast_malbatross")` | 放置后变成世界实体 `mast_malbatross`；拆除返料另有一条拆解配方。[部署](../../../raw/current/scripts/prefabs/mast.lua#L585-L589)、[注册](../../../raw/current/scripts/prefabs/mast.lua#L617-L618)、[拆解](../../../raw/current/scripts/recipes.lua#L1630-L1630) |
| 羽毛的其它去向 | `oceanfishingbobber_malbatross` 1 根 | 除了帆布，羽毛只进这一条配方。[L652](../../../raw/current/scripts/recipes.lua#L652-L652) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable`／`fuel`／`burnable` | 两者各按自己的常量挂载，见[差异表](#两者差异逐项对照) | 主模拟 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L35-L43)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L39) |
| 组件：`inventoryitem` | 羽毛未设 `nobounce`，帆布显式 `SetSinks(false)`，两者都落在组件默认值上 | 主模拟 | [帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L33-L34)、[默认](../../../raw/current/scripts/components/inventoryitem.lua#L66-L66)、[sinks 默认](../../../raw/current/scripts/components/inventoryitem.lua#L75-L75) |
| 组件：`hauntable` | 两者都是 `MakeHauntableLaunch`：只抛飞 | 主模拟 | [羽毛](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L44-L44)、[帆布](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L41-L41) |
| 组件：`snowmandecor` | 只有羽毛有；帆布没有 | 主模拟 | [条目](../../../raw/current/scripts/components/snowmandecoratable.lua#L30-L30) |
| 标签：`cattoy`／`birdfeather` | 只有羽毛有；消费端与四色鸟羽同一处 | 两端均有 | [[reference/prefabs/bird-feathers#关联机制导航]] |
| 关联 Prefab：`malbatross_feather_fall` | 落体载体，`animover` 生成真羽毛后自删 | 服务端；每次 `spawnfeather` 一个 | [工厂](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L51-L79) |
| 关联 Prefab：`malbatross` | **宿主侧**羽毛产出：受击 4 次 0.05、撞桅 3＋2×0.3（有门禁与 3 秒去重）；它**不是**全库唯一来源，另有弃船宝箱整堆 | 服务端 | [受击](../../../raw/current/scripts/prefabs/malbatross.lua#L143-L147)、[撞桅](../../../raw/current/scripts/prefabs/malbatross.lua#L252-L290) |
| 关联 Prefab：`mast_malbatross_item`／`mast_malbatross` | 帆布唯一的成品去向 | 部署后为世界实体 | [部署](../../../raw/current/scripts/prefabs/mast.lua#L587-L587) |
| 资源：`anim/malbatross_feather.zip`／`anim/malbatross_feathered_weave.zip` | 各带自己一份 animation 包；落体复用羽毛的 bank／build | 原版资源，不随页分发 | [羽毛 assets](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L1-L4)、[帆布 assets](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L1-L4) |

## 典型接入

### 让邪天翁每次受击掉毛判定命中时掉两根

- 前提与选择：服务端 `modmain.lua` postinit。**只有受击路径**经实例字段调用——`OnAttacked` 里写的是 `inst.spawnfeather(inst, 0.4)`；撞桅路径写的是**文件局部** `spawnfeather(inst, 0.4)`，绕开实例字段，包 `inst.spawnfeather` 对它无效。所以本示例只改受击：4 次 0.05 判定**次数不变**，每次命中时生成 2 根。[受击调用](../../../raw/current/scripts/prefabs/malbatross.lua#L143-L147)、[撞桅调用](../../../raw/current/scripts/prefabs/malbatross.lua#L275-L283)
- 生效与撤销：只影响之后生成的邪天翁；卸载模组后新个体回到"一次判定一根"。
- 原版依据：`spawnfeather(inst, time)` 只做"生成落体＋累加计数"，不读任何消耗上限，因此重复调用即线性增产；它在构造末尾被绑到 `inst.spawnfeather`，所以受击回调能读到被替换后的字段。[实现](../../../raw/current/scripts/prefabs/malbatross.lua#L188-L206)、[绑定](../../../raw/current/scripts/prefabs/malbatross.lua#L555-L555)

```lua
-- modmain.lua：只作用于受击路径——每次 0.05 判定命中时生成两根（撞桅走文件局部函数，不受影响）
AddPrefabPostInit("malbatross", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local old = inst.spawnfeather
    inst.spawnfeather = function(mb, time)
        old(mb, time)
        old(mb, time)
    end
end)
```

- 调整与验证：验收对照两条——① 让 4 次 0.05 判定里**恰好命中 1 次**：原版掉 1 根、装模组后掉 2 根（一次攻击仍掷满 4 次判定，不是「打一下掉一根」）；② 一次满足门禁的撞桅：仍是 3 根保底＋2 次 30% 额外判定，与装模组前相同。四次 0.05 判定仍各自独立掷，不要把它读成"每次受击固定掉 2 根"。本例未运行，只核对两处调用形式与字段绑定顺序。

## 常见陷阱与限制

- **误区：杀掉邪天翁就能拿羽毛。** `malbatross` 掉落表没有羽毛这一行，击杀只给肉、喙、宝石与素描。羽毛只从受击、撞桅（限可做工的桅杆、同一目标 3 秒一次）与弃船宝箱来。[表](../../../raw/current/scripts/prefabs/malbatross.lua#L396-L412)
- **边界：`malbatross_feather_fall` 不是可捡物品。** 它只有 Transform／AnimState／SoundEmitter／Network，没有 `inventoryitem` 与标签；捡不到，只是在落点播一段下落动画后换成真羽毛。[工厂](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L51-L79)
- **边界：羽毛帆布没有玩法标签。** 它只从 `fuel` 组件间接拿到 `BURNABLE_fuel`，既不是 `birdfeather`（不能供奉猫年神龛）也不是 `cattoy`，与羽毛本体不同。[组件链](../../../raw/current/scripts/prefabs/malbatross_feathered_weave.lua#L28-L41)、[对照](../../../raw/current/scripts/prefabs/malbatross_feather.lua#L23-L24)
- **边界：两者堆叠相同但燃料不同。** 同为 20 叠，羽毛 7.5 秒燃料／3 秒燃烧，帆布 15 秒／7.5 秒；别按"羽毛类"套一个值。[常量](../../../raw/current/scripts/tuning.lua#L1673-L1674)、[燃烧常量](../../../raw/current/scripts/tuning.lua#L1680-L1681)
- **边界：`SetSinks(false)` 不是差异点。** `inventoryitem.sinks` 默认就是 false，这行只是显式写默认值。[默认](../../../raw/current/scripts/components/inventoryitem.lua#L75-L75)、[setter](../../../raw/current/scripts/components/inventoryitem.lua#L310-L311)
- **边界：撞树、或撞不可做工的桅杆，都不掉毛。** 先要过 `workable`／`CanBeWorked()`／动作非 `NET` 这道门才轮到 `mast` 标签；反例是一个**只带 `mast` 标签但没有 `workable`** 的目标——它在门禁处就被挡下，一根都不生成。[门禁](../../../raw/current/scripts/prefabs/malbatross.lua#L252-L263)
- **未核验**：未运行游戏。18.55% 与 3.6 根是源码概率推导，未验证实际手感；`inst.feathers` 是否被外部系统（成就、采集册）读取未逐一排查。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/malbatross_feather.lua`（两工厂）与 `prefabs/malbatross_feathered_weave.lua` 全文；`prefabs/malbatross.lua` 的 `OnAttacked`／`spawnfeather`／`onothercollide`／`SetSharedLootTable('malbatross')`／`OnSave`／`OnLoad`；`prefabs/mast.lua` 的部署接线；`components/floater.lua` 的 `OnLandedServer` 与 `SetScale` 表缩放；`components/inventoryitem.lua` 的 `sinks` 默认值与 setter；`components/snowmandecoratable.lua` 的条目；`componentutil.lua` 的 `TryLuckRoll`；`tuning.lua` 的燃料／燃烧／掉毛概率常量；`recipes.lua` 的 4 行；`chinese_s.po` 两个名称。
  检索范围：`malbatross_feather`／`malbatross_feathered_weave` 定向检索覆盖 `recipes.lua`、`malbatross.lua`、`mast.lua`、`scrapbookdata.lua`、`snowmandecoratable.lua`；未覆盖：邪天翁本体的俯冲与换区、桅杆与帆的完整行为、幸运倍率曲线。
- 游戏运行：未运行。概率与期望根数为源码推导，基准 `seg_time`＝30 秒、`total_day_time`＝480 秒。
- 复审修订：2026-11-10，R2 条目：P2-1（补弃船宝箱来源、去「全部」绝对句、summary 同步）、P3-7（补 `size_class: standard`）。
- 外部复检修订：2026-11-10，R01（示例限定受击路径、验收按 4 次判定计）、R02（撞桅门禁与 3 秒去重、导航区分宿主与宝箱）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 同族羽毛：[[reference/prefabs/bird-feathers]]、[[reference/prefabs/goose-feather]]
- 丝线一侧：[[reference/prefabs/silk]]
- 海航设施（帆与桅杆）：Wiki 待建（`prefabs/mast.lua`）

返回 [[reference/prefabs/index|Prefab 索引]]。
