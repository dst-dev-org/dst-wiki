---
title: "糖果袋（candybag）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "原版糖果袋：BODY 槽十四格万圣节专用过滤背包容器，只收 halloweencandy／halloween_ornament 标签或 trinket_ 前缀物品；配方草×6 锁 HALLOWED_NIGHTS 活动档、活动期自动解锁，本体可燃烧、燃尽成灰。"
aliases: ["candybag", "糖果袋", "Candy Bag"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/candybag.lua", "containers.lua", "components/container.lua", "components/container_replica.lua", "components/builder.lua", "widgets/widgetutil.lua", "recipes.lua", "constants.lua", "simutil.lua", "gamelogic.lua", "map/forest_map.lua", "map/boons.lua", "prefabs/halloweencandy.lua", "prefabs/halloween_ornaments.lua", "standardcomponents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 糖果袋（candybag）

## 快速判断

- 职责与适用：可拾取、装备 BODY 槽的十四格（2×7）专用过滤背包容器，万圣节活动（Hallowed Nights）糖果与饰品的随身收纳；独立工厂，与 backpack／piggyback 无共享组装。适合借鉴"priorityfn 过滤容器"与"活动档科技解锁"两类组装。[工厂](../../../raw/current/scripts/prefabs/candybag.lua#L51-L101)、[过滤参数](../../../raw/current/scripts/containers.lua#L1741-L1764)
- 优先使用：改接收条件或入包优先 → 本页"调整糖果袋接收条件"；配方锁定／解锁原理 → 本页"配方与 HALLOWED_NIGHTS 活动档"；穿脱开关、溢出容器与燃烧生命周期等通用背包机制复用 [[reference/prefabs/backpack]]，同款十四格布局见 [[reference/prefabs/krampus_sack]]。
- 重要副作用：点燃即锁 `canbeopened=false`、熄灭恢复（[火情回调](../../../raw/current/scripts/prefabs/candybag.lua#L39-L49)）；燃尽先 `DropEverything` 再生成 `ash` 并移除本体（[onburnt](../../../raw/current/scripts/prefabs/candybag.lua#L28-L37)）。
- 修改边界：单一注册名，无皮肤注册、无运行时形态切换；本页不覆盖 piggyback／krampus_sack／icepack／seedpouch 等同族 BODY 容器；未逐项审校 container／equippable 组件内部与多人开关细节。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `candybag` | `fn`：公共端 Transform／AnimState／SoundEmitter／Network、inventory physics、bank／build `candybag`、`backpack` 标签、floater → `SetPristine`（客户端到此返回）→ 主模拟 inspectable／inventoryitem／equippable／container／burnable／propagator／hauntable helper | 物品＋装备（BODY，[constants.lua#L658](../../../raw/current/scripts/constants.lua#L658)）＋过滤容器三合一；`cangoincontainer=false` 不能再入其他容器；`prefabs` 依赖表只声明 `ash`；无 SG／Brain。[工厂](../../../raw/current/scripts/prefabs/candybag.lua#L51-L101)、[注册](../../../raw/current/scripts/prefabs/candybag.lua#L103) |

名称键 `STRINGS.NAMES.CANDYBAG` 对应"糖果袋"／Candy Bag。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L340095-L340097)

## 配方与 HALLOWED_NIGHTS 活动档

配方：`Recipe2("candybag", {Ingredient("cutgrass", 6)}, TECH.HALLOWED_NIGHTS, {hint_msg = "NEEDSHALLOWED_NIGHTS"})`，与 madscience_lab、pumpkinhat、witchbroom 同段登记。[配方](../../../raw/current/scripts/recipes.lua#L1411-L1415)

锁定与解锁全链：

1. `TECH.HALLOWED_NIGHTS = { SCIENCE = 10 }`，源码注释即"ApplySpecialEvent() will change this from lost to 0"。[定义](../../../raw/current/scripts/constants.lua#L1311)
2. SCIENCE=10 高于任何 builder 可得档：`KnowsRecipe` 逐维检查 `recipe.level[v] > bonusvalue` 即判否，配方锁定、提示 `NEEDSHALLOWED_NIGHTS`。[判定](../../../raw/current/scripts/components/builder.lua#L896-L907)
3. 活动期 `ApplyEvent` 把匹配事件的 `TECH[k].SCIENCE` 置 0（[ApplyEvent](../../../raw/current/scripts/simutil.lua#L624-L633)），0 在 `KnowsRecipe` 与 `CanPrototypeRecipe`（[widgetutil.lua#L16-L24](../../../raw/current/scripts/widgets/widgetutil.lua#L16-L24)）中都不再设限 → 活动期全体玩家无原型机可造。
4. 解锁在世界生成与读档各施加一次：生成时 `ApplySpecialEvent(current_gen_params.specialevent)`（[forest_map.lua](../../../raw/current/scripts/map/forest_map.lua#L455)）、读档时 `ApplySpecialEvent(savedata_overrides.specialevent or "default")`（[gamelogic.lua](../../../raw/current/scripts/gamelogic.lua#L368)）。两处随后都会遍历 `SPECIAL_EVENTS`，把设置里为 `enabled` 的**额外事件**再 `ApplyExtraEvent(event_name)`——所以除主事件外另开的额外活动同样把该科技档置 0（[生成侧](../../../raw/current/scripts/map/forest_map.lua#L456-L460)、[读档侧](../../../raw/current/scripts/gamelogic.lua#L365-L373)、[ApplyExtraEvent](../../../raw/current/scripts/simutil.lua#L649-L659)）；服务器设置为 default 时沿用常量默认 `WORLD_SPECIAL_EVENT = HALLOWED_NIGHTS`（[constants.lua](../../../raw/current/scripts/constants.lua#L858)）。事件常量 `"hallowed_nights"`（[constants.lua](../../../raw/current/scripts/constants.lua#L844)），活动判定 `IsSpecialEventActive`（[constants.lua](../../../raw/current/scripts/constants.lua#L1095-L1097)）。

另一获取链：地图 boon 布景在活动期把 candybag 放进 WoodBoon／GrassBoon／TwigsBoon 的 item_area 战利品（[boons.lua](../../../raw/current/scripts/map/boons.lua#L12)、[GrassBoon](../../../raw/current/scripts/map/boons.lua#L48)、[TwigsBoon](../../../raw/current/scripts/map/boons.lua#L66)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| inspectable／inventoryitem | 检视、拾取；`cangoincontainer=false` 禁止收入其他容器 | 主模拟；不阻止放入人物主物品栏 | [[reference/components/inventoryitem#关键接口]]；[配置](../../../raw/current/scripts/prefabs/candybag.lua#L77-L80) |
| equippable | BODY 槽；onequip 双符号（`backpack`＋`swap_body` 都从 candybag build 覆盖）＋`Open`，onunequip 清符号＋`Close`，onequiptomodel 仅 `Close` | 主模拟；入模型展示不开包 | [[reference/components/equippable#关键接口]]；[回调](../../../raw/current/scripts/prefabs/candybag.lua#L12-L26)、[绑定](../../../raw/current/scripts/prefabs/candybag.lua#L82-L87) |
| container | `WidgetSetup("candybag")`：两列七行 14 格、bank／build `ui_krampusbag_2x8`（与 krampus_sack 同款）、`issidewidget`、type `"pack"`、`openlimit=1`；`itemtestfn` 只收 `halloweencandy`／`halloween_ornament` 标签或 `trinket_` 前缀；`priorityfn` 同一函数 | 两端共享 containers.params；WidgetSetup 调用时现拷参数（[widgetsetup](../../../raw/current/scripts/containers.lua#L9-L17)）；服务端过滤 [container.lua](../../../raw/current/scripts/components/container.lua#L295)、Replica 过滤 [container_replica.lua](../../../raw/current/scripts/components/container_replica.lua#L373) | [[reference/components/container#新容器的客户端布局]]；[参数](../../../raw/current/scripts/containers.lua#L1741-L1764)、[同款布局对照](../../../raw/current/scripts/containers.lua#L1061-L1078) |
| container priorityfn 消费端 | 服务端 `ShouldPrioritizeContainer` 把匹配物品标记为优先送入本容器，并使本袋成为"专用口袋容器"（ValidateSpecializedContainer 要求 priorityfn）；Replica 同名判定 | 两端；priorityfn 原 与 itemtestfn 同一函数 | [服务端](../../../raw/current/scripts/components/container.lua#L308-L320)、[专用容器判定](../../../raw/current/scripts/components/container.lua#L789-L798)、[Replica](../../../raw/current/scripts/components/container_replica.lua#L386-L398) |
| burnable／propagator | `MakeSmallBurnable`（默认燃时 10 秒）＋`MakeSmallPropagator`，随后三回调全部覆盖：onignite 锁包、onextinguish 恢复、onburnt 掉物＋ash＋移除 | 主模拟；覆盖发生在 helper 之后，最终生效的是 candybag 自己的回调 | [[reference/components/burnable#生命周期]]；[[reference/components/propagator#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[覆盖](../../../raw/current/scripts/prefabs/candybag.lua#L92-L96) |
| hauntable／floater | 漂浮 `MakeInventoryFloatable("med", 0.125, 0.65)` 在 `SetPristine` **前**、公共端执行；作祟 `MakeHauntableLaunchAndDropFirstItem` 在**主模拟**分支（本 build 掉物分支注释，仅发射） | 两个 helper 的初始化端不同 | [[reference/components/hauntable#按效果选工厂]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1530)、[漂浮](../../../raw/current/scripts/prefabs/candybag.lua#L69) |
| API／数据：`backpack` 标签 | `SetPristine` 前加标，家族识别用；消费端（熊獾打劫、掉落保留、阿比盖尔花跳过监听）见 piggyback 页同节 | 公共端标签，两端可见 | [[reference/prefabs/piggyback#关联机制导航]]；[加标](../../../raw/current/scripts/prefabs/candybag.lua#L67) |
| 关联 Prefab：`ash` | onburnt 显式生成 | 燃尽时服务端生成；`prefabs` 依赖表同步声明 | [生成](../../../raw/current/scripts/prefabs/candybag.lua#L34-L36)、[声明](../../../raw/current/scripts/prefabs/candybag.lua#L7-L10) |
| 关联 Prefab：`halloweencandy_N` 家族（Wiki 待建） | itemtestfn 按 `halloweencandy` 标签接收 | 万圣节糖果，标签在家族共享工厂加注 | [加标](../../../raw/current/scripts/prefabs/halloweencandy.lua#L43)、[注册名](../../../raw/current/scripts/prefabs/halloweencandy.lua#L77) |
| 关联 Prefab：`halloween_ornament_N` 家族（Wiki 待建） | 按 `halloween_ornament` 标签接收 | 万圣节装饰球 | [加标](../../../raw/current/scripts/prefabs/halloween_ornaments.lua#L35)、[注册名](../../../raw/current/scripts/prefabs/halloween_ornaments.lua#L67) |
| 关联 Prefab：`trinket_N` 家族（Wiki 待建） | 按 prefab 前缀接收（`string.sub(prefab, 1, 8) == "trinket_"`） | 全部小玩意，无需标签 | [过滤](../../../raw/current/scripts/containers.lua#L1760-L1762) |
| 资源 | anim `candybag.zip`、UI bank／build `ui_krampusbag_2x8.zip`；foleysound `dontstarve/movement/foley/backpack` | 声明并实际用于 bank/build 与音效；未验证压缩包内容 | [资产](../../../raw/current/scripts/prefabs/candybag.lua#L1-L5)、[音效](../../../raw/current/scripts/prefabs/candybag.lua#L65) |

本文件未安装 SG／Brain，也无小地图图标；穿戴不减移速（equippable 回调只做外观与容器开关）。

## 典型接入

### 调整糖果袋接收条件

- 前提与选择：`modmain.lua` 顶部直接改 `require("containers")` 返回的共享表。服务端 container 与客户端 replica 都在模块顶层 require 同一模块（[container.lua](../../../raw/current/scripts/components/container.lua#L1)、[container_replica.lua](../../../raw/current/scripts/components/container_replica.lua#L1)），WidgetSetup 调用时从 `params[prefab]` 现拷字段，因此改一处即两端生效，无需 AddPrefabPostInit 双端接线；注册新键的替代做法见 [[reference/components/container#让原版背包只接收具有食物类型标签的物品]]。
- 生效与撤销：modmain 早于实体构造执行，改动对之后构造的全部 candybag 生效；组件字段在 WidgetSetup 后加只读保护（[container.lua](../../../raw/current/scripts/components/container.lua#L94-L96)），但模块表本身可写。撤销须恢复原函数并重载。
- 原版依据：itemtestfn／priorityfn 及其两端消费见上文[关联机制导航](#关联机制导航)。

```lua
-- modmain.lua：让糖果袋额外接收带 "benign" 标签的物品
local containers = GLOBAL.require("containers")
local previous_test = containers.params.candybag.itemtestfn
containers.params.candybag.itemtestfn = function(container, item, slot)
    return previous_test(container, item, slot) or item:HasTag("benign")
end
containers.params.candybag.priorityfn = containers.params.candybag.itemtestfn
```

- 调整与验证：`priorityfn` 原与 `itemtestfn` 是同一函数（[containers.lua](../../../raw/current/scripts/containers.lua#L1764)），替换后必须同步重指，否则自动入包的优先目标仍按旧判断。mod 环境默认无 `__index=_G`，`require` 须经 `GLOBAL` 调用。示例未在游戏内运行，机制依据上文源码审阅。

## 常见陷阱与限制

- **误区：糖果袋是普通背包的换皮或同工厂变体**：prefabs/candybag.lua 为独立工厂（103 行，不 require 任何背包 prefab 文件）；与家族的共享点只在 `backpack` 标签、`ui_krampusbag_2x8` 同款 UI 布局与通用 helper。家族对照见 [[reference/prefabs/backpack#关联机制导航]] 与 [[reference/prefabs/krampus_sack]]。
- **误区：装备糖果袋会自动收集或转化糖果**：onequip 只做外观覆盖与 Open（[回调](../../../raw/current/scripts/prefabs/candybag.lua#L12-L16)）；在**已核验的装备回调与 grep 检索范围**内（容器参数、配方、boon 战利品、scrapbook／debug 登记），没有发现穿戴触发的收集逻辑；糖果的进食行为属于 halloweencandy 家族自身（未检索引擎侧）。
- **误区：活动结束 TECH 会自动恢复锁定**：`ApplyEvent` 只有“把匹配档位 `SCIENCE` 置 0”这一条路径，脚本里没有对应的恢复分支（[simutil.lua](../../../raw/current/scripts/simutil.lua#L624-L633)），锁定态来自常量初始值 `{SCIENCE = 10}`。解锁发生在世界生成／读档时按 specialevent 与额外事件设置整体施加；同会话内切换活动是否恢复，本快照未核到恢复实现。
- **边界：燃尽掉物后本体成灰**：onburnt 先 DropEverything、Close，再生成 ash 并 Remove；掉出的可燃内容物若留在火场会继续烧掉。
- **边界：作祟不掉内容物**：MakeHauntableLaunchAndDropFirstItem 的掉物分支在本 build 被注释，实际只发射本体（与 [[reference/prefabs/backpack#常见陷阱与限制]] 同源）。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。全文实读 prefabs/candybag.lua（103 行）；核验注册名、公共端／主模拟分界、双符号穿戴回调、14 格过滤布局与 krampus_sack 同款对照（containers.lua L1741-L1764／L1061-L1078）、itemtestfn／priorityfn 两端消费（container.lua L295／L308-L320／L789-L798、container_replica.lua L373／L386-L398）、燃烧三回调与 helper 覆盖、活动档全链（constants.lua L844／L858／L1095-L1097／L1311、simutil.lua L624-L659、gamelogic.lua L368、forest_map.lua L455、widgetutil.lua L16-L24、builder.lua L896-L907）、boon 三处（boons.lua L12／L48／L66）、接受物品家族加标（halloweencandy.lua L43／L77、halloween_ornaments.lua L35／L67）。
- 名称核验：`chinese_s.po` 以 python 按 `msgctxt "STRINGS.NAMES.CANDYBAG"` 定位（L340095-L340097），msgid "Candy Bag" → "糖果袋"。
- 消费端检索：scripts/ 全目录 grep `candybag` 命中 10 个文件（containers.lua、debugcommands.lua、prefablist.lua、recipes.lua、recipes_filter.lua、scrapbook_prefabs.lua、screens/redux/scrapbookdata.lua、skins_defs_data.lua、map/boons.lua、prefabs/candybag.lua），无穿戴收集类消费端；prefabskins.lua／skinprefabs.lua 同法检索无 candybag 皮肤注册；未检索引擎侧（C++）。
- 游戏运行：未运行；未验证多人开关细节、活动切换的运行表现、资源压缩包内容与存档恢复。
- 结构检查：validate_page.py 通过（size_class standard，实际 101 行／15445 字节）；validate_source_links.py --strict 0 error 0 warning；validate_source_hashes.py check 未记录属预期。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
