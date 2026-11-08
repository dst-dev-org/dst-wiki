---
title: "苔藓与蕨叶三注册（cutlichen、foliage、quagmire_foliage_cooked）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "cutlichen（苔藓）与 foliage（蕨叶）是两张直接工厂产出的可食物品：苔藓取自洞穴苔藓、2 天保质且负理智触发 badfood；蕨叶取自洞穴蕨类、6 天保质，可作燃料并在晾肉架干制成 foliage_dried；quagmire_foliage_cooked（熟蕨叶）是 Gorge 专用注册，公共骨架只给 quagmire_stewable 标签，其余由事件数据补全。"
aliases: ["cutlichen", "foliage", "quagmire_foliage_cooked", "苔藓", "蕨叶", "熟蕨叶", "洞穴苔藓", "蕨类"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/cutlichen.lua", "prefabs/lichen.lua", "prefabs/foliage.lua", "prefabs/cave_fern.lua", "prefabs/quagmire_fern.lua", "prefabs/tumbleweed.lua", "prefabs/stalker_ferns.lua", "prefabs/driedplants.lua", "prefabs/driedplants_defs.lua", "prefabs/tree_rock_data.lua", "prefabs/tree_rocks.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/tradable.lua", "components/pickable.lua", "components/dryable.lua", "components/dryingrack.lua", "components/quagmire_recipebook.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "cooking.lua", "preparedfoods.lua", "languages/chinese_s.po", "map/customize.lua", "map/forest_map.lua", "map/rooms/cave/ruins.lua", "prefabs/cave.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 苔藓与蕨叶三注册（cutlichen、foliage、quagmire_foliage_cooked）

## 快速判断

- 职责与适用：`prefabs/cutlichen.lua` 与 `prefabs/foliage.lua` 各用一个直接工厂产出物品；两者的宿主植株 `lichen`（洞穴苔藓）与 `cave_fern`（蕨类植物）只作来源说明。`quagmire_foliage_cooked`（熟蕨叶）由 `foliage.lua` 的 `cooked_fn` 注册，属 Gorge 专用，公共骨架不直接给可食组件。[苔藓工厂](../../../raw/current/scripts/prefabs/cutlichen.lua#L6-L52)、[蕨叶工厂](../../../raw/current/scripts/prefabs/foliage.lua#L18-L83)、[熟蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L85-L110)
- 优先使用：查数值与保质看[数值与接线](#数值与接线)；干燥与燃料看[干燥、燃料与烹饪用途](#干燥燃料与烹饪用途)；来源与宿主看[获得路径](#获得路径)；Gorge 数据缺口看[常见陷阱与限制](#常见陷阱与限制)。
- 重要副作用：苔藓的生命为正但理智 −5，赋值即给实体打 `badfood`；蕨叶可作燃料并在晾肉架干制成 `foliage_dried`；熟蕨叶在构造时调用 Gorge 事件数据，事件文件不在本快照内。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[干燥](../../../raw/current/scripts/prefabs/foliage.lua#L70-L74)、[事件调用](../../../raw/current/scripts/prefabs/foliage.lua#L107-L107)
- 修改边界：只讲三件物品的构造、数值与接线；`lichen`／`cave_fern`／`quagmire_fern` 只列来源；`foliage_dried` 作为干燥产物列关系；Gorge 的事件数据与堆叠数值未核验，不写作结论。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cutlichen` | 直接 `fn`：bank／build＝`algae`、`PlayAnimation("idle")`、`pickupsound="vegetation_grassy"`、`MakeInventoryFloatable(inst)` → `SetPristine` → 主模拟挂组件 | 只挂 stackable／inspectable／inventoryitem／tradable／edible／perishable／hauntable；**没有** burnable／fuel／cookable／dryable／farmplantable。资源仅 `anim/algae.zip`。[全文](../../../raw/current/scripts/prefabs/cutlichen.lua#L1-L54) |
| `foliage` | 直接 `fn`：bank／build＝`foliage`、`PlayAnimation("anim")`、`pickupsound="vegetation_grassy"`、公共端 tag `cattoy`／`dryable`、`MakeInventoryFloatable(inst)` → `SetPristine` → 主模拟挂组件 | stackable／tradable／inspectable／fuel／burnable／propagator／inventoryitem／edible／perishable／dryable／hauntable；Gorge 模式另调事件钩子。[全文](../../../raw/current/scripts/prefabs/foliage.lua#L1-L83) |
| `quagmire_foliage_cooked` | 直接 `cooked_fn`：沿用 `foliage` bank／build、`PlayAnimation("cooked")`、`MakeInventoryFloatable(inst)` → 在 pristine **前**打 `quagmire_stewable` tag → `SetPristine` → 主模拟调 `event_server_data("quagmire", "prefabs/foliage").master_postinit_cooked(inst)` | 公共骨架**不挂**任何组件；可食、堆叠、鲜度等由事件数据补全（本快照无该文件）。依赖 `quagmire_burnt_ingredients`。[熟蕨叶工厂](../../../raw/current/scripts/prefabs/foliage.lua#L85-L110)、[依赖](../../../raw/current/scripts/prefabs/foliage.lua#L13-L16) |

两件普通物品都在主模拟分界后挂组件；客户端只读公共端的动画、浮水与 tag。`foliage` 的 `cattoy`／`dryable` tag 写在 `SetPristine` 之前，供客户端与其它系统识别；组件本体仍只在主模拟存在。[tag](../../../raw/current/scripts/prefabs/foliage.lua#L33-L39)、[端分界](../../../raw/current/scripts/prefabs/foliage.lua#L39-L43)、[苔藓端分界](../../../raw/current/scripts/prefabs/cutlichen.lua#L23-L27)

## 数值与接线

| 注册 | 饥饿／生命／理智 | 保质 | 堆叠 | 可燃／燃料 | 备注 |
|---|---|---|---|---|---|
| `cutlichen` | 12.5（`CALORIES_SMALL`）／3（`HEALING_SMALL`）／**−5**（`−SANITY_TINY`） | 2 天（960 秒，`PERISH_TWO_DAY`） | 40 | 无 | 负理智触发 `badfood`；无 `cookable` |
| `foliage` | 0／1（`HEALING_TINY`）／0 | 6 天（2880 秒，`PERISH_FAST`） | 40 | 可燃烧 3 秒；燃料 7.5（`TINY_FUEL`） | 可干制；无 `cookable` |
| `quagmire_foliage_cooked` | 未核验 | 未核验 | 未核验 | 未核验 | 只核到 `quagmire_stewable` tag 与事件钩子 |

时间基准 `seg_time`＝30、`total_day_time`＝480 秒；`PERISH_TWO_DAY`＝2×480、`PERISH_FAST`＝6×480；堆叠统一 `STACK_SIZE_SMALLITEM`＝40。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[保质常量](../../../raw/current/scripts/tuning.lua#L2043-L2049)、[堆叠](../../../raw/current/scripts/tuning.lua#L83-L83)

- 两件都把食物类型设为 `FOODTYPE.VEGGIE`；苔藓的 `sanityvalue=-5` 会让 `edible` 的赋值监听立即加上 `badfood`，即便它的生命是 +3。[食性](../../../raw/current/scripts/constants.lua#L2026-L2030)、[苔藓三维](../../../raw/current/scripts/prefabs/cutlichen.lua#L38-L42)、[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 鲜度都用 `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food`；`SetPerishTime` 会把剩余时间同时拉回满值。[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L44-L47)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L65-L68)、[实现](../../../raw/current/scripts/components/perishable.lua#L188-L194)
- 堆叠上限都在构造后显式写 40，而非依赖组件默认 20。[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L29-L30)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L45-L46)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- 两件都只挂 `tradable`、不设 `goldvalue`，默认 0，猪王拒收。[挂载](../../../raw/current/scripts/prefabs/cutlichen.lua#L36-L36)、[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)
- 两件构造里都没有 `farmplantable`／`plantable`，也没有任何种植调用；它们不是可种物。[苔藓组件段](../../../raw/current/scripts/prefabs/cutlichen.lua#L29-L49)、[蕨叶组件段](../../../raw/current/scripts/prefabs/foliage.lua#L45-L80)
- 作祟 helper 的名字有误导：`MakeHauntableLaunchAndPerish` 的腐坏分支整段被注释，`MakeHauntableLaunchAndIgnite` 的点燃分支也被注释，所以苔藓与蕨叶实际只会被弹飞，不会因作祟腐坏或起火。[LaunchAndPerish](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)、[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)

### 干燥、燃料与烹饪用途

- 蕨叶的 `dryable` 声明产物 `foliage_dried`、干燥时间 `DRY_FAST`＝1 天（480 秒），并指定晾肉架上的展示 build 为 `meat_rack_food_petals`。[接线](../../../raw/current/scripts/prefabs/foliage.lua#L70-L74)、[常量](../../../raw/current/scripts/tuning.lua#L2055-L2055)
- 晾肉架的 `GetProduct()`／`GetDryTime()` 正是从 `dryable` 读取产物与时长；干制完成由架子的计时任务结算。[消费](../../../raw/current/scripts/components/dryingrack.lua#L245-L262)、[接口](../../../raw/current/scripts/components/dryable.lua#L16-L46)
- `foliage_dried` 由 `driedplants.lua` 的统一工厂生成：默认生命 1、饥饿 0、理智 0、堆叠 40、燃料 15、可燃烧约 7.5 秒；`foliage` 这一项没有覆盖值，全走默认。[产物](../../../raw/current/scripts/prefabs/driedplants.lua#L51-L68)、[定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L17-L18)
- 蕨叶本体可作燃料（7.5）并 3 秒烧完；苔藓本体没有 `fuel`／`burnable`，其宿主 `lichen` 才是可燃的。[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L52-L56)、[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L29-L49)、[宿主](../../../raw/current/scripts/prefabs/lichen.lua#L63-L68)
- crockpot 侧：`cutlichen` 被登记为 `veggie=1`，并与鳗鱼／池塘鳗组合出 `unagi`；`foliage` **没有** `AddIngredientValues` 条目，不能直接下锅；`foliage_dried` 登记为 `decoration=1, dried=1`，并用于珍珠奶茶配方。[食材表](../../../raw/current/scripts/cooking.lua#L100-L104)、[unagi](../../../raw/current/scripts/preparedfoods.lua#L339-L349)、[干叶](../../../raw/current/scripts/cooking.lua#L167-L170)、[茶配方](../../../raw/current/scripts/recipes.lua#L1511-L1511)

## 获得路径

| 注册 | 已核验来源 | 产出与条件 | 证据 |
|---|---|---|---|
| `cutlichen` | `lichen`（洞穴苔藓）采摘 | 每次 1 个；`pickable:SetUp("cutlichen", LICHEN_REGROW_TIME)`，`LICHEN_REGROW_TIME`＝5 天（2400 秒）；植株留在原地再生 | [宿主](../../../raw/current/scripts/prefabs/lichen.lua#L56-L61)、[常量](../../../raw/current/scripts/tuning.lua#L1772-L1772) |
| `cutlichen` | 巨石枝 `tree_rock` 的藤蔓（显示名走 `SetPrefabNameOverride("tree_rock")`） | 藤蔓按所在区域加权抽取；`RUINS_ENTRANCE_AREA` 的 `cutlichen` 权重 10，出 5 条藤蔓中的若干条即得；属区域限定掉落，非常规采集 | [区域表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L130-L145)、[藤蔓映射](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L258-L258)、[抽取与掉落](../../../raw/current/scripts/prefabs/tree_rocks.lua#L126-L128)、[SpawnVineLoot](../../../raw/current/scripts/prefabs/tree_rocks.lua#L615-L625) |
| `foliage` | `cave_fern`（蕨类植物）采摘 | 每次 1 个；`SetUp("foliage", 10)` 同时 `remove_when_picked=true`、`quickpick=true`，摘后植株移除，10 秒再生不会发生 | [宿主](../../../raw/current/scripts/prefabs/cave_fern.lua#L51-L55)、[移除](../../../raw/current/scripts/components/pickable.lua#L577-L579) |
| `foliage` | `stalker_fern` | 开花后采摘，产物 `foliage`，数量 1 或 2（变体 2 给 1、其余给 2）；摘后植株消失 | [接线](../../../raw/current/scripts/prefabs/stalker_ferns.lua#L69-L75)、[移除](../../../raw/current/scripts/prefabs/stalker_ferns.lua#L27-L35) |
| `foliage` | 风滚草 `tumbleweed` | 打开时的随机战利品表里 `foliage` 权重 1（与草／树枝 40 等同表） | [战利品表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L147-L153) |
| `quagmire_foliage_cooked` | Gorge 事件内容 | 只核到基础注册与 `quagmire_stewable` tag；未熟形态 `quagmire_foliage`、烹饪路径与事件组件均不在本快照的 Lua 内 | [注册](../../../raw/current/scripts/prefabs/foliage.lua#L112-L113)、[配方书引用](../../../raw/current/scripts/components/quagmire_recipebook.lua#L23-L23) |

宿主侧补充：`lichen` 在洞穴遗迹房间与迷宫中按房间权重生成，`customize` 只允许洞穴世界；`cave_fern` 走洞穴房间与静态布局（蝙蝠洞等）。两者都不在 `regrowthmanager` 的再生表里；苔藓靠自身的 5 天 `pickable` 计时再生，蕨类摘后直接移除。[房间权重](../../../raw/current/scripts/map/rooms/cave/ruins.lua#L22-L23)、[世界限制](../../../raw/current/scripts/map/customize.lua#L362-L362)、[洞穴清单](../../../raw/current/scripts/prefabs/cave.lua#L33-L48)、[蕨类映射](../../../raw/current/scripts/map/forest_map.lua#L209-L209)、[苔藓映射](../../../raw/current/scripts/map/forest_map.lua#L218-L218)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维与 `FOODTYPE.VEGGIE`；苔藓负理智触发 `badfood` | 主模拟；蕨叶无理智赋值，默认 0 | [[reference/components/edible#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L38-L42)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L60-L63) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`；腐坏替换 `spoiled_food` | 苔藓 2 天、蕨叶 6 天；主模拟 | [[reference/components/perishable#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L44-L47)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L65-L68) |
| 组件：stackable | 显式 `maxsize`＝40 | 两件一致；默认 20 | [[reference/components/stackable#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L29-L30)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L45-L46) |
| 组件：dryable | 只挂在蕨叶；声明产物、时长与晾架 build | 主模拟；`dryable` tag 同时在公共端 | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/foliage.lua#L70-L74) |
| 组件：fuel／burnable | 蕨叶燃料 7.5、燃烧 3 秒；苔藓无 | 主模拟 | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L52-L56) |
| 组件：tradable | 只挂载，`goldvalue` 默认 0 | 两件都有；猪王拒收 | [[reference/components/tradable#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L36-L36)、[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：hauntable | 苔藓 `LaunchAndPerish`、蕨叶 `LaunchAndIgnite`；两条的腐坏／点燃分支均已注释 | 两件都只会被弹飞 | [[reference/components/hauntable#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L49-L49)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L80-L80)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 组件：floater／inventoryitem | `MakeInventoryFloatable(inst)` 用默认小号；`pickupsound` 均为 `vegetation_grassy` | 公共端设置 | [[reference/components/floater#关键接口]]；[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L19-L21)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L31-L37) |
| 组件：pickable（宿主） | 苔藓宿主设 5 天再生；蕨类宿主摘后移除 | 仅宿主，不是物品自身 | [[reference/components/pickable#关键接口]]；[苔藓宿主](../../../raw/current/scripts/prefabs/lichen.lua#L56-L61)、[蕨类宿主](../../../raw/current/scripts/prefabs/cave_fern.lua#L51-L55) |
| 数据：crockpot 食材 | 苔藓 `veggie=1`；干蕨叶 `decoration+dried`；生蕨叶无条目 | 静态表 | [[reference/components/stewer#关键接口]]；[表](../../../raw/current/scripts/cooking.lua#L100-L104)、[干叶](../../../raw/current/scripts/cooking.lua#L167-L170) |
| 关联 Prefab：`foliage_dried` | 蕨叶的干燥产物；独立工厂生成 | 晾肉架 480 秒 | [产物](../../../raw/current/scripts/prefabs/driedplants.lua#L51-L68)、[定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L17-L18) |
| 关联 Prefab：`lichen`／`cave_fern`／`quagmire_fern` | 前两件物品的来源宿主；`quagmire_fern` 是 Gorge 蕨宿主 | 宿主页 Wiki 待建 | [苔藓宿主](../../../raw/current/scripts/prefabs/lichen.lua#L28-L73)、[蕨类宿主](../../../raw/current/scripts/prefabs/cave_fern.lua#L25-L69)、[Gorge 蕨](../../../raw/current/scripts/prefabs/quagmire_fern.lua#L11-L38) |
| 资源：`anim/algae.zip`／`anim/foliage.zip` | 苔藓用 `algae` bank／build；蕨叶与熟蕨叶共用 `foliage` bank／build，另加 `meat_rack_food_petals.zip` 供晾架展示 | 熟蕨叶不新增资产 | [苔藓资产](../../../raw/current/scripts/prefabs/cutlichen.lua#L1-L4)、[蕨叶资产](../../../raw/current/scripts/prefabs/foliage.lua#L1-L5) |

## 典型接入

### 调整苔藓与蕨叶的堆叠和保质

- 前提与选择：服务端 `modmain.lua`；两件独立注册，逐件 postinit。苔藓原保质只有 2 天，适合先按需求延长再改堆叠。
- 生效与撤销：postinit 每次构造运行；停用模组恢复工厂原值。用 `SetNewMaxPerishTime` 保留已有剩余比例；直接调 `SetPerishTime` 会把剩余拉满。[接口](../../../raw/current/scripts/components/perishable.lua#L188-L208)
- 原版依据：原始数值见[数值与接线](#数值与接线)；两件均在构造末尾 `StartPerishing`，postinit 时仍满鲜。[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L44-L47)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L65-L68)

```lua
-- modmain.lua：苔藓与蕨叶统一堆叠 20、保质 10 天
local function retune(prefab)
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        if inst.components.stackable ~= nil then
            inst.components.stackable.maxsize = 20
        end
        if inst.components.perishable ~= nil then
            inst.components.perishable:SetNewMaxPerishTime(GLOBAL.TUNING.PERISH_MED)
        end
    end)
end

retune("cutlichen")
retune("foliage")
```

- 调整与验证：单位为「个」与「秒」；`maxsize` 影响新构造实例，`SetNewMaxPerishTime` 对已有实例按比例保留。示例把两件都改为 10 天；若只想改苔藓，删掉 `retune("foliage")` 即可。

## 常见陷阱与限制

- **边界：苔藓不是纯正收益**：它给 +3 生命、12.5 饥饿，但 −5 理智会立即打上 `badfood`；`badfood` 只看负生命／负理智。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- **边界：`foliage` 不能直接下锅**：`cooking.lua` 没有为生蕨叶登记食材值；只有干蕨叶被登记为 `decoration+dried` 并用于珍珠奶茶。[食材表](../../../raw/current/scripts/cooking.lua#L100-L104)、[干叶](../../../raw/current/scripts/cooking.lua#L167-L170)
- **边界：`quagmire_foliage_cooked` 不是普通世界物品**：构造时无条件调用 `event_server_data("quagmire", "prefabs/foliage")`，本快照没有该事件文件，无法核验其组件与数值；不要在非 Gorge 环境把它当可食物引用。[调用](../../../raw/current/scripts/prefabs/foliage.lua#L107-L107)
- **边界：蕨类宿主摘后不再生**：`cave_fern` 的 `remove_when_picked=true` 让 `SetUp("foliage", 10)` 的 10 秒再生失效，摘一次少一株；`lichen` 才按 5 天原地再生。[蕨类](../../../raw/current/scripts/prefabs/cave_fern.lua#L51-L55)、[苔藓](../../../raw/current/scripts/prefabs/lichen.lua#L56-L61)
- **边界：可燃的是宿主与蕨叶，不是苔藓**：`cutlichen` 没有 `burnable`／`fuel`，烧宿主掉落的是灰而不是苔藓。[苔藓](../../../raw/current/scripts/prefabs/cutlichen.lua#L29-L49)、[宿主](../../../raw/current/scripts/prefabs/lichen.lua#L63-L68)
- **误区：作祟会点燃或加速腐坏**：两个 helper 的对应分支都被注释，实际只有弹飞；不要据此写「作祟烧掉」或「作祟回鲜」的结论。[LaunchAndPerish](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)、[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **未核验**：`tree_rock` 藤蔓的区域权重只核到表值，未验证实际生成概率与可重复性；`quagmire_foliage` 未熟形态、Gorge 烹饪与熟蕨叶的组件均缺事件数据；`stalker_fern` 的生成条件未追。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `cutlichen.lua`、`lichen.lua`、`foliage.lua`、`cave_fern.lua`、`quagmire_fern.lua`、`tumbleweed.lua` 战利品段、`stalker_ferns.lua` 全文、`driedplants.lua` 与 `driedplants_defs.lua`、`tree_rock_data.lua` 的藤蔓表与 `tree_rocks.lua` 的抽取／掉落，以及 `edible`／`perishable`／`stackable`／`tradable`／`pickable`／`dryable`／`dryingrack` 的相关分支；`standardcomponents` 的燃烧、作祟、浮水与冬季 helper；`tuning` 的时间／三维／保质／堆叠／干燥／苔藓再生常量；`constants` 的 `FOODTYPE`；`cooking.lua`／`preparedfoods.lua`／`recipes.lua` 的食材与配方引用；`map/customize.lua`／`map/forest_map.lua`／`map/rooms/cave/ruins.lua`／`prefabs/cave.lua` 的生成接线。
  名称核验：`NAMES.CUTLICHEN`＝苔藓、`NAMES.FOLIAGE`＝蕨叶、`NAMES.QUAGMIRE_FOLIAGE_COOKED`＝熟蕨叶；宿主为 `LICHEN`＝洞穴苔藓、`CAVE_FERN`＝蕨类植物；干燥产物 `FOLIAGE_DRIED`＝干蕨叶。[CUTLICHEN](../../../raw/current/scripts/languages/chinese_s.po#L341955-L341957)、[FOLIAGE](../../../raw/current/scripts/languages/chinese_s.po#L342940-L342942)、[QUAGMIRE_FOLIAGE_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L347525-L347527)、[LICHEN](../../../raw/current/scripts/languages/chinese_s.po#L344700-L344702)、[CAVE_FERN](../../../raw/current/scripts/languages/chinese_s.po#L340755-L340757)、[FOLIAGE_DRIED](../../../raw/current/scripts/languages/chinese_s.po#L342945-L342947)
  未覆盖：`quagmire_foliage` 无 Lua 注册与名称键，Gorge 事件目录不在本快照；`tree_rock` 藤蔓的生成频率与实际可重复性未验证；`stalker_fern` 的触发条件未追；未穷举模组注入与活动脚本。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`PERISH_TWO_DAY`＝960 秒、`PERISH_FAST`＝2880 秒、`DRY_FAST`＝480 秒），非运行测量。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 无 error／warning；页内 Lua 示例通过 `parser_probe.exe --syntax-only`。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 同类采集食材与干制：[[reference/prefabs/kelp]]、[[reference/prefabs/wild-berries]]
- 可食、鲜度与干燥：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/dryable]]
- 采摘与世界生成：[[reference/components/pickable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
