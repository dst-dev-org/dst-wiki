---
title: "红灯笼（redlantern）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "原版手持照明装备 redlantern 与蛇年活动变体 yots_redlantern：HANDS 槽 MAGIC 燃料一次性提灯，装备或落地常亮、入袋即熄、不可开关不可补燃料，光照由附属 FX redlanternlight 提供、装备时另有 redlanternbody 跟随外观；耗尽可当火堆燃料，也可被火焰（含点火工具与闪电）点燃烧毁。"
aliases: ["redlantern", "红灯笼", "yots_redlantern", "redlanternlight", "redlanternbody", "蛇年红灯笼", "手持红灯笼照明"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/redlantern.lua", "prefabs/worm.lua", "prefabs/yots_worm_lantern.lua", "prefabs/forest.lua", "components/hunter.lua", "components/equippable.lua", "components/fueled.lua", "components/fuel.lua", "components/burnable.lua", "components/floater.lua", "components/inventoryitem.lua", "components/inventory.lua", "components/inspectable.lua", "components/machine.lua", "components/weather.lua", "components/lighter.lua", "standardcomponents.lua", "componentactions.lua", "componentutil.lua", "recipes.lua", "tuning.lua", "constants.lua", "strings.lua", "widgets/redux/craftingmenu_details.lua", "stringutil.lua", "entityscript.lua", "mods.lua", "util/sourcemodifierlist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 红灯笼（redlantern）

## 快速判断

- 职责与适用：`prefabs/redlantern.lua` 同文件注册四件——本体 `redlantern`（春节供品兑换的手持照明装备，物品＋装备）、蛇年活动变体 `yots_redlantern`（同工厂 `common_fn`，只换 build／持握 swap／图标／浮态缩放）、附属光 FX `redlanternlight` 与装备外观 FX `redlanternbody`。定位是提灯的一次性替代：12 游戏日燃料，但不可开关、不可补燃料、亮度不随燃料渐变。同文件无 quagmire 分支（全文核验）。[注册](../../../raw/current/scripts/prefabs/redlantern.lua#L422-L425)
- 优先使用：复刻“一次性照明装备＋跟随 FX 外观＋装备者状态切换”按[关联机制导航](#关联机制导航)组装并查 [[cookbook/equipment/hand]]；改燃料／光照数值查[数值表](#注册与构造)；对照可开关可充电的 [[reference/prefabs/lantern|提灯]] 与武器型 [[reference/prefabs/torch|火炬]]。
- 重要副作用：燃料只在点亮时消耗——装备或躺地上常亮、入袋／入容器立即熄灭、丢弃重新点亮；雨天按降水率加速（遮蔽或雨免保持 1）；耗尽熄灭并广播 `torchranout`（全库无原版监听者）；本体可被任何火源（环境火焰／玩家点火工具／闪电）点燃、3 秒烧毁成灰，残灯可当燃料（15）投入火堆；蛇年手持红灯笼会被 yots_worm 优先攻击。[turnon](../../../raw/current/scripts/prefabs/redlantern.lua#L59-L84)、[nofuel](../../../raw/current/scripts/prefabs/redlantern.lua#L228-L240)、[雨衰](../../../raw/current/scripts/prefabs/redlantern.lua#L242-L258)
- 修改边界：标签与 floater 在公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/redlantern.lua#L321-L323)）；光参数硬编码在 `redlanternlight` 构造与 `OnUpdateFlicker`（非 TUNING 键）；`redlanternbody` 是外观 FX、与 BODY 槽无关，槽位见下表。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `redlantern` | `fn()` → `common_fn("redlantern", "redlantern")`，无 postinit | 物品＋装备；build 与库存图标名 `redlantern`，持握符号用默认 `swap_redlantern`。[fn](../../../raw/current/scripts/prefabs/redlantern.lua#L381-L383)、[注册](../../../raw/current/scripts/prefabs/redlantern.lua#L422) |
| `yots_redlantern` | `yots_fn()` → `common_fn("yots_redlantern", "yots_redlantern", yots_fn_client)`，无 postinit | 蛇年活动变体：`_swapfile="yots_swap_redlantern"`、`SetPrefabNameOverride("redlantern")` 共用名称、浮态缩放覆写 {0.55, 0.4, 0.55}；其余初始化与本体一致。[yots_fn_client](../../../raw/current/scripts/prefabs/redlantern.lua#L411-L416)、[yots_fn](../../../raw/current/scripts/prefabs/redlantern.lua#L418-L420) |
| `redlanternlight` | `lanternlightfn()` 附属光 FX，随文件注册 | 详见[附属 FX](#redlanternlight-与-redlanternbody附属-fx同页简述)；`persists=false` 不进存档。[构造](../../../raw/current/scripts/prefabs/redlantern.lua#L271-L297) |
| `redlanternbody` | `lanternbodyfn()` 附属跟随 FX，随文件注册 | 同上；仅装备期间由 onequip 生成。[构造](../../../raw/current/scripts/prefabs/redlantern.lua#L385-L408) |

共同初始化（[common_fn](../../../raw/current/scripts/prefabs/redlantern.lua#L299-L379)）：实体组件＋`MakeInventoryPhysics` → bank `redlantern`＋`idle_loop` → 公共端标签 `light`／`redlantern` → `MakeInventoryFloatable("med", nil, {0.775, 0.5, 0.775})` → `client_pre`（仅 yots）→ `SetPristine`（非主模拟返回）→ 主模拟：随机帧 → inspectable → inventoryitem（掉落／入袋回调）→ equippable 三回调 → fueled → fuel → 浮态事件 → 雨监听 → MakeSmallBurnable＋MakeSmallPropagator＋MakeHauntableLaunch → `burnable.ignorefuel=true` → OnRemoveEntity／OnLoad → 构造即 `turnon` → `_onownerequip` 兜底监听。装备槽两注册都未显式赋值，用组件默认 `EQUIPSLOTS.HANDS`（[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[常量](../../../raw/current/scripts/constants.lua#L656)）。

名称经简中 .po `msgctxt` 核验：`STRINGS.NAMES.REDLANTERN`＝红灯笼（Red Lantern）；通用检视语＝“这个灯笼感觉比其他灯笼特别。”。po 无 `YOTS_REDLANTERN` 键：`yots_redlantern` 经 `SetPrefabNameOverride("redlantern")` 回退共用本体名称与检视语。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L348529-L348532)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L18609-L18612)、[名称覆写](../../../raw/current/scripts/entityscript.lua#L846-L850)、[描述回退](../../../raw/current/scripts/stringutil.lua#L375)

数值（tuning 键＋实际值；`seg_time`＝30 秒、`total_day_time`＝480 秒，[tuning.lua#L20](../../../raw/current/scripts/tuning.lua#L20)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 燃料总量 | `TUNING.REDLANTERN_LIGHTTIME`＝`total_day_time`×12 | 5760 燃料单位＝rate 1 下 5760 秒＝12 游戏日（[tuning.lua#L3644](../../../raw/current/scripts/tuning.lua#L3644)） |
| 雨中消耗增速 | `TUNING.REDLANTERN_RAIN_RATE` | 0.25：雨中 rate＝1＋0.25×降水率，降水率 0.1~1（[weather.lua#L400-L411](../../../raw/current/scripts/components/weather.lua#L400-L411)），满雨 1.25；持有者 `sheltered.sheltered` 或有 `rainimmunity` 组件则保持 1（[onupdatefueledraining](../../../raw/current/scripts/prefabs/redlantern.lua#L242-L248)） |
| 点亮首扣 | `TUNING.TURNON_FUELED_CONSUMPTION`／`TURNON_FULL_FUELED_CONSUMPTION` | 0.5 秒量／满燃料 1/60 秒量（[tuning.lua#L1668-L1669](../../../raw/current/scripts/tuning.lua#L1668-L1669)） |
| 光照参数 | `redlanternlight` 硬编码（非 TUNING 键） | 半径 1.2（闪烁 ±0.1）、强度 0.8、衰减 0.5、颜色 (200,100,100)/255，0.1 秒闪烁任务公共端双端运行（[常量](../../../raw/current/scripts/prefabs/redlantern.lua#L24-L36)、[构造](../../../raw/current/scripts/prefabs/redlantern.lua#L280-L287)） |
| 残灯燃料值 | `inst.components.fuel.fuelvalue`＝`TUNING.SMALL_FUEL`＝`seg_time`×0.5 | 15，fueltype 默认 BURNABLE（→`burnable_fuel` 标签）（[tuning.lua#L1674](../../../raw/current/scripts/tuning.lua#L1674)、[挂载](../../../raw/current/scripts/prefabs/redlantern.lua#L345-L346)、[默认](../../../raw/current/scripts/components/fuel.lua#L10-L16)） |
| 自身点燃时长 | `MakeSmallBurnable(inst, TUNING.TINY_BURNTIME)`＝`seg_time`×0.1 | 3 秒，烧毁成灰并 `Remove`（[tuning.lua#L1680](../../../raw/current/scripts/tuning.lua#L1680)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L176)、[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L46)） |
| 浮态 | `MakeInventoryFloatable("med", nil, {0.775,0.5,0.775})`；yots 覆写 | 缩放 {0.775,0.5,0.775}、无纵向偏移；`yots_redlantern` 改 {0.55,0.4,0.55}（[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790)、[覆写](../../../raw/current/scripts/prefabs/redlantern.lua#L415)） |
| 作祟 | `MakeHauntableLaunch(inst)` | 作祟弹飞（[helper](../../../raw/current/scripts/standardcomponents.lua#L1053)、[调用](../../../raw/current/scripts/prefabs/redlantern.lua#L356)） |

获取途径（两注册独立核验）：

| 注册名 | 途径与条件 | 证据 |
|---|---|---|
| `redlantern` | 配方：幸运金块 `lucky_goldnugget`×3，科技 `TECH.PERDOFFERING_ONE`＝`{PERDOFFERING=1}`，按钮文案 actionstr="PERDOFFERING"（"Offer"），无科技时提示 NEEDSSHRINE；供品科技由春节神龛 prototyper 提供——perdshrine 给 3 级、其余九座春节神龛各给 1 级，任一神龛旁即可制作 | [配方](../../../raw/current/scripts/recipes.lua#L1186)、[科技](../../../raw/current/scripts/constants.lua#L1272-L1273)、[供品科技表](../../../raw/current/scripts/tuning.lua#L1320-L1364)、[按钮文案](../../../raw/current/scripts/strings.lua#L9552) |
| `yots_redlantern` | 无配方；仅蛇年活动（YOTS）期间由 `yots_worm` 掉落表 50% 概率掉落；`yots_worm` 由假灯笼 `yots_worm_lantern` 被激活或点燃时 `ReplacePrefab` 转变，假灯笼由 `yots_worm_lantern_spawner` 首帧生成 3 个（`TUNING.YOTS_WORM_COUNT`）；猎踪在蛇年活动开启且蛇年神龛有效（`IsSnakeShrineActive`）时才把 spawner 当猎物返回并 `SpawnPrefab` 生成——这是 scripts 内唯一可核验的生成路径；`forest.lua` 只是 prefab 依赖声明，没有世界布点证据 | [掉落表](../../../raw/current/scripts/prefabs/worm.lua#L45-L53)、[装配](../../../raw/current/scripts/prefabs/worm.lua#L498)、[激活转变](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L23-L45)、[生成](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L124-L144)、[数量](../../../raw/current/scripts/tuning.lua#L8581)、[声明](../../../raw/current/scripts/prefabs/forest.lua#L508-L509)、[猎踪分诊](../../../raw/current/scripts/components/hunter.lua#L369-L378)、[神龛门禁](../../../raw/current/scripts/components/hunter.lua#L573-L575)、[猎踪生成](../../../raw/current/scripts/components/hunter.lua#L418-L431) |

材料侧注：`lucky_goldnugget` 为春节活动货币，产出点如 YOTS 蠕虫掉落（[lootsetfn](../../../raw/current/scripts/prefabs/worm.lua#L260-L264)），完整获取链不在本页范围。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | 全套回调 SetOnEquip／SetOnUnequip／SetOnEquipToModel；槽位未显式赋值＝组件默认 HANDS | 主模拟；两注册一致，无运行时切槽 | [[reference/components/equippable#关键接口]]；[默认槽](../../../raw/current/scripts/components/equippable.lua#L29)、[接线](../../../raw/current/scripts/prefabs/redlantern.lua#L334-L337) |
| 组件：fueled | `fueltype=FUELTYPE.MAGIC`（源码注释：无关联燃料、非可燃燃料）、`InitializeFuelLevel(REDLANTERN_LIGHTTIME)`、`SetDepletedFn(nofuel)`、FirstPeriod；`accepting` 默认 false → `CanAcceptFuelItem` 恒 false，不可补燃料 | 主模拟；Start/StopConsuming 由 turnon/turnoff 控制，period 1 秒；雨衰经 `SetUpdateFn` 每秒重算 rate | [[reference/components/fueled#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/redlantern.lua#L339-L343)、[消耗](../../../raw/current/scripts/components/fueled.lua#L325-L337)、[拒收](../../../raw/current/scripts/components/fueled.lua#L169-L175) |
| 组件：fuel | `fuelvalue = TUNING.SMALL_FUEL`（15），残灯可作燃料投入火堆 | 主模拟；源码注释即“toss depleted lanterns into a firepit” | [[reference/components/fuel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/redlantern.lua#L345-L346) |
| 组件：burnable＋propagator | `MakeSmallBurnable(TINY_BURNTIME)`＋`MakeSmallPropagator`；`ignorefuel=true` → 点燃/熄灭不联动照明燃料；`canlight` 不覆写＝默认 true 并镜像出 `canlight` 标签（[burnable.lua:6-8](../../../raw/current/scripts/components/burnable.lua#L6-L8)、[L50](../../../raw/current/scripts/components/burnable.lua#L50)）→ LIGHT 动作与闪电均可点燃；无 `wildfireprotected` | 主模拟；环境火焰传播、玩家点火工具、闪电三条点燃路径，3 秒烧毁成灰 | [[reference/components/burnable#关键接口]]；[调用](../../../raw/current/scripts/prefabs/redlantern.lua#L354-L358)、[ignorefuel 分支](../../../raw/current/scripts/components/burnable.lua#L380-L382)、[LIGHT 收集](../../../raw/current/scripts/componentactions.lua#L1481)、[闪电门禁](../../../raw/current/scripts/componentutil.lua#L1080-L1081) |
| 组件：inventoryitem | `SetOnDroppedFn(ondropped)`（落地 turnoff＋turnon 重亮）、`SetOnPutInInventoryFn(turnoff)`（入袋/入容器熄灭）；`ChangeImageName(_lit)` 切换点亮图标 | 主模拟回调；卸下走 Unequip→GiveItem→OnPutInInventory 链即熄 | [[reference/components/inventoryitem#关键接口]]；[回调](../../../raw/current/scripts/prefabs/redlantern.lua#L330-L332)、[OnPutInInventory](../../../raw/current/scripts/components/inventoryitem.lua#L348-L358)、[GiveItem 调用](../../../raw/current/scripts/components/inventory.lua#L1150) |
| 组件：floater | 公共端 `MakeInventoryFloatable`；监听 `floater_startfloating/stopfloating` 切 `float`／`idle_loop` 动画；yots 覆写缩放 | 公共端构造；前后浮态粒子 `idle_front_med`/`idle_back_med` 由 floater 生成 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/redlantern.lua#L315)、[动画事件](../../../raw/current/scripts/prefabs/redlantern.lua#L267-L268)、[FX](../../../raw/current/scripts/components/floater.lua#L181-L189) |
| 组件：inspectable／hauntable | 默认检视（无 SetDescription，描述走名称键）；`MakeHauntableLaunch` 作祟弹飞 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[挂载](../../../raw/current/scripts/prefabs/redlantern.lua#L328)、[作祟](../../../raw/current/scripts/prefabs/redlantern.lua#L356) |
| API／事件：`torchranout` | `nofuel` 熄灭时对装备者推送，data＝{prefab, equipslot} | 仅装备态推送；全库无原版监听者（grep 核验） | [nofuel](../../../raw/current/scripts/prefabs/redlantern.lua#L228-L240) |
| API／事件：`israining` 世界状态 | `WatchWorldState("israining", onisraining)` 接线雨衰，构造时立即同步一次 | 主模拟；停雨恢复 rate 1 并清 updatefn | [监听](../../../raw/current/scripts/prefabs/redlantern.lua#L351-L358) |
| API／数据：`redlantern` 标签 | yots_worm 的 retarget/keepTarget 检查 HANDS 装备的该标签物品，优先把持灯者当攻击目标 | 仅 YOTS 蠕虫消费（普通 worm 用原 retargetfn） | [retarget](../../../raw/current/scripts/prefabs/worm.lua#L402-L420)、[keepTarget](../../../raw/current/scripts/prefabs/worm.lua#L433-L449) |
| API／数据：`light`＋`turnedon` 组合 | 影之手寻光按 `light`＋`turnedon` 组合检查；红灯笼有 `light` 无 `turnedon`（无 machine 组件）→不被选中（提灯开灯后会被选中） | 对比消费端，双注册一致 | [影之手](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L318-L321)、[turnedon 来源](../../../raw/current/scripts/components/machine.lua#L2) |
| 关联 Prefab：`redlanternlight`／`redlanternbody` | 两件附属 FX 实体，见下节简述 | turnon/onequip 生成，turnoff/onunequip/OnRemove 清理；均 `persists=false` | [声明](../../../raw/current/scripts/prefabs/redlantern.lua#L18-L22)、[OnRemove 清理](../../../raw/current/scripts/prefabs/redlantern.lua#L108-L115) |
| 关联 Prefab：`yots_worm_lantern` | 蛇年假灯笼：借红灯笼名称与检视语（nameoverride），激活/点燃变 `yots_worm`；非本文件注册，仅名称复用方 | YOTS 期间由 spawner 放置 | [名称借用](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L86)、[检视借用](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L111) |
| 资源：anim／图标 | redlantern.zip（bank `redlantern`＋`float`/`idle_body_loop`）、swap_redlantern.zip（持握符号 `swap_redlantern`／`_stick`）；yots 另用 yots_redlantern.zip、yots_swap_redlantern.zip；INV_IMAGE `redlantern_lit`/`yots_redlantern_lit`；redlantern_water.zip 声明但脚本内无引用 | 静态声明；zip 内容不可从 scripts 核验 | [本体 assets](../../../raw/current/scripts/prefabs/redlantern.lua#L1-L7)、[yots assets](../../../raw/current/scripts/prefabs/redlantern.lua#L9-L16) |

### redlanternlight 与 redlanternbody（附属 FX，同页简述）

- `redlanternlight`（[构造](../../../raw/current/scripts/prefabs/redlantern.lua#L271-L297)）：Transform＋Light＋Network 的纯光 FX（FX 标签、`persists=false`）；光参数如数值表；`EnableClientModulation(true)`，0.1 秒 `OnUpdateFlicker` 公共端双端运行。由 `turnon` 生成后 `SetParent` 到（inventoryitem.owner or `_body` or 本体），监听其 onremove 清 `_light`；turnoff／本体 OnRemove 时移除。[光生成](../../../raw/current/scripts/prefabs/redlantern.lua#L63-L68)
- `redlanternbody`（[构造](../../../raw/current/scripts/prefabs/redlantern.lua#L385-L408)）：Transform＋AnimState＋Network 的跟随 FX（FX 标签、`persists=false`）；bank `redlantern`、build `redlantern`、anim `idle_body_loop`。onequip 生成并 `SetParent` 到装备者＋`AddFollower`＋`FollowSymbol(GUID, "swap_object", 68, -126, 0)`；`newstate` 监听 `ToggleOverrideSymbols`：`nodangle` 或骑乘（非 forcedangle）→ 藏 body、覆写完整符号＋按燃料显示 `LANTERN_OVERLAY`，否则覆写 `swap_redlantern_stick`＋显 body；yots 装备时 body build 跟随本体；bloomer/colouradder 挂接。onunequip/onequip 替换/本体 OnRemove 均移除 body。[onequip](../../../raw/current/scripts/prefabs/redlantern.lua#L142-L185)、[ToggleOverrideSymbols](../../../raw/current/scripts/prefabs/redlantern.lua#L122-L136)

## 典型接入

### 延长红灯笼燃烧时长（雨天加速同比例减慢）

- 前提与选择：服务端 `modmain.lua`。红灯笼无 torch 的 `SetFuelRateMult` 钩子，且雨天 `onupdatefueledraining` 每秒重写 `fueled.rate`，直接改 rate 会被覆盖；`DoUpdate` 消耗乘 `rate×rate_modifiers:Get()`，在 `rate_modifiers` 挂常驻修正可与雨衰链路解耦。`AddPrefabPostInit`/`TUNING` 是 modmain 环境直挂项，`TheWorld` 无全局回退须 `GLOBAL.` 前缀（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L317)、[GLOBAL 字段](../../../raw/current/scripts/mods.lua#L289)）。
- 生效与撤销：postinit 只影响之后新建的实例；存档只保存燃料余量，重载重建实例时会重新执行 postinit 重挂，`rate_modifiers` 本身不进存档。停用模组恢复原值。

```lua
-- modmain.lua：红灯笼燃烧时长加倍（雨天加速同比例减慢）
AddPrefabPostInit("redlantern", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.fueled.rate_modifiers:SetModifier(inst, 0.5, "mymod_redlantern_half")
end)

AddPrefabPostInit("yots_redlantern", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.fueled.rate_modifiers:SetModifier(inst, 0.5, "mymod_redlantern_half")
end)
```

- 调整与验证：0.5 → 5760→11520 秒＝12→24 游戏日，满雨消耗倍率 1.25→0.625；两注册是独立 prefab 须分别挂。修正为乘法链，与技能等其他 `rate_modifiers` 修正相乘（[SetModifier](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L75)、[DoUpdate](../../../raw/current/scripts/components/fueled.lua#L325-L337)）。

## 常见陷阱与限制

- **误区：红灯笼可像提灯一样开关或补燃料**：无 machine 组件（无右键开关、无 `turnedon` 标签）；fueled `fueltype=MAGIC` 且 `accepting=false` → ADDFUEL 拒收；耗尽只能当燃料（15）投入火堆或丢弃；亮度不随燃料渐变（对比提灯）。[挂载](../../../raw/current/scripts/prefabs/redlantern.lua#L339-L346)、[拒收](../../../raw/current/scripts/components/fueled.lua#L169-L175)
- **边界：只在“装备或躺在地上”时耗燃料**：入袋/入容器立即熄灭（OnPutInInventory→turnoff→StopConsuming），落地重新点亮；燃耗只随点亮发生，背包存放不计时。[入袋熄灭](../../../raw/current/scripts/prefabs/redlantern.lua#L330-L332)、[落地重亮](../../../raw/current/scripts/prefabs/redlantern.lua#L117-L120)
- **边界：雨中烧得更快，遮蔽或雨免可免**：rate＝1＋0.25×降水率（降水率 0.1~1）；`sheltered.sheltered` 或 `rainimmunity` 组件存在即保持 1，判断每秒重算、进出树下即时生效。[雨衰](../../../raw/current/scripts/prefabs/redlantern.lua#L242-L258)、[降水率范围](../../../raw/current/scripts/components/weather.lua#L400-L411)
- **边界：蛇年活动手持红灯笼吸引蠕虫攻击**：yots_worm 的 retarget/keepTarget 优先把 HANDS 装备红灯笼的玩家当目标（必须装备在 HANDS，放背包无效）；这是 `redlantern` 标签的已核验消费端。[retarget](../../../raw/current/scripts/prefabs/worm.lua#L402-L420)
- **边界：地上的“红灯笼”可能是假灯笼**：蛇年假灯笼 `yots_worm_lantern` 借红灯笼名称与检视语（nameoverride），拾取/点燃会激活并变成 yots_worm（600 HP，`TUNING.YOTS_WORM_HEALTH`）。[名称借用](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L86)、[转变](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L23-L30)、[生命](../../../raw/current/scripts/tuning.lua#L8580)
- **边界：红灯笼可被玩家点火工具与闪电点燃并烧毁**：`canlight` 默认 true 未覆写（见上表 burnable 行）→ lighter 的 LIGHT 动作收集与执行均放行（[lighter.lua:18-21](../../../raw/current/scripts/components/lighter.lua#L18-L21)），闪电因 `_inventoryitem` 在引燃标签集同样可点燃（[componentutil.lua:1080-1081](../../../raw/current/scripts/componentutil.lua#L1080-L1081)）；3 秒烧毁成灰并 `Remove`——火会直接毁掉灯笼；`ignorefuel=true` 保证点燃不启动照明燃料。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L213)
- **边界：无物品皮肤**：prefabskin.lua 与 misc_items.lua 无两注册皮肤条目（仅 YOTP 头像挂件同名）；`_swapfile` 机制保留（yots 自定义 swap build），无官方皮肤使用。

## 验证记录

- 源码审阅：2026-11-07（第二轮修订），Steam buildid 26335394。复核 hunter.lua 的 GetHuntedBeast 活动分支与 SpawnPrefab 生成点。早前核验 redlantern.lua 全文（四注册、common_fn、回调、OnLoad、_onownerequip）、equippable／fueled／fuel／burnable／inventory／inventoryitem／floater、weather、standardcomponents、获取链（recipes/tuning/constants/craftingmenu_details/strings）、YOTS 链、名称链、消费端（worm、charlie_boss_trial、componentutil、prefabskin/misc_items）、mods 与 sourcemodifierlist；canlight 断言已按源码修正。
- 游戏运行：未运行。未覆盖：光照实际视效与闪烁幅度、12 日燃料与雨衰为源码推导而非运行测量、神龛与假灯笼的世界生成布点（数据侧）、活动期间实机表现。
- 结构检查：validate_page.py 0 error（126 行，standard）与 validate_source_links.py --strict 0 error 0 warning（101 引用／27 源文件）通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]；同批手持照明对照 [[reference/prefabs/torch|火炬]] 与 [[reference/prefabs/lantern|提灯]]。
