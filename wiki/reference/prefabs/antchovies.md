---
title: "蚁头凤尾鱼与鱼群（antchovies、antchovies_group）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/antchovies.lua 与 antchovies_group.lua 的两个注册：每个鱼群被围网收成 0–3 条可食物品（单群上限 3，一网可同时波及多群），当前新世界未检出自然生成入口，旧档迁移仍可生成。"
aliases: ["antchovies", "antchovies_group", "蚁头凤尾鱼", "蚁头凤尾鱼群", "鱼群", "围网捕鱼"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/antchovies.lua", "prefabs/antchovies_group.lua", "prefabs/forest.lua", "prefabs/fishingnetvisualizer.lua", "components/fishschool.lua", "components/fishingnetvisualizer.lua", "components/fishingnet.lua", "components/retrofitforestmap_anr.lua", "stategraphs/SGantchovies.lua", "prefablist.lua", "scrapbook_prefabs.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蚁头凤尾鱼与鱼群（antchovies、antchovies_group）

## 快速判断

- 职责与适用：`prefabs/antchovies.lua` 注册**物品** `antchovies`（吃了回 9.375 饥饿的肉类），`prefabs/antchovies_group.lua` 注册**世界实体** `antchovies_group`（贴着水面的鱼群，挂 `fishschool`）；两者都不在**当前新世界**的任何生成表里。[物品注册](../../../raw/current/scripts/prefabs/antchovies.lua#L51-L51)、[鱼群注册](../../../raw/current/scripts/prefabs/antchovies_group.lua#L42-L42)
- 优先使用：想知道"这东西现在能不能拿到"直接看[生成入口盘点](#生成入口盘点)；改鱼群产出看[鱼群与围网](#鱼群与围网)；改物品数值看[注册与构造](#注册与构造)。
- 重要副作用：**当前新世界未检出自然生成入口（shipped but unused），旧档迁移仍可生成**——文件第一行就写着 `This prefab is shipped but not used anymore.`；唯一还能 `SpawnPrefab` 出鱼群的路径是旧存档的 Turn of Tides retrofit，而更晚的 retrofit 又会把**当时位于三个指定海域**（`OCEAN_SWELL`／`OCEAN_ROUGH`／`OCEAN_BRINEPOOL`）的鱼群删掉，不是全图清理。[注释](../../../raw/current/scripts/prefabs/antchovies.lua#L1-L1)、[retrofit 生成](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L163-L206)、[retrofit 清理](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L208-L216)
- 修改边界：围网本体（`fishingnet`／`fishingnetvisualizer`）与 `fishschool` 的行为分别归其组件页；本页只写这两个注册、它们之间的关系、以及"哪里真的会生成"。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `antchovies` | 单一 `fn()`，没有共用工厂；pristine → 主模拟两段 | **物品**：`MakeInventoryPhysics`、bank／build 都是 `antchovy`、播 `idle_ground`、`SetTwoFaced()`、动态阴影 0.65×0.25；主模拟只有 `inventoryitem`＋`edible`（0／9.375／0、`FOODTYPE.MEAT`）＋`stackable`（40），**没有** `inspectable`／`tradable`／`perishable`／`cookable`／`fuel`。[fn](../../../raw/current/scripts/prefabs/antchovies.lua#L7-L49) |
| `antchovies_group` | 单一 `fn()`；pristine → 主模拟两段 | **世界实体**：只加 `ignorewalkableplatforms` 标签，bank `antchovies`、build `water_antchovies`、`LAYER_BACKGROUND`＋`SetSortOrder(1)`、颜色乘 0.4 压暗；主模拟挂 `fishschool`，`SetNettedPrefab("antchovies")` 后 `StartReplenish(10)`。没有 `inventoryitem`／`edible`／`health`。[fn](../../../raw/current/scripts/prefabs/antchovies_group.lua#L11-L40) |

两个文件都把状态机注释掉了——`antchovies.lua#L37` 与 `antchovies_group.lua#L34` 都是 `--inst:SetStateGraph("SGantchovies")`，因此 `stategraphs/SGantchovies.lua`（负责 `caught_in_net` 等三个状态）虽然存在，却没有任何实体挂载它。[SG](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L6-L46)

数值来源：`CALORIES_TINY = calories_per_day/8`＝75/8＝**9.375**；堆叠 `STACK_SIZE_SMALLITEM`＝40。[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2058)、[堆叠](../../../raw/current/scripts/tuning.lua#L83-L83)

### 名称核验

`NAMES.ANTCHOVIES`＝**蚁头凤尾鱼**（Ant-Chovy）；**没有** `NAMES.ANTCHOVIES_GROUP` 这个键，鱼群也没有 `inspectable`，所以它不显示名字。物品同样没有 `inspectable`，名字只出现在界面（背包／掉落）里。[名称](../../../raw/current/scripts/languages/chinese_s.po#L338684-L338687)、[采集册里被注释](../../../raw/current/scripts/scrapbook_prefabs.lua#L252-L252)

## 生成入口盘点

把 `scripts/` 里与生成相关的 `antchovies`／`antchovies_group` 出现按"算不算生成"逐条判定（检索范围与其余名称／对话引用见[验证记录](#验证记录)）：

| 出现位置 | 性质 | 判定 |
|---|---|---|
| `prefablist.lua#L35-L36` | 静态注册清单 | 只是把两个 prefab 载入，**不生成** |
| `prefabs/forest.lua#L330` | 森林世界的 `prefabs` 加载表（"ocean" 分组） | 预载资源／prefab 声明，**不生成** |
| `prefabs/antchovies_group.lua#L8` | `prefabs` 依赖表里写 `"antchovies"` | 声明依赖，**不生成** |
| `components/fishschool.lua#L36` | `SpawnPrefab(self.fish_prefab_name)` | **真实生成**，但要先有 `on_pre_net` 事件 |
| `components/retrofitforestmap_anr.lua#L177` | `TurnOfTidesRetrofitting_PopulateOcean` 的 `distributeprefabs` | **真实生成**，仅旧存档 `retrofit_turnoftides` |
| `components/retrofitforestmap_anr.lua#L211` | `items_to_remove = { "seastack", "antchovies_group", … }` | **删除**，但只清当时位于指定三个海域的个体 |
| `stategraphs/SGantchovies.lua` | 未被 `SetStateGraph` 引用的状态机 | 死代码 |
| `scrapbook_prefabs.lua#L252` | `scrapbook_prefabs.lua` 里的注释行 | 采集册不收录 |

`map/` 目录下检索不到 `antchovies`：没有房间、任务或地形把 `antchovies_group` 放进当前世界生成。结论按"当前新世界未检出自然生成入口，旧档迁移仍可生成"处理——**不要把"注册了"写成"新档里可获得"**。另外，后续的清理 retrofit 只扫指定海域（`OCEAN_SWELL`／`OCEAN_ROUGH`／`OCEAN_BRINEPOOL`），不是全图清理，所以更早的旧档里仍可能留着鱼群。[清理海域](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L211-L215)

## 鱼群与围网

| 步骤 | 结算 | 证据 |
|---|---|---|
| 1. 鱼群初始 | `fishschool` 构造时 `fish_level = max_fish_level`＝3，并播 `group_pre`→`group_loop1` | [构造](../../../raw/current/scripts/components/fishschool.lua#L1-L12) |
| 2. 收网触发 | `fishingnetvisualizer:BeginOpening` 对 `collect_radius + MAX_FISH_SCHOOL_SIZE`＝2+2＝4 格内**所有**实体推 `on_pre_net`（以开网位置为圆心、世界坐标单位） | [推事件](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L105)、[半径常量](../../../raw/current/scripts/tuning.lua#L4042-L4042) |
| 3. 鱼群吐出 | `OnPreNet` 按**自己**的 `fish_level` 次 `SpawnPrefab("antchovies")`（每群独立结算，单群 0–3 条），位置取**网**的坐标，再给每条推 `on_caught_in_net`；随后 `fish_level = 0` 并播 `group_pst` | [吐出](../../../raw/current/scripts/components/fishschool.lua#L29-L45) |
| 4. 收进网 | 同一帧第二次 `FindEntities`（半径 2）把带 `inventoryitem` 的掉落物登记为捕获物 | [捕获](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L107-L113) |
| 5. 落地 | `BeginFinalPickup` 逐条 `DropItem`：`ReturnToScene`、推 `on_release_from_net`、按 0.25 秒间隔抛到投掷者脚边 | [投放](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L116-L184) |
| 6. 补充 | `StartReplenish(10)` 起 10 秒周期任务，每次 `Replenish` 让 `fish_level` +1 直到 3 | [补充](../../../raw/current/scripts/components/fishschool.lua#L13-L16)、[上限](../../../raw/current/scripts/components/fishschool.lua#L47-L56) |

- `on_caught_in_net` 在 `fishschool` 之外**没有监听者**：唯一监听它的是那份没被挂载的 `SGantchovies`，所以这条事件现在只是空推。[监听处](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L14-L18)
- 鱼群实体本身不会消失：收网只把 `fish_level` 归零、播一段 `group_pst`，之后靠补充任务长回去。[吐出](../../../raw/current/scripts/components/fishschool.lua#L42-L44)
- 推导：`fishschool` 没有 `OnSave`／`OnLoad`，读档后 `fish_level` 回到构造时的 3；同一次收网如果 4 格内有两群鱼，两群会一起被清空（事件按实体逐个推）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`fishschool` | 鱼群的"存量—吐出—补充"全部由它驱动；`SetNettedPrefab` 决定吐什么 | 仅 `antchovies_group`、仅服务端 | [[reference/components/fishschool]]；[挂载](../../../raw/current/scripts/prefabs/antchovies_group.lua#L35-L37) |
| 组件：`edible`／`inventoryitem`／`stackable` | 物品侧只有这三件：9.375 饥饿的肉类、可拾取、40 一叠 | 仅 `antchovies`、仅主模拟 | [[reference/components/edible]]、[[reference/components/inventoryitem]]；[赋值](../../../raw/current/scripts/prefabs/antchovies.lua#L39-L46) |
| 组件：`fishingnetvisualizer` | 抛出—张开—回收的时序宿主，`BeginOpening` 是鱼群唯一的外部触发 | 由 `fishingnet` 生成，挂在 `fishingnetvisualizer` 实体上 | [[reference/components/fishingnetvisualizer]]、[[reference/components/fishingnet]]；[推事件](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L105) |
| SG：`SGantchovies` | 三个状态（`idle`／`caught_in_net`／`released_from_net`）都在，但没有实体挂载 | 死代码，见上 | [SG](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L6-L46) |
| 标签：`ignorewalkableplatforms` | 让鱼群不被当作可行走平台的一部分 | 仅鱼群、pristine | [加标签](../../../raw/current/scripts/prefabs/antchovies_group.lua#L20-L20) |
| 数据：`TUNING.MAX_FISH_SCHOOL_SIZE` | 只用于把"找鱼群"的半径从 2 撑到 4 | 与 `collect_radius` 相加 | [常量](../../../raw/current/scripts/tuning.lua#L4042-L4042) |
| 关联 Prefab：`fishschool` 的吐出品 | `antchovies` 是唯一产物 | `SetNettedPrefab("antchovies")` 写死 | [写死](../../../raw/current/scripts/prefabs/antchovies_group.lua#L36-L36) |
| 资源：`anim/antchovy.zip`（`PKGREF`）／`anim/water_antchovies.zip` | 物品与鱼群各一套 bank／build | 原版资源，不随页分发 | [物品 assets](../../../raw/current/scripts/prefabs/antchovies.lua#L2-L5)、[鱼群 assets](../../../raw/current/scripts/prefabs/antchovies_group.lua#L1-L4) |

## 典型接入

### 用 `fishschool` 做一个"可捕捞鱼群"

- 前提与选择：新建一个只在水面生成的自定义 prefab，主模拟挂 `fishschool`，再用 `SetNettedPrefab` 指定吐出的物品、`StartReplenish` 指定补充周期。这是 `antchovies_group` 唯一被验证过的接线方式。
- 生效与撤销：`on_pre_net` 由 `fishingnetvisualizer` 广播，所以只要鱼群落在网的 4 格内就会结算；`fish_level` 不存档，重启服务器后回到 3。
- 原版依据：见[鱼群与围网](#鱼群与围网)。

```lua
-- 自建 prefab（服务端段）：照搬 antchovies_group 的接线
local inst = CreateEntity()
inst.entity:AddTransform()
inst.entity:AddAnimState()
inst.entity:AddNetwork()
inst.AnimState:SetBank("antchovies")
inst.AnimState:SetBuild("water_antchovies")
inst.entity:SetPristine()
if not TheWorld.ismastersim then
    return inst
end
inst:AddComponent("fishschool")
inst.components.fishschool:SetNettedPrefab("antchovies")
inst.components.fishschool:StartReplenish(10)
return inst
```

- 调整与验证：`StartReplenish` 的参数是**秒**（周期），`max_fish_level` 固定 3、要改只能在 `fishschool` 上加字段。**测试用网没有配方**——`Recipe("fishingnet", …)` 在 recipes.lua 里被注释掉，只能靠调试生成（如 `c_spawn("fishingnet")`）或旧存档已有的网。三个样本都要以**开网位置**为圆心量距离（世界坐标单位）：①一群满员（`fish_level = 3`）落在 4 格内 → 收 3 条；②两群都满员且都在 4 格内 → 收 6 条（单网按群累加，不是 3 条封顶）；③空群（`fish_level = 0`）→ 收 0 条，且不播 `group_pst`。[无配方](../../../raw/current/scripts/recipes.lua#L671-L671)、[空群分支](../../../raw/current/scripts/components/fishschool.lua#L29-L45)

## 常见陷阱与限制

- **误区：源码里有 `antchovies` 就代表海里能捞到。** 当前新世界没有自然生成入口；能"遇到"它的只剩旧存档的 Turn of Tides retrofit，而同一文件里更晚的 `retrofit_turnoftides_seastacks` 会把指定海域里的它列进 `items_to_remove` 删掉（并非全图）。[清理](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L208-L216)
- **误区：鱼群是生物、会自己游。** 它没有 `locomotor`／`health`／Brain，只是一段贴水的动画＋`fishschool` 计数，产出靠网触发。[构造](../../../raw/current/scripts/prefabs/antchovies_group.lua#L11-L40)
- **边界：`SGantchovies` 没被使用。** 两个 prefab 的 `SetStateGraph` 都被注释，`on_caught_in_net` 与 `on_release_from_net` 都没有监听者；这两条事件现在是空推。[注释](../../../raw/current/scripts/prefabs/antchovies.lua#L37-L37)、[SG](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L14-L30)
- **边界：物品不可检查、不会腐坏、不能烤。** 它只有 `inventoryitem`／`edible`／`stackable` 三件，没有 `inspectable`／`perishable`／`cookable`，所以既看不到检查描述，也不会变成腐烂物。[fn](../../../raw/current/scripts/prefabs/antchovies.lua#L35-L46)
- **边界：3 条是单群上限，不是单网上限。** 每群按自己的 `fish_level`（上限 `max_fish_level = 3`）吐鱼，同一网 4 格内若有两群满员就是 6 条；补充每 10 秒 1 条。[上限](../../../raw/current/scripts/components/fishschool.lua#L47-L56)、[逐群推事件](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L105)
- **未核验**：未运行游戏。`fish_level` 不存档、4 格内两群同时清空、以及"旧存档 retrofit 后还能不能看见鱼群"（取决于该存档是否已执行过清理 retrofit，且清理时鱼群是否位于 `biomes_to_cleanup` 所列海域）都是源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/antchovies.lua` 与 `prefabs/antchovies_group.lua` 全文、`components/fishschool.lua` 全文、`stategraphs/SGantchovies.lua` 全文；`components/fishingnetvisualizer.lua` 的 `BeginOpening`／`DropItem`／`BeginFinalPickup`；`components/retrofitforestmap_anr.lua` 的 `TurnOfTidesRetrofitting_PopulateOcean`／`CleanupOceanPoution` 与 `retrofit_turnoftides*` 派发段；`prefablist.lua`、`prefabs/forest.lua` 的加载表、`scrapbook_prefabs.lua` 的注释行；`tuning.lua` 的 `MAX_FISH_SCHOOL_SIZE`／`STACK_SIZE_SMALLITEM`／`CALORIES_TINY`；`chinese_s.po` 名称。
   检索范围：在 `scripts/` 下检索 `antchovies`，其中与生成相关的 8 个文件已逐一判定为注册／声明／生成／删除；另有 `strings.lua`、`speech_*.lua` 等名称与对话引用（如 strings.lua 的 `ANTCHOVIES = "Ant-Chovy"`、speech_wilson.lua 的 `--unimplemented` 台词），均非生成入口；`map/` 下无命中。旧档里能不能清掉还取决于鱼群**所在海域**：清理只对 `biomes_to_cleanup`（`OCEAN_SWELL`／`OCEAN_ROUGH`／`OCEAN_BRINEPOOL`）里的个体生效，其余海域的鱼群会留下来。[清理解析](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L211-L215)
- 修订记录：2026-11-09 第 1 轮独立检视回源复核后按点名条目修正——P3-1 检索范围改为“生成相关 8 个文件，另有 strings／speech 名称与对话引用”（L125、L42）。外部复检轮再改：P1 把“3 条”限定为单群上限并补“两群满员 6 条”样本（L6、L119、L111）、P2 统一口径为“新世界无自然生成、旧档迁移可生成”并补清理只扫指定海域、渔网无配方与半径单位（L22、L55、L62、L111、L115）。D 组第二轮再改：快速判断与生成表把清理限定为“当时位于三个指定海域的鱼群”（L22、L51）。第三轮再改：验证记录补“旧档能否清掉取决于 `biomes_to_cleanup` 所列海域”（L125）。第四轮再改：L20 限定为“当前新世界”、L120 未核验补“还要看清理时鱼群所在海域”；其余段落未动。
- 未覆盖：`components/fishingnet.lua` 的完整投网动作与消耗、`speech_webber.lua` 里那句 `--unimplemented` 台词的触发条件。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 鱼群驱动：[[reference/components/fishschool]]、[[reference/components/fishingnetvisualizer]]、[[reference/components/fishingnet]]
- 物品组件：[[reference/components/edible]]、[[reference/components/inventoryitem]]、[[reference/components/stackable]]
- 海钓与鱼：[[reference/prefabs/fishing-tools]]、[[reference/prefabs/ocean-fish]]、[[reference/prefabs/freshwater-fish]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
