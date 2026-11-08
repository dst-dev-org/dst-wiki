---
title: "催长剂起子与超级催长剂（soil_amender／soil_amender_fermented）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "原版园艺肥料族参照：soil_amender 用 perishable 作 3 游戏日发酵计时并按三态分档肥力，到时替换为独立构造的 soil_amender_fermented（finiteuses 5 次，替换物尝试回填原位置／持有格）；两态部署到可耕土壤按养分表加地块养分（也可走普通 FERTILIZE 动作），施用完返还空瓶子，全程不可堆叠、不可食用。"
aliases: ["soil_amender", "soil_amender_fermented", "催长剂起子", "超级催长剂", "已发酵催长剂", "正在发酵的催长剂", "发酵计时", "Growth Formula Starter", "Super Growth Formula"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/soil_amender.lua", "prefabs/fertilizer_nutrient_defs.lua", "prefabs/wormwood.lua", "components/perishable.lua", "components/fertilizer.lua", "components/finiteuses.lua", "components/inventoryitem.lua", "components/farming_manager.lua", "components/pickable.lua", "components/crop.lua", "components/grower.lua", "standardcomponents.lua", "widgets/itemtile.lua", "componentactions.lua", "mainfunctions.lua", "entityscript.lua", "actions.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 催长剂起子与超级催长剂（soil_amender／soil_amender_fermented）

## 快速判断

- 职责与适用：`prefabs/soil_amender.lua` 同文件注册的两个物品类肥料（[注册](../../../raw/current/scripts/prefabs/soil_amender.lua#L251-L252)）：`soil_amender`（催长剂起子）挂 `perishable` 当 3 游戏日的发酵计时用，按 fresh／stale／spoiled 三态给出低／中／高三档肥力，计时结束后替换为独立构造、无新鲜度的 `soil_amender_fermented`（超级催长剂，5 次施用；替换物尝试回填原位置／持有格）；两态都不可堆叠、不可食用，主用途是部署到可耕土壤给农田地块加养分，也可经普通 FERTILIZE 动作喂给符合条件的旧目标（未成熟作物、贫瘠采集物等）或自肥者（沃姆伍德挂 `fertilizable` 组件，[wormwood.lua#L845-L846](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L846)）。
- 优先使用：给农田补养分、复刻"物品随时间发酵升级"或"施用返还容器"时参照本页；改发酵时长与替换物读[注册与构造](#注册与构造)与[典型接入](#典型接入)；肥料数值如何被消费见 [[reference/components/fertilizer]]。
- 重要副作用：每次施用完成都触发 `bottlereturnfn`，最后一次施用移除本体并返还 1 个 `messagebottleempty`——优先交给有物品栏组件的使用者，否则在目标或物品位置生成（[返还](../../../raw/current/scripts/prefabs/soil_amender.lua#L41-L53)、[消费](../../../raw/current/scripts/components/fertilizer.lua#L30-L48)）；soil_amender 的"spoiled"态只是三档中肥力最高的一档，不会产生腐烂食物。
- 修改边界：本页覆盖两注册的工厂、三态发酵链、部署施肥与返还链；farm_plant 作物生长消费、堆肥桶转化等下游设施只给入口不展开（作物侧数值见 [[reference/prefabs/farm-crops-staple]]、种子侧见 [[reference/prefabs/farm-crop-seeds]]）；客户端只读 `fertilizer_index` 净变量与标签，施肥结算全在服务端（[部署](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `soil_amender` | `fn`：Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank／build `soil_amender`、`fresh_idle` → `MakeInventoryFloatable`＋`MakeDeployableFertilizerPristine`（`tile_deploy`／`gridplacer_farmablesoil`）→ Pristine → 主模拟 inspectable→inventoryitem→fertilizerresearchable→fertilizer→perishable→`MakeDeployableFertilizer`→`MakeHauntableLaunch` | 物品；带 `perishable`（`PERISH_SUPERFAST`）、`displaynamefn` 三态切名、`fertilizer_index` 净变量同步；[工厂](../../../raw/current/scripts/prefabs/soil_amender.lua#L131-L195) |
| `soil_amender_fermented` | `fermented_fn`：同上骨架，`fermented_idle` 动画，`fertilizerkey` 直接定死 `"soil_amender_fermented"` → Pristine → 主模拟 inspectable→inventoryitem→fertilizerresearchable→fertilizer→finiteuses→`MakeDeployableFertilizer`→`MakeHauntableLaunch` | 物品；无 `perishable`、无 `displaynamefn`／净变量，构造期一次性写死高档肥力＋5 次 `finiteuses`；[工厂](../../../raw/current/scripts/prefabs/soil_amender.lua#L197-L249) |

两态均不挂 `stackable`——整文件无 `AddComponent("stackable")`，已全文检索（也见[验证记录](#验证记录)），两态均 `MakeInventoryFloatable`＋`MakeHauntableLaunch`；assets 共用同一张表（`soil_amender.zip` 动画＋`soil_amender_stale`／`_spoiled` 两个 `INV_IMAGE`）（[assets](../../../raw/current/scripts/prefabs/soil_amender.lua#L1-L7)）。

名称（简中 `.po` `msgctxt` 定向核验，[L349714-L349737](../../../raw/current/scripts/languages/chinese_s.po#L349714-L349737)）：`SOIL_AMENDER`＝催长剂起子（配方与默认键）；`_FRESH`＝催长剂起子；`_STALE`＝正在发酵的催长剂；`_SPOILED`＝已发酵催长剂；`SOIL_AMENDER_FERMENTED`＝超级催长剂。`soil_amender` 实显名走 `displaynamefn` 按新鲜度标签在 FRESH／STALE／SPOILED 三键间切换（[getdisplayname](../../../raw/current/scripts/prefabs/soil_amender.lua#L59-L63)）。

### 三态发酵链

1. 构造期即定初态：事件监听先注册，随后 `SetPerishTime(PERISH_SUPERFAST)` 赋值触发 perishable 的标签刷新（比例 ≥0.5 判 fresh），推 `forceperishchange`，`update_fertilizer` 依此写入低档肥力——监听若晚于 `SetPerishTime` 注册，初值将停留组件默认（[标签驱动](../../../raw/current/scripts/components/perishable.lua#L1-L27)、[接线](../../../raw/current/scripts/prefabs/soil_amender.lua#L184-L189)）。
2. 运行期按剩余比例跨过 0.5／0.2（`PERISH_FRESH`／`PERISH_STALE`）阈值时切标签并再推 `forceperishchange`，`update_fertilizer` 按新档重写 fertilizer 数值、切换 `percolate` 动画与 `ChangeImageName` 图标、置 `fertilizer_index` 供客户端同步（[update_fertilizer](../../../raw/current/scripts/prefabs/soil_amender.lua#L81-L114)、[阈值](../../../raw/current/scripts/tuning.lua#L1647-L1648)）。
3. 计时归零走 `Perishable:Perish`：无食物意义上的"变坏"，而是按 `onperishreplacement` 生成 `soil_amender_fermented`，`onreplacedfn` 为其补播放 `fermented_pre` 动画与音效，然后旧实例移除——地面实例在旧坐标生成替换物，库存态把旧槽传给对应库存或容器的 `GiveItem` 尝试回填，能否落回原槽由收纳规则决定（[替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)、[onreplacedfn](../../../raw/current/scripts/prefabs/soil_amender.lua#L71-L79)）。
4. `soil_amender_fermented` 独立构造、无 perishable，不再变化也不可逆；发酵计时受容器保鲜／冰箱／季节等通用倍率影响（[Update](../../../raw/current/scripts/components/perishable.lua#L60-L154)），在物品栏与地面上都持续。

**与食材腐坏的区别**：`soil_amender` 无 `edible` 组件，perishable 只当发酵倒计时，三态只改肥力档与显示名，永不产出 `spoiled_food`；常规食材靠 `edible`＋health/hunger 走腐烂替换（`spoiled_food` 是另一注册，见 [[reference/components/perishable]]）。物品栏的"腐坏度"条只属于 `soil_amender` 起子（`show_spoilage` 标签＋fresh/stale/spoiled 标签的组合，[标签接线](../../../raw/current/scripts/prefabs/soil_amender.lua#L151-L162)、[物品栏腐坏条](../../../raw/current/scripts/widgets/itemtile.lua#L732-L747)）；超级催长剂无 perishable，栏内显示的是耐久（finiteuses，[挂载](../../../raw/current/scripts/prefabs/soil_amender.lua#L233-L243)）。

### 数值与获取

（tuning 键＋实际值；`day_time`＝300 秒、`total_day_time`＝480 秒、`perish_warp`＝1，[tuning.lua#L20-L47](../../../raw/current/scripts/tuning.lua#L20-L47)）

| 项 | 键／定义 | 实际值 |
|---|---|---|
| 发酵总时长 | `TUNING.PERISH_SUPERFAST`＝3×`total_day_time` | 1440 秒＝3 游戏日（[tuning.lua#L2043-L2045](../../../raw/current/scripts/tuning.lua#L2043-L2045)） |
| 三态养分（公式一） | `SOILAMENDER_NUTRIENTS_LOW/MED/HIGH` | {8,0,0}／{16,0,0}／{32,0,0}（[tuning.lua#L5868-L5870](../../../raw/current/scripts/tuning.lua#L5868-L5870)） |
| `fertilizervalue`（旧版 crop 生长进度） | `SOILAMENDER_FERTILIZE_LOW/MED/HIGH` | `day_time/8`＝37.5／`day_time/2`＝150／`day_time*1.5`＝450（[tuning.lua#L1626-L1645](../../../raw/current/scripts/tuning.lua#L1626-L1645)） |
| `soil_cycles`（旧版农田地力） | `SOILAMENDER_SOILCYCLES_LOW/MED/HIGH` | 1／2／5 |
| `withered_cycles`（枯萎植物复壮） | `SOILAMENDER_WITHEREDCYCLES_LOW/MED/HIGH` | 0.25／0.5／1 |
| 超级催长剂 | 三项均取 `*_HIGH` 档＋`SOILAMENDER_FERMENTED_USES`＝5 次 | [tuning.lua#L1636-L1641](../../../raw/current/scripts/tuning.lua#L1636-L1641) |

获取：仅 `soil_amender` 有配方——空瓶子 1＋海带 1＋灰 1，`TECH.SCIENCE_TWO`，园艺筛选组（[recipes.lua#L445](../../../raw/current/scripts/recipes.lua#L445)）；`soil_amender_fermented` 无配方，唯一入栏路径是发酵替换（[Perish](../../../raw/current/scripts/components/perishable.lua#L293-L318)）。养分表对应 `FERTILIZER_DEFS` 键 `soil_amender_low/med/high/fermented`，fermented 条目另带 `uses＝5`（[fertilizer_nutrient_defs.lua#L7-L10](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L7-L10)）；这些键供植物图鉴施肥页按 `SORTED_FERTILIZERS` 排序列出（[fertilizer_nutrient_defs.lua#L54-L77](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L54-L77)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：perishable | `SetPerishTime(PERISH_SUPERFAST)`＋`StartPerishing`；`onperishreplacement`＝`soil_amender_fermented`、`onreplacedfn` 播替换动画 | 仅 soil_amender；计时全程有效，含栏内与地面 | [[reference/components/perishable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/soil_amender.lua#L184-L189) |
| 组件：fertilizer | 三态 `update_fertilizer` 动态写 `fertilizervalue/soil_cycles/withered_cycles`＋`SetNutrients`；fermented 构造期写死高档 | 主模拟；`onappliedfn`＝`bottlereturnfn` 承接返还 | [[reference/components/fertilizer#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/soil_amender.lua#L180-L181)、[fermented](../../../raw/current/scripts/prefabs/soil_amender.lua#L233-L238) |
| 组件：finiteuses | fermented 专属：上限＝当前＝5；未设 `SetConsumption`，由 `Fertilizer:OnApplied` 每次 `Use(1)`，耗尽即"最后一次施用" | 仅 soil_amender_fermented；`SetOnFinished` 注释掉，由 `onappliedfn` 路径自毁 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/soil_amender.lua#L240-L243)、[消费](../../../raw/current/scripts/components/fertilizer.lua#L30-L48) |
| 组件：deployable | `MakeDeployableFertilizerPristine`（公共端 `deployable`／`tile_deploy` 标签、placer 名）＋`MakeDeployableFertilizer`（`DEPLOYMODE.CUSTOM`、`keep_in_inventory_on_deploy`） | 两态同接；仅可部署在 `IsFarmableSoilAtPoint` 处 | [[reference/components/deployable#关键接口]]；[pristine](../../../raw/current/scripts/standardcomponents.lua#L1812-L1821)、[部署](../../../raw/current/scripts/standardcomponents.lua#L1823-L1831) |
| 组件：fertilizerresearchable | `SetResearchFn` 返回 `fertilizerkey`（low/med/high/fermented），供植物图鉴登记养分样本 | 两态同接；主模拟 | [[reference/components/fertilizerresearchable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/soil_amender.lua#L177-L178) |
| 组件：inventoryitem | 承接 `ChangeImageName` 三态图标与`keep_in_inventory_on_deploy` 的栏内部署 | 主模拟；`imagename` 变化推 `imagechange` 事件 | [[reference/components/inventoryitem#关键接口]]；[ChangeImageName](../../../raw/current/scripts/components/inventoryitem.lua#L470-L473) |
| 组件：inspectable | soil_amender 的 `getstatus` 按 spoiled/stale 标签返回检视态（换 DESCRIBE 文案） | 主模拟；fermented 用默认检视 | [[reference/components/inspectable#关键接口]]；getstatus [定义](../../../raw/current/scripts/prefabs/soil_amender.lua#L65-L69) |
| 组件：floater／hauntable | `MakeInventoryFloatable("small",…)`；`MakeHauntableLaunch` 作祟弹飞 | 公共端 floater／主模拟 hauntable | [[reference/components/hauntable#按效果选工厂]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1059) |
| Replica／同步：`fertilizer_index` | `net_tinybyte`＋`onfertilizerindexdirty` 事件回读，把 1/2/3 档映射回 `fertilizerkey` | 构造于 Pristine 前，客户端可读；服务端经 `update_fertilizer` 写 | [净变量](../../../raw/current/scripts/prefabs/soil_amender.lua#L158-L162) |
| API／数据：`show_spoilage` 标签 | 让 soil_amender 起子的物品栏腐坏条无视"不可食用"照常显示（fermented 无此标签与保质阶段标签，栏内显示耐久） | 公共端标签；itemtile 结合 fresh/stale/spoiled 标签判定 | [标签](../../../raw/current/scripts/prefabs/soil_amender.lua#L153)、[判定](../../../raw/current/scripts/widgets/itemtile.lua#L732-L747) |
| API／数据：`fertilizer_ondeploy` | 部署点取地块坐标，`farming_manager:AddTileNutrients` 加养分，再 `Fertilizer:OnApplied` 结算消耗与返还 | 主模拟；目标须 `IsFarmableSoilAtPoint` | [[reference/components/fertilizer#典型接入]]；[实现](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)、[加养分](../../../raw/current/scripts/components/farming_manager.lua#L394-L398) |
| API／数据：FERTILIZE 动作 | 对 crop/grower/pickable 目标施肥的旧路径：分别消费 `fertilizervalue`／`soil_cycles`／`withered_cycles`；也是自肥者（fertilizable）的喂食入口 | 主模拟；宿主为遗留作物（plant_normal）、遗留农田（farmplot）、可枯萎采集物、自肥者（沃姆伍德 [wormwood.lua#L845-L846](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L846)） | [[reference/components/fertilizable]]；[动作](../../../raw/current/scripts/actions.lua#L1812-L1842)、[收集器](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[crop](../../../raw/current/scripts/components/crop.lua#L89-L104)、[grower](../../../raw/current/scripts/components/grower.lua#L68-L84)、[pickable](../../../raw/current/scripts/components/pickable.lua#L259-L277) |
| 关联 Prefab：`messagebottleempty` | 施用完毕返还的空瓶子（配方原料可循环） | 最后一次施用时生成并 `GiveItem` 入栏，无主时落地 | [返还](../../../raw/current/scripts/prefabs/soil_amender.lua#L41-L53)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L345414-L345417) |
| 关联 Prefab：`poopcloud`／`gridplacer_farmablesoil` | poopcloud 仅 `prefabs` 依赖声明（本文件无生成调用）；gridplacer 为部署网格 placer | 依赖声明不等于生成 | [声明](../../../raw/current/scripts/prefabs/soil_amender.lua#L9-L14) |
| 资源：anim `soil_amender.zip` | bank／build `soil_amender`；fresh/stale/spoiled/fermented 四组 idle／pre／loop 动画＋`percolate` 随机间隔循环 | 静态声明；`INV_IMAGE` 两枚切图标 | [assets](../../../raw/current/scripts/prefabs/soil_amender.lua#L1-L7)、[percolate](../../../raw/current/scripts/prefabs/soil_amender.lua#L18-L39) |

## 典型接入

### 修改发酵时长

- 前提与选择：服务端 `modmain.lua` 用 `AddPrefabPostInit`。发酵时长在构造期经 `SetPerishTime` 写入，运行期只由 perishable 自身推进，改 `TUNING.PERISH_SUPERFAST` 会波及所有快腐物品，故选 postinit 后重设本实例。
- 生效与撤销：存档重建先 `SpawnPrefab` 跑 postinit（[SpawnSaveRecord](../../../raw/current/scripts/mainfunctions.lua#L464-L503)、[prefab postinit 链](../../../raw/current/scripts/mainfunctions.lua#L394-L411)），随后 `SetPersistData` 逐组件调 `OnLoad`（[entityscript.lua#L2005-L2007](../../../raw/current/scripts/entityscript.lua#L2005-L2007)），`Perishable:OnLoad` 把存档的剩余秒数与暂停态覆盖回去（[perishable.lua#L337-L348](../../../raw/current/scripts/components/perishable.lua#L337-L348)）。因此新做的实例按新上限满值起算；存量存档实例则是"上限取 postinit 新值、剩余秒数取存档值"，进度百分比不按新时长缩放——既不会清回起子态，也不保证保比例。在 postinit 当场改用 `SetNewMaxPerishTime` 同样解决不了顺序：它按当时 `GetPercent` 换算，随后仍被 OnLoad 覆盖剩余值（[SetNewMaxPerishTime](../../../raw/current/scripts/components/perishable.lua#L204-L208)）。要严格保比例，须等组件读回完成后再按旧上限换算剩余值（如延迟一帧或挂在读档完成时机，此处只指方向不展开）。撤销即移除模组。
- 原版依据：perishable 只在标签跨档时推 `forceperishchange`，改总时长不触发肥力档立即重算，跨过阈值才按新时长节奏切档（[标签驱动](../../../raw/current/scripts/components/perishable.lua#L1-L27)）。

```lua
-- modmain.lua：发酵总时长改为 2 游戏日（原版 3 日）；肥力档数值不变
AddPrefabPostInit("soil_amender", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetPerishTime(GLOBAL.TUNING.TOTAL_DAY_TIME * 2)
    end
end)
```

- 调整与验证：`TUNING.TOTAL_DAY_TIME`＝480（秒），为运行值口径；示例对新做物品按 960 秒满值生效，存量存档实例的剩余秒数仍为存档值（见"生效与撤销"的读档顺序）；替换物（`onperishreplacement`）与肥力档可在同 postinit 里改 `perishable.onperishreplacement` 或经 `update_fertilizer` 同款写法重挂 `forceperishchange` 监听，替换行为见 [Perish](../../../raw/current/scripts/components/perishable.lua#L293-L318)。

## 常见陷阱与限制

- **边界：三态名字里的"已发酵"不是超级催长剂**：`soil_amender` 的 spoiled 态显示"已发酵催长剂"（`SOIL_AMENDER_SPOILED`），仍是可继续用的起子（高档肥、单次施用）；只有 perishable 走完 3 日替换出的 `soil_amender_fermented` 才是"超级催长剂"（5 次施用）。两键语义易混。[.po](../../../raw/current/scripts/languages/chinese_s.po#L349714-L349737)
- **边界：施用返还的是空瓶子不是起子**：`bottlereturnfn` 只在最后一次施用（`final_use`）返还 `messagebottleempty`；起子单次施用即耗尽并返还，超级催长剂第 5 次才返还，前 4 次物品留在栏内继续可换格／掉落。[返还](../../../raw/current/scripts/prefabs/soil_amender.lua#L41-L53)、[OnApplied](../../../raw/current/scripts/components/fertilizer.lua#L30-L48)
- **边界：发酵替换的落点由收纳规则决定**：`Perishable:Perish` 在旧实例坐标生成替换物，库存态把旧槽传给对应库存或容器的 `GiveItem` 尝试回填，能否落回原槽由收纳规则决定；两态都不可堆叠，不存在跨堆叠转移数量。[替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)
- **边界：部署点限可耕土壤**：`fertilizer_candeploy` 只认 `IsFarmableSoilAtPoint`，placer 为 `gridplacer_farmablesoil`；对非土壤目标（未成熟作物、贫瘠旧农田、贫瘠采集物、自肥者）走 FERTILIZE 动作路径而非部署路径，两条路径消费同一 fertilizer 组件。[candeploy](../../../raw/current/scripts/standardcomponents.lua#L1812-L1821)、[FERTILIZE](../../../raw/current/scripts/actions.lua#L1812-L1842)、[收集器](../../../raw/current/scripts/componentactions.lua#L1241-L1254)
- **边界：三档肥力只在跨档时重写**：`update_fertilizer` 由 `forceperishchange` 驱动，运行中改 `TUNING.SOILAMENDER_*` 不会立刻影响已定档实例，需等下次跨档（或自己推事件）。[update_fertilizer](../../../raw/current/scripts/prefabs/soil_amender.lua#L81-L114)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394（`raw/current/BUILD.txt`）。核验 `prefabs/soil_amender.lua` 全文（两注册工厂、三态 `update_fertilizer`、`percolate`、`bottlereturnfn`、`onreplacedfn`、净变量与标签接线、无 stackable）；`fertilizer_nutrient_defs.lua`（四键养分／图标／名称／排序）；`perishable.lua`（标签驱动、阈值、SetPerishTime、Perish 替换链、Update 倍率）；`fertilizer.lua`（OnApplied 消耗与移除）；`finiteuses.lua`（Use／OnUsedAsItem／onfinished）；`standardcomponents.lua` 的 `fertilizer_ondeploy`／candeploy／`MakeDeployableFertilizer*`；`farming_manager.lua` 的 `AddTileNutrients`；FERTILIZE 动作与 crop／grower／pickable 三条旧路径消费点；`inventoryitem.lua` 的 `ChangeImageName`；`itemtile.lua` 腐坏条判定；recipes.lua 配方与 GARDENING 筛选；tuning 各键及 `day_time`／`total_day_time`／`perish_warp` 换算；chinese_s.po 五个名称键与空瓶子键定向核验；plant_normal／farmplot 确认为 crop／grower 组件宿主。
- 游戏运行：未运行。未覆盖：三态动画与音效实际表现、发酵在冰箱／冬季的实际速率、部署网格交互与施肥实际耗时、`fertilizervalue` 等旧路径对遗留农田实体的现网生效范围；37.5／150／450 秒与 1440 秒为源码推导值。
- 结构检查：validate_page.py（standard 档）与 validate_source_links.py 实际执行通过（见交付说明）。
- 修订：2026-11-08（独立检视意见，主代理回源码复核采纳）：删除正文 I07 内部台账编号引用，改为"整文件无 `AddComponent("stackable")`，已全文检索"的源码表述；全文检索依据已记于上文源码审阅条。
- 外部复检修订：2026-11-08（docs/reports/w05/prefab-inventory-review-b01-20261108，主代理逐项回源码核实采纳）：典型接入"生效与撤销"按读档顺序重写——SpawnSaveRecord 先 SpawnPrefab 跑 postinit、随后 SetPersistData 逐组件 OnLoad 把存档剩余秒数与暂停态覆盖回 perishable（mainfunctions.lua:394-503、entityscript.lua:2002-2008、perishable.lua:337-348），存量实例为"上限取 postinit 新值、剩余秒数取存档值"，既不清回起子态也不保比例，`SetNewMaxPerishTime` 同样被 OnLoad 覆盖；"只经部署施肥"扩展为部署＋FERTILIZE 动作双路径（补收集器 componentactions.lua:1241-1254 与自肥者沃姆伍德 wormwood.lua:845-846）；还瓶表述改为"优先交给有物品栏组件的使用者，否则在目标或物品位置生成"；"原位替换／优先回填原持有格"统一改为"尝试回填、落点由收纳规则决定"；腐坏条归属改准为仅 soil_amender 起子（fermented 无 perishable、栏内显示耐久）；frontmatter sources 补 prefabs/wormwood.lua、componentactions.lua、mainfunctions.lua、entityscript.lua。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
