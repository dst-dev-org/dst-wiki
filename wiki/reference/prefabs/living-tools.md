---
title: "生活工具族（bugnet／brush／wateringcan／pitchfork／minifan）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "五文件八注册的手持生活工具原版参照：捕虫网 NET 捕捉、刷子 BRUSH 梳理、浇水壶装水浇灌与空壶转燃料、干草叉 TERRAFORM 铲地（非 DIG）、小风扇装备制冷（移动才耗油）。"
aliases: ["bugnet", "thulecitebugnet", "brush", "wateringcan", "premiumwateringcan", "pitchfork", "goldenpitchfork", "minifan", "捕虫网", "铥矿捕虫网", "刷子", "浇水壶", "空浇水壶", "空鸟嘴壶", "鸟嘴壶", "黄金干草叉", "干草叉", "旋转的风扇", "生活工具"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bugnet.lua", "prefabs/brush.lua", "prefabs/wateringcan.lua", "prefabs/pitchfork.lua", "prefabs/minifan.lua", "prefabs/fan_wheel.lua", "prefabs/beefalo.lua", "prefabs/wortox_nabbag.lua", "components/tool.lua", "components/weapon.lua", "components/finiteuses.lua", "components/equippable.lua", "components/brushable.lua", "components/brush.lua", "components/wateryprotection.lua", "components/fillable.lua", "components/heater.lua", "components/fueled.lua", "components/temperature.lua", "components/terraformer.lua", "components/watersource.lua", "components/nabbag.lua", "components/witherable.lua", "components/playercontroller.lua", "actions.lua", "componentactions.lua", "componentutil.lua", "bufferedaction.lua", "constants.lua", "tuning.lua", "recipes.lua", "strings.lua", "standardcomponents.lua", "mods.lua", "prefabskins.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 生活工具族（bugnet／brush／wateringcan／pitchfork／minifan）

## 快速判断

- 职责与适用：五个独立 prefab 文件、八个注册的手持"生活类"工具族（五文件机制彼此独立）：捕虫网 NET 捕捉、刷子 BRUSH 梳理、浇水壶装水浇灌、干草叉 TERRAFORM 铲地、小风扇移动制冷；全部是 `equippable` 默认 `EQUIPSLOTS.HANDS` 槽可拾取物品（[equippable.lua:29](../../../raw/current/scripts/components/equippable.lua#L29)）。三组共用工厂的变体对（bugnet／thulecitebugnet、wateringcan／premiumwateringcan、pitchfork／goldenpitchfork）须与五条机制链同页对照，选 `complex` 档整体保留族结构。
- 优先使用：改耐久／伤害／单次消耗、新可刷目标等走"关联机制导航"对应组件页（多数已有案例）；查注册差异、官方名、配方与皮肤读"注册与构造"；新做同类手持工具走 [[cookbook/equipment/hand]]。
- 重要副作用：按工具分流——捕虫网 NET、刷子 BRUSH、干草叉 TERRAFORM 走 `BufferedAction:Do` → `OnUsedAsItem`，按 `finiteuses` 的 `consumption[action]` 折算耐久（[bufferedaction.lua:22-37](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[finiteuses.lua:101-114](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)）；浇水壶是例外，`POUR_WATER` 经 `wateryprotection.onspreadprotectionfn=onuse` 直接 `finiteuses:Use(1)`，没有 `SetConsumption` 表项，复用时不要再叠动作消耗表（[wateringcan.lua:69-72](../../../raw/current/scripts/prefabs/wateringcan.lua#L69-L72)）。捕虫网／刷子／干草叉耗尽 `inst.Remove`，浇水壶空壶转挂 `fuel` 可作燃料，minifan 用 `fueled` 耗尽自毁并通告。
- 修改边界：八注册都是公共端只放实体基础与标签、主模拟装组件；本页不覆盖 NET 可捕目标全量清单（仅抽样）、皮肤实体定义端（prefabskin／skinprefabs）、引擎侧动作与动画、scrapbook 数据端与 quagmire 生成点。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bugnet` | `fn()` 直接构造；公共端 tag `tool`＋`weapon`；主模拟 weapon→tool→finiteuses→equippable | 捕虫网：10 次耐久（`BUGNET_USES`）、伤害 `wilson_attack*.125`=4.25（源码推导值）、攻击磨损 3。[fn](../../../raw/current/scripts/prefabs/bugnet.lua#L28-L83)、[耐久](../../../raw/current/scripts/prefabs/bugnet.lua#L65-L70) |
| `thulecitebugnet` | `fn_thulecite()` 独立工厂，结构与 `fn` 相同 | 铥矿捕虫网：100 次耐久、伤害同 4.25、磨损 3；持握与漂浮换装用 `thulecitebugnet`／`swap_object` 符号。[fn](../../../raw/current/scripts/prefabs/bugnet.lua#L111-L166)、[换装](../../../raw/current/scripts/prefabs/bugnet.lua#L90-L100) |
| `brush` | `fn()` 直接构造；公共端仅 tag `weapon`（无 `tool`） | 刷子：75 次耐久（`BRUSH_USES`）、伤害 `wilson_attack*.8`=27.2、磨损 3；`brush` 组件只作标记无字段。[fn](../../../raw/current/scripts/prefabs/brush.lua#L18-L68)、[组件](../../../raw/current/scripts/prefabs/brush.lua#L49) |
| `wateringcan` | `MakeWateringCan(name, uses, water_amount)` 共用工厂（`prefabs` 声明 `gridplacer`） | 浇水壶：40 次耐久、水量 25；出厂 `SetUses(0)`＋`SetDoesNotStartFull(true)`。[工厂](../../../raw/current/scripts/prefabs/wateringcan.lua#L83-L172)、[耐久](../../../raw/current/scripts/prefabs/wateringcan.lua#L146-L149) |
| `premiumwateringcan` | 同上工厂（return 处两次调用） | 鸟嘴壶：160 次耐久、水量同 25；与普通仅配方和耐久不同，机制无差异。[return](../../../raw/current/scripts/prefabs/wateringcan.lua#L174-L175)、[数值](../../../raw/current/scripts/tuning.lua#L5786-L5794) |
| `pitchfork` | `common_fn(bank, build)`＋`normal()` 包装 | 干草叉：tag `sharp`＋`weapon`；`terraformer`＋`finiteuses`（TERRAFORM 消耗 0.125，总量 25）＋`AddInherentAction(TERRAFORM)`；伤害 `wilson_attack*.5`=17。[common_fn](../../../raw/current/scripts/prefabs/pitchfork.lua#L33-L88)、[normal](../../../raw/current/scripts/prefabs/pitchfork.lua#L103-L105) |
| `goldenpitchfork` | 同 `common_fn`＋`golden()` 追加 master 初始化 | 黄金干草叉：TERRAFORM 消耗与攻击磨损折算 1/`GOLDENTOOLFACTOR`(=4)、金装备音；漂浮换装引用 `swap_goldenshovel` 符号。[golden](../../../raw/current/scripts/prefabs/pitchfork.lua#L107-L122)、[折算基数](../../../raw/current/scripts/tuning.lua#L95) |
| `minifan` | `fn()` 直接构造；主模拟 weapon→heater→insulator→fueled→equippable | 旋转的风扇：heater `equippedheat`＝`TUNING.MINIFAN_COOLER`=55，构造时即 `SetThermics(false, true)` 并给夏季隔热 60——装备后按 (55−体温)×倍率提供降温贡献（最终体温还看环境等项），不必先移动；`fueled` 只在 `locomote` 前进时耗油（90 秒），停走即停耗并清零热学／隔热，耗尽 `itemranout` 自毁；配方树枝3＋花瓣1。[组装](../../../raw/current/scripts/prefabs/minifan.lua#L110-L140)、[配方](../../../raw/current/scripts/recipes.lua#L523) |

名称键核验（[chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po)，`STRINGS.NAMES.*`）：`BRUSH`="刷子"、`BUGNET`="捕虫网"（Bug Net）两键相邻 [L339964-L339972](../../../raw/current/scripts/languages/chinese_s.po#L339964-L339972)；`GOLDENPITCHFORK`="黄金干草叉"（Snazzy Pitchfork）[L343364-L343367](../../../raw/current/scripts/languages/chinese_s.po#L343364-L343367)；`MINIFAN`="旋转的风扇"（Whirly Fan）[L345449-L345452](../../../raw/current/scripts/languages/chinese_s.po#L345449-L345452)；`PITCHFORK`="干草叉" [L346979-L346982](../../../raw/current/scripts/languages/chinese_s.po#L346979-L346982)；`PREMIUMWATERINGCAN`="空鸟嘴壶"／`_NOT_EMPTY`="鸟嘴壶"（Waterfowl Can）[L347279-L347287](../../../raw/current/scripts/languages/chinese_s.po#L347279-L347287)；`THULECITEBUGNET`="铥矿捕虫网" [L350379-L350382](../../../raw/current/scripts/languages/chinese_s.po#L350379-L350382)；`WATERINGCAN`="空浇水壶"／`_NOT_EMPTY`="浇水壶"（Watering Can）[L351789-L351797](../../../raw/current/scripts/languages/chinese_s.po#L351789-L351797)。浇水壶显示名经 `displaynamefn` 按有无水切键（[wateringcan.lua:75-77](../../../raw/current/scripts/prefabs/wateringcan.lua#L75-L77)）。

配方（[recipes.lua](../../../raw/current/scripts/recipes.lua)，均无附加数据）：`bugnet` 树枝4＋丝绢2＋绳索1（科学1档）、`thulecitebugnet` 铥矿2＋精尘1（远古档）[L399-L400](../../../raw/current/scripts/recipes.lua#L399-L400)；`pitchfork` 树枝2＋燧石2（科学1档）、`goldenpitchfork` 树枝4＋金块2（科学2档）[L402-L403](../../../raw/current/scripts/recipes.lua#L402-L403)；`wateringcan` 木板2＋绳索1（科学1档）、`premiumwateringcan` 浮木桩2＋绳索1＋邪天翁喙1（科学2档）[L442-L443](../../../raw/current/scripts/recipes.lua#L442-L443)；`minifan` 树枝3＋花瓣1（0 档）[L523](../../../raw/current/scripts/recipes.lua#L523)；`brush` 钢丝绒1＋海象长牙1＋金块2（科学2档）[L685](../../../raw/current/scripts/recipes.lua#L685)。反向消费：`gestalt_cage` 以 `thulecitebugnet` 为原料（[L1016](../../../raw/current/scripts/recipes.lua#L1016)）。

皮肤（[prefabskins.lua](../../../raw/current/scripts/prefabskins.lua)，仅登记清单，实体定义端未核验）：`bugnet` 5 款 [L379-L385](../../../raw/current/scripts/prefabskins.lua#L379-L385)、`pitchfork` 3 款 [L1447-L1451](../../../raw/current/scripts/prefabskins.lua#L1447-L1451)、`goldenpitchfork` 1 款 [L933-L935](../../../raw/current/scripts/prefabskins.lua#L933-L935)、`wateringcan` 2 款 [L2247-L2250](../../../raw/current/scripts/prefabskins.lua#L2247-L2250)、`premiumwateringcan` 1 款 [L1504-L1506](../../../raw/current/scripts/prefabskins.lua#L1504-L1506)；`brush`／`minifan`／`thulecitebugnet` 无条目。

## 关联机制导航

八注册公共基座：`inspectable`／`inventoryitem`（浇水壶另设 `getstatus` 空壶态与 `displaynamefn`，[wateringcan.lua:79-81](../../../raw/current/scripts/prefabs/wateringcan.lua#L79-L81)）；`equippable` 装备回调换 `ARM_carry` 并按 `GetSkinBuild` 分支推皮肤事件（四个有皮肤槽的注册有该分支 [bugnet.lua:7-17](../../../raw/current/scripts/prefabs/bugnet.lua#L7-L17)，brush／minifan 无）；全部 `MakeInventoryFloatable`＋`MakeHauntableLaunch`；耐久统一 `finiteuses`（minifan 例外用 `fueled`）。

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`finiteuses` | `SetConsumption` 建"动作→消耗"表，成功动作经 `OnUsedAsItem` 折算；`SetConsumption(TERRAFORM)` 自动映射 `TERRAFORM_REMOVE` 同值；耗尽 `SetOnFinished(inst.Remove)` | 捕虫网 NET 1、刷子 BRUSH 1、干草叉 TERRAFORM 0.125（金 0.03125）；`efficientuser` 乘数可降耗 | [[reference/components/finiteuses#关键接口]]；[映射](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)、[Use](../../../raw/current/scripts/components/finiteuses.lua#L85-L87) |
| 组件：`weapon` | `SetDamage`＋`attackwear`（默认 1）：攻击按 `attackwear`×乘数扣 `finiteuses` | 磨损 3：两网＋刷；golden 1/4；wateringcan `attackwearmultipliers` ×0 不磨损 | [[reference/components/weapon#关键接口]]；[扣次](../../../raw/current/scripts/components/weapon.lua#L105-L121)、[免磨损](../../../raw/current/scripts/prefabs/wateringcan.lua#L157-L159) |

### 捕虫网（bugnet／thulecitebugnet）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`tool` | `SetAction(ACTIONS.NET)` 写入 `NET_tool` 标签，效率 1 | 两注册同；未 `AddInherentAction` | [[reference/components/tool#关键接口]]；[登记](../../../raw/current/scripts/prefabs/bugnet.lua#L61-L62)、[标签写入](../../../raw/current/scripts/components/tool.lua#L41-L45)、[TOOLACTIONS](../../../raw/current/scripts/constants.lua#L2187-L2196) |
| 动作：`ACTIONS.NET` | 提供走 componentactions `tool` 处理器；`fn` 要求目标 `workable` 工作动作==NET、可工作、非死亡，持 `NET_tool` 豁免 `grabbable` 检查，成功 `WorkedBy` | 官方名"捕捉"（[strings.lua:596](../../../raw/current/scripts/strings.lua#L596)、[.po L1364-L1367](../../../raw/current/scripts/languages/chinese_s.po#L1364-L1367)）；canforce | [[reference/components/workable#关键接口]]；[处理器](../../../raw/current/scripts/componentactions.lua#L2507-L2520)、[fn](../../../raw/current/scripts/actions.lua#L1859-L1878) |
| 消费端：NET 目标 | 蜜蜂／蝴蝶／萤火虫／蚊子等把 `workable` 工作动作设为 NET | 目标端资格归 workable；仅抽样，全量未核验 | [butterfly.lua:166](../../../raw/current/scripts/prefabs/butterfly.lua#L166)（bee/fireflies/mosquito 同型已 grep） |
| 关联 Prefab：`wortox_nabbag` | NET `fn` 对带 `nabbag` 组件的持有物触发 `ReplicateNetFromAct` 锥形复制 | 强抢袋经沃拓克斯技能当捕网，`NABBAG_USES_AS_BUGNET`=20 折算 | [[reference/prefabs/wortox-nabbag]]；[复制](../../../raw/current/scripts/components/nabbag.lua#L12-L45)、[折算](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L209-L217)、[TUNING](../../../raw/current/scripts/tuning.lua#L7435) |

### 刷子（brush）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`brush` | 空壳标记组件（无字段），仅供 componentactions 识别 | 仅主模拟挂载 | [[reference/components/brush]]；[全文](../../../raw/current/scripts/components/brush.lua#L1-L5) |
| 动作：`ACTIONS.BRUSH` | 提供＝componentactions `brush` 处理器（USEITEM／EQUIPPED）：目标带 `brushable` 标签、左键 | 官方名"梳洗"（[strings.lua:795](../../../raw/current/scripts/strings.lua#L795)、[.po L344-L347](../../../raw/current/scripts/languages/chinese_s.po#L344-L347)）；目标战斗中／死亡拒绝 | [USEITEM](../../../raw/current/scripts/componentactions.lua#L1082-L1085)、[EQUIPPED](../../../raw/current/scripts/componentactions.lua#L2247-L2250)、[fn](../../../raw/current/scripts/actions.lua#L4205-L4216) |
| 数据：目标侧 `brushable` | `Brush` 按"世界天数÷`cyclesperprize`，封顶 `max`"产出 `prize` 并推 `brushed` 事件 | 原版 prefab 消费者仅 `beefalo`：prize `beefalowool`、max 1；产出>0 加驯化与服从 | [[reference/components/brushable]]；[计算](../../../raw/current/scripts/components/brushable.lua#L31-L34)、[beefalo](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[OnBrushed](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L494) |

### 浇水壶（wateringcan／premiumwateringcan）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`wateryprotection` | 浇灌主链：`POUR_WATER`／`POUR_WATER_GROUNDTILE` → `SpreadProtection(AtPoint)`：半径 2.25（排除 `player` 等）灭火／防枯 30 秒／降温 5／加湿 25；土壤湿度走 `farming_manager`；`onspreadprotectionfn=onuse` 每浇一次 `Use(1)` | 地块动作仅 `FARMING_SOIL` 地皮；对目标要求 withered/fire/smolder/canpourwateron 标签 | [[reference/components/wateryprotection#关键接口]]、[[reference/components/farming-manager]]；[配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L131-L138)、[扩散](../../../raw/current/scripts/components/wateryprotection.lua#L46-L60)、[防枯单位](../../../raw/current/scripts/components/witherable.lua#L248-L256)、[门禁](../../../raw/current/scripts/componentactions.lua#L2232-L2236) |
| 组件：`fillable` | `FILL`（对 `watersource` 目标）→ `overrideonfillfn`：水源带 `override_fill_uses` 按点数回充（冰/雪球/水气球＝1/0.3/1），否则回满；`acceptsoceanwater=false` 使 `FILL_OCEAN` 报 `UNSUITABLE_FOR_PLANTS` | 两注册同；显示 `showoceanaction` 但故意拒海水 | [[reference/components/fillable#关键接口]]、[[reference/components/watersource]]；[Fill 链](../../../raw/current/scripts/components/fillable.lua#L33-L35)、[OnFill](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37)、[FILL fn](../../../raw/current/scripts/actions.lua#L2126-L2164)、[override 消费](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L117) |
| 组件：`fuel`＋`burnable` | 空壶（≤0）动态挂 `fuel`（`SMALL_FUEL`=15 秒、BURNABLE），有水即移除；常驻可燃，烧毁按 `DefaultBurntFn` 成灰移除，另听 `onburnt` 按 `ceil(剩余×25×0.1)` 补土壤湿度 | 空壶＝可投燃料，有水壶＝不可投但可被烧 | [切换](../../../raw/current/scripts/prefabs/wateringcan.lua#L39-L59)、[onburnt](../../../raw/current/scripts/prefabs/wateringcan.lua#L61-L67)、[烧毁](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[SMALL_FUEL](../../../raw/current/scripts/tuning.lua#L1674) |
| 边界：空壶动作 | `POUR_WATER` 对 `uses<=0` 拒绝并报 `OUT_OF_WATER`（仅 `wateringcan` 标签物品） | 空壶只能装水或当燃料 | [拒绝](../../../raw/current/scripts/actions.lua#L5416)、[标签](../../../raw/current/scripts/prefabs/wateringcan.lua#L113) |

### 干草叉（pitchfork／goldenpitchfork）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`terraformer` | `TERRAFORM` 右键点动作（POINT 门禁 `CanTerraformAtPoint`）：`Terraform` 换地块为 tile 下土或 `DIRT`，`HandleDugGround` 掉地皮或播 `sinkhole_spawn_fx_1..3`（prefabs 依赖），`soil` 实体收 `collapsesoil` | 两注册同；无 `plow` 标签走普通铲地 | [[reference/components/terraformer#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/pitchfork.lua#L79)、[fn](../../../raw/current/scripts/actions.lua#L2873-L2877)、[门禁](../../../raw/current/scripts/componentactions.lua#L2224-L2230)、[掉落](../../../raw/current/scripts/componentutil.lua#L31-L48) |
| 动作：`TERRAFORM`／`_REMOVE` | `AddInherentAction(TERRAFORM)`＋`tile_placer="gridplacer"`；EQUIPPED 侧对 `terraformerremoveable` 目标（嘉年华高尔夫草皮）提供 REMOVE，消耗经映射同值 | 官方名"挖"（[strings.lua:530](../../../raw/current/scripts/strings.lua#L530)、[.po L2149-L2152](../../../raw/current/scripts/languages/chinese_s.po#L2149-L2152)） | [登记](../../../raw/current/scripts/prefabs/pitchfork.lua#L73)、[REMOVE](../../../raw/current/scripts/componentactions.lua#L2501-L2504)、[映射](../../../raw/current/scripts/components/finiteuses.lua#L31-L32) |

### 小风扇（minifan）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`heater` | `equippedheat=55`＋构造期 `SetThermics(false, true)`（外放热关、内吸冷开）；temperature 装备槽按"heat<当前温度 且 endothermic"把 `(55−体温)×倍率` 计入体温增量，环境温度与其他修正仍参与 | 构造即生效；`locomote` 前进保持该态并开始耗油，停走按原回调关热学与隔热，卸装同样归零；再次装备不自动恢复，直到下次前进 | [[reference/components/heater#关键接口]]、[[reference/components/temperature]]；[配置](../../../raw/current/scripts/prefabs/minifan.lua#L126-L128)、[门控](../../../raw/current/scripts/prefabs/minifan.lua#L149-L163)、[判定](../../../raw/current/scripts/components/temperature.lua#L371-L381) |
| 组件：`insulator` | `SetInsulation(INSULATION_SMALL)`（=60 秒隔热当量）＋`SetSummer()` | 构造时设为 60；停走与卸装归 0，之后只有 `locomote` 前进才重设 | [[reference/components/insulator]]；[配置](../../../raw/current/scripts/prefabs/minifan.lua#L132-L134)、[归零](../../../raw/current/scripts/prefabs/minifan.lua#L50-L54)、[数值](../../../raw/current/scripts/tuning.lua#L2260) |
| 组件：`fueled` | `InitializeFuelLevel(MINIFAN_FUEL)`＝`day_time*0.3`=90 秒、`period=1`/`rate=1`（移动中 1 秒耗 1）、`SetFirstPeriod(1/60)` 抹平首拍；耗尽推 `itemranout`＋`ANNOUNCE_FAN_OUT` 并 `Remove` | `accepting` 未开＝不能用燃料物品续注；`fueltype` 默认 BURNABLE（非 USAGE，不出"需缝补"） | [[reference/components/fueled#关键接口]]；[配置](../../../raw/current/scripts/prefabs/minifan.lua#L142-L145)、[depleted](../../../raw/current/scripts/prefabs/minifan.lua#L68-L78)、[耗油](../../../raw/current/scripts/components/fueled.lua#L325-L337)、[FUEL](../../../raw/current/scripts/tuning.lua#L1717)、[时间基数](../../../raw/current/scripts/tuning.lua#L19-L27) |
| 关联 Prefab：`fan_wheel` | 装备时 `SpawnPrefab`＋`SetParent` 到 owner，随 `locomote` 决定旋转；`onremove` 清理 | 纯表现实体（net_bool 同步）；卸装 `StartUnequipping` | [生成](../../../raw/current/scripts/prefabs/minifan.lua#L20-L37)、[SetSpinning](../../../raw/current/scripts/prefabs/fan_wheel.lua#L64)、[注册](../../../raw/current/scripts/prefabs/fan_wheel.lua#L153) |

## 典型接入

### 解除 minifan 的移动门控（站立也制冷耗油）

- 前提与选择：modmain `AddPrefabPostInit` 回调在实体构造后执行，晚于工厂初始化；选包装 `SetOnEquip` 而非整体替换，保留原持握外观、轮子与监听登记。
- 生效与撤销：影响此后生成与读档重建的 minifan；每次装备时移除 `locomote` 监听，原 onunequip 停转／归零链保留，卸装仍停耗、空油仍自毁。
- 原版依据：门控本体是 `locomote` 监听（上文小风扇表）；modmain 模组环境（[mods.lua:258-317](../../../raw/current/scripts/mods.lua#L258-L317)）无 `__index=_G`，`TheWorld`／引擎全局须 `GLOBAL.` 前缀，`TUNING` 由环境直供。

```lua
-- modmain.lua（模组环境：AddPrefabPostInit/TUNING 由环境提供；TheWorld 用 GLOBAL.）
local function OneEquipKeepCooling(inst, owner)
    inst._orig_onequip(inst, owner) -- 原持握换装、fan_wheel 生成与 locomote 监听登记
    inst:RemoveEventCallback("locomote", inst._onlocomote, owner) -- 监听登记在 inst、事件源是 owner
    inst.components.heater:SetThermics(false, true)
    inst.components.insulator:SetInsulation(TUNING.INSULATION_SMALL)
    inst.components.fueled:StartConsuming()
    if inst._wheel ~= nil then
        inst._wheel:SetSpinning(true)
    end
end

AddPrefabPostInit("minifan", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local equippable = inst.components.equippable
    if equippable ~= nil then
        inst._orig_onequip = equippable.onequipfn
        equippable:SetOnEquip(OneEquipKeepCooling)
    end
end)
```

- 调整与验证：`onequipfn` 是 equippable 已核验字段（[equippable.lua:68-70](../../../raw/current/scripts/components/equippable.lua#L68-L70)）；解绑必须用 `inst:RemoveEventCallback("locomote", inst._onlocomote, owner)`——监听方是 inst、事件源是 owner（[entityscript.lua:1249-1256](../../../raw/current/scripts/entityscript.lua#L1249-L1256)），写成 `owner:RemoveEventCallback(...)` 不会解除原监听；原 `onequip` 不恢复隔热，示例补 `SetInsulation` 才保留站立隔热。站立制冷仍只把体温拉向 55（装备槽 endothermic 条件），低温环境无升温。浇水量／海边装水／铲地保裸地等组件级案例见 [[reference/components/wateryprotection#典型接入]]、[[reference/components/fillable#典型接入]]、[[reference/components/terraformer#典型接入]]。

## 常见陷阱与限制

- **误区：干草叉能"挖干草"**：无 `tool` 组件、不走 DIG，只做 `TERRAFORM` 地块置换（不产生草类掉落），收割草是目标 `pickable` 侧行为。[工厂](../../../raw/current/scripts/prefabs/pitchfork.lua#L33-L88)、[Terraform](../../../raw/current/scripts/components/terraformer.lua#L12-L48)
- **误区：捕虫网动作供给缺失**：`bugnet` 未 `AddInherentAction`，NET 资格完全来自 `NET_tool` 标签处理器（与 shovel 的 DIG 处理器同机制）；改动作供给应加该标签或改处理器，不能假设 `inherentactions` 生效。[处理器](../../../raw/current/scripts/componentactions.lua#L2507-L2520)、[shovel 对照](../../../raw/current/scripts/prefabs/shovel.lua#L111)
- **brushable 的 `regrowthdays` 不被读取**：beefalo 设置了 `brushable.regrowthdays=1`，但产出公式只用 `cyclesperprize`（默认 0）：距上次梳理 ≥1 天时"天数/0"→∞，封顶 `max=1`——每天至多刷出 1 团牛毛、同日重刷无产出；写模组以 `cyclesperprize` 为准。[beefalo](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[公式](../../../raw/current/scripts/components/brushable.lua#L31-L34)
- **浇水壶的"燃料"是条件态**：可燃性常驻，烧尽成灰无 `_burnt` 版；"能当燃料投入火堆"仅空壶成立（`fuel` 组件 ≤0 才挂载、>0 即移除），烧毁瞬间按剩余水量给农田补湿。[切换](../../../raw/current/scripts/prefabs/wateringcan.lua#L53-L59)、[onburnt](../../../raw/current/scripts/prefabs/wateringcan.lua#L61-L67)
- **minifan 耐久是 `fueled` 不是 `finiteuses`**：无 `SetConsumption` 表、不吃 `efficientuser`；`accepting=false` 不能用燃料物品续注，90 秒移动耗尽即毁；耗尽事件是 `itemranout` 而非 `toolbroke`。[depleted](../../../raw/current/scripts/prefabs/minifan.lua#L68-L78)、[CanAcceptFuelItem](../../../raw/current/scripts/components/fueled.lua#L169-L175)
- **边界：minifan 有三个热学状态，别都当"移动才降温"**。工厂在挂 heater 后立刻 `SetThermics(false, true)` 并给夏季隔热 60，装备后按 (55−体温)×倍率提供降温贡献，此时还没开始耗油；最终体温仍由环境等项决定；`locomote` 前进才开始耗油并保持该态；停走或卸装才关热学、把隔热归 0，重新装备不会自动恢复，直到再次前进。读档重建也按构造态起算。[构造](../../../raw/current/scripts/prefabs/minifan.lua#L119-L134)、[装备回调](../../../raw/current/scripts/prefabs/minifan.lua#L20-L37)、[停走归零](../../../raw/current/scripts/prefabs/minifan.lua#L149-L163)、[判定](../../../raw/current/scripts/components/temperature.lua#L368-L381)
- **goldenpitchfork 漂浮换装引用异库符号**：`SetBankSwapOnFloat(true, 7, {sym_build="swap_goldenshovel"})`，本文件 assets 未声明该 zip（疑为从 goldenshovel 复制的笔误）；实际漂浮视觉是否回退未核验。[golden](../../../raw/current/scripts/prefabs/pitchfork.lua#L119)、[assets](../../../raw/current/scripts/prefabs/pitchfork.lua#L1-L7)
- **playercontroller 的 `brush` 标签分支对原版刷子不生效**：`GetLeftClickAction` 有 `tool:HasTag("brush")` 分支，但原版 `brush` 未加该标签，动作仍由 `brush` 组件处理器供给；模组自行加标签才走该分支。[分支](../../../raw/current/scripts/components/playercontroller.lua#L2059)、[brush 无标签](../../../raw/current/scripts/prefabs/brush.lua#L18-L68)

## 验证记录

- 源码审阅：2026-11-07，steam-buildid 26335394（`raw/current/BUILD.txt`）。完整读取五个 prefab 文件并核对全部 return 注册；第二轮检视修订复核 minifan 构造期 `SetThermics`／`SetSummer`、`locomote` 监听与解绑签名、wateringcan `onuse` 直接扣次；核验 equippable 默认 HANDS、finiteuses 消耗链与 TERRAFORM_REMOVE 映射、weapon attackwear、NET/TERRAFORM/BRUSH/FILL/POUR_WATER 定义与 fn、componentactions 六类处理器（USEITEM/POINT/EQUIPPED 定位）、brushable 计算与 beefalo 消费、wateryprotection 扩散与 farming_manager 土壤湿度、fillable 装水与拒海水、heater/temperature 装备制冷判定、fueled 耗油与 depleted、fan_wheel 生成清理、nabbag 复制入口、HandleDugGround 与 DefaultBurntFn、mods.lua 模组环境、tuning 引用键（时间基数 day_time=300／total_day_time=480 单独复核）、recipes 八条配方、.po 十个名称键与三个动作键、prefabskins 皮肤清单。
- 游戏运行：未运行。未覆盖：NET 可捕目标全量清单与各自掉落、皮肤实体定义端与皮肤事件消费端、引擎侧 inherentactions 读取与动画、scrapbook specialinfo 数据端、quagmire/节日差异、goldenpitchfork 漂浮视觉实际表现、POUR_WATER 对 `canpourwateron` 目标的原版分布、.po 中 POUR_WATER 系动作键。
- 结构检查：validate_page.py 与 validate_source_links.py 已实际运行（结果见交付说明）；validate_page 实测 147 行／27296 字节（补 minifan 注册行后），`complex` 档（48 KiB／480 行）——按"快速判断"所记的多变体族结构选档，非字节触发；validate_source_links --strict 121 条引用 0 error 0 warning。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
