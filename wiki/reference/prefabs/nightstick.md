---
title: "晨星锤（nightstick）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版手持电击装备 nightstick（官方中文名晨星锤）：HANDS 槽电属性近战武器，装备时由 nightstickfire 跟随 FX 提供照明并按 fueled 计时耗电，耗尽整件销毁，可在 Winona 电池／避雷针等 battery 电源上 CHARGE_FROM 回充燃料。"
aliases: ["nightstick", "晨星锤", "Morning Star", "光杖", "电击照明武器", "电池充电武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/nightstick.lua", "prefabs/nightstickfire.lua", "prefabs/sparks.lua", "prefabs/player_common.lua", "prefabskins.lua", "prefabs/skinprefabs.lua", "prefabs/winona_battery_high.lua", "prefabs/winona_battery_low.lua", "prefabs/lightningrod.lua", "prefabs/scienceprototyper.lua", "components/weapon.lua", "components/equippable.lua", "components/fueled.lua", "components/burnable.lua", "components/batteryuser.lua", "components/battery.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/inventory.lua", "components/combat.lua", "components/wildfires.lua", "componentutil.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "mainfunctions.lua", "entityscript.lua", "tuning.lua", "recipes.lua", "constants.lua", "mods.lua", "skin_strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 晨星锤（nightstick）

## 快速判断

- 职责与适用：`prefabs/nightstick.lua` 顶层 `fn` 直接注册的单注册可拾取手持装备（物品＋装备）：HANDS 槽电属性近战武器；照明由独立 FX Prefab `nightstickfire` 跟随宿主提供，装备才亮；燃料仅装备燃烧时计时、耗尽整件销毁；`batteryuser` 支持从 battery 电源回充。同文件无 quagmire 分支，scripts 内 grep 无其他 nightstick 专属变体。[注册](../../../raw/current/scripts/prefabs/nightstick.lua#L124-L198)
- 优先使用：复刻"电击武器＋装备照明＋电池回充"按[关联机制导航](#关联机制导航)组装并查 [[cookbook/equipment/hand]]；改伤害／照明／燃料／充电参数查[数值表](#注册与构造)；改充电行为查 [[reference/components/batteryuser#关键接口]]。
- 重要副作用：装备即 `burnable:Ignite`→`fueled:StartConsuming` 每秒扣 1 燃料，卸下／入袋即停；攻击命中生成电击火花，电属性伤害对非免疫目标 ×1.5（湿目标最高 ×2.5）；耗尽先 `Remove` 再广播 `itemranout`（自动补装＋播报）。[onfuelchange](../../../raw/current/scripts/prefabs/nightstick.lua#L78-L101)、[onattack](../../../raw/current/scripts/prefabs/nightstick.lua#L103-L105)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/nightstick.lua#L148-L152)）；照明 Light 在 `nightstickfire` 实体而非本体；充电倍率与接收回调在本 prefab 内，电源侧消费规则归 [[reference/components/battery#原版宿主与消费边界]]，本页不展开电池宿主机制。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightstick` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；`nightstickfire` 是独立 FX Prefab（`prefabs` 数组声明且装备时实际 `SpawnPrefab`）[注册](../../../raw/current/scripts/prefabs/nightstick.lua#L198)、[声明](../../../raw/current/scripts/prefabs/nightstick.lua#L8-L12) |

共同初始化（[nightstick.lua#L124-L196](../../../raw/current/scripts/prefabs/nightstick.lua#L124-L196)）：实体组件＋`MakeInventoryPhysics` → bank `nightstick`＋build `nightstick`＋`idle` → 公共端标签 `wildfireprotected`／`batteryuser`／`weapon` → `MakeInventoryFloatable("med", 0.05, {1.1,0.4,1.1}, swap_bank, -19, {sym_build="swap_nightstick"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → inventoryitem → equippable → inspectable → burnable → fueled → batteryuser → `MakeHauntableLaunch` → `OnRemoveEntity`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.NIGHTSTICK`＝晨星锤（英文 msgid "Morning Star"）；通用检视语＝"它带电！"（"It's electric!"）。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L346174-L346177)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L16494-L16496)

数值（tuning 键＋实际值；`seg_time`＝30 秒、`night_time`＝30×2＝60 秒、`wilson_attack`＝34×倍率 1，[tuning.lua#L19-L37](../../../raw/current/scripts/tuning.lua#L19-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.NIGHTSTICK_DAMAGE`＝`wilson_attack`×.85 | 28.9 生命值（源码推导，[tuning.lua#L376](../../../raw/current/scripts/tuning.lua#L376)） |
| 电伤乘区 | `TUNING.ELECTRIC_DAMAGE_MULT`／`ELECTRIC_WET_DAMAGE_MULT` | 1.5＋1×目标 `GetWetMultiplier()`：干燥 ×1.5≈43.35，湿目标最高 ×2.5≈72.25；`SetElectric()` 无参即用默认乘区（[tuning.lua#L659-L660](../../../raw/current/scripts/tuning.lua#L659-L660)、[结算](../../../raw/current/scripts/components/combat.lua#L1200-L1210)） |
| 燃料总量 | `TUNING.NIGHTSTICK_FUEL`＝`night_time`×6 | 360 燃料单位＝rate 1 下装备燃烧 360 秒（0.75 游戏日，[tuning.lua#L1722](../../../raw/current/scripts/tuning.lua#L1722)） |
| 点亮首扣时长 | `TUNING.TURNON_FUELED_CONSUMPTION`／`TURNON_FULL_FUELED_CONSUMPTION` | 0.5 秒量／满燃料 1/60 秒量（[tuning.lua#L1668-L1669](../../../raw/current/scripts/tuning.lua#L1668-L1669)） |
| 充电倍率 | `CalcBatteryChargeMult` 硬编码 | `1 − fueled:GetPercent()`（缺失比例，夹 0~1）；满电或倍率非正拒绝 `CHARGE_FULL`（[倍率](../../../raw/current/scripts/prefabs/nightstick.lua#L107-L110)、[接收](../../../raw/current/scripts/prefabs/nightstick.lua#L112-L122)） |
| 照明参数 | nightstickfire 硬编码 | Intensity 0.75、Colour (252,251,237)/255、Falloff 0.6、Radius 4（[nightstickfire.lua#L10-L14](../../../raw/current/scripts/prefabs/nightstickfire.lua#L10-L14)） |
| 漂浮参数 | floater 配置 | "med"、垂直偏移 0.05、缩放 {1.1,0.4,1.1}、漂浮换 bank（float_index −19，sym_build `swap_nightstick`）（[调用](../../../raw/current/scripts/prefabs/nightstick.lua#L146)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)） |
| 作祟 | `MakeHauntableLaunch` 默认 | 必弹飞、冷却 `HAUNT_COOLDOWN_SMALL`、`LAUNCH_SPEED_SMALL`、`HAUNT_TINY`（[调用](../../../raw/current/scripts/prefabs/nightstick.lua#L191)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)） |

获取途径：配方 伏特羊角（lightninggoathorn）1＋电子元件（transistor）2＋硝石（nitre）2，`TECH.SCIENCE_TWO`（SCIENCE 树 2 级，由炼金引擎 `researchlab2` 提供该档）。[配方](../../../raw/current/scripts/recipes.lua#L482)、[科技键](../../../raw/current/scripts/constants.lua#L1246)、[档位映射](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299-L300)、[ALCHEMYMACHINE 树](../../../raw/current/scripts/tuning.lua#L1267-L1270)。scripts 内 grep `nightstick` 无其他玩法消费端（`recipes_filter`／`debugcommands`／`misc_items` 为配方过滤、调试与皮肤换购名单）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(NIGHTSTICK_DAMAGE)`＋`SetOnAttack(onattack)`＋`SetElectric()`（乘区参数 nil，用默认电伤乘区） | 主模拟；无 finiteuses、无磨损、无射程 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L154-L157)、[SetElectric](../../../raw/current/scripts/components/weapon.lua#L73-L78) |
| 组件：inventoryitem／inspectable | 默认挂载，无额外配置 | 主模拟 | [[reference/components/inventoryitem#关键接口]]、[[reference/components/inspectable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L159)、[挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L170) |
| 组件：equippable | 四回调：onequip（点火＋持握符号＋循环音＋生成 fire）、onunequip、onpocket（Extinguish 兜底）、onequiptomodel | 主模拟；槽默认 HANDS；装备态入袋先走 `Inventory:Unequip`，onpocket 在 GiveActiveItem 等路径兜底 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/nightstick.lua#L162-L166)、[ToPocket](../../../raw/current/scripts/components/equippable.lua#L109-L113)、[GiveItem 先 Unequip](../../../raw/current/scripts/components/inventory.lua#L1035-L1039) |
| 组件：burnable | 状态联动载体：`canlight=false`（无 `canlight` 标签）、`fxprefab=nil`（无自身燃烧 FX） | 主模拟；Ignite→`fueled:StartConsuming`（[启动](../../../raw/current/scripts/components/burnable.lua#L380-L382)），Extinguish→`StopConsuming`（[停止](../../../raw/current/scripts/components/burnable.lua#L486-L488)） | [[reference/components/burnable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/nightstick.lua#L174-L176) |
| 组件：fueled | `InitializeFuelLevel(NIGHTSTICK_FUEL)`、`SetDepletedFn(inst.Remove)`、分段回调 onfuelchange、`SetFirstPeriod` | 主模拟；默认 `fueltype=BURNABLE`、`accepting=false` 不可喂料、sections=1、period=1、rate=1 | [[reference/components/fueled#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L180-L184)、[消耗](../../../raw/current/scripts/components/fueled.lua#L325-L337) |
| 组件：batteryuser | 倍率＝缺失燃料比例、接收回调回充＋电火花、`SetAllowPartialCharge(true)` | 主模拟；电源须带 `battery` 标签／组件 | [[reference/components/batteryuser#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L186-L189) |
| 组件：hauntable／floater | `MakeHauntableLaunch` 作祟弹飞；漂浮态换持握符号 | hauntable 主模拟；floater 公共端 | [[reference/components/hauntable#按效果选工厂]]、[[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/nightstick.lua#L191) |
| API／动作：`CHARGE_FROM` | 持拿（USEITEM）或装备（EQUIPPED）本物右键 battery 目标、或对 battery 场景物（SCENE）执行 → `BatteryUser:ChargeFrom` | 右键路径均要求双方标签；动作优先取 invobject 为接收者 | [[reference/components/batteryuser#关键接口]]；[动作](../../../raw/current/scripts/actions.lua#L5852-L5874)、[收集](../../../raw/current/scripts/componentactions.lua#L1058-L1062)、[收集](../../../raw/current/scripts/componentactions.lua#L2241-L2245) |
| API／事件：`itemranout` | 装备态耗尽先 `Remove` 再推事件，data 带 `prefab`／`equipslot`／`announce="ANNOUNCE_TORCH_OUT"` | 玩家侧监听自动补装同 prefab 工具 | [耗尽块](../../../raw/current/scripts/prefabs/nightstick.lua#L84-L100)、[OnItemRanOut](../../../raw/current/scripts/prefabs/player_common.lua#L612-L623) |
| API／数据：`battery` 宿主 | 原版可充电目标：winona_battery_low（L943-946）、winona_battery_high（紫宝石等 gems 供电）、lightningrod（L191-194） | 紫宝石喂 G.E.M.erated 电池发电后经 CHARGE_FROM 回充；电源侧倍率消费规则见目标页 | [[reference/components/battery#原版宿主与消费边界]]；[gems 接受测试](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1152-L1171) |
| API／数据：`wildfireprotected` 标签 | 夏季野火事件排除该实体 | 全端读取 | [排除表](../../../raw/current/scripts/components/wildfires.lua#L27) |
| 关联 Prefab：`nightstickfire` | 照明 FX 实体，装备才存在（见下小节） | onequip 生成、卸下／入袋路径／本体移除即 `Remove` | [声明](../../../raw/current/scripts/prefabs/nightstick.lua#L8-L12)、[生成](../../../raw/current/scripts/prefabs/nightstick.lua#L39-L44) |
| 关联 Prefab：`electrichitsparks` | 攻击命中（onattack）与充电成功时 `SpawnElectricHitSparks`；目标电免疫时换 `electrichitsparks_electricimmune` | 服务端 `SpawnPrefab` 后 `AlignToTarget` | [helper](../../../raw/current/scripts/componentutil.lua#L1022-L1030)、[注册](../../../raw/current/scripts/prefabs/sparks.lua#L177-L179)、[电免疫判定](../../../raw/current/scripts/componentutil.lua#L22-L24) |
| 资源：anim／音效 | nightstick.zip（bank/build `nightstick`）、swap_nightstick.zip（持握符号 `swap_object`）、common.fsb；装备循环音 `dontstarve_DLC001/common/morningstar`（loop 名 "torch"） | 静态声明；皮肤 build 经 `GetSkinBuild` 走 `OverrideItemSkinSymbol` | [声明](../../../raw/current/scripts/prefabs/nightstick.lua#L1-L6)、[音效](../../../raw/current/scripts/prefabs/nightstick.lua#L36) |

### nightstickfire（照明 FX 实体，同页简述）

`Prefab("nightstickfire", fn)` 单文件注册（[nightstickfire.lua#L27](../../../raw/current/scripts/prefabs/nightstickfire.lua#L27)）：纯 FX 实体（`FX` 标签、无 AnimState），`Transform`＋`Light`＋`Network`；主模拟端 `persists=false` 不进存档（[L22](../../../raw/current/scripts/prefabs/nightstickfire.lua#L22)）。Light 参数在 `SetPristine` 前硬编码、两端构造时同样执行（见[数值表](#注册与构造)照明行）。生命周期：onequip 时 `SpawnPrefab` 并互设引用（`inst.fire`／`fire.nightstick`），`ListenForEvent("onremove")` 清引用，`fire.entity:SetParent(owner.entity)` 引擎父子挂宿主；onunequip、onequiptomodel、本体 `OnRemoveEntity` 均 `fire:Remove()`（[onequip](../../../raw/current/scripts/prefabs/nightstick.lua#L39-L44)、[清理](../../../raw/current/scripts/prefabs/nightstick.lua#L47-L76)）。因此照明严格随装备存在：卸下／入袋／耗尽销毁即灭，重进世界不保留 fire，重新装备时重建。皮肤仅换 build（`equipskinneditem`／`unequipskinneditem` 事件＋`OverrideItemSkinSymbol`，[onequip](../../../raw/current/scripts/prefabs/nightstick.lua#L25-L31)），无 FX 变体名单；唯一皮肤 `nightstick_swamp`（"Primal Morning Star"，Woven Elegant，item 类）。[皮肤名单](../../../raw/current/scripts/prefabskins.lua#L1346-L1349)、[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8656-L8665)、[皮肤名](../../../raw/current/scripts/skin_strings.lua#L2563)

## 典型接入

### 调整晨星锤照明光圈（Light 在 nightstickfire 实体上）

- 前提与选择：目标模组根 `modmain.lua`，两端加载，无需新增资源。光参数是 `nightstickfire` 构造时硬编码（[nightstickfire.lua#L10-L14](../../../raw/current/scripts/prefabs/nightstickfire.lua#L10-L14)），本体组件与 tuning 均无光圈入口，故 postinit 该 FX Prefab。`AddPrefabPostInit` 回调在 `SpawnPrefab` 构造后于当前端执行（[mainfunctions.lua#L388-L407](../../../raw/current/scripts/mainfunctions.lua#L388-L407)），客户端与服务端都会构造 `nightstickfire`，因此**不要**加 `ismastersim` 门禁，两端同改才与原版 pristine 前设置等价。
- 生效与撤销：每次装备都会重新 `SpawnPrefab("nightstickfire")`，postinit 对每个新实例生效；停用模组恢复半径 4。

```lua
-- modmain.lua：晨星锤光圈半径 4 → 6（照明在 nightstickfire FX 实体上，须两端生效）
AddPrefabPostInit("nightstickfire", function(inst)
    inst.Light:SetRadius(6)
end)
```

- 调整与验证：半径单位为游戏长度单位；Intensity／Falloff／Colour 同在 pristine 前硬编码，可同法修改。若另改服务端专属行为（weapon／fueled／batteryuser 等），回调须加 `GLOBAL.TheWorld.ismastersim` 门禁：modmain 环境只直挂 `TUNING` 等白名单、无 `__index=_G` 回退（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L317)），`TheWorld`／`SpawnPrefab` 等须 `GLOBAL.` 前缀，写法参考 [[reference/prefabs/ruins-bat#典型接入]]。未做游戏运行验证。

## 常见陷阱与限制

- **误区：可以像火把一样点燃或续燃料**：`canlight=false`→无 `canlight` 标签，LIGHT 收集与闪电等外部点火路径都触不到它；`fueled.accepting=false`→无 `burnable_fueled` 标签，ADDFUEL 不可用；fueltype 默认 `BURNABLE`（非 `USAGE`）也不会加 `needssewing`，无缝补修复。补能只有电池回充一条路。[标签设置](../../../raw/current/scripts/components/burnable.lua#L6-L8)、[接受判定](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[配置](../../../raw/current/scripts/prefabs/nightstick.lua#L174-L180)
- **边界：照明与耗电都严格随装备态**：`nightstickfire` 不进存档、只由 onequip 生成；卸下／入袋／本体移除即灭即停耗，装回重建。改照明必须落在 FX 实体或两端 postinit，只改本体无效。[生命周期](#nightstickfire照明-fx-实体同页简述)
- **边界：攻击无磨损**：无 finiteuses，`weapon:OnAttack` 的耐久分支不触达，攻击不耗耐久也不额外耗燃料；唯一消耗是装备时每秒 1 燃料。[OnAttack 分支](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **边界：电伤加成取决于目标**：带 `electricdamageimmune` 标签或背包绝缘（`inventory:IsInsulated`）的目标无乘区且火花换免疫版；湿目标乘区最高 ×2.5（`GetWetMultiplier` 返回 0~1）。[免疫判定](../../../raw/current/scripts/componentutil.lua#L22-L24)、[湿度系数](../../../raw/current/scripts/entityscript.lua#L770-L791)
- **边界：满电充电被拒绝且不扣电源**：接收回调满电或倍率非正时返回 `CHARGE_FULL`，`BatteryUser:ChargeFrom` 不到成功分支就不调电源 `OnUsed`；部分充电靠电源侧 `ResolvePartialChargeMult` 降倍率。[接收链](../../../raw/current/scripts/components/batteryuser.lua#L31-L48)
- **边界：耗尽先 `Remove` 再发 `itemranout`**：自动补装依赖此顺序；装备态耗尽播报 `ANNOUNCE_TORCH_OUT`。[耗尽块](../../../raw/current/scripts/prefabs/nightstick.lua#L84-L100)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394。核验 prefabs/nightstick.lua 全文、nightstickfire.lua 全文；weapon／equippable／fueled 全文与 burnable（Ignite/Extinguish/标签）、batteryuser／battery 全文、inspectable 全文、inventoryitem 头部 replica 接线；CHARGE_FROM 定义与 SCENE/USEITEM/EQUIPPED 三条收集路径；combat 电伤乘区、IsEntityElectricImmune、GetWetMultiplier；SpawnElectricHitSparks 与 sparks.lua 注册；tuning 键值与换算（seg_time/night_time/wilson_attack/TURNON/ELECTRIC_*）、配方行与科技档（constants、scienceprototyper、PROTOTYPER_TREES）；名称与检视语 po 键；player_common OnItemRanOut；皮肤注册三处；充电宿主 battery 挂载（winona_battery_low/high、lightningrod）与 G.E.M.erator gems 接受测试；scripts 顶层与 prefabs 目录 grep `nightstick` 复核无其他玩法消费端。未覆盖：battery 宿主完整供电机制（归 [[reference/components/battery]]）、G.E.M.erator 供电细节、皮肤实装外观。
- 游戏运行：未运行。360 秒燃尽、干燥 ×1.5／湿 ×2.5 伤害为源码推导而非运行测量；光圈观感与充电实机流程未测。
- 结构检查：2026-11-06 validate_page.py 0 error（104 行／20120 字节，standard）与 validate_source_links.py --strict 0 error 0 warning（74 条源码引用、28 个来源文件）通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]；同批照明对照 [[reference/prefabs/torch|火炬]]、[[reference/prefabs/lighter|薇洛的打火机]]；充电机制详见 [[reference/components/batteryuser]]、[[reference/components/battery]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
