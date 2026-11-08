---
title: "浮木桩与活木头（driftwood_log、livinglog）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "两件木质堆叠材料：浮木桩 driftwood_log（prefabs/driftwood.lua）来自海上漂流木与水下树根、自带 waterproofer 与 0 防雨效果；活木头 livinglog（prefabs/livinglog.lua）来自树精、活木树与毒桦栗树等，进食／燃烧／焚烧各有惨叫。两者均为 WOOD 食性 0／0、燃料 45、小号可燃 15 秒、只修 health 的 WOOD 修补件（12.5／37.5）。"
aliases: ["driftwood_log", "livinglog", "浮木桩", "活木头", "漂流木", "浮木", "活木"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/driftwood.lua", "prefabs/livinglog.lua", "prefabs/driftwood_trees.lua", "prefabs/watertree_root.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "prefabs/walls.lua", "prefabs/dock_damage.lua", "prefabs/boat_bumpers.lua", "prefabs/livingtree.lua", "prefabs/livingtree_halloween.lua", "prefabs/leif.lua", "prefabs/deciduoustrees.lua", "prefabs/mushgnome.lua", "prefabs/wanderingtrader.lua", "prefabs/cookiecutter.lua", "prefabs/forest.lua", "map/rooms/forest/terrain_ocean.lua", "map/rooms/forest/terrain_moonisland.lua", "map/static_layouts/bullkelpfarmmedium.lua", "scenarios/chest_abandonedboat.lua", "recipes.lua", "components/fuel.lua", "components/fueled.lua", "components/edible.lua", "components/eater.lua", "components/incinerator.lua", "components/repairer.lua", "components/repairable.lua", "components/health.lua", "components/acidlevel.lua", "components/waterproofer.lua", "components/inventoryitem.lua", "components/inventory.lua", "prefabs/compostingbin.lua", "components/stackable.lua", "map/terrain.lua", "map/forest_map.lua", "map/ocean_gen.lua", "standardcomponents.lua", "componentactions.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 浮木桩与活木头（driftwood_log、livinglog）

## 快速判断

- 职责与适用：两件同骨架的物品（浮木桩 `driftwood_log` 与活木头 `livinglog`；`MakeInventoryPhysics` → bank／build＝注册名 → `SetPristine` → 主模拟组件），差异只在来源、修补量与活木头的音效／无防潮。`driftwood.lua` 注册的是 `driftwood_log`（文件名与注册名不同），`livinglog.lua` 注册 `livinglog`。[driftwood 注册](../../../raw/current/scripts/prefabs/driftwood.lua#L6-L62)、[livinglog 注册](../../../raw/current/scripts/prefabs/livinglog.lua#L22-L76)
- 优先使用：改堆叠／燃料／燃烧／修补看[数值与接线](#数值与接线)；查来源看[获得链](#获得链)；改食性、燃料结算与修补目标看[消费端](#食性燃料与修补的消费端)；音效与作祟看[尖叫与作祟](#尖叫与作祟)。
- 重要副作用：两者都带 `edible`（WOOD、0／0）所以会进入读取 `foodtype` 的判定，但玩家默认食性不含 WOOD；都只给 `healthrepairvalue`，目标满血时 REPAIR 被拒；作祟只会把物品弹飞，**不会点燃**（`#HAUNTFIX` 注释段）。
- 修改边界：树木宿主（浮木树、水下树根、活木树、毒桦栗树）各自的砍伐与掉落细节归宿主页；`rock_petrified_tree`／树苗归 [[reference/prefabs/evergreen-trees]] 与 B14；配方消费只列入口不列全表。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `driftwood_log` | `Prefab("driftwood_log", fn, assets)`；公共端 `MakeInventoryPhysics`、bank／build `driftwood_log`、`pickupsound = "wood"`、预加 `waterproofer` 标签、`MakeInventoryFloatable("med", 0.03, 0.65)` | 主模拟才挂 `edible`／`fuel`／燃烧／`hauntable`／`inspectable`／`inventoryitem`／`stackable`／`waterproofer`／`repairer`。[构造](../../../raw/current/scripts/prefabs/driftwood.lua#L13-L59) |
| `livinglog` | 同骨架但多一个 `SoundEmitter`；bank／build `livinglog`、浮水 `("med", 0.1, 0.7)` | 主模拟多挂 `fuel:SetOnTakenFn`、`onignite` 监听、`eatensound`／`incineratesound`；不带 `waterproofer`。[构造](../../../raw/current/scripts/prefabs/livinglog.lua#L30-L73) |

- 两者都没有 `tradable`，也不设 `stackable.maxsize`，堆叠取组件默认 `TUNING.STACK_SIZE_MEDITEM`＝20；`pickupsound` 与浮水在 `SetPristine()` 之前设置（两端生效），其余组件全在主模拟。[堆叠默认](../../../raw/current/scripts/components/stackable.lua#L23-L29)
- 命名：注册名即文件名规则只对活木头成立；浮木桩来自 `prefabs/driftwood.lua`，实显名取 `STRINGS.NAMES.DRIFTWOOD_LOG`＝浮木桩、`LIVINGLOG`＝活木头。[po](../../../raw/current/scripts/languages/chinese_s.po#L342275-L342277)、[po](../../../raw/current/scripts/languages/chinese_s.po#L344755-L344757)

## 数值与接线

| 项 | `driftwood_log` | `livinglog` | 消费者与条件 |
|---|---|---|---|
| 堆叠 | 20（默认 MEDITEM） | 20 | `stackable`；组件不设 `maxsize` |
| 可食 | `FOODTYPE.WOOD`、health 0、hunger 0 | 同 | `edible`；默认食性不含 WOOD（见下节） |
| 燃料 | `MED_FUEL`＝45 | 同，另设 `SetOnTakenFn` | `fueled` 结算时乘 `WET_FUEL_PENALTY`（0.75）与其他倍率 |
| 可燃 | `MakeSmallBurnable(inst, MED_BURNTIME)`＝15 秒、`MakeSmallPropagator` | 同 | 小号燃烧 FX 等级 2、蔓延半径 3–5 |
| 修补 | `healthrepairvalue = REPAIR_LOGS_HEALTH`＝12.5 | 同值 ×3＝37.5 | 只修 health 通道；均带 `boatrepairsound` |
| 浮水 | `("med", 0.03, 0.65)` | `("med", 0.1, 0.7)` | `floater` 的尺寸／垂直偏移／缩放 |
| 防潮 | 预加 `waterproofer` 标签、组件 `SetEffectiveness(0)` | 无 | 组件构造即 `inventoryitem:EnableMoisture(false)` |

- 燃料数值：`MED_FUEL = seg_time * 1.5`，`seg_time`＝30 秒 → 45；湿燃料按 `fueled` 的 `WET_FUEL_PENALTY`＝0.75 折减。活木头会受潮（无 `waterproofer`），浮木桩不会。[常量](../../../raw/current/scripts/tuning.lua#L1671-L1682)、[结算](../../../raw/current/scripts/components/fueled.lua#L200-L214)
- 修补：`repairer` 只写 `healthrepairvalue`，因此只挂 `health_wood` 标签；目标必须同时带 `repairable_wood` 与 `healthrepairable` 才出现 REPAIR 动作（`MATERIALS` 枚举值是小写：`MATERIALS.WOOD`＝`"wood"`）。`address_repair_amount` 先看 health——目标满血时直接返回 false；即使动作被列出，`Repair` 也会失败。[标签](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[结算](../../../raw/current/scripts/components/repairable.lua#L106-L146)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1688-L1698)、[满血判定](../../../raw/current/scripts/components/health.lua#L18-L26)
- `MATERIALS.WOOD` 的实际可修目标有三种赋值路径：直赋 `repairable.repairmaterial = MATERIALS.WOOD` 的是 `boat`、`yotd_boats`、`dock_damage`；`wall_wood` 由 `walls.lua` 用 `data.name` 赋（该值即 `MATERIALS.WOOD`）；`boat_bumpers` 的 `yotd` 变体经数据表 `data.material` 赋 WOOD。`boat`／`yotd_boats`／`wall_wood`／`boat_bumpers` 都有 `health`，12.5／37.5 落在 health 通道；`dock_damage` 没有 health，只手动置 `healthrepairable` 与 `justrunonrepaired`，修补量不生效、直接走兜底返回成功。`repairable` 另有 perish 通道（`perishrepairpercent`）与 work／finiteuses 通道，三者都没有原版 WOOD 目标。[boat](../../../raw/current/scripts/prefabs/boat.lua#L572-L583)、[墙体](../../../raw/current/scripts/prefabs/walls.lua#L324-L341)、[护舷](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L264-L267)、[护舷数据](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L366-L374)、[dock_damage](../../../raw/current/scripts/prefabs/dock_damage.lua#L78-L82)、[兜底](../../../raw/current/scripts/components/repairable.lua#L144-L146)
- 一次修补消耗 1 件（有 `stackable` 就 `Get():Remove()`，否则 `Remove()`）；目标满值或修补量落在不存在的通道时，`Repair` 在消耗前返回 false。[消耗](../../../raw/current/scripts/components/repairable.lua#L164-L183)

### 浮木桩的 waterproofer 边界

- 预加标签只是优化：组件构造会再 `AddTag("waterproofer")`，并在 `inventoryitem` 存在时调用 `EnableMoisture(false)`，**删掉** `inventoryitemmoisture`——所以浮木桩自身永远不会湿。[组件](../../../raw/current/scripts/components/waterproofer.lua#L1-L20)、[moisture 开关](../../../raw/current/scripts/components/inventoryitem.lua#L116-L125)
- `SetEffectiveness(0)` 只参与「装备挡雨」这一路：两个消费者（`acidlevel` 的酸雨分摊与 `Inventory:GetWaterproofness`）都只遍历 `equipslots`，浮木桩没有 `equippable`，进不了任一列表；所以它既不挡雨，也不因这 0 值得益，只保留「自身不吸水」。[效果值](../../../raw/current/scripts/components/waterproofer.lua#L22-L28)、[酸雨](../../../raw/current/scripts/components/acidlevel.lua#L159-L183)、[防水查询](../../../raw/current/scripts/components/inventory.lua#L2714-L2734)

## 获得链

### 浮木桩 driftwood_log

| 来源 | 实际产出 | 证据 |
|---|---|---|
| 海上漂流木 `driftwood_tall`（大） | 砍倒按 `driftwood_tree` 共享表：2 `twigs`＋4 `driftwood_log`（每项概率 1.0）；换桩后 DIG 再给 1 个 | [表](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L26-L34)、[挖桩](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L53-L56) |
| 海上漂流木 `driftwood_small1`／`small2`（小） | 按 `driftwood_small` 表：1 `twigs`＋2 `driftwood_log`；小树砍倒直接碎掉、不留桩 | [表](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L36-L41)、[砍倒](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L99-L104) |
| 水下树根 `watertree_root` | 概率表：1 个 `driftwood_log`（1.0）＋1 个 30% `driftwood_log`＋30% `twigs`；3 次 CHOP，物块落点被设为「在实体内部」 | [表](../../../raw/current/scripts/prefabs/watertree_root.lua#L12-L18)、[产出](../../../raw/current/scripts/prefabs/watertree_root.lua#L99-L111) |
| 世界生成的散落物品 | 浅海房间 `OceanCoastal` 的 `distributeprefabs` 有 `driftwood_log = 3`；月岛碎片房间 0.5；`BullkelpFarmMedium` 布局摆 1 个；沉船宝箱 66% 抽 2–5 件（与 `log` 同池） | [浅海](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L15-L33)、[月岛](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L2-L29)、[布局](../../../raw/current/scripts/map/static_layouts/bullkelpfarmmedium.lua#L109-L116)、[宝箱](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L35-L38) |

- 烧毁的漂流木改走 `on_burnt`：`SetChanceLootTable(nil)`＋`SetLoot({"charcoal"})`，砍倒只给木炭，不再给 `driftwood_log`。[烧毁](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L124-L138)
- `prefabs/forest.lua` 只把 `driftwood_log`／`driftwood_small1`／`small2`／`tall` 列进依赖表（资源预载），不负责生成。[依赖](../../../raw/current/scripts/prefabs/forest.lua#L337-L341)

### 活木头 livinglog

| 来源 | 实际产出 | 证据 |
|---|---|---|
| 树精 `leif`／`leif_sparse` 死亡 | 掉落表 6 个 `livinglog`＋1 个 `monstermeat` | [loot](../../../raw/current/scripts/prefabs/leif.lua#L183-L184) |
| `livingtree` 砍倒 | 战利品表 2 个 `livinglog`（CHOP 20）；DIG 树桩再给 1 个 | [loot](../../../raw/current/scripts/prefabs/livingtree.lua#L188-L198)、[挖桩](../../../raw/current/scripts/prefabs/livingtree.lua#L53-L56) |
| `livingtree_halloween`（万圣夜替换形态） | 砍倒时按阶段 `SetLoot`：1／2 个 `livinglog`；DIG 树桩 1 个 | [阶段](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L15-L29)、[砍倒](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L228-L231)、[挖桩](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L159-L162) |
| 毒桦栗树（怪物形态 `deciduoustrees`） | 三阶段战利品表都是 `livinglog`＋`acorn`；砍倒额外各 20% 掷 `livinglog` 与 `nightmarefuel`；DIG 树桩给 `livinglog` | [表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L103-L105)、[额外](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L577-L580)、[挖桩](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L461-L464) |
| `mushgnome` 死亡 | 共享表：1 个 100%＋1 个 50% `livinglog` | [loot](../../../raw/current/scripts/prefabs/mushgnome.lua#L18-L29)、[接线](../../../raw/current/scripts/prefabs/mushgnome.lua#L186) |
| 沃姆伍德配方 | `Recipe2("livinglog", {HEALTH 20}, TECH.NONE)`，`builder_tag="plantkin"`、`sg_state="form_log"` | [配方](../../../raw/current/scripts/recipes.lua#L338) |
| 流浪商人 | `RANDOM_UNCOMMONS` 商品：1 个 `livinglog`，售价 3 个 `beefalowool` | [商品](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L22-L32)、[配方](../../../raw/current/scripts/recipes.lua#L1003) |

- 作为材料，`livinglog` 被夜剑、传送法杖与焦点、蘑菇农场、暗影/月亮法杖等配方消费；本页只确认「有消费、不是纯燃料」，不维护配方全表。[夜剑](../../../raw/current/scripts/recipes.lua#L481-L483)、[蘑菇农场](../../../raw/current/scripts/recipes.lua#L449)

## 食性、燃料与修补的消费端

- **可食 ≠ 玩家可食。** 默认 `eater` 的 `preferseating` 是 `FOODGROUP.OMNI`，`OMNI.types` 只有 MEAT／VEGGIE／INSECT／SEEDS／GENERIC／GOODIES，不含 WOOD；玩家因此吃不了这两件。全 scripts 内显式把 WOOD 写进食性的只有 `cookiecutter`（`SetDiet({FOODTYPE.WOOD}, ...)`），所以这份「0／0 可食」的实际意义是能进读取 `foodtype` 的非进食判定（例如堆肥桶把 WOOD／ROUGHAGE 记为 browns），但不构成玩家食物。[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)、[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[WOOD 食性](../../../raw/current/scripts/prefabs/cookiecutter.lua#L192)、[堆肥桶](../../../raw/current/scripts/prefabs/compostingbin.lua#L202-L207)
- **燃料**：`fuel` 只提供 `fuelvalue`；被 `fueled` 消耗时按 `fuelvalue × bonus × mult × 湿度 × 精通` 结算，随后 `fuel:Taken(self.inst)`——回调里的 `taker` 是**接收燃料的设施**（火堆、火坑等），不是玩家。活木头的 `FuelTaken` 因此在设施位置播惨叫。[Taken](../../../raw/current/scripts/components/fuel.lua#L31-L36)、[结算](../../../raw/current/scripts/components/fueled.lua#L209-L214)
- **修补**：只走 health 通道，12.5／37.5；一次一件，满血目标被拒（见上节）。

## 尖叫与作祟

- 活木头有四处音效接线：① `fuel:SetOnTakenFn(FuelTaken)`——作为燃料被投入设施时，由**设施**播 `dontstarve/creatures/leif/livinglog_burn`；② `ListenForEvent("onignite", onignite)`——自身被点燃时播同一段；③ `inst.eatensound`——`edible:OnEaten` 在进食者位置播放；④ `inst.incineratesound`——焚化炉销毁物品时播放（优先级高于 `murdersound`）。[音效接线](../../../raw/current/scripts/prefabs/livinglog.lua#L6-L20)、[吃](../../../raw/current/scripts/components/edible.lua#L210-L213)、[焚化](../../../raw/current/scripts/components/incinerator.lua#L16-L25)
- 两者的 `hauntable` 走 `MakeHauntableLaunchAndIgnite`：`Launch` 会把物品弹飞（并清 `is_landed`），但点燃分支已被 `#HAUNTFIX` 整段注释、回调恒返回 false；所以「作祟点燃活木头／浮木桩」在本 build **不会发生**，只有弹飞。[作祟](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[只弹飞的同族 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | WOOD、0／0；提供 `eatensound`（活木头） | 主模拟；玩家默认不可食 | [[reference/components/edible#关键接口]]；[接线](../../../raw/current/scripts/prefabs/livinglog.lua#L46-L49) |
| 组件：`eater`／食性 | 决定「谁能吃 WOOD」 | 主模拟；只有 `cookiecutter` 显式设 WOOD | [[reference/components/eater#关键接口]]；[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063) |
| 组件：`fuel`／`fueled` | 45 燃料；活木头另挂 `OnTakenFn` | 主模拟；湿燃料 ×0.75 | [[reference/components/fuel#关键接口]]；[接线](../../../raw/current/scripts/prefabs/livinglog.lua#L51-L53) |
| 组件：`repairer`／`repairable` | 只修 health：12.5／37.5 | 主模拟；目标需 `healthrepairable`＋`repairable_wood` | [[reference/components/repairer#关键接口]]、[[reference/components/repairable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/driftwood.lua#L56-L59) |
| 组件：`waterproofer` | 自身不吸水；效果值 0 | 主模拟；浮木桩专属 | [[reference/components/waterproofer#关键接口]]；[接线](../../../raw/current/scripts/prefabs/driftwood.lua#L50-L54) |
| 组件：`stackable`／`inventoryitem` | 默认 20；浮水与拾取音 | 两端（浮水）／主模拟（堆叠） | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/components/stackable.lua#L23-L29) |
| 组件：`burnable`／`propagator`／`hauntable` | 15 秒小火、小号蔓延、弹飞 | 主模拟；作祟不点燃 | [[reference/components/burnable#关键接口]]；[可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| 关联 Prefab：`driftwood_tall`／`small1`／`small2` | 浮木桩的主要采集宿主 | 世界生成；烧毁只给木炭 | 源码 [driftwood_trees.lua](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L271-L273)（Wiki 待建） |
| 关联 Prefab：`watertree_root` | 水下树根，含 30% 追加浮木桩 | 世界生成；沉底 | 源码 [watertree_root.lua](../../../raw/current/scripts/prefabs/watertree_root.lua#L131)（Wiki 待建） |
| 关联 Prefab：`leif`／`leif_sparse`、`livingtree` | 活木头的主要来源 | 主模拟 | 源码 [leif.lua](../../../raw/current/scripts/prefabs/leif.lua#L216-L217)、[livingtree.lua](../../../raw/current/scripts/prefabs/livingtree.lua#L219)（Wiki 待建） |

## 典型接入

### 把活木头改成更高燃料值（保留惨叫）

- 前提与选择：模组环境不暴露 `TheWorld`，只通过 `GLOBAL` 取全局表，所以示例要写 `GLOBAL.TheWorld.ismastersim`；`AddPrefabPostInit` 只在实体创建时执行，而 `fueled` 在**每次投入时**读 `fuel_obj.components.fuel.fuelvalue`。[模组环境](../../../raw/current/scripts/mods.lua#L258-L292)、[回调登记](../../../raw/current/scripts/modutil.lua#L593-L599)、[燃料读取](../../../raw/current/scripts/components/fueled.lua#L197-L221)
- 生效与撤销：PostInit 只改**之后创建**的实例，同一次运行里已存在且未被直接改写字段的活木头不会自动变；读档重建（`SpawnPrefab`）会再次执行 PostInit，所以「启用后新建」与「读档重建」两类都是 120 秒。要让注册前就存在的实例也生效，只能直接改该实例的 `components.fuel.fuelvalue`——只有这种「已改写该实例字段」的情形才不必重载；撤掉模组后新建／重建回到 45，已改写字段的现存实例保持 120 直到被重建。[实体构造](../../../raw/current/scripts/mainfunctions.lua#L464-L503)
- 原版依据：`fuelvalue` 在 `livinglog.lua` 主模拟写死为 `TUNING.MED_FUEL`，`fueled` 结算直接读该字段。[定义](../../../raw/current/scripts/prefabs/livinglog.lua#L51-L53)、[结算](../../../raw/current/scripts/components/fueled.lua#L209-L212)

```lua
-- modmain.lua：活木头燃料 45 → 120 秒，尖叫接线保留
AddPrefabPostInit("livinglog", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.fuel.fuelvalue = 120
end)
```

- 调整与验证：单位是「燃料秒」。验收用**干燥**活木头（`wetmult`＝1）、容量足够且无 `fuelmaster`／`multfn` 等倍率的火堆：单件燃料增量应为 120 秒（旧示例的 ×2＝90 秒不再作为本页推荐值），且 `fuel:Taken` 的惨叫仍在（只改数值，不动 `SetOnTakenFn`）。湿着投入仍乘 `WET_FUEL_PENALTY`＝0.75；已存在实例未改写字段时仍是 45。

## 常见陷阱与限制

- **误区：可食就能吃。** 两件的 WOOD 食性只让它们通过「有 edible」的检查；默认 `OMNI` 不含 WOOD，玩家吃不了，原版唯一 WOOD 食性消费者是饼干切割机。[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)
- **误区：浮木桩的 `waterproofer` 会替玩家挡雨。** 它自身不吸水来自 `EnableMoisture(false)`；`SetEffectiveness(0)` 加上不可装备，使这份防雨数值在原版没有落点。[组件](../../../raw/current/scripts/components/waterproofer.lua#L9-L12)
- **边界：修补量只有 health 一条。** 12.5／37.5 只对材质 `wood`（`MATERIALS.WOOD` 的值）且 health 未满的目标有效；满血目标动作即便出现也会在 `address_repair_amount` 返回 false。[判定](../../../raw/current/scripts/components/repairable.lua#L106-L115)
- **边界：作祟只弹飞、不点燃。** `MakeHauntableLaunchAndIgnite` 的点燃段被注释，回调返回 false；不要按「会点燃」设计联动。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **边界：燃料 `Taken` 的 taker 是设施。** 写「玩家听到惨叫」会错；音效播在接收燃料的 `fueled` 实体上。[FuelTaken](../../../raw/current/scripts/prefabs/livinglog.lua#L8-L12)、[调用](../../../raw/current/scripts/components/fueled.lua#L214)
- **未核验**：`map/terrain.lua` 的 `TERRAIN_FILTER` 只影响世界生成落点过滤（`forest_map`／`ocean_gen` 消费），不涉及手动部署；世界生成权重总量未核对。[过滤表](../../../raw/current/scripts/map/terrain.lua#L13-L14)、[消费](../../../raw/current/scripts/map/forest_map.lua#L25-L33)、[海面](../../../raw/current/scripts/map/ocean_gen.lua#L745)；活木头的配方全表、`leif` 的战斗细节与蜡封／皮肤未覆盖。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `driftwood.lua`／`livinglog.lua` 全文、`driftwood_trees.lua` 的三张共享掉落表与烧毁分支、`watertree_root.lua` 的概率表、`livingtree.lua`／`livingtree_halloween.lua` 的砍伐与挖桩、`leif.lua` 的掉落、`deciduoustrees.lua` 的怪物战利品、`mushgnome.lua` 的共享表、`wanderingtrader.lua` 的商品表与 `recipes.lua` 的活木头配方、`cookiecutter.lua` 的 WOOD 食性、`stackable`／`fuel`／`fueled`／`edible`／`eater`／`incinerator`／`repairer`／`repairable`／`waterproofer`／`inventoryitem` 的相关分支、`standardcomponents.lua` 的燃烧与作祟 helper、`tuning.lua` 的 `MED_FUEL`／`MED_BURNTIME`／`REPAIR_LOGS_HEALTH`／`WET_FUEL_PENALTY`／`STACK_SIZE_MEDITEM`。
  名称核验：`STRINGS.NAMES.DRIFTWOOD_LOG`＝浮木桩、`LIVINGLOG`＝活木头；没有 `DRIFTWOOD` 键，注册名是 `driftwood_log`。[po](../../../raw/current/scripts/languages/chinese_s.po#L342275-L342277)、[po](../../../raw/current/scripts/languages/chinese_s.po#L344755-L344757)
  未覆盖：全配方表、活木头与树精的战斗细节、皮肤与蜡封交互、世界生成权重总量。
- 游戏运行：未运行。燃料与修补数值为源码常量；未实测湿燃料与作祟表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 均通过（计数见交付回报）。

## 相关链接

- 树木宿主：[[reference/prefabs/evergreen-trees]]（同批）、[[reference/prefabs/deciduous-trees]]
- 木头与炭、灰：[[reference/prefabs/basic-gathered-materials]]、[[reference/prefabs/ashes-and-charcoal]]
- 机制页：[[reference/components/fuel]]、[[reference/components/repairer]]、[[reference/components/repairable]]、[[reference/components/waterproofer]]、[[reference/components/eater]]

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|Prefab 索引]]。
