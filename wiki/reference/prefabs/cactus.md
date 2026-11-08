---
title: "仙人掌（cactus、oasis_cactus）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "同一 MakeCactus 工厂的两个注册：绿洲种的建造与名字重定向、采摘仙人掌肉的 6 点刺伤门禁与 4 天再生、只在夏季出现的花及其被摘后的状态、燃烧与世界再生，以及恶地／绿洲房间与静态布景的分布。"
aliases: ["cactus", "oasis_cactus", "仙人掌", "绿洲仙人掌", "沙漠仙人掌", "Cactus"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/cactus.lua", "prefabs/plantables.lua", "prefabs/player_common.lua", "prefabs/armor_dragonfly.lua", "components/dynamicmusic.lua", "components/pickable.lua", "components/combat.lua", "components/wisecracker.lua", "components/regrowthmanager.lua", "standardcomponents.lua", "entityscript.lua", "tuning.lua", "map/forest_map.lua", "map/customize.lua", "map/rooms/forest/DLCrooms.lua", "map/static_layouts/trap_summer.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 仙人掌（cactus、oasis_cactus）

## 快速判断

- 职责与适用：`prefabs/cactus.lua` 用 `MakeCactus(name)` 产出 `cactus`（恶地／沙漠）与 `oasis_cactus`（绿洲）两个世界植物，末尾另注册物品 `cactus_flower`（见 [[reference/prefabs/petals]]）。两种仙人掌共用一套 `fn`，差异只在 `name` 决定的 bank／build／小地图图标与地形来源。[工厂](../../../raw/current/scripts/prefabs/cactus.lua#L74-L127)、[导出](../../../raw/current/scripts/prefabs/cactus.lua#L177-L179)
- 优先使用：改采摘收益／再生看[采摘、刺伤与再生](#采摘刺伤与再生)；问「花什么时候有」看[夏季开花](#夏季开花)；问「长在哪」看[生成地](#生成地)。产物数值（`cactus_meat`／`cactus_meat_cooked`）与生熟正负见 [[reference/prefabs/tropical-gathered-fruits]]，`cactus_flower` 的食用与干制见 [[reference/prefabs/petals]]。
- 重要副作用：采摘者满足门禁时被刺 `CACTUS_DAMAGE`＝6 点并推 `thorns`（护甲照常吸收，且会播一句台词）；有采摘者且植株带花（`has_flower`，只在夏季置真）时，独立于刺伤门禁额外追加 1 朵 `cactus_flower` 给它。采完 `Physics:SetActive(false)`，秃株不再挡路。[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L43)
- 修改边界：本页只写两个宿主：构造、采摘门禁、开花、燃烧、再生与布点。产物数值、食谱与干制归产物页；`cactus_flower` 注册本身归 [[reference/prefabs/petals]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cactus` | `MakeCactus("cactus")` 的闭包：transform／anim／minimap／network → `MiniMapEntity:SetIcon("cactus.png")` → `SetBuild("cactus")`＋`SetBank("cactus")`＋`idle` 循环 → tag `plant`／`thorny` → `MakeObstaclePhysics(inst, .3)` → `SetPrefabNameOverride("cactus")` → `SetPristine` → 主模拟：随机起始帧、pickable、inspectable、大燃可燃、世界再生、中传播、`OnEntityWake`、作祟 | 普通沙漠种；`thorny` 是给装备与音乐系统读的标记 tag（消费端见[采摘、刺伤与再生](#采摘刺伤与再生)），挡路半径 0.3。[工厂](../../../raw/current/scripts/prefabs/cactus.lua#L74-L127)、[tag 与物理](../../../raw/current/scripts/prefabs/cactus.lua#L89-L94) |
| `oasis_cactus` | 同一闭包、只换 `name`：`MiniMapEntity:SetIcon("oasis_cactus.png")`、`SetBuild("oasis_cactus")`、`SetBank("oasis_cactus")`；其余逐行相同 | 绿洲沙漠种；同样 `SetPrefabNameOverride("cactus")`，所以玩家看到的名字与普通种一样是「仙人掌」；图鉴另有一条 `oasis_cactus` 条目。[调用](../../../raw/current/scripts/prefabs/cactus.lua#L83-L94)、[图鉴条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1024-L1024) |
| `cactus_flower` | `cactusflowerfn`（同文件末尾） | 物品，不属本页主题；注册与数值见 [[reference/prefabs/petals]]。[工厂](../../../raw/current/scripts/prefabs/cactus.lua#L129-L175) |

名称：`STRINGS.NAMES.CACTUS`＝仙人掌，`CACTUS_FLOWER`＝仙人掌花；`STRINGS.NAMES.OASIS_CACTUS` 在简中 po 里没有条目，绿洲种靠 `SetPrefabNameOverride("cactus")` 落到「仙人掌」这条译文上（`GetBasicDisplayName` 先查 `nameoverride`）。[CACTUS](../../../raw/current/scripts/languages/chinese_s.po#L340055-L340057)、[CACTUS_FLOWER](../../../raw/current/scripts/languages/chinese_s.po#L340060-L340062)、[名称回退](../../../raw/current/scripts/entityscript.lua#L687-L692)

### 采摘、刺伤与再生

- 接线：`pickable:SetUp("cactus_meat", TUNING.CACTUS_REGROW_TIME)`，只给产物与再生时长；`picksound = "dontstarve/wilson/harvest_sticks"`；四个回调 `onregenfn`／`onpickedfn`／`makeemptyfn`／`ontransplantfn`。[接线](../../../raw/current/scripts/prefabs/cactus.lua#L104-L111)、[SetUp](../../../raw/current/scripts/components/pickable.lua#L222-L228)
- 时长：`CACTUS_REGROW_TIME = total_day_time * 4`＝4 天＝1920 秒，春季乘 `SPRING_GROWTH_MODIFIER`＝0.75（1440 秒）。注意它和世界再生的 `CACTUS_REGROWTH_TIME`（20 天）只差一个字母，两条链不同。[采摘常量](../../../raw/current/scripts/tuning.lua#L1768-L1768)、[再生常量](../../../raw/current/scripts/tuning.lua#L3179-L3179)
- 刺伤结算（`onpickedfn`）：外层先判 `picker ~= nil`，再要求**三合一**——采摘者有 `combat` 组件、其背包没有带 `bramble_resistant` tag 的装备、本人也没有 `shadowminion` tag；三条都满足才 `combat:GetAttacked(inst, TUNING.CACTUS_DAMAGE)` 并 `PushEvent("thorns")`。[门禁](../../../raw/current/scripts/prefabs/cactus.lua#L23-L27)
- 伤害点数：`CACTUS_DAMAGE = wilson_health * 0.04`，`wilson_health`＝150，即 6 点；走常规受击流程，护甲在 `inventory:ApplyDamage` 里照常吸收。[常量](../../../raw/current/scripts/tuning.lua#L765-L765)、[基准血量](../../../raw/current/scripts/tuning.lua#L38-L38)、[护甲结算](../../../raw/current/scripts/components/combat.lua#L624-L632)
- `thorns` 的可见效果：玩家身上的 `wisecracker`（`player_common` 统一挂载）监听这个事件，播 `ANNOUNCE_THORNS` 台词；没有其他玩法结算。[台词](../../../raw/current/scripts/components/wisecracker.lua#L119-L121)、[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2779-L2779)
- 动画与物理：采摘先 `Physics:SetActive(false)`（秃株不再有障碍），播 `picked_flower` 或 `picked` 再推 `empty`；再生时先恢复物理，夏季播 `grow_flower`→`idle_flower`，其余季节播 `grow`→`idle`；`makeemptyfn`（读档／`MakeEmpty`）统一停在 `empty`。[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L21)、[再生](../../../raw/current/scripts/prefabs/cactus.lua#L45-L56)、[空株](../../../raw/current/scripts/prefabs/cactus.lua#L58-L62)
- 采摘后：`Pick` 生成 1 个 `cactus_meat`（继承世界湿度）并起 4 天再生任务；植株留在原地，不枯萎、不计轮次。[Pick](../../../raw/current/scripts/components/pickable.lua#L535-L583)
- `thorny` 的其它消费端（花丛的玫瑰变体也带这个 tag）：龙鳞甲受击时不会点燃带此 tag 的攻击者；危险音乐在受击回调里用 `NON_DANGER_TAGS` 排除带此 tag 的攻击者，所以被仙人掌刺到不会切战斗曲。[龙鳞甲](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L10-L18)、[音乐分类](../../../raw/current/scripts/components/dynamicmusic.lua#L716-L717)

### 夏季开花

- `has_flower` 只在两处置真：`onregenfn` 里 `TheWorld.state.issummer` 为真时，以及 `OnEntityWake` 里「`pickable.canbepicked` 且夏季」时；其余路径（采摘后、`makeemptyfn`）一律置 false。[再生开花的判断](../../../raw/current/scripts/prefabs/cactus.lua#L45-L54)、[唤醒校正](../../../raw/current/scripts/prefabs/cactus.lua#L64-L72)
- 生成条件可以概括为「**只在夏季、且只在再生或唤醒时**」：没有独立的计时器与概率，形态就是植株自己的 `idle_flower` 动画加上这个布尔字段；世界生成出来的仙人掌初始没有花，夏季读档／激活时由 `OnEntityWake` 补上。
- 采摘时有采摘者且 `has_flower` 为真：额外 `SpawnPrefab("cactus_flower")`、`InheritWorldWetnessAtTarget(inst)`，有背包就 `GiveItem`，没背包就 `DoDropPhysics` 落地；这一支只看 `inst.has_flower`，不再查季节，也不受刺伤门禁影响。随后 `has_flower = false`，植株照常进入 `empty`。[给花](../../../raw/current/scripts/prefabs/cactus.lua#L29-L42)
- 花不是 `pickable.product`：改产物、`numtoharvest`、`droppicked` 都影响不到它；花只在 `onpickedfn` 里按字段追加。[边界](../../../raw/current/scripts/prefabs/cactus.lua#L104-L111)
- 被摘走后植株本身没有额外变化（不回退外观、不掉生命），下一轮 4 天再生若仍在夏季就再开一朵。

### 燃烧、世界再生与不可移植

- 燃烧：`MakeLargeBurnable(inst)`（默认 30 秒）＋`MakeMediumPropagator(inst)`（引燃点 15～25、传播半径 5～7、传播造成伤害）；作祟用 `MakeHauntableIgnite`，其点火分支在当前构建被注释，实际只设冷却。[燃烧与传播](../../../raw/current/scripts/prefabs/cactus.lua#L115-L121)、[大燃可燃](../../../raw/current/scripts/standardcomponents.lua#L188-L198)、[中传播](../../../raw/current/scripts/standardcomponents.lua#L215-L228)、[作祟工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)
- 烧完：`DefaultBurntFn` 在原地生成 1 份 `ash` 再移除本体（沙漠不属海面，会留灰）。[烧毁](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- 世界再生：`AddToRegrowthManager(inst)` 把移除事件接到再生管理器；`cactus` 与 `oasis_cactus` 各自登记，基础 `CACTUS_REGROWTH_TIME`＝20 天，倍率为夏季 ×2、冬季 ×0、其余 ×1，下雨再 ×1.2（倍率乘在「时间累积速度」上，数值越大越快）。[挂接](../../../raw/current/scripts/prefabs/cactus.lua#L116-L116)、[登记与倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L182-L192)、[常量](../../../raw/current/scripts/tuning.lua#L6037-L6040)
- 不可铲除／不可移植：`cactus.lua` 没有 `workable`，也没有对应的 `dug_cactus` 种植件——`plantables.lua` 的种植表只含浆果丛、树苗、草、沼泽丛、石头果丛、香蕉丛与猴尾，构造函数统一注册 `dug_<name>`，其中没有仙人掌。因此 `ontransplantfn`（`MakeEmpty`）在原生内容里不可达，只有模组手动 `pickable:OnTransplant()` 才会走到。[种植表](../../../raw/current/scripts/prefabs/plantables.lua#L119-L182)、[注册循环](../../../raw/current/scripts/prefabs/plantables.lua#L186-L191)、[移植回调](../../../raw/current/scripts/components/pickable.lua#L418-L424)

### 生成地

| 种 | 布点与证据 |
|---|---|
| `cactus` | 恶地系房间的散布表：`BGBadlands` 0.2、`Lightning` 0.8、`Badlands` 0.7、`DragonflyArena` 0.7（房间地块都是 `DIRT_NOISE`）。[BGBadlands](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L1-L20)、[Lightning](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L22-L38)、[Badlands](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L40-L58)、[DragonflyArena](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L60-L75) |
| `oasis_cactus` | 带 `sandstorm`＋`RoadPoison` tag 的绿洲系房间：`BGLightningBluff` 0.2、`LightningBluffAntlion` 0.1、`LightningBluffOasis` 0.02、`LightningBluffLightning` 0.8。[沙尘暴一侧](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L330-L345)、[蚁狮](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L347-L361)、[绿洲本体](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L363-L378)、[闪电羊沙丘](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L380-L395) |
| 两种 | 世界生成设置 `cactus` 一次翻译成两个 prefab；另有独立的 `cactus_regrowth` 开关控制再生。`map/terrain.lua` 没有给仙人掌登记地块白名单，落点只由房间散布表决定。[设置翻译](../../../raw/current/scripts/map/forest_map.lua#L193-L193)、[两个设置项](../../../raw/current/scripts/map/customize.lua#L360-L360) |
| 静态布景 | `trap_summer`（夏季陷阱）固定摆 2 株 `cactus`。[第 1 株](../../../raw/current/scripts/map/static_layouts/trap_summer.lua#L150-L158)、[第 2 株](../../../raw/current/scripts/map/static_layouts/trap_summer.lua#L161-L169) |

本页只给分布出处与权重，房间与地形机制归世界生成主题。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 产物 `cactus_meat`、4 天再生、四个回调；无轮次、无 `remove_when_picked` | 主模拟 | [[reference/components/pickable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/cactus.lua#L104-L111) |
| 组件：inspectable | 空配置挂载 | 主模拟 | [挂载](../../../raw/current/scripts/prefabs/cactus.lua#L113-L113) |
| 组件：combat（仅被调用） | 采摘者用自己的 `combat:GetAttacked` 结算刺伤；本 prefab 不挂 combat | 主模拟；被刺的是采摘者 | [[reference/components/combat#关键接口]]；[调用](../../../raw/current/scripts/prefabs/cactus.lua#L25-L26) |
| 组件：burnable／propagator | 大燃可燃（默认 30 秒）＋中传播 | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[接线](../../../raw/current/scripts/prefabs/cactus.lua#L115-L117) |
| 组件：hauntable | `MakeHauntableIgnite`：点火分支被注释，只剩冷却 5 秒与恒 false 回调 | 主模拟 | [[reference/components/hauntable#关键接口]]；[工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189) |
| API／数据：`AddToRegrowthManager` | 移除时向再生管理器登记补种 | 主模拟；两注册各一条 | [[reference/components/regrowthmanager#原版注册与其季节倍率]]；[挂接](../../../raw/current/scripts/prefabs/cactus.lua#L116-L116)、[登记](../../../raw/current/scripts/components/regrowthmanager.lua#L182-L192) |
| API／数据：`OnEntityWake` | 唤醒时按季节修正 `has_flower` 与待机动画 | 主模拟；只在 `canbepicked` 时开花 | [实现](../../../raw/current/scripts/prefabs/cactus.lua#L64-L72) |
| 关联 Prefab：`cactus_meat`／`cactus_meat_cooked` | 采摘产物与烤制产物 | 主模拟生成 | [[reference/prefabs/tropical-gathered-fruits#仙人掌肉的生熟正负]]；[产物](../../../raw/current/scripts/prefabs/cactus.lua#L107-L107) |
| 关联 Prefab：`cactus_flower` | 夏季开花的额外产物，不是 pickable 产物 | 主模拟；有采摘者且 `has_flower` 为真才生成 | [[reference/prefabs/petals#注册与构造]]；[生成](../../../raw/current/scripts/prefabs/cactus.lua#L29-L38) |
| 资源：anim | `anim/cactus.zip`（bank／build `cactus`）、`anim/oasis_cactus.zip`、`anim/cactus_flower.zip` | 两个宿主动画名相同（`idle`／`idle_flower`／`picked`／`picked_flower`／`empty`／`grow`／`grow_flower`） | [声明](../../../raw/current/scripts/prefabs/cactus.lua#L1-L6) |

## 典型接入

### 让仙人掌不再刺伤采摘者

- 前提与选择：服务端 `modmain.lua`。刺伤写在 `onpickedfn` 这个**字段**上，postinit 里把它换成自己的函数即可；原函数是文件内 local，无法从外部包裹，重写时要自己补回动画与开花逻辑。[原实现](../../../raw/current/scripts/prefabs/cactus.lua#L18-L43)
- 生效与撤销：postinit 每次构造时运行，换掉字段只影响之后发生的采摘；停用模组即恢复。注意 `thorns` 事件与 `cactus_flower` 的生成都在原函数里，整段替换会一起丢掉。
- 原版依据：门禁与伤害值见[采摘、刺伤与再生](#采摘刺伤与再生)。

```lua
-- modmain.lua：保留采摘与开花，只跳过刺伤结算
local function peaceful_picked(inst, picker)
    inst.Physics:SetActive(false)
    inst.AnimState:PlayAnimation(inst.has_flower and "picked_flower" or "picked")
    inst.AnimState:PushAnimation("empty", true)

    if picker ~= nil and inst.has_flower then
        local loot = GLOBAL.SpawnPrefab("cactus_flower")
        loot.components.inventoryitem:InheritWorldWetnessAtTarget(inst)
        local inventory = picker.components.inventory
        if inventory ~= nil then
            inventory:GiveItem(loot, nil, inst:GetPosition())
        else
            local x, y, z = inst.Transform:GetWorldPosition()
            loot.components.inventoryitem:DoDropPhysics(x, y, z, true)
        end
    end

    inst.has_flower = false
end

AddPrefabPostInit("cactus", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
        return
    end
    inst.components.pickable.onpickedfn = peaceful_picked
end)
```

- 调整与验证：`oasis_cactus` 是独立注册，要同样处理才能覆盖绿洲种；`DoDropPhysics` 的签名是 (x, y, z, randomdir)，示例从世界坐标拆参更稳妥。示例只去掉刺伤，未动 `cactus_meat` 与再生。未运行游戏验证。

## 常见陷阱与限制

- **误区：两个注册要分别写两套逻辑**：它们是同一个闭包，`name` 只决定 bank／build／小地图图标；改行为要么同时 postinit 两个注册名，要么只改一个并明确接受另一形态不生效。[工厂](../../../raw/current/scripts/prefabs/cactus.lua#L74-L127)
- **边界：`CACTUS_REGROW_TIME` 与 `CACTUS_REGROWTH_TIME` 是两条链**：前者是采后再生（4 天，春季 ×0.75），后者是世界再生管理器补种被毁植株（20 天，夏季 ×2／冬季 ×0）。[采摘](../../../raw/current/scripts/tuning.lua#L1768-L1768)、[补种](../../../raw/current/scripts/tuning.lua#L3179-L3179)
- **边界：冬季既不开花也不补种**：`has_flower` 只看 `issummer`，冬季世界再生的倍率为 0，被烧掉的仙人掌要等非冬季才可能补回来。[开花](../../../raw/current/scripts/prefabs/cactus.lua#L45-L54)、[倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L182-L190)
- **边界：刺伤有三道豁免**：无 `combat` 的实体（多数非玩家）、背包里有 `bramble_resistant` 装备的、以及带 `shadowminion` tag 的暗影仆从都不吃这一下；豁免存在时连 `thorns` 台词都不播。[门禁](../../../raw/current/scripts/prefabs/cactus.lua#L23-L27)
- **边界：仙人掌不能挖走**：没有 `workable`、没有 `dug_cactus`，`ontransplantfn` 是原生内容不可达的接线；想搬走只能靠模组自己加种植件或直接 `SpawnPrefab`。[种植表](../../../raw/current/scripts/prefabs/plantables.lua#L119-L182)
- **未核验**：`thorny` 除龙鳞甲与危险音乐外是否还有其它消费者未穷举（按 tag 检索另有珍珠装饰评分与老存档 retrofit 两处，未展开）；刺伤与花期的实机表现未运行验证。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检开花与刺伤门禁解耦）。核对 `cactus.lua` 全文（工厂闭包、四个回调、唤醒、燃烧／再生／传播／作祟接线、`cactus_flower` 工厂、导出）、`pickable` 的 `SetUp`／`Pick`／`OnTransplant`、`combat` 的受击与护甲段、`wisecracker` 的 `thorns` 监听、`regrowthmanager` 的仙人掌登记与季节倍率、`standardcomponents` 的大燃／中传播／烧毁／作祟工厂、`plantables` 的种植名单与注册循环、`map` 的房间散布与静态布景、`scrapbookdata` 的两条图鉴、`tuning` 的相关常量与 `wilson_health`。
- 名称核验：`STRINGS.NAMES.CACTUS`＝仙人掌、`CACTUS_FLOWER`＝仙人掌花；`OASIS_CACTUS` 无简中条目，绿洲种靠 `SetPrefabNameOverride("cactus")` 显示为「仙人掌」。[CACTUS](../../../raw/current/scripts/languages/chinese_s.po#L340055-L340057)、[CACTUS_FLOWER](../../../raw/current/scripts/languages/chinese_s.po#L340060-L340062)
- 未覆盖：`cactus_meat`／`cactus_meat_cooked`／`cactus_flower` 的数值与去向、食谱与干制；绿洲房间与沙尘暴的生成流程。
- 游戏运行：未运行。6 点刺伤、4 天与 20 天两档再生、夏季开花均为源码推导。
- 结构检查：见交付回报。

## 相关链接

- 产物：[[reference/prefabs/tropical-gathered-fruits]]、[[reference/prefabs/petals]]
- 采摘与再生：[[reference/components/pickable]]、[[reference/components/regrowthmanager]]
- 战斗结算：[[reference/components/combat]]

返回 [[reference/prefabs/index|Prefab 索引]]。
