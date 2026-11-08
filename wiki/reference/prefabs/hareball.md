---
title: "兔毛球（hareball）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "兔年活动里舒适的兔人作为奖品递出的 hareball：cattoy／renewable 标签、GOODIES 三维 0/0/0、6 天鲜度腐坏成 spoiled_food、可点燃并蔓延且作祟只会抛飞，以及 brain 生成与 SG 吃／丢交互。"
aliases: ["hareball", "兔毛球", "兔球", "舒适的兔人奖品"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hareball.lua", "prefabs/cozy_bunnyman.lua", "brains/cozy_bunnymanbrain.lua", "stategraphs/SGcozy_bunnyman.lua", "prefablist.lua", "tuning.lua", "constants.lua", "components/edible.lua", "components/eater.lua", "components/perishable.lua", "components/tradable.lua", "components/floater.lua", "components/propagator.lua", "components/inventoryitem.lua", "standardcomponents.lua", "prefabs/catcoon.lua", "prefabs/grass.lua", "components/forestresourcespawner.lua", "scrapbook_prefabs.lua", "actions.lua", "componentactions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 兔毛球（hareball）

## 快速判断

- 职责与适用：`prefabs/hareball.lua` 只注册一件 `hareball`，是兔年活动（YoTR 2023）里舒适的兔人给获胜玩家递上的奖品：可食（`FOODTYPE.GOODIES`，三维全 0）、6 天鲜度、腐坏成 `spoiled_food`；带 `cattoy` 与 `renewable` 标签；可点燃、会传播火焰，被作祟只会抛飞。[工厂](../../../raw/current/scripts/prefabs/hareball.lua#L10-L58)
- 优先使用：改数值看[数值与交互](#数值与交互)；要看它怎么被生成和递出看[来源与交互](#来源与交互)；要改食用收益看[典型接入](#典型接入)。
- 重要副作用：`MakeSmallBurnable` 加 `MakeSmallPropagator` 让它能起火并向外传热，火焰还会按 `damages = true`、距离 2 对附近「同时带 `propagator` 与 `health`」的目标结算烧伤（[传播器](../../../raw/current/scripts/components/propagator.lua#L219-L229)）；`MakeHauntableLaunchAndIgnite` 的点火分支在快照里被 `#HAUNTFIX` 注释掉，作祟实际只抛飞（[作祟](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)）。
- 修改边界：本页只写物品本身与它和兔人的交互关系；舒适的兔人本体（Brain 其余节点、活动流程、金块结算）Wiki 待建（源码入口 `prefabs/cozy_bunnyman.lua`、`brains/cozy_bunnymanbrain.lua`）；没有堆叠组件，每格只放 1 个。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hareball` | `fn()` 单工厂：物理与动画 → 两个标签 → `MakeInventoryFloatable(inst, "med", 0.05, 0.68)` → `SetPristine`；非主模拟在 `SetPristine` 之后返回，主模拟再挂组件 | 没有变体、没有皮肤分支。主模拟挂 `inventoryitem`、`edible`（`GOODIES`、0／0、未写理智）、`inspectable`、`tradable`（未写 `goldvalue`，用默认 0）、`perishable`（`PERISH_FAST`＋`spoiled_food`），再 `MakeSmallBurnable(SMALL_BURNTIME)`、`MakeSmallPropagator`、`MakeHauntableLaunchAndIgnite`，最后监听 `on_landed` 播 `yotr_2023/common/goop_place`。[工厂](../../../raw/current/scripts/prefabs/hareball.lua#L10-L58) |

两端的分界有一处例外：`AddSoundEmitter` 与 `MakeInventoryFloatable` 都在 `SetPristine` **之前**，所以客户端副本也有 `SoundEmitter` 与 `floater` 组件。但 `on_landed` 的监听写在主模拟分支之后，只挂在服务端实例上，落地音效由服务端触发（事件来自 `inventoryitem:SetLanded`）；其余组件也只在主模拟。[工厂头](../../../raw/current/scripts/prefabs/hareball.lua#L13-L33)、[落地回调](../../../raw/current/scripts/prefabs/hareball.lua#L6-L8)、[事件源](../../../raw/current/scripts/components/inventoryitem.lua#L513-L536)

`stackable` 从未挂载：一件占一格，堆叠上限就是 1。`stackable` 组件的 20 是**挂载后**的默认 `maxsize`，本件没有该组件，所以不适用。[工厂](../../../raw/current/scripts/prefabs/hareball.lua#L35-L43)、[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)

### 名称核验

`hareball` 的简中实显是**兔毛球**（Hare Ball），键命中一次，注册名与 `STRINGS.NAMES.HAREBALL` 同名；采集册把该条目注释掉（`--["hareball"]=true`），所以游戏内采集册查不到它。[HAREBALL](../../../raw/current/scripts/languages/chinese_s.po#L343735-L343737)、[采集册清单](../../../raw/current/scripts/scrapbook_prefabs.lua#L180)

## 数值与交互

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度按 480 秒／天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47)

| 项 | 值 | 证据 |
|---|---|---|
| 饥饿／生命／理智 | 0／0／0（理智未写，用 `edible` 默认 0） | [赋值](../../../raw/current/scripts/prefabs/hareball.lua#L37-L40)、[默认](../../../raw/current/scripts/components/edible.lua#L19-L44) |
| 食物类型 | `FOODTYPE.GOODIES`，同时属于 `FOODGROUP.OMNI` 与 `FOODGROUP.VEGETARIAN` 的可食类型 | [类型](../../../raw/current/scripts/constants.lua#L2042)、[分组](../../../raw/current/scripts/constants.lua#L2049-L2108) |
| 鲜度 | `PERISH_FAST`＝6 天＝2880 秒，腐坏替换 `spoiled_food` | [赋值](../../../raw/current/scripts/prefabs/hareball.lua#L45-L48)、[常量](../../../raw/current/scripts/tuning.lua#L2043-L2051) |
| 堆叠 | 无 `stackable` 组件，每格 1 个 | [工厂](../../../raw/current/scripts/prefabs/hareball.lua#L35-L43) |
| 漂浮 | `med`、垂直偏移 0.05、缩放 0.68，两端同参 | [漂浮](../../../raw/current/scripts/prefabs/hareball.lua#L27)、[默认值](../../../raw/current/scripts/components/floater.lua#L30-L45) |
| 交易值 | 未写，`tradable` 默认 0，猪王不收 | [组件默认](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 燃烧 | 小火，燃烧时长 `SMALL_BURNTIME`＝7.5 秒；会传播火焰并结算烧伤 | [燃烧](../../../raw/current/scripts/prefabs/hareball.lua#L50)、[常量](../../../raw/current/scripts/tuning.lua#L1681)、[传播器默认](../../../raw/current/scripts/standardcomponents.lua#L200-L213) |

采食判定走 `eater` 的默认 diet：`caneat = { FOODGROUP.OMNI }`，`GOODIES` 在 OMNI 的类型表里，所以玩家与吃 OMNI 的生物都能吃；三维全 0，赋值不产生负值，因此不会带 `badfood`。「能喂给其他玩家」只解决食品安全这一层：`spoiled`／`badfood`／`unsafefood` 会被喂食动作的可用性判定挡掉，旁路有三条：目标带 `possessedbody`、PVP 开启、或目标带 `ignoresspoilage` 且物品不是 `badfood`／`unsafefood`（兔毛球三维非负、不会带这两个标签，所以腐坏的那份仍可喂给这类目标）；前两条跳过的只是食品安全这一层（[安全门槛](../../../raw/current/scripts/actions.lua#L2397-L2410)、[动作可用性](../../../raw/current/scripts/componentactions.lua#L1164-L1171)）；目标还须处于可被喂的状态（idle、不在忙／攻击／睡觉、非 `wereplayer`，骑乘关系也要允许），且其食性组接受 `edible_GOODIES`。[吃者默认](../../../raw/current/scripts/components/eater.lua#L22-L24)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 来源与交互

| 环节 | 行为 | 证据 |
|---|---|---|
| 生成 | 舒适的兔人在 `carrotgamestatus == "prizedeliver"` 时 `SpawnPrefab("hareball")`，`GiveItem` 给自己，并用 `entitytracker` 记为 `"goop"` | [brain](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L531-L538) |
| 递出 | 目标玩家记在 `goop.yotr_targeteater`，并监听 `oneaten`：**计时器 `yotr_waitforplayertoeat` 仍在、且 `data.eater` 就是该目标**才推进到 `givenuggets`；`goopeaten(inst)` 写在 `if` 之外，无条件执行（结束本局）。球还在兔人手上（`goop.components.inventoryitem.owner == inst`）时它就把球 `Drop` 向获胜玩家，没有获胜者则丢自己脚边 | [判定](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L540-L548)、[丢球](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L553-L564) |
| 吃回 | 当 `eat` 的目标是 `hareball` 时，SG 把该 buffered action 存进 `statemem.barf`：4 帧后先塞回背包，20 帧后再 `PushEvent("oneaten", {eater = 兔人})` 并 `Remove` 球，随即转 `disgust`——球最终被销毁。原版兔人的 diet 只认 `VEGGIE`／`RAW`，`eater:CanEat` 对 `GOODIES` 为假，玩家喂不进去，这条分支只在别的入口对球发 `EAT` 时生效 | [eat 状态](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L439-L456)、[20 帧](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L467-L475)、[diet](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L439)、[接受判定](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L43-L52) |
| 丢弃被嫌弃 | 兔人自己执行 `DROP`（把奖品丢向玩家，见上格）时，`DROP` 被映射到 `pickup` 状态，于是播 `disgust` 动画与 `bunnyman_disgust` 音效，而不是普通拾取 | [映射](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L14)、[pickup 状态](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L865-L871) |
| 依赖声明 | 兔人的 `prefabs` 表列了 `hareball`，只保证预载，不等于生成 | [依赖](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L15-L23) |

`cattoy` 让它与「猫粮」区分开：浣猫的 `ShouldAcceptItem` 对 `cattoy`／`catfood`／`cattoyairborne` 任一标签都返回真，所以兔毛球是可被浣猫收下的进贡物，与它能不能吃无关。[浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | `GOODIES`、饥饿 0、生命 0；理智与 `ismeat` 都未写 | 仅主模拟 | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/hareball.lua#L37-L40) |
| 组件：perishable | `SetPerishTime(PERISH_FAST)`＋`StartPerishing`，腐坏替换成 `spoiled_food` | 仅主模拟；到点 `SpawnPrefab` 替换并按原堆叠数给予（本件恒为 1） | [[reference/components/perishable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/hareball.lua#L45-L48)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L302) |
| 组件：inventoryitem | 未设 `SetSinks`、未设丢弃回调：`sinks` 用默认 `false`，掉进水里不会沉 | 仅主模拟 | [[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hareball.lua#L35)、[默认](../../../raw/current/scripts/components/inventoryitem.lua#L75) |
| 组件：floater | `med` 尺寸、偏移 0.05、缩放 0.68 | `MakeInventoryFloatable` 在 `SetPristine` 之前，两端都有 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/hareball.lua#L27) |
| 组件：burnable／propagator | `MakeSmallBurnable(SMALL_BURNTIME)` 加 `MakeSmallPropagator`：小火、7.5 秒、能向外传热并烧伤 | 主模拟；烧伤只打「同时有 `propagator` 与 `health`」且 `vulnerabletoheatdamage ~= false` 的目标 | [[reference/components/burnable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/hareball.lua#L50-L51)、[伤害条件](../../../raw/current/scripts/components/propagator.lua#L219-L229) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`：作祟只 `Launch` | 点火分支被注释，函数名与行为不一致 | [[reference/components/hauntable#关键接口]]；[作祟](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 标签：`cattoy` | 浣猫 `ShouldAcceptItem` 直接接受 | 标签两端都有；接受判定在服务端（`trader:SetAcceptTest` 挂在主模拟段） | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| 标签：`renewable` | 快照内没有 `HasTag("renewable")` 的直接消费者；`ForestResourceSpawner` 只把它当 `FindEntities` 的粗筛，随后仍按 `matches` 里的 prefab 名比对 | 同标签也出现在割下的草、树枝、燧石等物上 | [粗筛](../../../raw/current/scripts/components/forestresourcespawner.lua#L73-L113) |
| 关联 Prefab：`spoiled_food` | 腐坏产物 | 到点替换 | [赋值](../../../raw/current/scripts/prefabs/hareball.lua#L48) |
| 资源：`anim/hareball.zip` | bank 与 build 都是 `hareball`，单动画 `idle` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/hareball.lua#L1-L4) |

## 典型接入

### 让奖品吃下去有点收益

- 前提与选择：服务端 `modmain.lua` 的 postinit。奖品原本三维全 0，只在活动流程里当令牌用；要它顺便回血加饥饿就走 `edible` 字段，不要改 `FOODTYPE`（改成 `MEAT` 等会同时改 `edible_*` 标签与谁愿意吃）。
- 生效与撤销：只影响之后生成的主模拟副本；已经拿在手里的旧物品不回溯。卸模组后新生成的仍为 0／0。
- 原版依据：`edible` 的 `hungervalue`／`healthvalue` 是普通字段，构造时直接赋值；`sanityvalue` 保持 0 才不会因负值被自动加上 `badfood`。[赋值](../../../raw/current/scripts/prefabs/hareball.lua#L37-L40)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7)

```lua
-- modmain.lua：兔毛球吃了回 12.5 饥饿、1 生命
AddPrefabPostInit("hareball", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local edible = inst.components.edible
    if edible ~= nil then
        edible.hungervalue = GLOBAL.TUNING.CALORIES_SMALL
        edible.healthvalue = GLOBAL.TUNING.HEALING_TINY
    end
end)
```

- 调整与验证：`CALORIES_SMALL`＝12.5、`HEALING_TINY`＝1；实测吃一口看饥饿条与血条，并确认物品没有带上 `badfood`。

## 常见陷阱与限制

- **误区：`MakeHauntableLaunchAndIgnite` 会把它点着。** 点火那段在 `#HAUNTFIX` 注释里，作祟只抛飞并给 `HAUNT_TINY`。[作祟](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **误区：带 `renewable` 标签就等于可再生。** 快照内没有 `HasTag("renewable")` 的消费者；资源再生器只拿它做 `FindEntities` 的粗筛，判定仍按 `matches` 里的具体 prefab 名，割草变蜥蜴的判定还额外要求 `v.prefab == "grass"`。[再生器](../../../raw/current/scripts/components/forestresourcespawner.lua#L73-L113)、[割草](../../../raw/current/scripts/prefabs/grass.lua#L32-L61)
- **边界：小火也会伤人。** `MakeSmallPropagator` 设了 `damages = true`、`damagerange` 2，烧伤按「目标有 `propagator` 与 `health`」结算；把它堆在可燃物旁边会连营。[传播器](../../../raw/current/scripts/standardcomponents.lua#L200-L213)、[伤害条件](../../../raw/current/scripts/components/propagator.lua#L219-L229)
- **边界：吃掉它是流程的一部分。** 三维全 0 不代表没用：兔人靠 `oneaten` 推进阶段，而且只有 `data.eater` 等于获胜玩家、计时器还在时才发金块；所以「谁来吃」是判定的一部分，`yotr_waitforplayertoeat` 的兜底分支本页未核。[brain](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L540-L548)
- **边界：没堆叠。** 一件一格，别按 20 或 40 规划背包。[工厂](../../../raw/current/scripts/prefabs/hareball.lua#L35-L43)
- **未核验**：本页未运行游戏；活动流程只按 Brain 与 SG 源码读出，未实测整局兔年奖品链。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/hareball.lua` 全文；核对 `brains/cozy_bunnymanbrain.lua` 的奖品生成与递出、`stategraphs/SGcozy_bunnyman.lua` 的 `eat`／`pickup` 状态、`prefabs/cozy_bunnyman.lua` 的依赖表；`edible`／`eater`／`perishable`／`tradable`／`floater`／`propagator`／`inventoryitem` 的相关分支与 `standardcomponents.lua` 的三个 helper；`constants.lua` 的 `FOODTYPE.GOODIES` 与 `FOODGROUP`；`tuning` 的鲜度与燃烧时长；`prefabs/catcoon.lua` 的接受条件、`prefabs/grass.lua` 与 `components/forestresourcespawner.lua` 的 `renewable` 用法。
  名称核验：`languages/chinese_s.po` 的 `NAMES.HAREBALL`＝兔毛球；采集册条目被注释，未额外查动态选名。
  未覆盖：兔人的 Brain 其余节点、活动金块结算与刷新；`yotr` 其它物品；模组注入。
- 游戏运行：未运行。天数、秒数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/eater]]、[[reference/components/perishable]]
- 火与作祟：[[reference/components/burnable]]、[[reference/components/hauntable]]
- 携带与漂浮：[[reference/components/inventoryitem]]、[[reference/components/floater]]

返回 [[reference/prefabs/index|Prefab 索引]]。
