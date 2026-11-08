---
title: "钓鱼与水上工具族（桨／淡水钓竿／海钓竿／渔网）"
created: "2026-11-07"
updated: "2026-11-10"
type: prefab
summary: "四个源码文件注册的手持水上工具：船桨（ROW 动作推动船）、淡水钓竿（对鱼群 FISH/REEL）、海钓竿（浮子＋鱼饵的海洋钓鱼系统）与渔网（CAST_NET 拖网），覆盖注册清单、动作链、耐久、饵具附属与获取途径。"
aliases: ["oar", "oar_driftwood", "malbatross_beak", "oar_monkey", "yotd_oar", "fishingrod", "oceanfishingrod", "fishingnet", "桨", "船桨", "浮木桨", "战桨", "邪天翁喙", "利爪船桨", "淡水钓竿", "海钓竿", "渔网", "鱼竿", "钓竿", "划桨", "钓鱼工具", "船桨耐久", "海洋钓鱼", "拖网"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/oar.lua", "prefabs/fishingrod.lua", "prefabs/oceanfishingrod.lua", "prefabs/fishingnet.lua", "debugcommands.lua", "prefabs/oceanfishingbobber.lua", "prefabs/oceanfishinglure.lua", "prefabs/fishingnetvisualizer.lua", "prefabs/hats.lua", "prefabs/malbatross.lua", "components/oar.lua", "components/fishingrod.lua", "components/oceanfishingrod.lua", "components/fishingnet.lua", "components/fishingnetvisualizer.lua", "components/oceanfishable.lua", "components/oceanfishinghook.lua", "components/equippable.lua", "components/piratespawner.lua", "components/playercontroller.lua", "components/playeractionpicker.lua", "containers.lua", "componentactions.lua", "actions.lua", "tuning.lua", "recipes.lua", "constants.lua", "prefabskins.lua", "modutil.lua", "mods.lua", "componentutil.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 钓鱼与水上工具族（桨／淡水钓竿／海钓竿／渔网）

## 快速判断

- 职责与适用：四个 prefab 文件各自注册的手持水上工具，均为可拾取装备（HANDS 槽，[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)）。`oar.lua` 五个船桨（ROW／ROW_FAIL／ROW_CONTROLLER 推动所在船）；`fishingrod.lua` 淡水钓竿（对 `fishable` 目标 FISH／REEL）；`oceanfishingrod.lua` 海钓竿（容器装浮子＋鱼饵，OCEAN_FISHING_CAST 抛投、收线张力系统）；`fishingnet.lua` 渔网（CAST_NET 生成拖网实体收集漂浮物品）。[桨注册](../../../raw/current/scripts/prefabs/oar.lua#L181-L185)、[钓竿注册](../../../raw/current/scripts/prefabs/fishingrod.lua#L103)、[海钓竿注册](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L213)、[渔网注册](../../../raw/current/scripts/prefabs/fishingnet.lua#L83)
- 优先使用：改船桨推力／速度查[数值表](#注册与构造)与 oar 组件（[[reference/components/oar]]）；复刻淡水钓鱼链用 fishingrod 组件（[[reference/components/fishingrod]]）；海洋钓鱼系统查[海钓竿机制索引](#海钓竿海洋钓鱼链)；造同类手持工具查 [[cookbook/equipment/hand]]。
- 重要副作用：船桨 RowFail 按磨损扣耐久并淋湿周围实体；海钓竿抛投/中鱼会打湿容器内饵具、断线销毁全部饵具；渔网拖网会把收集物丢在收网者脚下。[RowFail](../../../raw/current/scripts/components/oar.lua#L35-L51)、[断线销毁](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L108-L116)、[拖网收集](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L114)
- 修改边界：公共端打标签与漂浮外观，组件在主模拟挂载；`fishingrod` 在 quagmire 游戏模式走 event_server_data 替换（无武器/耐久）；渔网 `fishingnet` 无配方、常规玩法无生成链（仅调试上架清单收录，见[陷阱](#常见陷阱与限制)）；海钓竿附属饵具本页只给注册清单与机制索引，鱼群 AI／charm 细节见 [[reference/prefabs/ocean-fish]]。[quagmire 分支](../../../raw/current/scripts/prefabs/fishingrod.lua#L77-L89)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oar`（桨） | `fn(TUNING.BOAT.OARS.BASIC, "oar", "swap_oar", TUNING.MED_FUEL, true)` | 木桨：edible（FOODTYPE.WOOD）、fuel 45 秒；force 0.3／max_velocity 2。 [工厂](../../../raw/current/scripts/prefabs/oar.lua#L151-L153)、[BASIC 表](../../../raw/current/scripts/tuning.lua#L4194-L4202) |
| `oar_driftwood`（浮木桨） | `fn(...DRIFTWOOD, "oar_driftwood", "swap_oar_driftwood", MED_FUEL, true, true)` | 第六参 `is_waterproof`：pristine 打 `waterproofer` 标签＋主模拟挂 waterproofer（effectiveness 0）；force 0.5／vel 3.5／耐久 400。 [工厂](../../../raw/current/scripts/prefabs/oar.lua#L155-L157)、[防水分支](../../../raw/current/scripts/prefabs/oar.lua#L113-L116) |
| `malbatross_beak`（邪天翁喙） | `fn(...MALBATROSS, "malbatross_beak", "swap_malbatross_beak", nil, nil)` | 非木（无 edible/fuel）、不防水；force 0.8／vel 5／耐久 1500／RowFail 磨损 6。 [工厂](../../../raw/current/scripts/prefabs/oar.lua#L159-L161)、[MALBATROSS 表](../../../raw/current/scripts/tuning.lua#L4214-L4222) |
| `oar_monkey`（战桨） | `fn(...MONKEY, "oar_monkey", "swap_oar_monkey", MED_FUEL, true)` | 木桨；damage `wilson_attack`×1.5=51、attackwear 5、vel 3。 [工厂](../../../raw/current/scripts/prefabs/oar.lua#L163-L165)、[MONKEY 表](../../../raw/current/scripts/tuning.lua#L4224-L4232) |
| `yotd_oar`（利爪船桨） | `fn(...DRIFTWOOD, "yotd_oar", "yotd_oar", MED_FUEL, true, true)` 后覆盖 equippable 回调 | 数值同浮木桨；装备 symbol 用 `"swap_oar"`（build 为 `yotd_oar`）。 [覆盖](../../../raw/current/scripts/prefabs/oar.lua#L167-L179) |
| `fishingrod`（淡水钓竿） | `fn()`：fishingrod 组件 SetWaitTimes(4,40)／SetStrainTimes(0,5)；非 quagmire 挂 weapon＋finiteuses | finiteuses 9 次，监听 `fishingcollect` 每次收鱼 Use(1)，耗尽 `inst.Remove`；quagmire 走 event_server_data。 [组件设置](../../../raw/current/scripts/prefabs/fishingrod.lua#L73-L89)、[onfished](../../../raw/current/scripts/prefabs/fishingrod.lua#L32-L36) |
| `oceanfishingrod`（海钓竿） | `fn()`：oceanfishingrod 组件 SetDefaults（默认浮子投射物 `oceanfishingbobber_none_projectile`、BASE 抛投、HOOK 裸钩）＋container（WidgetSetup `oceanfishingrod`） | 无 finiteuses（无使用耐久）；容器 2 槽：槽 1 浮子、槽 2 鱼饵（itemtestfn 按 `oceanfishing_bobber`／`oceanfishing_lure` 标签）。 [SetDefaults](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L181-L186)、[容器参数](../../../raw/current/scripts/containers.lua#L1318-L1346) |
| `fishingnet`（渔网） | `fn()`：finiteuses 22 次、fishingnet 组件 | 消耗挂 ACTIONS.CAST_NET=1；burnable 组件但 `canlight=false`；MiniMap 图标 `boat_net.png`；assets 用 PKGREF（注释标注 unused）。 [finiteuses](../../../raw/current/scripts/prefabs/fishingnet.lua#L59-L63)、[PKGREF](../../../raw/current/scripts/prefabs/fishingnet.lua#L3-L5) |

四个文件共 8 个主注册。桨族共同工厂 `fn(data, build, swap_build, fuel_value, is_wooden, is_waterproof)`（[oar.lua#L65-L149](../../../raw/current/scripts/prefabs/oar.lua#L65-L149)）：实体组件＋`MakeInventoryPhysics` → bank/build → 防水标签（按参）→ `MakeInventoryFloatable` → `scrapbook_specialinfo="PADDLE"` → SetPristine（非主模拟返回）→ 主模拟：edible（木桨）→ inventoryitem → oar 组件（force/max_velocity）→ inspectable → waterproofer（effectiveness 0）→ weapon（damage/attackwear 按 data）→ equippable（onequip 覆盖 `swap_object`、显隐 ARM_carry，[皮肤分支](../../../raw/current/scripts/prefabs/oar.lua#L31-L42)）→ MakeSmallBurnable/Propagator → fuel（木桨）→ finiteuses（ROW／ROW_CONTROLLER 消耗 1、ROW_FAIL 消耗 `ROW_FAIL_WEAR`、`SetModifyUseConsumption` 船长减半）→ MakeHauntableLaunch。四注册名均带 `allow_action_on_impassable` 标签，让玩家可对不可通行点（水面）发起动作：[playercontroller 消费](../../../raw/current/scripts/components/playercontroller.lua#L5405)、[actionpicker 消费](../../../raw/current/scripts/components/playeractionpicker.lua#L463)。

名称键经简中 .po `msgctxt` 核验：`NAMES.OAR`=桨、`NAMES.OAR_DRIFTWOOD`=浮木桨、`NAMES.OAR_MONKEY`=战桨、`NAMES.MALBATROSS_BEAK`=邪天翁喙、`NAMES.YOTD_OAR`=利爪船桨、`NAMES.FISHINGROD`=淡水钓竿、`NAMES.OCEANFISHINGROD`=海钓竿、`NAMES.FISHINGNET`=渔网。[OAR](../../../raw/current/scripts/languages/chinese_s.po#L346220-L346222)、[OAR_DRIFTWOOD](../../../raw/current/scripts/languages/chinese_s.po#L346225-L346227)、[OAR_MONKEY](../../../raw/current/scripts/languages/chinese_s.po#L346230-L346232)、[MALBATROSS_BEAK](../../../raw/current/scripts/languages/chinese_s.po#L344970-L344972)、[YOTD_OAR](../../../raw/current/scripts/languages/chinese_s.po#L352985-L352987)、[FISHINGROD](../../../raw/current/scripts/languages/chinese_s.po#L342835-L342837)、[OCEANFISHINGROD](../../../raw/current/scripts/languages/chinese_s.po#L346370-L346372)、[FISHINGNET](../../../raw/current/scripts/languages/chinese_s.po#L342830-L342832)

数值（tuning 实测；`wilson_attack`=34×乘数 1，[tuning.lua#L37](../../../raw/current/scripts/tuning.lua#L37)；桨表 [BOAT.OARS](../../../raw/current/scripts/tuning.lua#L4192-L4233)）：

| 注册名 | 装备槽 | 推力 FORCE | 最大桨速 MAX_VELOCITY | 耐久 USES | 每 ROW 消耗 | ROW_FAIL 磨损 | 武器伤害 | 攻击磨损 |
|---|---|---|---|---|---|---|---|---|
| `oar` | HANDS | 0.3 | 2 | 500 | 1 | 25 | 17（×0.5） | 25 |
| `oar_driftwood` | HANDS | 0.5 | 3.5 | 400 | 1 | 25 | 17 | 25 |
| `malbatross_beak` | HANDS | 0.8 | 5 | 1500 | 1 | 6 | 27.2（×0.8） | 6 |
| `oar_monkey` | HANDS | 0.6 | 3 | 500 | 1 | 25 | 51（×1.5） | 5 |
| `yotd_oar` | HANDS | 0.5 | 3.5 | 400 | 1 | 25 | 17 | 25 |

装备 `master_crewman` 标签（猴子小帽授予，[hats.lua#L2778-L2786](../../../raw/current/scripts/prefabs/hats.lua#L2778-L2786)）时每次划桨消耗 ×`MASTER_CREWMAN_MULT.OAR_CONSUMPTION`=0.5（[修改函数](../../../raw/current/scripts/prefabs/oar.lua#L57-L63)、[调参](../../../raw/current/scripts/tuning.lua#L6752-L6758)）。

钓竿/渔网耐久：`fishingrod` 9（[FISHINGROD_USES](../../../raw/current/scripts/tuning.lua#L170)）、`fishingnet` 22（[FISHING_NET_USES](../../../raw/current/scripts/tuning.lua#L4048)）、`oceanfishingrod` 无使用耐久（仅攻击磨损 4，无 finiteuses，[组件区](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L188-L190)）。武器伤害三竿同值 `FISHINGROD_DAMAGE`=`wilson_attack`×0.125=4.25（[调参](../../../raw/current/scripts/tuning.lua#L371)）。

获取途径（配方行均实读 [recipes.lua](../../../raw/current/scripts/recipes.lua#L614-L615)）：

| 注册名 | 配方与科技 | 其他生成点 |
|---|---|---|
| `oar` | 原木 1，`TECH.NONE`（[L614](../../../raw/current/scripts/recipes.lua#L614)） | — |
| `oar_driftwood` | 浮木 1（[[reference/prefabs/driftwood-livinglog]]），`TECH.NONE`（[L615](../../../raw/current/scripts/recipes.lua#L615)） | — |
| `oar_monkey` | 无配方 | 海盗船生成时 SpawnPrefab（[piratespawner.lua#L306](../../../raw/current/scripts/components/piratespawner.lua#L306)） |
| `malbatross_beak` | 无配方 | 邪天翁战利品表 100%（[malbatross.lua#L405](../../../raw/current/scripts/prefabs/malbatross.lua#L405)） |
| `yotd_oar` | 金块 1，`TECH.DRAGONOFFERING_THREE`＋`nounlock`＋`actionstr="PERDOFFERING"`（蜻蜓年活动供奉，[L1107](../../../raw/current/scripts/recipes.lua#L1107)） | — |
| `fishingrod` | 树枝 2＋丝绸 2，`TECH.SCIENCE_ONE`（[L641](../../../raw/current/scripts/recipes.lua#L641)） | — |
| `oceanfishingrod` | 木板 1＋丝绸 3，`TECH.SCIENCE_ONE`（[L642](../../../raw/current/scripts/recipes.lua#L642)） | — |
| `fishingnet` | 无配方 | 无常规生成点；仅调试上架清单收录（见陷阱） |

海钓竿附属饵具注册清单（同批注册、独立 prefab，行为页 Wiki 待建；本页只列注册与入口）：

- 浮子（[BOBBERS 定义表](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L2-L16)，每项注册 `<名>_projectile`＋`<名>_floater` 两个 prefab，`make_inv_item` 者另注册物品本体）：`oceanfishingbobber_none`／`_twig`／`_plug`（仅 projectile+floater）与 `oceanfishingbobber_ball`／`_oval`／`_crow`／`_robin`／`_robin_winter`／`_canary`／`_goose`／`_malbatross`（三件套），共 30 个注册名。
- 鱼饵（[LURES 定义表](../../../raw/current/scripts/prefabs/oceanfishinglure.lua#L9-L37)，循环注册本体，[L106-L120](../../../raw/current/scripts/prefabs/oceanfishinglure.lua#L106-L120)）：`oceanfishinglure_spoon_red/_green/_blue`、`oceanfishinglure_spinner_red/_green/_blue`、`oceanfishinglure_hermit_rain/_snow/_drowsy/_heavy`，另有 WIP 注释组 `oceanfishinglure_spoon_brown/_yellow/_silver`、`oceanfishinglure_spinner_orange/_yellow/_white`（同样注册），共 16 个；浆果/种子类饵直接用食物物品（style 映射见注释）。
- 依赖声明：`oceanfishingrod` 的 prefabs 数组列 `oceanfishingbobber_none_projectile`（[oceanfishingrod.lua#L7-L10](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L7-L10)）；`fishingnet` 列 `fishingnetvisualizer`（[fishingnet.lua#L8-L11](../../../raw/current/scripts/prefabs/fishingnet.lua#L8-L11)，注册于 [fishingnetvisualizer.lua#L45](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L45)）。

饵具配方（`TECH.FISHING_ONE`，[recipes.lua#L645-L664](../../../raw/current/scripts/recipes.lua#L645-L664)）：浮子按对应材料 1（球浮子原木、椭圆浮子浮木、各羽毛浮子对应羽毛）；亮片饵燧石 2＋对应蘑菇 1；复合饵燧石 1＋对应蘑菇 1＋牛毛 1；隐者饵（`TECH.LOST`）饼干切割壳 1＋对应材料 1。皮肤：`fishingrod` 3 款、`oar`／`oar_driftwood` 各 2 款、`oceanfishingrod` 4 款（[prefabskins.lua#L866-L870](../../../raw/current/scripts/prefabskins.lua#L866-L870)、[L1358-L1381](../../../raw/current/scripts/prefabskins.lua#L1358-L1381)）；`fishingnet` 无皮肤条目。

## 关联机制导航

### 桨与划船动作链（oar 组件）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：oar` | 挂 force/max_velocity；Row 按持桨者平台调 `boatphysics:ApplyRowForce`，推 `rowing`／`rowed` 事件 | 主模拟；需站在带 `boat` 标签的平台上 | [[reference/components/oar]]；[Row](../../../raw/current/scripts/components/oar.lua#L11-L33) |
| `组件：finiteuses` | ROW/ROW_CONTROLLER 消耗 1、ROW_FAIL 消耗 data.ROW_FAIL_WEAR、`SetModifyUseConsumption` 按 master_crewman 折半 | 主模拟；耗尽推 `toolbroke` 并 Remove | [[reference/components/finiteuses]]；[消耗设置](../../../raw/current/scripts/prefabs/oar.lua#L137-L144) |
| `组件：weapon／equippable／edible／fuel／waterproofer` | 伤害与磨损按 OARS 表；装备换手持外观；木桨可作木头食物（0 血 0 饱食）与燃料（MED_FUEL=45 秒，[seg_time=30](../../../raw/current/scripts/tuning.lua#L19)、[MED_FUEL](../../../raw/current/scripts/tuning.lua#L1675)）；浮木系防水组件效果 0 | 装备槽 HANDS；防水仅标签＋0 效果组件 | [[reference/components/weapon]]、[[reference/components/equippable]]、[[reference/components/edible]]、[[reference/components/fuel]]、[[reference/components/waterproofer]] |
| `API／数据：ACTIONS.ROW/ROW_FAIL/ROW_CONTROLLER` | componentactions 对 oar 组件给动作候选：需平台为船、目标点在船外，ROW_FAIL 按动画帧窗口判定；fn 调 `oar:Row`／`oar:RowFail` | 客户端给候选、服务端执行；`overriderowaction` 目标（如拖网渔船）近距覆盖 | [候选逻辑 Row()](../../../raw/current/scripts/componentactions.lua#L15-L47)、[CheckRowOverride](../../../raw/current/scripts/componentactions.lua#L102-L124)、[ROW_FAIL.fn](../../../raw/current/scripts/actions.lua#L1341-L1354)、[ROW.fn](../../../raw/current/scripts/actions.lua#L1356-L1384) |
| `关联 Prefab：玩家 SG` | RowFail 推 `working` 事件（湿手掉武器链消费）；Row 推 `rowing` | 消费端为玩家 SG/其他系统，本页未展开 | 待建（玩家 SG 页）；[RowFail 淋湿](../../../raw/current/scripts/components/oar.lua#L35-L51) |

### 淡水钓鱼链（fishingrod 组件）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：fishingrod` | SetWaitTimes(4,40)（咬钩等待随鱼群剩余量线性取值）、SetStrainTimes(0,5)（钩住后断竿时间随耐久取值）；StartFishing 需目标有 fishable 组件 | 主模拟；replica 同步 target/hookedfish/caughtfish | [[reference/components/fishingrod]]；[等待计算](../../../raw/current/scripts/components/fishingrod.lua#L112-L124)、[Hook](../../../raw/current/scripts/components/fishingrod.lua#L146-L157)、[Replica 挂钩](../../../raw/current/scripts/components/fishingrod.lua#L1-L34) |
| `组件：finiteuses` | 不按动作消耗，监听自身 `fishingcollect` 事件每次收鱼 Use(1)；耗尽 Remove | 主模拟 | [onfished](../../../raw/current/scripts/prefabs/fishingrod.lua#L32-L36)、[Collect 推事件](../../../raw/current/scripts/components/fishingrod.lua#L187-L201) |
| `API／数据：ACTIONS.FISH/REEL` | componentactions 对 `fishable` 目标给 FISH（新目标）或 REEL（已抛竿）；REEL 按 hasHookedFish→Reel／nibble 状态→Hook／否则 StopFishing 分流 | 目标需 `fishable` 标签；`fishingrod` pristine 标签为优化 | [候选](../../../raw/current/scripts/componentactions.lua#L2339-L2347)、[FISH.fn](../../../raw/current/scripts/actions.lua#L1888-L1894)、[REEL.fn](../../../raw/current/scripts/actions.lua#L1896-L1921) |
| `API／数据：ACTIONS.FISH_OCEAN` | 持竿对水面点候选该动作，服务端 fn 直接失败并返回 "TOODEEP" | 仅右键点动作（[point 段](../../../raw/current/scripts/componentactions.lua#L2122-L2126)） | [FISH_OCEAN.fn](../../../raw/current/scripts/actions.lua#L1884-L1886) |

### 海钓竿（海洋钓鱼链）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：oceanfishingrod` | Cast 生成投射物浮子并设为 target；OnUpdate 维持线距/张力；Reel 收线（高张力+达 `REELING_SNAP_TENSION` 0.92 断线）；CatchFish 经 `oceanfishable:MakeProjectile` 弹出鱼 | 主模拟；SetDefaults 注入默认浮子投射物与裸钩 [HOOK](../../../raw/current/scripts/tuning.lua#L294)（charm 0.1／radius 1） | [[reference/components/oceanfishingrod]]；[Cast](../../../raw/current/scripts/components/oceanfishingrod.lua#L168-L188)、[Reel](../../../raw/current/scripts/components/oceanfishingrod.lua#L230-L258)、[CatchFish](../../../raw/current/scripts/components/oceanfishingrod.lua#L337-L361) |
| `组件：container` | WidgetSetup `oceanfishingrod`：槽 1 浮子、槽 2 鱼饵，`type="hand_inv"`，不接受堆叠；装备时 Open、卸下/抛竿 Close，收鱼/断线按情况重开或销毁内容 | 主模拟；`stay_open_on_hide=true`、`canbeopened=false`（由装备流程控制） | [[reference/components/container]]；[参数](../../../raw/current/scripts/containers.lua#L1318-L1346)、[装备开关](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L28-L53)、[OnStartedFishing/OnDoneFishing](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L102-L116) |
| `组件：reticule` | 抛投准星：targetfn 用 replica 最大抛距（含浮子/鱼饵 dist_max 加成）+ `oceanfishingfocus` 实体吸附；validfn 要求海面或虚拟海洋实体 | 客户端；`accepts_oceanfishingtackle` 标签用于换饵 | [[reference/components/reticule]]；[targetfn](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L72-L92)、[validfn](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L94-L96) |
| `组件：weapon` | 伤害同淡水钓竿（4.25）、attackwear 4；无 finiteuses，攻击不耗使用次数 | 主模拟 | [设置](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L188-L190) |
| `Replica／同步：oceanfishingrod` | 客户端读 GetMaxCastDist 与目标；target/line_tension 经 netvar 同步（组件 setter 调 replica `_SetTarget` 等） | 注册于组件构造 | [setter](../../../raw/current/scripts/components/oceanfishingrod.lua#L4-L43)、[UpdateClientMaxCastDistance](../../../raw/current/scripts/components/oceanfishingrod.lua#L129-L137) |
| `API／数据：ACTIONS.OCEAN_FISHING_CAST/REEL/STOP/CATCH` | 两条门禁不同：首次抛竿在竿的 target 为 nil 时只要求海面点（`CanCastFishingNetAtPoint`，距离 >2，[L4-L11](../../../raw/current/scripts/componentactions.lua#L4-L11)、[右键分支](../../../raw/current/scripts/componentactions.lua#L2160-L2175)）；已有 target 后才由 `GetFishingAction` 读持有者 `fishing_idle` 标签，按目标是否为投射物／`oceanfishing_catchable` 选 REEL／STOP／CATCH（[L86-L100](../../../raw/current/scripts/componentactions.lua#L86-L100)）。REEL 调 `oceanfishingrod:Reel`；STOP/CATCH 进 SG 状态后调 StopFishing("reeledin")/CatchFish | 服务端执行；`partiallyhooked` 目标显示 SETHOOK 文案 | [CAST.fn](../../../raw/current/scripts/actions.lua#L1398-L1406)、[REEL.fn](../../../raw/current/scripts/actions.lua#L1415-L1420)、[STOP.fn](../../../raw/current/scripts/actions.lua#L1422-L1430)、[CATCH.fn](../../../raw/current/scripts/actions.lua#L1432-L1440)、[候选 GetFishingAction](../../../raw/current/scripts/componentactions.lua#L86-L100) |
| `API／数据：钩饵数据流` | GetTackle 读容器槽 1/2 → Cast 缓存 → 浮子落水后 `oceanfishinghook:SetLureData`（charm×时段×天气×鱼偏好）；target 变化回调 OnHookedSomething：新目标为浮子（有 oceanfishinghook）则湿化饵具（OCEAN_WETNESS=75），为鱼等非投射物则移除一次性饵具 | 主模拟；itemget/itemlose 事件触发 UpdateClientMaxCastDistance | [GetTackle](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L55-L68)、[OnHookedSomething](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L118-L136)、[SetLureData 调用点](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L52-L55) |
| `关联 Prefab：oceanfishingbobber_*` | 投射物 complexprojectile 飞行，落海生成 floater（oceanfishable+oceanfishinghook），floater 经 `SetRod` 反向成为竿的 target；鱼吃浮子时经 ACTIONS.EAT 把 rod 移交给鱼 | 浮子鱼饵机制见 [[reference/components/oceanfishinghook]] 与 [[reference/prefabs/ocean-fish]]；TACKLE 调参表见 [tuning.lua#L263-L282](../../../raw/current/scripts/tuning.lua#L263-L282) | [OnProjectileLand](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L36-L78)、[EAT 移交](../../../raw/current/scripts/actions.lua#L799-L807)、[[reference/components/oceanfishinghook]]、[[reference/components/oceanfishable]] |
| `关联 Prefab：oceanfish` 等 | 鱼群脑以 `fishinghook` 标签找浮子并积累兴趣（charm），执行 EAT 上钩 | [[reference/prefabs/ocean-fish]] | [鱼脑寻食](../../../raw/current/scripts/brains/oceanfishbrain.lua#L91-L134) |

### 拖网链（fishingnet 组件）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：fishingnet` | CastNet 生成 `fishingnetvisualizer` 并 BeginCast | 主模拟 | [[reference/components/fishingnet]]；[CastNet](../../../raw/current/scripts/components/fishingnet.lua#L5-L12) |
| `组件：finiteuses` | ACTIONS.CAST_NET 消耗 1，耗尽 Remove | 主模拟 | [设置](../../../raw/current/scripts/prefabs/fishingnet.lua#L59-L63) |
| `API／数据：ACTIONS.CAST_NET` | 候选需海面/虚拟海洋点且距离 >2；fn 调 fishingnet:CastNet | 服务端执行 | [候选](../../../raw/current/scripts/componentactions.lua#L2116-L2120)、[CAST_NET.fn](../../../raw/current/scripts/actions.lua#L4954-L4970) |
| `关联 Prefab：fishingnetvisualizer` | 行为由 `SGfishingnetvisualizer` 推进：casting → opening → retrieving → final_pickup，开网阶段生成 `fishingnetvisualizerfx` 水花；组件负责飞行（速度 10）、收集 collect_radius=2 内 inventoryitem 实体（先对 radius+`MAX_FISH_SCHOOL_SIZE` 范围广播 `on_pre_net`）、回收（速度 12）与把收集物逐个丢到收网者脚边并自删 | 主模拟装 SG，无 Brain；`on_release_from_net`/`on_pre_net` 事件供消费方 | [工厂装 SG](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L38-L40)、[状态机](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L24-L84)、[组件收集](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L114)、[回收投放](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L156-L184)、[注册](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L45) |

### 共用装配

- 漂浮：四注册名均 `MakeInventoryFloatable`（参数各异，见注册表）；装备换手（`swap_object`）＋ARM_carry 显隐是三竿/桨共同回调写法，皮肤经 `OverrideItemSkinSymbol` 并推 `equipskinneditem`/`unequipskinneditem` 事件（[桨 onequip](../../../raw/current/scripts/prefabs/oar.lua#L31-L47)、[钓竿](../../../raw/current/scripts/prefabs/fishingrod.lua#L7-L30)、[海钓竿](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L12-L47)）。
- 耐久耗尽即整件 Remove：桨（`toolbroke` 事件后 Remove）、淡水钓竿与渔网（`inst.Remove`）；海钓竿无使用耐久。掉落/ HauntableLaunch 为标准组装。
- `allow_action_on_impassable` 标签（四注册名全有）让玩家能对不可通行点（水面）发起动作，是水面工具动作链的前置（[playercontroller 消费](../../../raw/current/scripts/components/playercontroller.lua#L5405)）。

## 典型接入

### 用 AddPrefabPostInit 强化原版船桨

- 前提与选择：`modmain.lua`，需要改已有注册名的组件参数（推力/桨速）而不重建实体时选 postinit；AddPrefabPostInit 在 prefab 构造后对每份实体回调（[modutil.lua#L593-L599](../../../raw/current/scripts/modutil.lua#L593-L599)）。
- 生效与撤销：随实体构造常驻；`inst.components.oar` 仅主模拟存在，用非空判断区分端，客户端副本自动跳过。
- 原版依据：oar 组件的 force/max_velocity 即划船推力消费点（[Row 调 ApplyRowForce](../../../raw/current/scripts/components/oar.lua#L29)）；modmain 环境无 `__index=_G`，`TUNING` 由环境直供（[mods.lua#L263-L292](../../../raw/current/scripts/mods.lua#L263-L292)），`TheWorld`/`ACTIONS` 等需 `GLOBAL.` 前缀（本例未用到）。

```lua
-- modmain.lua：把基础桨的推力与桨速对齐邪天翁喙
AddPrefabPostInit("oar", function(inst)
    if inst.components.oar ~= nil then
        inst.components.oar.force = TUNING.BOAT.OARS.MALBATROSS.FORCE
        inst.components.oar.max_velocity = TUNING.BOAT.OARS.MALBATROSS.MAX_VELOCITY
    end
end)
```

- 调整与验证：数值单位为船物理推力标量与速度上限（对比表见上方数值表）；若要同步改耐久/磨损应挂 `finiteuses`（消耗映射见[桨工厂](../../../raw/current/scripts/prefabs/oar.lua#L137-L144)），运行验证未做（本页 validation=source-reviewed）。

## 常见陷阱与限制

- **`fishingnet` 不是拖网渔船**：手持渔网与 `ocean_trawler`（拖网渔船建筑）是两个系统；渔网无配方，常规玩法无生成链（prefabs／map 布局检索无命中，仅 [forest.lua#L342](../../../raw/current/scripts/prefabs/forest.lua#L342) 持久化声明与文件头 "unused" 注释），但调试命令 `d_spawnallhandequipment_onstands` 的整手装备清单收录 `fishingnet`（[L1851](../../../raw/current/scripts/debugcommands.lua#L1851)，经 `d_spawnlist` 上架生成）——"无生成点"只限常规玩法。复刻拖网玩法应参考 ocean_trawler（Wiki 待建）或本页 visualizer 链。
- **ROW_FAIL 比正常划桨磨损高一个量级**：节奏没踩中按 `ROW_FAIL_WEAR`（多数 25 次/次）扣耐久，而成功划桨只扣 1；RowFail 还会把 2 距离内实体增加 9 点湿度（[RowFail](../../../raw/current/scripts/components/oar.lua#L35-L51)、[BASIC 表](../../../raw/current/scripts/tuning.lua#L4194-L4202)）。
- **浮木桨的"防水"是 0 效果**：`is_waterproof` 只给 `waterproofer` 标签＋`SetEffectiveness(0)` 组件，不提供实际防水加成（[分支](../../../raw/current/scripts/prefabs/oar.lua#L113-L116)）；别按标签推断装备后防水。
- **海钓竿断线销毁全部饵具**：`StopFishing(reason, lost_tackle=true)`（断线、超距、被打断且有鱼等）走 OnDoneFishing `container:DestroyContents()`（[回调](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L108-L116)、[触发点](../../../raw/current/scripts/components/oceanfishingrod.lua#L303-L330)）；收集物/沉底类正常收线不销毁。
- **quagmire 游戏模式下淡水钓竿无武器/耐久**：master 分支整体换成 event_server_data 的 quagmire 实现，武器标签也不打（[分支](../../../raw/current/scripts/prefabs/fishingrod.lua#L57-L89)）；通用规则判断需排除该模式。
- **`yotd_oar` 是活动限定**：配方挂 `TECH.DRAGONOFFERING_THREE`＋`nounlock`＋PERDOFFERING 动作（蜻蜓年供奉），常规科技树不可见（[配方](../../../raw/current/scripts/recipes.lua#L1107)）。
- **边界**：海钓竿本体耐久无限（无 finiteuses），消耗压力在饵具（一次性饵中鱼即消失、断线全灭、饵具入海打湿 75）；`oceanfishingbobber_none`/`_twig`/`_plug` 无物品形态，只能经竿的默认设置或代码获得，非玩家可拾取（[BOBBERS 表 make_inv_item=false](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L2-L16)）。

## 验证记录

- 源码审阅：2026-11-07（第二轮检视修订），steam-buildid:26335394。复核 SGfishingnetvisualizer 的状态推进、海钓竿抛竿与已下竿两条门禁、debugcommands 的手持装备上架清单。逐行实读四主文件（oar.lua 185 行／fishingrod.lua 103 行／oceanfishingrod.lua 213 行／fishingnet.lua 83 行）与其直接组件（components/oar、fishingrod、oceanfishingrod、fishingnet、fishingnetvisualizer）、饵具两文件（oceanfishingbobber.lua 356 行、oceanfishinglure.lua 121 行）、oceanfishable/oceanfishinghook 关键函数、containers.lua WidgetSetup、componentactions.lua 动作候选、actions.lua 各 fn、tuning/recipes/constants/prefabskins 调参与配方行、hats/malbatross/piratespawner 生成点、mods.lua/modutil.lua 环境与 postinit、componentutil.lua FindVirtualOceanEntity。8 个主注册名与 47 个附属注册名（30 浮子系＋16 鱼饵＋1 拖网实体）均从 return/循环实读核对；.po 名称键 8 条经 python UTF-8 读取核对。未覆盖：oceanfishingbobber/lure 内 FX 细节、鱼群 charm/AI 全流程、`fishingnetvisualizerfx.lua`（未读）、玩家 SG 钓鱼状态机、quagmire event_server_data 内容、皮肤贴图内容。
- complex 选档理由：海钓竿为"容器数据 → 投射物/浮子 → 鱼移交 → 张力收线/断线"的多状态生命周期且含两类附属变体族，需整体保留协作链才可读，非因篇幅超限自动升档。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过，实测 163 行／30662 字节（复杂档上限 480 行／48 KiB）；validate_source_links.py --strict 0 error 0 warning（130 条引用）；validate_source_hashes.py check 通过（基线已由主代理 record）。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
