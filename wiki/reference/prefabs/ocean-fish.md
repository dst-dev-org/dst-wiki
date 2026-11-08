---
title: "海鱼：小型与中型鱼群及物品态（oceanfish_small_1～9、oceanfish_medium_1～9）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/oceanfish.lua 的 18 个世界注册与 18 个 _inv 物品注册：water_common／inv_common 两工厂共用 oceanfishdef 数据表；鱼群按季节与地皮加权、herd 与 shoalspawner 两条宿主；上钩后的 partiallyhooked、体力与挣扎；钓起出水投射与落点水陆分流；物品态 1 天腐坏、宰杀两套掉落表（≥70% 重量整体换重鱼表）、烤制与温度／光照／幸运／灭火等特殊件。"
aliases: ["oceanfish_small_1", "oceanfish_medium_1", "oceanfish_small_2", "oceanfish_medium_2", "oceanfish_small_3", "oceanfish_medium_3", "oceanfish_small_4", "oceanfish_medium_4", "oceanfish_small_5", "oceanfish_medium_5", "oceanfish_small_6", "oceanfish_medium_6", "oceanfish_small_7", "oceanfish_medium_7", "oceanfish_small_8", "oceanfish_medium_8", "oceanfish_small_9", "oceanfish_medium_9", "oceanfish_small_1_inv", "oceanfish_medium_1_inv", "oceanfish_small_2_inv", "oceanfish_medium_2_inv", "oceanfish_small_3_inv", "oceanfish_medium_3_inv", "oceanfish_small_4_inv", "oceanfish_medium_4_inv", "oceanfish_small_5_inv", "oceanfish_medium_5_inv", "oceanfish_small_6_inv", "oceanfish_medium_6_inv", "oceanfish_small_7_inv", "oceanfish_medium_7_inv", "oceanfish_small_8_inv", "oceanfish_medium_8_inv", "oceanfish_small_9_inv", "oceanfish_medium_9_inv", "小孔雀鱼", "针鼻喷墨鱼", "小饵鱼", "三文鱼苗", "爆米花鱼", "落叶比目鱼", "花朵金枪鱼", "炽热太阳鱼", "口水鱼", "泥鱼", "斑鱼", "浮夸狮子鱼", "黑鲶鱼", "玉米鳕鱼", "花锦鲤", "金锦鲤", "冰鲷鱼", "甜味鱼", "海鱼", "海洋鱼", "海钓鱼", "鱼群"]
tags: [prefab, creature, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/oceanfish.lua", "prefabs/oceanfishdef.lua", "prefabs/oceanfish_shoalspawner.lua", "prefabs/schoolherd.lua", "components/schoolspawner.lua", "brains/oceanfishbrain.lua", "stategraphs/SGoceanfish.lua", "components/oceanfishable.lua", "components/oceanfishingrod.lua", "components/oceanfishinghook.lua", "prefabs/oceanfishingbobber.lua", "prefabs/oceanfishingrod.lua", "prefabs/oceanfishinglure.lua", "components/weighable.lua", "components/lootdropper.lua", "components/perishable.lua", "components/preserver.lua", "components/cookable.lua", "components/edible.lua", "components/eater.lua", "components/tradable.lua", "components/heater.lua", "components/firedetector.lua", "components/wateryprotection.lua", "components/propagator.lua", "components/luckitem.lua", "components/murderable.lua", "components/childspawner.lua", "components/complexprojectile.lua", "components/malbatrossspawner.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "containers.lua", "prefabs/fish_box.lua", "prefabs/waterplant.lua", "prefabs/chum.lua", "prefabs/books.lua", "prefabs/waterprojectiles.lua", "prefabs/forest.lua", "map/rooms/forest/terrain_ocean.lua", "map/forest_map.lua", "tiledefs.lua", "tilemanager.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 海鱼：小型与中型鱼群及物品态（oceanfish_small_1～9、oceanfish_medium_1～9）

## 快速判断

- 职责与适用：prefabs/oceanfish.lua 用一个循环把 oceanfishdef.lua 的 18 条鱼定义各注册两次——世界态 oceanfish_small_1～9／oceanfish_medium_1～9（游在海里、被海钓）与物品态同名加 `_inv`（进背包、可宰杀／腐坏／烤）。两种形态共用同一份 `FISH_DEFS` 表，字段含义只在“谁读它”上分叉。[注册循环](../../../raw/current/scripts/prefabs/oceanfish.lua#L577-L605)、[18 条定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L184-L1021)
- 优先使用：查某条鱼掉什么、烤成什么、吃多少看[谱系与产出](#18-种谱系与产出)；查“为什么钓上来是熟的”“为什么会有金块”看[掉落、烤制与食用](#掉落烤制与食用)；查上钩到出水的完整顺序看[世界态生命周期](#世界态生命周期)与[出水与投射](#出水与投射)；只关心背包形态看[物品态](#物品态)。
- 重要副作用：出水成功会**换实体**——世界鱼在陆地落点被删除、原地生成 `_inv` 并由 `CopyWeighable` 搬走重量（S4／S5）；掉回水面则不换实体，鱼留在水里并标记 `leaving`。[落点分流](../../../raw/current/scripts/prefabs/oceanfish.lua#L81-L118)
- 修改边界（S2／S8）：只覆盖海钓链这 36 个注册与鱼侧参数——竿与张力、`oceanfishable` 接口、鱼群选型、鱼肉数值各归其它页，逐项见[关联机制导航](#关联机制导航)；池塘钓竿＋`fishable` 的 `pondfish`／`pondeel` 不是这里的注册。
- 范围说明（`complex` 档）：本页整体承载 18 种×两态共 36 个注册的对照，并为了回答“为什么是这一条鱼”把鱼群选型、上钩、出水、物品态四段一起写完，拆开会失去“共享数据表”这条主线。

## 注册与构造

两个工厂都由 `MakeFish(data)` 成对注册：`Prefab(data.prefab, water_common(data))` 与 `Prefab(data.prefab.."_inv", inv_common(data))`；`_inv` 那条在 `MakeFish` 里没有 assets／prefabs 依赖表，资源由世界态那条带齐（bank、build、`extra_anim_assets`）。[成对注册](../../../raw/current/scripts/prefabs/oceanfish.lua#L577-L601)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oceanfish_small_1`～`9`、`oceanfish_medium_1`～`9`（18 个） | `water_common(data)`：实体→可选 Light→Physics→标签与动画→`SetPristine`→非主模拟返回→主模拟挂组件→SG／Brain | 六面朝向、`COLLISION.CHARACTERS`＋游泳碰撞掩码、水下排序层；主模拟挂 `locomotor`／`oceanfishable`／`eater`／`knownlocations`／`timer`／`herdmember`／`weighable`，`firesuppressant` 的鱼再加 `firedetector`＋`wateryprotection`。[water_common](../../../raw/current/scripts/prefabs/oceanfish.lua#L275-L407) |
| `oceanfish_small_1_inv`～`9_inv`、`oceanfish_medium_1_inv`～`9_inv`（18 个） | `inv_common(fish_def)`：实体→可选 Light／DynamicShadow→`MakeInventoryPhysics`→动画（`flop_pst`）→改名→标签→`SetPristine`→非主模拟返回→主模拟挂物品组件 | 两面朝向、播 `flop_pst`；用 `SetPrefabNameOverride(fish_def.prefab)` 直接继承世界态名字，所以 18 个 `_inv` 显示的都是鱼名而不是 `_inv` 名。[inv_common](../../../raw/current/scripts/prefabs/oceanfish.lua#L415-L573)、[改名](../../../raw/current/scripts/prefabs/oceanfish.lua#L449-L449) |
| `oceanfish_shoalspawner`（关联宿主，不单开页） | 独立文件：`childspawner` 的 `childname = "oceanfish_medium_2"`、`wateronly = true` | 只产中型 2 号“斑鱼”；睡眠时停释放任务、醒来 0.1 秒后 `ReleaseAllChildren`；带 `oceanshoalspawner` 标签与小地图图标，`OnInit` 推 `ms_registerfishshoal` 供邪天翁刷新器收集。[宿主](../../../raw/current/scripts/prefabs/oceanfish_shoalspawner.lua#L10-L75) |
| `schoolherd_<鱼名>`（关联宿主，18 个） | 独立文件按 `FISH_DATA.fish` 逐个注册，`herd:SetMemberTag("herd_"..prefab)` | 鱼群内部机制：6 个导航点、`herd_offset` 散布、`lifetime` 到期后向全体推 `doleave`；成员全部休眠则整群删除。选型与摆位细节见 [[reference/components/schoolspawner]]。[schoolherd](../../../raw/current/scripts/prefabs/schoolherd.lua#L126-L178) |

两条工厂的共用点与分界（按执行顺序）：

- **标签分层（S3）**。世界态在 `SetPristine` 前打 `ignorewalkableplatforms`／`notarget`／`NOCLICK`／`NOBLOCK`／`oceanfishable`／`oceanfishable_creature`／`oceanfishinghookable`／`oceanfish`／`swimming`／`herd_<prefab>`，有 `fishtype` 时再加 `ediblefish_<fishtype>`；物品态打 `weighable_fish`／`fish`／`oceanfish`／`catfood`／`smallcreature`／`smalloceancreature`，有 `heater` 时补 `HASHEATER`、有 `luckitem` 时按正负补 `luckyitem`／`unluckyitem`。[世界态标签](../../../raw/current/scripts/prefabs/oceanfish.lua#L303-L320)、[物品态标签](../../../raw/current/scripts/prefabs/oceanfish.lua#L452-L472)
- **物理与渲染**。世界态质量 1、摩擦 0、阻尼 5、胶囊 0.5×1，`SetSixFaced`，排序层 `LAYER_WIP_BELOW_OCEAN`／`UNDERWATER`；物品态走 `MakeInventoryPhysics`、`SetTwoFaced`，不设水下层。[物理](../../../raw/current/scripts/prefabs/oceanfish.lua#L292-L301)、[层](../../../raw/current/scripts/prefabs/oceanfish.lua#L328-L329)
- **主模拟组件清单**。世界态只挂“行为用”的件：`locomotor`（`allowocean`／`ignoreLand`、走跑速取鱼数据）、`oceanfishable`（三个钩子＋`StrugglingSetup`＋`catch_distance`）、`eater`（按 `diet` 设可食与偏好）、`knownlocations`、`timer`、`herdmember`（`Enable(false)`、`herdprefab = "schoolherd_"..prefab`）、`weighable`（只 `Initialize`＋`SetWeight`，**不设** `type`，注释写明重量只为拷给物品）。[世界态主模拟](../../../raw/current/scripts/prefabs/oceanfish.lua#L336-L376)
- **世界态没有战斗与掉落**。两条工厂里世界态都不加 `health`／`combat`／`lootdropper`：鱼不会被杀死，`fish_def.loot`／`heavy_loot` 只在物品态被消费。[世界态组件](../../../raw/current/scripts/prefabs/oceanfish.lua#L340-L398)、[物品态掉落](../../../raw/current/scripts/prefabs/oceanfish.lua#L498-L503)
- **`_inv` 没有 `stackable`**。`inv_common` 全文没有 `AddComponent("stackable")`，所以一格一条、无法叠堆；它也没有 `health`，被宰杀走 `murderable`＋`lootdropper`。[物品态组件](../../../raw/current/scripts/prefabs/oceanfish.lua#L485-L536)
- **共享数据表**。`fish_def` 由 `MakeFish(data)` 原样传进两个工厂，两个工厂都存成 `inst.fish_def`；`_inv` 的掉落、烤制、腐坏、食用三维全读同一份定义，所谓“两套表”是同一张 `FISH_DEFS` 的不同字段被不同工厂读取。[数据入口](../../../raw/current/scripts/prefabs/oceanfish.lua#L336-L336)、[物品侧](../../../raw/current/scripts/prefabs/oceanfish.lua#L483-L483)

### 名称核验

（S7）18 条简中名与注册名一一对应，`STRINGS.NAMES` 里基础名与 `_INV` 名各有一份且文本相同；因为 `_inv` 走 `SetPrefabNameOverride(基础名)`，实际显示取的是基础名那份。九条小型：小孔雀鱼、针鼻喷墨鱼、小饵鱼、三文鱼苗、爆米花鱼、落叶比目鱼、花朵金枪鱼、炽热太阳鱼、口水鱼；九条中型：泥鱼、斑鱼、浮夸狮子鱼、黑鲶鱼、玉米鳕鱼、花锦鲤、金锦鲤、冰鲷鱼、甜味鱼。[小型名](../../../raw/current/scripts/languages/chinese_s.po#L346469-L346557)、[中型名](../../../raw/current/scripts/languages/chinese_s.po#L346379-L346467)、[显示名解析](../../../raw/current/scripts/entityscript.lua#L687-L692)

## 18 种谱系与产出

oceanfishdef.lua 先把档位换算成常量再由每条鱼引用。读表前先记这几组：群规模 `TINY` 1–3／`SMALL` 2–5／`MEDIUM` 4–6／`LARGE` 6–10；散布半径 `TINY` 2／`SMALL` 3／`MEDIUM` 6／`LARGE` 10；群体寿命 `SHORT` 240–480 秒（8–16 个 `seg_time`）／`MEDIUM` 480–960 秒（1–2 天，本文件的 `DAY` 硬写为 16×30 秒）。[档位表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L65-L104)

### 出现地皮与季节权重

四张季节表同构：先建秋季基础表，再 `deepcopy` 给冬／春／夏，最后按季节往个别地皮追加一条。[季节表构造](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1030-L1084) 权重档为 `VERY_COMMON` 4／`COMMON` 2／`UNCOMMON` 1／`RARE` 0.25。

| 地皮 | 秋季基础（鱼→权重） | 其它季节追加 |
|---|---|---|
| `OCEAN_COASTAL` | small_1 1、small_2 2、small_3 0.25、small_4 4、small_5 2、medium_1 1 | 春：small_7 1 |
| `OCEAN_SWELL` | small_1 1、small_3 0.25、medium_1 2、medium_2 1、medium_4 1、medium_5 2 | 秋：small_6 1；冬：medium_8 1；夏：small_8 1 |
| `OCEAN_ROUGH` | small_3 0.25、medium_2 2、medium_3 1、medium_4 2、medium_5 1 | — |
| `OCEAN_BRINEPOOL` | （空表） | — |
| `OCEAN_HAZARDOUS` | medium_3 4 | — |
| `OCEAN_WATERLOG` | small_2 1、small_4 0.25、medium_9 4 | 秋：small_6 2；春：small_7 2 |

追加行写的是 `GROUND.OCEAN_*`，与基础表的 `WORLD_TILES.OCEAN_*` 是同一批编号（原版海洋地皮在 `GROUND` 与 `WORLD_TILES` 里同值）：[常量](../../../raw/current/scripts/constants.lua#L753-L763)、[注册沿用 id](../../../raw/current/scripts/tilemanager.lua#L128-L157)、[海洋地皮](../../../raw/current/scripts/tiledefs.lua#L152-L155)。所以追加落进的是同一条地皮。选中逻辑是“当前季节表→按 `GetTileAtPoint` 取该格→加权抽一条”，见 [[reference/components/schoolspawner]]。**`OCEAN_BRINEPOOL` 是空表，鱼群不会在此类水面自然生成**。口水鱼（small_9）**不在任何季节表里**，鱼书与诱鱼剂都走 `SpawnSchool`→`PickSchool`，同样刷不出它；它唯一的主动来源是海草 `waterplant` 的 `childspawner`（`childname = "oceanfish_small_9"`，上限 1 条、半径 4.5、240 秒再生、白花 120 秒），花粉落地时 `ReleaseAllChildren` 放出；另两条只是“物品变回世界鱼”：`_inv` 落水、砸开鱼箱（最多 5 条）。[鱼书](../../../raw/current/scripts/prefabs/books.lua#L558-L583)、[诱鱼剂](../../../raw/current/scripts/prefabs/chum.lua#L19-L33)、[海草](../../../raw/current/scripts/prefabs/waterplant.lua#L13-L13)、[鱼群入口](../../../raw/current/scripts/prefabs/waterplant.lua#L469-L476)、[花粉放出](../../../raw/current/scripts/prefabs/waterplant.lua#L495-L497)、[砸箱](../../../raw/current/scripts/prefabs/fish_box.lua#L56-L85)、[回水](../../../raw/current/scripts/prefabs/oceanfish.lua#L62-L66)

### 谱系与捕获参数

“饵偏好”是该鱼在 `TUNING.OCEANFISH_LURE_PREFERENCE` 里的乘数档，值为 0 的风格对它**完全无效**（如素食鱼的 `spoon`＝0）；“食性”是 `eater:SetDiet` 的可食组，决定它主动去吃哪类东西。[偏好表](../../../raw/current/scripts/tuning.lua#L322-L332)、[应用点](../../../raw/current/scripts/components/oceanfishinghook.lua#L31-L46)、[设食性](../../../raw/current/scripts/prefabs/oceanfish.lua#L354-L360)

| 简中名（注册名） | 重量 | 走／跑 | 群档／散布 | 群寿命 | 饵偏好 | 食性 |
|---|---|---|---|---|---|---|
| 小孔雀鱼（small_1） | 48.34–60.30 | 0.8／2.5 | LARGE（6–10）／2 | MEDIUM | SMALL_OMNI | OMNI |
| 针鼻喷墨鱼（small_2） | 37.54–57.62 | 1.5／3.0 | MEDIUM（4–6）／3 | MEDIUM | SMALL_OMNI | OMNI |
| 小饵鱼（small_3） | 39.66–63.58 | 1.2／2.5 | MEDIUM／3 | MEDIUM | SMALL_MEAT | MEAT |
| 三文鱼苗（small_4） | 39.70–56.26 | 1.2／2.0 | MEDIUM／3 | MEDIUM | SMALL_OMNI | OMNI |
| 爆米花鱼（small_5） | 33.08–47.74 | 1.0／2.0 | MEDIUM／3 | MEDIUM | SMALL_VEGGIE | VEGGIE |
| 落叶比目鱼（small_6） | 28.87–44.44 | 1.5／2.0 | MEDIUM／3 | MEDIUM | SMALL_VEGGIE | VEGGIE |
| 花朵金枪鱼（small_7） | 53.64–63.36 | 0.8／2.5 | MEDIUM／3 | MEDIUM | SMALL_VEGGIE | VEGGIE |
| 炽热太阳鱼（small_8） | 41.14–56.78 | 1.5／3.0 | MEDIUM／3 | MEDIUM | SMALL_MEAT | MEAT |
| 口水鱼（small_9） | 31.15–49.20 | 1.0／2.0 | TINY（1–3）／2 | SHORT | SMALL_VEGGIE | VEGGIE |
| 泥鱼（medium_1） | 154.32–214.97 | 1.2／3.0 | MEDIUM／3 | MEDIUM | OMNI | OMNI |
| 斑鱼（medium_2） | 172.41–228.88 | 2.5／2.5 | SMALL（2–5）／6 | MEDIUM | MEAT | MEAT |
| 浮夸狮子鱼（medium_3） | 246.77–302.32 | 2.2／3.5 | MEDIUM／3 | MEDIUM | MEAT | MEAT |
| 黑鲶鱼（medium_4） | 193.27–278.50 | 1.4／2.5 | MEDIUM／3 | MEDIUM | OMNI | OMNI |
| 玉米鳕鱼（medium_5） | 161.48–241.80 | 1.3／2.8 | MEDIUM／3 | MEDIUM | VEGGIE | VEGGIE |
| 花锦鲤（medium_6） | 188.88–238.88 | 1.2／3.0 | MEDIUM／3 | MEDIUM | OMNI | OMNI |
| 金锦鲤（medium_7） | 188.88–238.88 | 1.2／3.0 | MEDIUM／3 | MEDIUM | OMNI | OMNI |
| 冰鲷鱼（medium_8） | 190.90–270.70 | 1.5／3.0 | MEDIUM／3 | MEDIUM | OMNI | OMNI |
| 甜味鱼（medium_9） | 210.50–315.14 | 1.2／3.0 | SMALL（2–5）／3 | MEDIUM | BERRY | BERRY |

上表逐条出自 [小型定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L188-L602) 与 [中型定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L604-L1021)。`walkspeed`／`runspeed` 既写进 `locomotor`，又作为 [`oceanfishable:StrugglingSetup`](../../../raw/current/scripts/components/oceanfishable.lua#L56-L65) 的速度上限：与竿夹角超过 90° 时按当前张力减速（最多减 0.8 倍），朝着竿时按张力的一半加速，见 [UpdateRunSpeed](../../../raw/current/scripts/components/oceanfishable.lua#L105-L118)。

### 注册名对照（世界态／物品态）

36 个注册由同一条鱼定义成对生成（`MakeFish`），下表供按任一注册名检索；`schoolherd_<鱼名>` 与 `oceanfish_shoalspawner` 不在这 36 个之内（见[注册与构造](#注册与构造)）。

| 简中名 | 世界态注册名 | 物品态注册名 |
|---|---|---|
| 小孔雀鱼 | `oceanfish_small_1` | `oceanfish_small_1_inv` |
| 针鼻喷墨鱼 | `oceanfish_small_2` | `oceanfish_small_2_inv` |
| 小饵鱼 | `oceanfish_small_3` | `oceanfish_small_3_inv` |
| 三文鱼苗 | `oceanfish_small_4` | `oceanfish_small_4_inv` |
| 爆米花鱼 | `oceanfish_small_5` | `oceanfish_small_5_inv` |
| 落叶比目鱼 | `oceanfish_small_6` | `oceanfish_small_6_inv` |
| 花朵金枪鱼 | `oceanfish_small_7` | `oceanfish_small_7_inv` |
| 炽热太阳鱼 | `oceanfish_small_8` | `oceanfish_small_8_inv` |
| 口水鱼 | `oceanfish_small_9` | `oceanfish_small_9_inv` |
| 泥鱼 | `oceanfish_medium_1` | `oceanfish_medium_1_inv` |
| 斑鱼 | `oceanfish_medium_2` | `oceanfish_medium_2_inv` |
| 浮夸狮子鱼 | `oceanfish_medium_3` | `oceanfish_medium_3_inv` |
| 黑鲶鱼 | `oceanfish_medium_4` | `oceanfish_medium_4_inv` |
| 玉米鳕鱼 | `oceanfish_medium_5` | `oceanfish_medium_5_inv` |
| 花锦鲤 | `oceanfish_medium_6` | `oceanfish_medium_6_inv` |
| 金锦鲤 | `oceanfish_medium_7` | `oceanfish_medium_7_inv` |
| 冰鲷鱼 | `oceanfish_medium_8` | `oceanfish_medium_8_inv` |
| 甜味鱼 | `oceanfish_medium_9` | `oceanfish_medium_9_inv` |

### 掉落、烤制与食用

四张表分别是 `loot`（普通）、`heavy_loot`（重鱼）、`cooking_product`（上火烤）、`perish_product`（腐坏替换）；`loot`／`heavy_loot` 是多件固定列表而不是概率表，**列出的每件都会掉**。[四张表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L106-L147)

| 简中名 | 普通掉落 | 重鱼掉落（≥70% 重量） | 烤制产物 | 腐坏产物 | 食用（饥／生／智） | FOODTYPE |
|---|---|---|---|---|---|---|
| 小孔雀鱼 | fishmeat_small | fishmeat | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 针鼻喷墨鱼 | fishmeat_small | fishmeat | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 小饵鱼 | fishmeat_small | fishmeat | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 三文鱼苗 | fishmeat_small | fishmeat | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 爆米花鱼 | corn_cooked | — | corn_cooked | corn_cooked | 12.5／3／0 | VEGGIE |
| 落叶比目鱼 | plantmeat | — | plantmeat_cooked | spoiled_food | 12.5／0／−10 | MEAT |
| 花朵金枪鱼 | plantmeat | — | plantmeat_cooked | spoiled_food | 12.5／0／−10 | MEAT |
| 炽热太阳鱼 | fishmeat_small_cooked | fishmeat_cooked | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 口水鱼 | fishmeat_small | fishmeat | fishmeat_small_cooked | fishmeat_small | 12.5／1／0 | MEAT |
| 泥鱼 | fishmeat | fishmeat＋fishmeat_small | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 斑鱼 | fishmeat | fishmeat＋fishmeat_small | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 浮夸狮子鱼 | fishmeat | fishmeat＋fishmeat_small | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 黑鲶鱼 | fishmeat | fishmeat＋fishmeat_small | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 玉米鳕鱼 | corn | — | corn_cooked | corn | 25／3／0 | VEGGIE |
| 花锦鲤 | fishmeat（活动期＋金块×2） | fishmeat＋fishmeat_small（活动期＋金块×3） | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 金锦鲤 | 同上 | 同上 | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 冰鲷鱼 | fishmeat＋ice×2 | fishmeat＋fishmeat_small＋ice×2 | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |
| 甜味鱼 | fishmeat | fishmeat＋fishmeat_small | fishmeat_cooked | fishmeat | 25／8／0 | MEAT |

三维取自 `EDIBLE_VALUES_*` 五档：小型肉 12.5／1／0、小型素 12.5／3／0、叶肉 12.5／0／−10、中型肉 25／8／0、中型素 25／3／0；对应常量 `HEALING_TINY` 1／`HEALING_SMALL` 3／`HEALING_MEDSMALL` 8、`CALORIES_SMALL` 12.5／`CALORIES_MED` 25、`SANITY_SMALL` 10。[五档定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L168-L172)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2061)。锅食材值另算：小型 `meat 0.5／fish 0.5`、中型 `meat 1／fish 1`、冰鲷另加 `frozen 1`，两条素食鱼写 `veggie 1`。[食材档](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L163-L166)

`FOODTYPE` 只有两条素食鱼（爆米花鱼、玉米鳕鱼）是 `VEGGIE`，其余都是 `MEAT`；`foodtype == MEAT` 时物品态还会补 `edible.ismeat = true`。注意 `fishtype` 只决定 `ediblefish_<fishtype>` 标签，与 `foodtype` 不是同一件事：落叶比目鱼／花朵金枪鱼的 `fishtype` 是 `veggie`、被吃时却按 `MEAT` 结算。[赋值](../../../raw/current/scripts/prefabs/oceanfish.lua#L505-L520)、[标签](../../../raw/current/scripts/prefabs/oceanfish.lua#L313-L315)

### 特殊件

- **炽热太阳鱼（small_8）是行走的热源与灯**：`heater = { carriedheat = 70, heat = 70, carriedheatmultiplier = 2 }`、`propagator = { propagaterange = 3, heatoutput = 2 }`、`light = { r = 0.6, i = 0.5, f = 1, c = 235/255,165/255,12/255 }`；small_8 与 small_9 是仅有的两条没有 `dynamic_shadow` 的鱼。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L507-L553)、[挂载](../../../raw/current/scripts/prefabs/oceanfish.lua#L538-L559)
- **冰鲷鱼（medium_8）方向相反**：`heater = { endothermic = true, carriedheat = -5, heat = -5, carriedheatmultiplier = 2 }`，构造时 `SetThermics(false, true)` 把组件标成吸热而非放热；它没有 `light` 也没有 `propagator`。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L928-L974)、[吸热分支](../../../raw/current/scripts/prefabs/oceanfish.lua#L546-L548)、[组件语义](../../../raw/current/scripts/components/heater.lua#L24-L35)
- **口水鱼（small_9）是移动灭火器**：`firesuppressant = true` 让世界态加 `firedetector`（`range` 7、`detectPeriod` 4、`fireOnly`）与 `wateryprotection`，并把 `inst.LaunchProjectile` 指向 `launch_water_projectile`（生成 `waterstreak_projectile`）；定义里还带 `extra_anim_assets = {"oceanfish_small_sprinkler"}`、`extra_prefabs = {"waterstreak_projectile"}`。[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L378-L395)、[水弹](../../../raw/current/scripts/prefabs/oceanfish.lua#L215-L232)、[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L555-L602)
- **黑鲶鱼（medium_4）扣幸运**：`luckitem = { luck = TUNING.BLACKCATFISH_LUCK }` 即 −0.25，物品态按正负打 `unluckyitem` 并 `SetLuck(−0.25)`；它不是装备，走 `GetLuck` 分支，所以**放在背包里就对持有者生效**。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L788-L788)、[常量](../../../raw/current/scripts/tuning.lua#L9305-L9305)、[生效路径](../../../raw/current/scripts/components/luckitem.lua#L39-L53)
- **花锦鲤／金锦鲤（medium_6／7）的 YOT 改写**：活动期由 `SpecialEventSetup()` 把两者的 `loot`／`heavy_loot` 换成 `MEDIUM_YOT`（普通表＝`fishmeat`＋2 块 `lucky_goldnugget`；重鱼表＝`fishmeat`＋`fishmeat_small`＋3 块）、`oncooked_fn` 设为烤制时额外掉 2 块金块，并以 0.5 的权重**新增**到近岸／中浪／汹涌三格（基线表里本来没有这两条，不是把已有权重压低）。该函数在世界 `specialeventsetup:OnPostInit` 里调一次，改的是模块级共享表；没有 else 分支回滚，活动结束后同一进程内再调一次也不会还原，只能重启进程／重载模块。[改写](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1087-L1106)、[调用](../../../raw/current/scripts/components/specialeventsetup.lua#L303-L304)、[烤制回调](../../../raw/current/scripts/prefabs/oceanfish.lua#L527-L531)

## 世界态生命周期

### 群体：选型、摆位与游荡

世界态的“群”由三样东西拼起来：`schoolspawner`（世界组件）决定何时何地选哪条鱼，`schoolherd_<鱼名>`（非联网实体）持有导航点与群体寿命，鱼自己挂 `herdmember` 并记住 `herd_offset`。选型、海域比例、阻挡与逐鱼落点全部在 [[reference/components/schoolspawner]]，此处只列与鱼侧字段有关的接口：`schoolmin`／`schoolmax` 是群大小、`schoolrange` 是散布半径、`schoollifetimemin`／`max` 是群体寿命、`herdwandermin`／`max` 与 `herdwanderdelaymin`／`max` 决定游荡步长与停顿、`herdarrivedist` 决定“算到点”的距离。[消费者](../../../raw/current/scripts/components/schoolspawner.lua#L206-L239)、[群参数](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L79-L104)

`schoolherd` 一侧每次更新都会重算 `herd_offset`（半径 8 的随机点），鱼 brain 的游荡目标就读这个点（没有则退到 `home`）；到点后按 `herdwanderdelay` 停顿再换下一个导航点，导航点用 `FindSwimmableOffset` 取可游泳位置，第 4 个点固定回原点。[导航与偏移](../../../raw/current/scripts/prefabs/schoolherd.lua#L20-L83)、[游荡目标](../../../raw/current/scripts/brains/oceanfishbrain.lua#L44-L56)。鱼群被选中时还会在中心放一个 `fishschoolspawnblocker`，默认一游戏日后消失，用来抑制附近继续自然刷群。

### 上钩、挣扎与破水

| 阶段 | 触发与条件 | 鱼侧结果与证据 |
|---|---|---|
| 咬钩 | 鱼在 `FindFoodAction` 里找到 `fishinghook`（要求 `TestInterest` 通过且在饵半径内）或拖网饵，走近后对钩发 `ACTIONS.EAT` | 双方都有 `oceanfishable` 时，`EAT.fn` 直接把钩子那根竿交给鱼：`doer.components.oceanfishable:SetRod(obj...:GetRod())`；旧目标（浮子）由 `SetTarget` 里的 `WasEatenByA` 移除。[EAT.fn](../../../raw/current/scripts/actions.lua#L799-L810)、[咬钩判定](../../../raw/current/scripts/brains/oceanfishbrain.lua#L118-L145)、[目标切换](../../../raw/current/scripts/components/oceanfishingrod.lua#L260-L301) |
| 半钩住 | `oceanfishable:SetRod(rod)` 触发 `onsetrodfn` | 加 `partiallyhooked` 与 `scarytooceanprey` 两个标签；brain 的“Hooked”节点里 `partiallyhooked` 分支只 `StandStill`，此时鱼不挣扎。[OnSetRod](../../../raw/current/scripts/prefabs/oceanfish.lua#L165-L173)、[站桩分支](../../../raw/current/scripts/brains/oceanfishbrain.lua#L192-L201) |
| 摘钩窗口 | 进入 `bitehook_loop` 时 `SetTimeout(base + random×var)` | `SET_HOOK_TIME_SHORT`＝1＋0～0.5 秒、`SET_HOOK_TIME_MEDIUM`＝2＋0～0.5 秒；超时且仍 `partiallyhooked` 就进 `bitehook_escape` 并 `StopFishing("linetooloose")`——**玩家不拉线，鱼会自己脱钩**。[档位](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L175-L176)、[状态](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L175-L209) |
| 拉紧 | 收线 `Reel()` 每次调 `OnReelingIn(fisher)` | 首次收线：移除 `partiallyhooked`、`ResetStruggling()`（体力归 1、重排挣扎计时）；若 `homeseeker.home` 是 `oceanfish_shoalspawner` 则推 `ms_shoalfishhooked_redux`，邪天翁刷新器据此以 0.3 概率把该浅滩排进下次刷新。[OnReelingIn](../../../raw/current/scripts/prefabs/oceanfish.lua#L151-L163)、[收线](../../../raw/current/scripts/components/oceanfishingrod.lua#L230-L258)、[邪天翁侧](../../../raw/current/scripts/components/malbatrossspawner.lua#L108-L116) |
| 挣扎／疲惫 | `stamina_def` 的 `drain_rate`／`recover_rate` 在张力高／低时改体力，体力再插值出下一次挣扎／疲惫时长 | 挣扎时鱼绕钓者快游，体力耗尽转“疲惫”慢速，两种状态反复翻转；`tiredout_angles` 决定疲惫时朝竿方向的偏角。[体力循环](../../../raw/current/scripts/components/oceanfishable.lua#L134-L176)、[brain 分支](../../../raw/current/scripts/brains/oceanfishbrain.lua#L196-L200) |
| 破水表现 | `breach`／`bitehook_escape`／`eat`／`shoot` 共用 `SpawnSplashFx` 与 `SetBreaching` | 出水时切两面朝向、排序层回 `LAYER_WORLD`、开灯；入水前恢复六面与水下层、关灯。FX 从 `fish_def.breach_fx` 随机取（小型 `ocean_splash_small1/2`、中型 `ocean_splash_med1/2`），压在船底时跳过。[FX 选择](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L38-L66)、[FX 表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L178-L182) |

### 结束：离群、休眠清除与消失

- **群体到期**：`schoolherd` 的 `lifetime`（`schoollifetimemin`～`max`，绝大多数鱼 1–2 个游戏日、口水鱼 240–480 秒）到点后向全体成员推 `doleave` 并删除自己；鱼收到后进 `leave` 状态播 `spawn_out` 并 `Remove`。[定时](../../../raw/current/scripts/prefabs/schoolherd.lua#L57-L83)、[到期](../../../raw/current/scripts/prefabs/schoolherd.lua#L117-L124)、[leave 状态](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L95-L107)
- **休眠清除**：鱼 `OnEntitySleep` 0.1 秒后执行 `HandleEntitySleep`——有有效 `homeseeker.home` 且未 `leaving` 且仍 `persists` 就交给宿主的 `childspawner:GoHome`，否则**直接删除自己**，所以离屏散鱼不会留在存档里。[清除](../../../raw/current/scripts/prefabs/oceanfish.lua#L181-L189)、[宿主回收](../../../raw/current/scripts/components/childspawner.lua#L701-L715)；群侧还有一条：成员全部休眠则整群连鱼一起删。[checkforremoval](../../../raw/current/scripts/prefabs/schoolherd.lua#L99-L115)
- **`leaving` 标记**：被钓起落回水面、或 `oceanfishing_stoppedfishing` 时置 `leaving`，brain 的顶层分支会周期性推 `doleave`，鱼随即进 `leave` 并 `persists = false`。[设置](../../../raw/current/scripts/prefabs/oceanfish.lua#L96-L97)、[状态图](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L25-L30)、[brain 离场](../../../raw/current/scripts/brains/oceanfishbrain.lua#L181-L190)
- **待核验（S10）**：`OnTimerDone` 里有 `lifespan` 分支（挂着竿就续 30 秒，否则移除 `oceanfishable` 并进 `leave`），但快照中**没有为海鱼启动该 timer 的代码**——全 `scripts` 搜 `"lifespan"` 共 6 处，除本函数两处外全在 `gelblob.lua`，`oceanfish.lua` 里唯一一次 `StartTimer` 就是这条续期。原版路径上它应属未接线／旧存档遗留，本页不把它当现行寿命机制。[处理分支](../../../raw/current/scripts/prefabs/oceanfish.lua#L140-L149)

## 出水与投射

钓上来的不是“鱼进背包”，而是**整只鱼被抛向岸上、在水陆分界处换形态**（S4）。顺序如下：

1. 玩家在可捕获距离内按 `ACTIONS.OCEAN_FISHING_CATCH`：`CatchFish` 把 `target` 传给 `oceanfishable:MakeProjectile()`，后者在鱼身上加 `complexprojectile`、把 `OnHit` 接到落点函数，并让鱼进 `launched_out_of_water`（循环播 `catching_loop`）、碰撞掩码切到只认地面、排序层回 `LAYER_WORLD`、开灯，先放一次水花。[制造投射物](../../../raw/current/scripts/prefabs/oceanfish.lua#L120-L138)、[CatchFish](../../../raw/current/scripts/components/oceanfishingrod.lua#L337-L361)
2. `launched_out_of_water` 里 `Locomotor:Stop`，鱼不再自己游；竿已在 `CatchFish` 末尾 `SetTarget(nil)` 解绑，所以飞行途中鱼与竿无引用。[状态](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L308-L317)、[解绑](../../../raw/current/scripts/components/oceanfishingrod.lua#L357-L357)
3. 弹道由 `complexprojectile` 统一结算：初速 12、重力 −30、发射偏移 (0,0.5,0)、目标高度 0.2；每帧把速度写进物理，`velocity.y < 0` 且世界 `y <= 0.05` 即视为落地并回调 `OnHit`——**判定只看高度，不看命中谁**。[发射参数](../../../raw/current/scripts/components/oceanfishingrod.lua#L95-L106)、[落地判定](../../../raw/current/scripts/components/complexprojectile.lua#L198-L211)
4. `OnProjectileLand` 用落点 `Map:IsPassableAtPoint` 二选一：

| 落点 | 处理 | 结果 |
|---|---|---|
| 可通行（岸／船面／陆地） | 生成 `fish_def.prefab.."_inv"`、继承位置与朝向、`SetLanded(true, false)`、`MakeMoistureAtLeast(MAX_WETNESS)` 打满湿、取消旧 flop 任务并立刻 `Flop`；若世界鱼有 `oceanfishable` 且物品有 `weighable` 则 `CopyWeighable` 搬重量，并把世界鱼的 owner 清空 | 世界鱼 `Remove`，得到一条**刚出水**的 `_inv`（满湿、有重量、带原钓者的 owner 记录）。[陆地分支](../../../raw/current/scripts/prefabs/oceanfish.lua#L102-L116) |
| 不可通行（水面） | 移掉 `complexprojectile`、碰撞掩码换回游泳组、排序层回水下、关灯、`weighable:SetPlayerAsOwner(nil)`、`leaving = true`、`persists = false`、回 `idle` 并 `RestartBrain("fish_out_of_water")`，放一次水花 | 不生成物品，鱼留在这片水面并进入离场流程，之后自行消失。[水面分支](../../../raw/current/scripts/prefabs/oceanfish.lua#L84-L100) |

反方向也有对称路径：物品态被丢到**水面**上时，`OnInventoryLanded` 会生成一只同名世界鱼（`leaving = true`、`persists = false`）、放水花、删掉物品；落到可通行处则只重置 flop 计时。[回水](../../../raw/current/scripts/prefabs/oceanfish.lua#L54-L72)

## 物品态

### 数值与转换

18 个 `_inv` 一致的部分：鲜度 `TUNING.PERISH_ONE_DAY`（1 天＝480 秒，S1／S9）、`ignorewentness = true`（湿了也不加速腐坏）、无堆叠、交易金值 `GOLD_VALUES.MEAT`＝1。差异只在下面几条链上。[鲜度](../../../raw/current/scripts/prefabs/oceanfish.lua#L490-L494)、[金值](../../../raw/current/scripts/prefabs/oceanfish.lua#L533-L534)、[金值常量](../../../raw/current/scripts/tuning.lua#L1946-L1952)

| 环节 | 规则 | 证据 |
|---|---|---|
| 腐坏 | 1 天后替换成 `perish_product`（多为 `fishmeat`／`fishmeat_small`；两条素食鱼是 `corn`／`corn_cooked`；叶肉两条是 `spoiled_food`） | [替换字段](../../../raw/current/scripts/prefabs/oceanfish.lua#L493-L493)、[Perish](../../../raw/current/scripts/components/perishable.lua#L277-L310) |
| 烤制 | 只有 `cooking_product` 非空才挂 `cookable`（18 条全有）；**因为带 `smallcreature` 标签，`Cook()` 的鲜度折算整段被跳过**，产物按满鲜生成 | [挂载](../../../raw/current/scripts/prefabs/oceanfish.lua#L527-L531)、[跳过条件](../../../raw/current/scripts/components/cookable.lua#L18-L38) |
| 宰杀 | `murderable` 只提供标签，实际由 `ACTIONS.MURDER.fn` 调 `lootdropper:GenerateLoot()` 逐件生成 | [组件](../../../raw/current/scripts/components/murderable.lua#L1-L11)、[MURDER](../../../raw/current/scripts/actions.lua#L3166-L3190) |
| 交易 | 全部 `goldvalue = 1`（含两条素食鱼），过猪王 `> 0` 门槛；交付者判定见 [[reference/components/tradable]] | [赋值](../../../raw/current/scripts/prefabs/oceanfish.lua#L533-L534) |
| 作祟 | `MakeHauntableLaunchAndPerish` 在本 build 只做抛飞（腐烂那段被 `#HAUNTFIX` 注释掉），抛飞会清 `is_landed`，落到水面即触发下面的“回水” | [helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 鱼箱 | 鱼箱容器只收 `smalloceancreature`，18 个 `_inv` 全中；箱内 `preserver` 给的是**基础倍率 −1/3**，它替换的是基准 1，随后按**环境温度**再乘：环境温度 < 0 时乘 0.75（鱼带 `frozen` 且无 `frozenfiremult` 时倍率直接取 0，既不回鲜也不腐坏）、环境温度 > `TUNING.OVERHEAT_TEMP` 时乘 1.25；再乘 `localPerishMultiplyer` 与 `PERISH_GLOBAL_MULT`（后两者默认 1）。潮湿那一步因 `ignorewentness` 被跳过——所以“`+dt/3` 回鲜”只是**没有其它倍率时的基准**，实际速率随环境温度与局部／全局倍率变化；封顶仍是 100% | [过滤](../../../raw/current/scripts/containers.lua#L1303-L1311)、[箱](../../../raw/current/scripts/prefabs/fish_box.lua#L158-L166)、[基础倍率](../../../raw/current/scripts/components/preserver.lua#L8-L14)、[读取点](../../../raw/current/scripts/components/perishable.lua#L78-L80)、[后续倍率与结算](../../../raw/current/scripts/components/perishable.lua#L107-L133)、[常量](../../../raw/current/scripts/tuning.lua#L2077-L2088) |

### 宰杀与重鱼表门槛

普通表与重鱼表是**二选一整体替换**，不是相加：`inv_common` 先用 `SetLoot(fish_def.loot)` 写下普通表，仅当该鱼定义了 `heavy_loot` 时才追加 `SetLootSetupFn(Inv_LootSetupFn)`；生成战利品时 `GenerateLoot` 先跑这个 setup 函数，若 `weighable:GetWeightPercent() >= TUNING.WEIGHABLE_HEAVY_LOOT_WEIGHT_PERCENT`（0.7）就 `SetLoot(heavy_loot)` 覆盖普通表。[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L498-L503)、[门槛函数](../../../raw/current/scripts/prefabs/oceanfish.lua#L234-L242)、[执行点](../../../raw/current/scripts/components/lootdropper.lua#L186-L191)、[常量](../../../raw/current/scripts/tuning.lua#L337-L338)

重量在构造时随机一次：`Lerp(weight_min, weight_max, math.random())`，`SetWeight` 会把值截到两位小数，`weight_percent` 由 min／max 重新映射，所以“≥70%”是相对该鱼自己的重量区间，不是绝对值——small_1 的 0.7 落在 56.7 以上（区间 48.34–60.30），medium_3 的 0.7 落在 285.7 以上（区间 246.77–302.32）。[抽重量](../../../raw/current/scripts/prefabs/oceanfish.lua#L11-L13)、[物品侧](../../../raw/current/scripts/prefabs/oceanfish.lua#L522-L525)、[截断与百分比](../../../raw/current/scripts/components/weighable.lua#L10-L12)

只有四条鱼没有 `heavy_loot`：small_5、small_6、small_7、medium_5——正好是 `fishtype = "veggie"` 的那四条（其中 small_6／7 的食用分类仍是 `MEAT`），宰杀只走普通表。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L372-L505)、[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L791-L834)

重量还会记“归谁”：`CatchFish` 时 `SetPlayerAsOwner(fisher)`；若物品被怪物当作掉落物抛出，`on_loot_dropped` 会把 `prefab_override_owner` 记成掉落者的 prefab。展示端与奖杯判定见 [[reference/components/weighable]]（S5）。[owner](../../../raw/current/scripts/components/oceanfishingrod.lua#L346-L348)、[归属](../../../raw/current/scripts/prefabs/oceanfish.lua#L175-L179)

### flop、落地与回水

`_inv` 的拍尾有三个排期点：构造时 `math.random()*2+1` 即 1～3 秒后第一次；`on_landed` 落地时重排为 `math.random()+2+0.5*math.random(3)` 即约 2.5～4.5 秒；此后每次自我续排也是 `math.random()+2+0.5*num`（`num` 为本次 `flop_loop` 重复 1–3 次），同为约 2.5～4.5 秒。每轮播 `flop_pre`＋若干 `flop_loop`＋`flop_pst`，并按帧排 4 次落地音效；被拿进背包时 `onputinventory` 取消当前任务，`animover` 时补一次音效。**不能拾取时（如被容器锁定）`Flop` 直接返回并取消任务**。[Flop](../../../raw/current/scripts/prefabs/oceanfish.lua#L23-L52)、[拾取](../../../raw/current/scripts/prefabs/oceanfish.lua#L74-L79)、[音效](../../../raw/current/scripts/prefabs/oceanfish.lua#L409-L413)

### 温度、光照、幸运与灭火

- **温度**：`heater` 的 `heat`（放在地上影响周围）与 `carriedheat`（在背包里影响持有者）都从鱼定义读，`carriedheatmultiplier` 缺省 1、两条特殊鱼写 2；冰鲷走 `SetThermics(false, true)`。[挂载](../../../raw/current/scripts/prefabs/oceanfish.lua#L538-L549)、[语义](../../../raw/current/scripts/components/heater.lua#L53-L78)
- **光照**：只有 small_8 的 `light` 非空；世界态与物品态都 `AddLight()` 并按 `{r,i,f,c}` 设置，世界态初始 `Enable(false)`，只在 `SetBreaching(in_air=true)`／`OnMakeProjectile` 时打开。[世界态](../../../raw/current/scripts/prefabs/oceanfish.lua#L283-L290)、[物品态](../../../raw/current/scripts/prefabs/oceanfish.lua#L422-L428)、[开关](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L50-L66)
- **热扩散**：small_8 的 `propagator` 带 `propagaterange 3`、`heatoutput 2`，落地 `StartSpreading`、进背包 `StopSpreading`——即**只在世界里当热源**。[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L551-L559)、[组件](../../../raw/current/scripts/components/propagator.lua#L98-L110)
- **灭火**：口水鱼的世界态每 4 秒扫一次半径 7 内的火；醒着进 `shoot` 状态吐水弹，睡着则就地 `SpreadProtectionAtPoint`（默认半径 4）。水弹命中后放 `waterstreak_burst`、按 `WATERSTREAK_AOE_DIST`＝3 灭火并加湿，**不含降温项**。[扫描](../../../raw/current/scripts/components/firedetector.lua#L139-L170)、[回调](../../../raw/current/scripts/prefabs/oceanfish.lua#L203-L213)、[射击状态](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L319-L359)、[水弹命中](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L339-L369)

### UI 与采集册

- 物品名恒为鱼名（见[名称核验](#名称核验)）；`no_wet_prefix = true` 让满湿的鱼不显示“湿的”前缀，但湿度值仍照常写入（`MakeMoistureAtLeast`）。[湿前缀](../../../raw/current/scripts/prefabs/oceanfish.lua#L474-L474)
- 采集册只登记 `_inv` 那一份（`oceanfish_*_inv`，`type = "creature"`、`anim = "flop_pst"`、`perishable = 480`、三维与依赖按鱼写死）；`scrapbook_adddeps` 取 `loot` 与 `heavy_loot` 的并集，所以重鱼表的产物也显示成“产出”。[采集册数据](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1028-L1045)、[依赖](../../../raw/current/scripts/prefabs/oceanfish.lua#L481-L481)
- 可用采集册交叉核对：小型肉 12.5／1／0（small_5 为 12.5／3／0）、中型肉 25／8／0（medium_5 为 25／3／0），叶肉两条 −10 理智，与源码一致。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：oceanfishable | 世界态的“被钓”一侧：`makeprojectilefn`／`onreelinginfn`／`onsetrodfn` 三个钩子＋`StrugglingSetup`＋`catch_distance = 2.5` | 主模拟；上竿才逐帧更新、才加 `oceanfishing_catchable` | [[reference/components/oceanfishable]]；[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L347-L352) |
| 组件：eater | 按 `diet` 设可食组与偏好；brain 用它筛可吃的钩／饵／拖网 | 主模拟 | [赋值](../../../raw/current/scripts/prefabs/oceanfish.lua#L354-L360)、[筛选](../../../raw/current/scripts/brains/oceanfishbrain.lua#L90-L109) |
| 组件：herdmember | `Enable(false)`＋`herdprefab = "schoolherd_"..prefab`；`schoolspawner` 生成时才 `Enable(true)` 并 `AddMember` | 主模拟；手动生成（非群）可以没有 herd | [挂载](../../../raw/current/scripts/prefabs/oceanfish.lua#L367-L369)、[入群](../../../raw/current/scripts/components/schoolspawner.lua#L193-L204) |
| 组件：weighable | 世界态只随机重量；物品态 `type = TROPHYSCALE_TYPES.FISH` 并接收拷贝 | 两端各一次 `Initialize`＋`SetWeight`，`type` 只在物品态 | [[reference/components/weighable]]；[世界态](../../../raw/current/scripts/prefabs/oceanfish.lua#L371-L376)、[物品态](../../../raw/current/scripts/prefabs/oceanfish.lua#L522-L525) |
| 组件：lootdropper | 普通表／重鱼表与 0.7 门槛；没有别的掉落入口 | 主模拟 | [[reference/components/lootdropper]]；[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L498-L503) |
| 组件：perishable | 1 天、`onperishreplacement`、`ignorewentness` | 主模拟；进鱼箱取 `preserver` 的基础负倍率，实际速率还受温度与局部／全局倍率影响 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L490-L494) |
| 组件：cookable／edible／tradable | 烤制产物、三维与 `foodtype`、金值 1 | 主模拟；`cookable` 因 `smallcreature` 不继承鲜度 | [[reference/components/cookable]]、[[reference/components/edible]]、[[reference/components/tradable]]；[段](../../../raw/current/scripts/prefabs/oceanfish.lua#L505-L534) |
| 组件：heater／propagator／luckitem | small_8 的 70 度热源与灯、medium_8 的吸热、medium_4 的 −0.25 幸运 | 主模拟；`propagator` 随身停止、`luckitem` 在背包即生效 | [[reference/components/heater]]、[[reference/components/propagator]]、[[reference/components/luckitem]]；[段](../../../raw/current/scripts/prefabs/oceanfish.lua#L538-L564) |
| 组件：firedetector／wateryprotection | 口水鱼扫火、吐水弹、就地保护 | 主模拟；`fireOnly` 只认火与阴燃 | [[reference/components/firedetector]]、[[reference/components/wateryprotection]]；[接线](../../../raw/current/scripts/prefabs/oceanfish.lua#L378-L395) |
| SG／Brain | `SGoceanfish` 管 arrive／idle／leave／bitehook／breach／eat／shoot；`oceanfishbrain` 管游荡、上钩、躲避水花 | 主模拟装 SG 与 Brain | [SG](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L68-L365)、[Brain](../../../raw/current/scripts/brains/oceanfishbrain.lua#L173-L229) |
| 组件：complexprojectile | 出水投射的飞行与落地回调 | 只在投射期存在，落到水里会被移除 | [[reference/components/complexprojectile]]；[移除](../../../raw/current/scripts/prefabs/oceanfish.lua#L86-L86) |
| 关联 Prefab：`oceanfish_shoalspawner` | 斑鱼的额外宿主；也是邪天翁刷新器的“浅滩”来源 | 世界生成只在 `OceanSwell` 房间按权重放置 | [宿主](../../../raw/current/scripts/prefabs/oceanfish_shoalspawner.lua#L55-L69)、[房间](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L35-L54) |
| 关联 Prefab：`waterplant`（海草） | 口水鱼唯一的主动来源：`childspawner.childname = "oceanfish_small_9"`，上限 1 条、半径 4.5、240 秒再生（白花 120 秒） | 主模拟；花粉落地 `pollenlanded` 才 `ReleaseAllChildren` 放出 | [配置](../../../raw/current/scripts/prefabs/waterplant.lua#L469-L476)、[放出](../../../raw/current/scripts/prefabs/waterplant.lua#L495-L497) |
| 关联 Prefab：`waterstreak_projectile` | 口水鱼吐出的水弹 | 由 `LaunchProjectile` 生成 | [注册](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L401-L405) |
| 关联 Prefab：破水水花 | `ocean_splash_small1/2`、`ocean_splash_med1/2` | 按 `breach_fx` 随机 | [[reference/prefabs/ocean_splash_med1]]；[表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L178-L182) |
| 关联 Prefab：鱼肉两链 | 所有掉落、烤制与腐坏产物的数值归属 | 本页只写指向哪张表 | [[reference/prefabs/fishmeat]] |
| 资源：`anim/oceanfish_small.zip`／`oceanfish_medium.zip` | 两种 bank；每条鱼另有自己的 build，bank≠build 时把 build 也加进 assets | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/oceanfish.lua#L577-L587) |

## 典型接入

### 给没有重鱼表的鱼补一张重鱼表

- 前提与选择：服务端 `modmain.lua` 的 postinit，目标 `oceanfish_small_5_inv`（爆米花鱼）。选它是因为原版只给 14 条鱼装了门槛函数，`fishtype = "veggie"` 的那 4 条拿不到重鱼表；直接用原版同一套 `SetLootSetupFn` 机制补，而不是改 `TUNING` 或改共享定义表。[原版门槛](../../../raw/current/scripts/prefabs/oceanfish.lua#L234-L242)
- 生效与撤销：只影响之后生成的物品；已在背包里的鱼没有 `lootsetupfn`，宰杀仍走普通表。卸模组后新物品回到只有普通表。
- 原版依据：`GenerateLoot` 在任何随机／固定段之前先跑 `lootsetupfn`；`SetLoot` 只清 `chanceloot`／`randomloot`／`numrandomloot`，**不清** `chanceloottable`／`ifnotchanceloot`，所以 setup 里换表替换的是固定表与随机表，概率表与兜底表仍按原样结算。

```lua
-- modmain.lua：给爆米花鱼补重鱼掉落（阈值沿用原版 0.7）
local HEAVY = { "fishmeat", "fishmeat_small" }

local function UseHeavyLoot(lootdropper)
    local inst = lootdropper.inst
    local weighable = inst.components.weighable
    if weighable ~= nil
        and weighable:GetWeightPercent() >= GLOBAL.TUNING.WEIGHABLE_HEAVY_LOOT_WEIGHT_PERCENT then
        lootdropper:SetLoot(HEAVY)
    end
end

AddPrefabPostInit("oceanfish_small_5_inv", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local lootdropper = inst.components.lootdropper
    if lootdropper ~= nil then
        lootdropper:SetLootSetupFn(UseHeavyLoot)
    end
end)
```

- 调整与验证：`WEIGHABLE_HEAVY_LOOT_WEIGHT_PERCENT` 是“相对该鱼 min～max 的百分比”，不是绝对重量；进游戏钓一条 35–40 的爆米花鱼看宰杀是否掉 `corn_cooked`，再钓一条接近 47 的看是否变成两件肉。本例未运行，只核对了 API 与调用顺序。

## 常见陷阱与限制

- **误区：世界里的鱼能杀掉取肉。** 世界态两条工厂都没有 `health`／`combat`／`lootdropper`，鱼只会在物品态被宰杀、腐坏或烤制；`fish_def.loot` 是给 `_inv` 读的数据。[世界态组件](../../../raw/current/scripts/prefabs/oceanfish.lua#L340-L398)
- **边界：`_inv` 不能叠、只放 1 天。** `inv_common` 没有 `stackable`，鲜度固定 `PERISH_ONE_DAY`＝480 秒（S9），别按“海鲜”套用池塘鱼或龙虾的天数。[鲜度](../../../raw/current/scripts/prefabs/oceanfish.lua#L490-L494)
- **误区：普通表与重鱼表相加。** `SetLoot` 是整体替换，重鱼表里多出的 `fishmeat_small`／`ice` 是**换掉**普通表后的结果；`loot` 列表内每件都必掉，没有概率。[替换](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[门槛](../../../raw/current/scripts/prefabs/oceanfish.lua#L234-L242)
- **边界：YOT 只在活动期生效，且改的是共享表。** `SpecialEventSetup()` 在 `OnPostInit` 调一次，直接改 `FISH_DEFS.oceanfish_medium_6/7` 的掉落与烤制回调，并以 0.5 权重把两者新增进三格地皮；函数没有 else 回滚，活动结束后同一进程内再 `OnPostInit` 也不会还原，只能重启进程／重载模块。[改写](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1087-L1106)、[调用时机](../../../raw/current/scripts/components/specialeventsetup.lua#L303-L304)
- **误区：口水鱼是肉食鱼。** 它的 `diet` 是 `VEGGIE`、饵偏好 `SMALL_VEGGIE`（勺子饵权重 0），只是物品态按 `MEAT` 结算（食用分类 `edible_MEAT`）；`ediblefish_meat` 是**世界态**按 `fishtype` 打的标签——食性、`fishtype`、`foodtype` 三者分属不同字段，别互相推断。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L594-L602)、[偏好](../../../raw/current/scripts/tuning.lua#L324-L324)
- **边界：冰鲷与太阳鱼方向相反。** 一个 `heat = 70`／`carriedheat = 70`，一个 `endothermic` 且 `heat = -5`，`carriedheatmultiplier` 都是 2；冰鲷没有灯也没有 `propagator`。[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L550-L552)、[定义](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L973-L973)
- **误区：作祟能加快腐烂。** 本 build 的 `MakeHauntableLaunchAndPerish` 只抛飞，腐烂段整段被注释；把鱼抛进水里反而会变回世界鱼。[helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- **误区：烤鱼继承鲜度。** 通用 `Cook()` 只在“产物与原物都有 `perishable` 且原物不带 `smallcreature`”时才折算鲜度，而 18 个 `_inv` 全带 `smallcreature`，所以烤出来的鱼块是满鲜。[跳过](../../../raw/current/scripts/components/cookable.lua#L29-L34)
- **未核验（S10）**：鱼群在海洋世界生成的实际密度与 `oceanfish_shoalspawner` 的每图落点数量。源码里能读到“`OceanSwell` 房间 `distributepercent` 0.004、宿主权重 0.08”与可开关的世界生成设置项 `ocean_shoal`，但没有逐图统计，也未运行游戏。[房间](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L35-L54)、[设置项](../../../raw/current/scripts/map/forest_map.lua#L237-L239)
- **未核验**：本页未运行游戏；挣扎体感、破水特效可见性、鱼箱回鲜速度与 YOT 掉落表现均为源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/oceanfish.lua`（629 行）与 `prefabs/oceanfishdef.lua`（1109 行），逐条核对 18 个定义的四张表、四季权重、`SpecialEventSetup` 与两条工厂的组件顺序；`prefabs/oceanfish_shoalspawner.lua`、`prefabs/schoolherd.lua`、`components/schoolspawner.lua`、`brains/oceanfishbrain.lua`、`stategraphs/SGoceanfish.lua` 全文；`components/oceanfishable`／`oceanfishingrod`／`oceanfishinghook` 与 `prefabs/oceanfishingbobber`／`oceanfishingrod`／`oceanfishinglure` 的入口与门禁；`weighable`／`lootdropper`／`perishable`／`preserver`／`cookable`／`edible`／`eater`／`tradable`／`heater`／`firedetector`／`wateryprotection`／`propagator`／`luckitem`／`murderable`／`childspawner`／`complexprojectile` 的相关分支；`actions.lua` 的 `EAT`／`MURDER`、`componentactions.lua` 的海钓动作筛选、`containers.lua` 与 `prefabs/fish_box.lua` 的鱼箱过滤与保鲜；`forest.lua` 的 `schoolspawner` 安装、`terrain_ocean.lua`／`forest_map.lua` 的宿主放置；`waterplant.lua` 的鱼群 childspawner 与花粉放出、`chum.lua`／`chum_aoe.lua`／`books.lua` 的鱼群入口（核对 small_9 的可达性）；`tuning.lua` 的 `OCEAN_FISHING`／`OCEANFISH`／`OCEANFISH_SHOAL`／`OCEANFISH_LURE_PREFERENCE`／鲜度与热量常量；`scrapbookdata.lua` 与 `chinese_s.po` 交叉核对。
- 检索范围：全 `scripts` 的 `"lifespan"`（6 处，仅 `oceanfish.lua` 与 `gelblob.lua`，确认无海鱼启动点）、`ediblefish_`（唯一消费者是 `gnarwail` 的 `ediblefish_meat`）、`smalloceancreature`（唯一消费者是鱼箱过滤）、`inst.heavy`（`OnSave`／`OnLoad` 之外无写入方）、`(?i)moon|halloween|conversion`（本页两文件 0 命中）。
- 名称核验：`languages/chinese_s.po` 的 18 条 `NAMES.OCEANFISH_*` 与 `_INV` 键，文本成对相同（小孔雀鱼…甜味鱼）。
- 未覆盖：`oceanfishingtackle` 的饵具数值与配方、拖网（`oceantrawler`）的鱼侧交接、邪天翁刷新器的完整节奏、`fishschoolspawnblocker` 的存档行为。
- 游戏运行：未运行。
- 结构检查：见交付回报（`validate_page.py` 与 `validate_source_links.py --strict`）。

## 相关链接

- 海钓链：[[reference/prefabs/fishing-tools]]、[[reference/components/oceanfishingrod]]、[[reference/components/oceanfishable]]
- 鱼群：[[reference/components/schoolspawner]]、[[reference/components/herdmember]]、[[reference/components/childspawner]]
- 物品侧：[[reference/components/weighable]]、[[reference/components/perishable]]、[[reference/components/lootdropper]]、[[reference/prefabs/fishmeat]]
- 破水 FX：[[reference/prefabs/ocean_splash_med1]]
- 同批海钓产物：[[reference/prefabs/wobster]]

返回 [[reference/prefabs/index|Prefab 索引]]。
