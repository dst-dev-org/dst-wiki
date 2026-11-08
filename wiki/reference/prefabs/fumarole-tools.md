---
title: "气孔工具族（fumaroleaxe、fumarolepickaxe、fumaroleshovel、fumarolehammer、fumarole_farm_hoe）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "气孔温度工具族：五件手持工具共用 MakeFumaroleTool 工厂协议接入物品温度档位——使用降温、冷却自毁、重热自修、分档外观与效率、对冻结目标增伤，攻击与工作均不磨损。"
aliases: ["fumaroleaxe", "fumarolepickaxe", "fumaroleshovel", "fumarolehammer", "fumarole_farm_hoe", "炽热斧", "烈焰鹤嘴锄", "呲啦铲", "炎头锤", "烧灼园艺锄", "气孔工具", "气孔斧", "气孔镐", "气孔铲", "气孔锤", "气孔园艺锄", "Fumarole Tool", "热工具", "温度工具"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
size_class: complex
sources: ["standardcomponents.lua", "prefabs/axe.lua", "prefabs/pickaxe.lua", "prefabs/shovel.lua", "prefabs/hammer.lua", "prefabs/farm_hoe.lua", "components/fumaroletool.lua", "components/inventoryitem.lua", "components/inventoryitemtemperature.lua", "components/heater.lua", "components/temperature.lua", "components/damagetypebonus.lua", "components/finiteuses.lua", "components/weapon.lua", "components/tool.lua", "components/equippable.lua", "components/inspectable.lua", "components/floater.lua", "components/trapfumaroleburning.lua", "components/temperatureoverrider.lua", "prefabs/trap_fumarole.lua", "componentutil.lua", "actions.lua", "bufferedaction.lua", "entityscript.lua", "widgets/itemtile.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwx78_common.lua", "stringutil.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 气孔工具族（fumaroleaxe、fumarolepickaxe、fumaroleshovel、fumarolehammer、fumarole_farm_hoe）

## 快速判断

- 职责与适用：五个手持工具注册（斧／镐／铲／锤／园艺锄）走同一 `MakeFumaroleTool` 工厂协议：物品温度（0~90 度）驱动四档外观与工作效率，≤0 度自毁进入破损形态、重新加热到 ≥60 度自动修复；对冻结目标伤害最高 ×3。都是可拾取、HANDS 槽装备物品；斧镐铲锤挂 `tool` 组件做工作动作，锄挂 `farmtiller` 走 TILL（无 `tool`）。[工厂](../../../raw/current/scripts/standardcomponents.lua#L2066-L2135)、[五注册](../../../raw/current/scripts/prefabs/axe.lua#L291-L294)
- 优先使用：做"靠温度工作、冷却失效、加热复用"的工具或武器 → 复刻下文[共享工厂协议](#共享工厂协议)的接线顺序并在 `fumarole_updatetemperaturerange` 回调里做分档表现；温度档位、灯光、破损 UI 的组件内部机制见 [[reference/components/fumaroletool]]。
- 重要副作用：每次成功使用（工作／攻击／锄地等缓冲动作）把工具降温 `heatonuse` 度并给持有者升温同等数值；攻击不磨损（`SetIgnoreCombatDurabilityLoss(true)`）、工作也不扣次数（`ClearAllConsumptions`），只有新破损才扣 1 次（读档恢复不扣）——4 次用尽整个实体移除；`heatrock` 标签使工具不被其他物品的换热扫描当作热源。[消耗语义](../../../raw/current/scripts/components/fumaroletool.lua#L121-L131)、[heatrock 排除](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L213-L214)
- 修改边界：组件与回调装配全在主模拟 master postinit，公共端只有 4 个优化标签（[pristine](../../../raw/current/scripts/standardcomponents.lua#L2057-L2064)）；皮肤分支代码就绪但本 build 无皮肤注册；quagmire 分支属基础工厂（仅 axe／shovel 有），本页未在 quagmire 模式运行验证。本页覆盖五注册与共享协议；`fumaroletool` 组件内部、锄地候选机制归各自页面。

## 注册与构造

### 共享工厂协议

`MakeFumaroleToolPristine(inst)`（[定义](../../../raw/current/scripts/standardcomponents.lua#L2057-L2064)）在公共端打 4 个标签：`heatrock`、`HASHEATER`、`inventoryitemtemperature`、`show_broken_ui`（后三个与组件自身打的标签同名，提前到 pristine 供客户端判断）。

`MakeFumaroleTool(inst, heatonuse, onbroken, onrepaired, onupdatetemperature)`（[定义](../../../raw/current/scripts/standardcomponents.lua#L2066-L2135)）在 master postinit 依次执行：

1. `inspectable.getstatus = GetFumaroleToolStatus`：按 fumaroletool 档位字段返回 `"LUKEWARM"`（档 2）／`"WARM"`（档 3）／`"HOT"`（档 4），档 1 返回 nil（[helper](../../../raw/current/scripts/standardcomponents.lua#L2031-L2036)）；
2. 物品温度接线：`inventoryitem:EnableTemperature(true)`（补挂 inventoryitemtemperature）、`SetTemperatureModifier("fumaroletool_mod", TUNING.FUMAROLETOOL_TEMP_MODIFIER)`（-30）、`SetMinTemperature(0)`／`SetMaxTemperature(90)`、`inherentinsulation = TUNING.INSULATION_MED_LARGE`（[接线](../../../raw/current/scripts/standardcomponents.lua#L2068-L2072)；[EnableTemperature 补组件](../../../raw/current/scripts/components/inventoryitem.lua#L191-L199)）；
3. 监听 `temperaturedelta` → `fumarole_UpdateTemperatureModifier`：按附近外热源功率（`externalheaterpower` 0..1）把 `fumaroletool_mod` 从 -30 线性缓动到 0（[helper](../../../raw/current/scripts/standardcomponents.lua#L2052-L2055)；[功率累加](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L268-L274)）；
4. `finiteuses:ClearAllConsumptions()` + `SetIgnoreCombatDurabilityLoss(true)`：工作与攻击都不再按动作扣次数（[调用](../../../raw/current/scripts/standardcomponents.lua#L2074-L2077)；[攻击跳过](../../../raw/current/scripts/components/weapon.lua#L111-L119)；[工作按 consumption 表扣、清空后不扣](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)）；
5. 挂 `heater`，`heatfn`／`equippedheatfn`／`carriedheatfn` 全部返回工具自身温度（[helper](../../../raw/current/scripts/standardcomponents.lua#L2027-L2029)、[挂载](../../../raw/current/scripts/standardcomponents.lua#L2122-L2125)）；玩家侧由 temperature 组件读装备／携带槽 heater 取暖（[消费端](../../../raw/current/scripts/components/temperature.lua#L370-L412)）；
6. 挂 `fumaroletool` 组件并注入 `SetOnBroken`／`SetOnRepaired`／`SetHeatOnUse(heatonuse)`／`SetOnUpdateTemperatureRange(onupdatetemperature)`（[挂载](../../../raw/current/scripts/standardcomponents.lua#L2127-L2131)）；
7. `damagetypebonus:AddBonusCallback(GetFumaroleDamageMultFn)`：对 `frozen` 标签或 `freezable` 已冻结目标按档查 `FUMAROLETOOL_FROZEN_DAMAGE_MULTS`，对 `coldness > 0` 的部分冻结目标按冻结比例 Lerp 查 `FUMAROLETOOL_FREEZING_DAMAGE_MULTS`（[helper](../../../raw/current/scripts/standardcomponents.lua#L2038-L2050)；[结算](../../../raw/current/scripts/components/weapon.lua#L94-L103)）。

前置条件：调用前实体须已有 `inspectable`、`inventoryitem`（`_onbroken` 还无条件读 `equippable` 与 `floater`），否则构造或破损回调报错（[取用点](../../../raw/current/scripts/standardcomponents.lua#L2067-L2068)、[破损取用](../../../raw/current/scripts/standardcomponents.lua#L2080-L2098)）。

### 五注册与差异

五注册分别位于各自基础工具文件的 `FUMAROLE_DATA`＋回调与注册行；官方中文名经简中 .po `msgctxt` 定向核验（[NAMES.FUMAROLE* 五键](../../../raw/current/scripts/languages/chinese_s.po#L343059-L343082)）；配方全部为 树枝 4＋螨腺 2＋硝石 3、`TECH.SCIENCE_TWO`（[recipes.lua:1077-1081](../../../raw/current/scripts/recipes.lua#L1077-L1081)）。伤害基数为源码推导值（`wilson_attack = 34`，[tuning.lua:358-365](../../../raw/current/scripts/tuning.lua#L358-L365)）。

| 注册名／官方名（.po 键） | 工厂链与分档回调 | HEAT_ON_USE（每次使用降温） | 修复时重建 | 证据 |
|---|---|---|---|---|
| `fumaroleaxe`／炽热斧（NAMES.FUMAROLEAXE） | `MakeAxe` → `fumarole_*` 回调 | -0.2 | tool `CHOP`(效率 1)＋weapon 27.2 | [注册](../../../raw/current/scripts/prefabs/axe.lua#L285-L294)、[修复](../../../raw/current/scripts/prefabs/axe.lua#L232-L243) |
| `fumarolepickaxe`／烈焰鹤嘴锄（NAMES.FUMAROLEPICKAXE） | `MakePickaxe` → 同构回调 | -0.6（-0.2×3） | tool `MINE`(1)＋weapon 27.2 | [注册](../../../raw/current/scripts/prefabs/pickaxe.lua#L238-L246)、[修复](../../../raw/current/scripts/prefabs/pickaxe.lua#L185-L196) |
| `fumaroleshovel`／呲啦铲（NAMES.FUMAROLESHOVEL） | `MakeShovel` → 同构回调 | -0.8（-0.2×4） | tool `DIG`（默认效率 1）＋weapon 17 | [注册](../../../raw/current/scripts/prefabs/shovel.lua#L243-L251)、[修复](../../../raw/current/scripts/prefabs/shovel.lua#L190-L201) |
| `fumarolehammer`／炎头锤（NAMES.FUMAROLEHAMMER） | `MakeHammer` → 同构回调 | -0.2×(4/3)≈-0.267 | tool `HAMMER`（默认效率 1）＋weapon 17 | [注册](../../../raw/current/scripts/prefabs/hammer.lua#L209-L216)、[修复](../../../raw/current/scripts/prefabs/hammer.lua#L156-L167) |
| `fumarole_farm_hoe`／烧灼园艺锄（NAMES.FUMAROLE_FARM_HOE） | `MakeFarmHoe` → 同构回调 | -0.8（-0.2×4） | `farmtiller`＋固有动作 TILL＋weapon 17（无 tool） | [注册](../../../raw/current/scripts/prefabs/farm_hoe.lua#L250-L258)、[修复](../../../raw/current/scripts/prefabs/farm_hoe.lua#L197-L208) |

分档表现（`fumarole_updatetemperaturerange` 每次档位变化时重设，档 1~4）：

- 动画 `idle_1`..`idle_4`、手持换装符号 `swap_fumaroleaxe_1`..`_4`（各文件同构，[axe.lua:254-279](../../../raw/current/scripts/prefabs/axe.lua#L254-L279)）；`AnimState` 亮度覆盖取 `FUMAROLETOOL_LIGHTOVERRIDES = { 0, 0, .15, .35 }`。
- 图标 `ChangeImageName(<基础名>_2/_3/_4)`，档 1 无后缀；基础名前三件与注册名同名，锄为 `fumarole_farm_hoe`（贴图 build 却是 `fumarolehoe`）（[axe.lua:260-261](../../../raw/current/scripts/prefabs/axe.lua#L254-L279)、[farm_hoe.lua:226-227](../../../raw/current/scripts/prefabs/farm_hoe.lua#L220-L244)）。
- 工作效率：斧 `CHOP`、镐 `MINE`、铲 `DIG`、锤 `HAMMER` 每次档位变化重设为 `FUMAROLE<工具>_EFFECTIVENESS = { 1, 1, 1.25, 1.5 }`，实际做功量在动作侧按 `tool:GetEffectiveness` 结算（[DoToolWork](../../../raw/current/scripts/actions.lua#L1703-L1717)）；园艺锄无 tool、无效率概念（[tuning.lua:9499](../../../raw/current/scripts/tuning.lua#L9496-L9500) 注释），锄地只降温不加速。
- 浮动换装帧：斧／镐／锤 `SetBankSwapOnFloat(true, -11, ...)`（负=left 帧 11）、铲 `(true, 7, ...)`（right 帧 7）、锄 `(true, -7, { bank = "fumarolehoe", ... })`；破损时各设 `(false, nil, FUMAROLE_SWAP_DATA_BROKEN)` 关闭换装（[floater 语义](../../../raw/current/scripts/components/floater.lua#L123-L143)）。
- 破损回调差异：斧镐铲锤移除 `equippable`＋`tool`＋`weapon`；锄移除 `equippable`＋`RemoveInherentAction(TILL)`＋`farmtiller`＋`weapon`（[axe.lua:247-252](../../../raw/current/scripts/prefabs/axe.lua#L245-L252)、[farm_hoe.lua:212-218](../../../raw/current/scripts/prefabs/farm_hoe.lua#L210-L218)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`fumaroletool` | 工厂挂载并注入 4 回调；档位字段随 `temperaturedelta` 重算并回调分档表现；≤0 破损、≥60 修复 | 五注册共用；主模拟 | [[reference/components/fumaroletool]]；[挂载](../../../raw/current/scripts/standardcomponents.lua#L2127-L2131) |
| 组件：`inventoryitem`＋`inventoryitemtemperature` | `EnableTemperature(true)`、mod 名 `fumaroletool_mod`、上下限 0/90、`inherentinsulation=180`；`externalheaterpower` 驱动 mod 缓动 | 五注册共用；扫描排除 `heatrock` 标签实体；fridge 分支独立于该排除 | [[reference/components/inventoryitemtemperature]]；[接线](../../../raw/current/scripts/standardcomponents.lua#L2068-L2072)、[排除与冰箱](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L213-L227) |
| 组件：`heater` | 三 heatfn 返回工具温度；玩家装备／携带即取暖 | 五注册共用；无 thermics 改动（默认 exothermic） | [[reference/components/heater]]；[挂载](../../../raw/current/scripts/standardcomponents.lua#L2122-L2125) |
| 组件：`damagetypebonus` | `AddBonusCallback` 按档增伤；`GetBonus` 把回调 nil 结果当 ×1 | 伤害结算经 `weapon:GetDamage`；broken 后无 weapon 即无加成 | [[reference/components/damagetypebonus]]；[回调](../../../raw/current/scripts/standardcomponents.lua#L2133-L2134) |
| 组件：`finiteuses` | `maxfiniteuses = FUMAROLETOOL_NUMUSES(4)`；`ClearAllConsumptions` 后只有破损 `Use(1)` 扣次 | 耗尽回调：斧镐铲锤 `inst.Remove`，锄 `OnFiniteUsesFinished`（推 toolbroke＋Remove） | [工厂设置](../../../raw/current/scripts/prefabs/axe.lua#L111-L117)、[锄回调](../../../raw/current/scripts/prefabs/farm_hoe.lua#L34-L40)、[[reference/components/finiteuses]] |
| 组件：`weapon` | 伤害按注册（27.2／17）；`OnAttack` 推 `weapononattack` → fumaroletool `OnUsed` 降温；攻击磨损被跳过 | broken 移除；斧镐铲锤锄都有 weapon | [事件](../../../raw/current/scripts/components/weapon.lua#L105-L121)、[消费](../../../raw/current/scripts/components/fumaroletool.lua#L33-L35)、[[reference/components/weapon]] |
| 组件：`tool`／`farmtiller`＋固有动作 | 斧镐铲锤 `SetAction` 分档效率；锄 `AddInherentAction(TILL)`＋farmtiller，`CanDoAction` 经固有动作通过 | tool 组件同时维护 `<动作>_tool` 标签供 CanDoAction | [[reference/prefabs/farm-hoe]]；[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45)、[CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1830) |
| 组件：`equippable` | 工厂挂 HANDS 槽＋皮肤分档换装回调；broken 移除、修复重建并显式设 `EQUIPSLOTS.HANDS`（与组件默认一致） | 五注册共用 | [重建](../../../raw/current/scripts/prefabs/axe.lua#L232-L243)、[默认槽](../../../raw/current/scripts/components/equippable.lua#L29)、[[reference/components/equippable]] |
| 组件：`floater` | 工厂初始浮动参数＋档位变化重设换装帧；broken 关换装 | 五注册共用 | [[reference/components/floater]]；[语义](../../../raw/current/scripts/components/floater.lua#L89-L92) |
| 组件：`inspectable` | `getstatus` 三档名；破损 `nameoverride = "BROKEN_FUMAROLETOOLITEM"` 改检视字符串（非显示名） | 五注册共用 | [getstatus](../../../raw/current/scripts/components/inspectable.lua#L27-L51)、[nameoverride 注释](../../../raw/current/scripts/components/inspectable.lua#L7) |
| UI：`itemtile`（show_broken_ui） | 库存格复用腐败表盘显示温度百分比；`broken` 标签时表盘归零 | 客户端读 pristine 标签与 net 化 `broken` 标签 | [表盘](../../../raw/current/scripts/widgets/itemtile.lua#L101-L112)、[破损置零](../../../raw/current/scripts/widgets/itemtile.lua#L943-L962) |
| 关联 Prefab：`heatrocklight` | fumaroletool 组件构造时生成灯光子实体，跟随 grand owner，入包维度容器／掩埋时隐藏 | 五注册共用；主模拟生成 | [[reference/components/fumaroletool]]；[生成](../../../raw/current/scripts/components/fumaroletool.lua#L47-L48) |
| 关联 Prefab：`trap_fumarole`（气孔陷阱） | 踩上活跃陷阱的实体被加 `trapfumaroleburning`，按 `TRAP_FUMAROLE_TEMPERATURE_RATE(3°/s)` 对物品 `AddTemperature`——是可靠的重热／修复途径 | 陷阱部署后 AOE 扫描（含地上物品） | [扫描加组件](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L45-L73)、[升温](../../../raw/current/scripts/components/trapfumaroleburning.lua#L133)、[[reference/components/trapfumaroleburning]] |
| API／数据：TUNING `FUMAROLETOOL_*`／`FUMAROLE<工具>_*` | 温度档、倍率、灯效、heatonuse 等全部取值见下文验证记录 | 五注册共用 | [tuning.lua:9467-9499](../../../raw/current/scripts/tuning.lua#L9468-L9500) |
| 资源：ANIM `fumaroleaxe/fumarolepickaxe/fumaroleshovel/fumarolehammer/fumarolehoe.zip` | 各文件 `fumarole_assets` 声明；动画 `idle_1..4`、`broken`、`broken_reheating_pre/loop/pst`、`repair` | 档位动画名在回调中拼接 | [axe 资源](../../../raw/current/scripts/prefabs/axe.lua#L19-L25) |
| 资源：INV_IMAGE `<基础名>_2/_3/_4` | 仅声明 2~4 档图；档 1 用无后缀基础名，其图集条目未在本文件声明（快照无资源目录，未验证） | 图标随档位 ChangeImageName | [axe 资源](../../../raw/current/scripts/prefabs/axe.lua#L19-L25) |

环境温度来源：物品温度的目标值用 `GetLocalTemperature`（实体覆写 → `fumarolelocaltemperature` 地热层 → 世界温度），地热生物群系会直接抬高环境温度（[温度查询](../../../raw/current/scripts/components/temperatureoverrider.lua#L8-L37)）；详见 [[reference/components/fumarolelocaltemperature]]。

## 典型接入

### 温度生命周期（制造 → 分档 → 冷却 → 破损 → 重热修复）

- 前提与选择：原版完整链路；理解各阈值后可改 tuning 或重定向加热源。
- 生效与撤销：全程主模拟；`broken` 与温度随存档恢复（fumaroletool `OnSave/OnLoad`）。
- 原版依据：本页共享协议与下列锚点；组件内部细节见 [[reference/components/fumaroletool]]。下列流程为改写的展示节选（非逐字源码，行号为核验依据）。

```text
制造    OnBuilt: SetTemperature(90 - 湿度惩罚[0..60])  -- fumaroletool.lua:155-158
分档    temperaturedelta → 未破损时 UpdateTempRange: temp>60→档4, >40→3, >20→2, 其余→1
        → 回调 onupdatetemperaturerange 重设动画/图标/效率/浮动
        → temp≥40 开灯: 强度 clamp(0.5*(temp-40)/60, 0, 0.5)  -- fumaroletool.lua:21-30
冷却    目标 = 环境温度(局部温度) + fumaroletool_mod(-30..0) + 湿度惩罚
        速率受 inherentinsulation 限幅: 30/(30+180)≈0.143°/s (源码推导)
        冰箱内: 直接按 1°/s(低冷 0.5) 压向 0  -- inventoryitemtemperature.lua:225-227
        漂在海面: 目标再向 -50 偏移; 落水瞬间按 1/3 量程截断(90→≤30)  -- floater.lua:146-163
破损    temp≤0 → SetBroken(true) → _onbroken:
        装备中→卸下 GiveItem 回背包+toolbroke; onbroken 移除 equippable/tool/weapon
        (锄另有 TILL 固有动作+farmtiller); 播 broken; nameoverride=...; 新破损 finiteuses:Use(1)
        读档恢复 broken=true 时 isloading=true，跳过 Use(1)
        破损态监听温度差>5 播 broken_reheating, ≤2 播回 broken  -- standardcomponents.lua:2010-2025
修复    temp≥60 → SetBroken(false) → onrepaired 重建组件, 未持有时播 repair+idle_4
        加热来源: 附近 HASHEATER 火源(10 距离内扫描), 或气孔陷阱 3°/s (tuning.lua:9511)
```

- 调整与验证：阈值在 `FUMAROLETOOL_TEMPS = { 0, 20, 40, 60 }`（[tuning.lua:9473](../../../raw/current/scripts/tuning.lua#L9468-L9479)）；1~59 度仍是破损态；修复判定用 `data.new >= 60`，档位显示用严格 `> 60`，所以恰好 60 度会修复但档位可能仍停在 3。

### 使用与消耗语义（攻击／工作只降温不磨损）

- 前提与选择：结算入口统一是缓冲动作成功后的 `OnUsedAsItem` 分发＋攻击事件，两个方向都只改温度。
- 生效与撤销：`heatonuse` 乘 `doer.aoediminishingreturns` 倍率后 `AddTemperature(heatonuse)`（负数降温）＋持有者 `temperature:DoDelta(-heatonuse)` 升温（[OnUsed](../../../raw/current/scripts/components/fumaroletool.lua#L121-L131)）。
- 原版依据（改写展示节选，非逐字源码；行号为核验依据）：

```text
工作   chop 等 SG 状态每挥击执行一次缓冲动作(SGwilson.lua:5097-5101)
       → BufferedAction:Do: 动作 fn(按 tool 效率做功) + invobject:OnUsedAsItem  -- bufferedaction.lua:22-37
       → EntityScript 分发: finiteuses(consumption 已清,不扣) + fumaroletool(CanDoAction 通过→OnUsed)
攻击   Weapon:OnAttack: 推 weapononattack(→OnUsed 降温) + 攻击磨损被 IgnoresCombatDurabilityLoss 跳过
锄地   TILL fn → farmtiller:Till, 无消耗项; 同一缓冲动作分发仍触发 OnUsed 降温
WX-78  扫描模块 harvest 直接调 fumaroletool:OnUsed(且两段 if 均命中→连调两次)  -- SGwx78_common.lua:557-572
```

- 调整与验证：想让工具“越用越坏”需在 `MakeFumaroleTool` 后恢复 `SetConsumption`，并 `finiteuses:SetIgnoreCombatDurabilityLoss(false)`＋设置 `weapon.attackwear`；破损重建武器后也要重设。默认协议下唯一扣次点是新破损（读档恢复不扣，4 次用尽 `onfinished` 移除实体）。

### 借鉴：用原版协议组装自定义热工具

- 前提与选择：服务端 prefab 文件内；复用 `MakeFumaroleToolPristine`＋`MakeFumaroleTool` 可一次接齐温度档、灯光、破损 UI 与增伤，无需自写档位逻辑。
- 生效与撤销：`common_postinit` 在 SetPristine 前调，`master_postinit` 在 mastersim 分支、组件齐备后调；移除温度机制需拆 `fumaroletool`／`heater` 并清理监听。
- 原版依据：五注册同构（[axe.lua:281-294](../../../raw/current/scripts/prefabs/axe.lua#L281-L294)）。

```lua
-- prefab 文件内；动画需自备 idle_1..4 / broken / repair
local function updatetemperaturerange(inst, owner, temprange)
    inst.AnimState:SetLightOverride(TUNING.FUMAROLETOOL_LIGHTOVERRIDES[temprange])
    inst.AnimState:PlayAnimation("idle_" .. temprange)
    local base = inst:GetSkinName() or "myhottool"
    inst.components.inventoryitem:ChangeImageName(base .. (temprange == 1 and "" or ("_" .. temprange)))
    if inst.components.tool ~= nil then
        inst.components.tool:SetAction(ACTIONS.MINE, TUNING.FUMAROLEPICKAXE_EFFECTIVENESS[temprange])
    end
end

local function onequip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_object", "swap_myhottool", "swap_myhottool")
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")
end

local function onunequip(inst, owner)
    owner.AnimState:Hide("ARM_carry")
    owner.AnimState:Show("ARM_normal")
end

local function onbroken(inst) -- 破损移除装备与功能组件，保留拾取/漂浮
    inst:RemoveComponent("equippable")
    inst:RemoveComponent("tool")
    inst:RemoveComponent("weapon")
end

-- 基础工厂已加 inspectable/inventoryitem/floater；首次生成与修复共用这套配置
local function setup_components(inst)
    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = EQUIPSLOTS.HANDS
    inst.components.equippable:SetOnEquip(onequip)
    inst.components.equippable:SetOnUnequip(onunequip)
    inst:AddComponent("tool")
    inst.components.tool:SetAction(ACTIONS.MINE, 1)
    inst:AddComponent("weapon")
    inst.components.weapon:SetDamage(TUNING.PICK_DAMAGE)
end

local function onrepaired(inst) -- 修复恢复与首次生成相同的组件配置
    setup_components(inst)
end

local function common_postinit(inst)
    MakeFumaroleToolPristine(inst)
end

local function master_postinit(inst)
    local finiteuses = inst:AddComponent("finiteuses")
    finiteuses:SetMaxUses(TUNING.FUMAROLETOOL_NUMUSES)
    finiteuses:SetUses(TUNING.FUMAROLETOOL_NUMUSES)
    finiteuses:SetOnFinished(inst.Remove)
    setup_components(inst)
    MakeFumaroleTool(inst, TUNING.FUMAROLEPICKAXE_HEAT_ON_USE, onbroken, onrepaired, updatetemperaturerange)
end
```

- 调整与验证：`heatonuse` 传负数表示使用降温；示例用气孔镐的 `FUMAROLEPICKAXE_HEAT_ON_USE`（与 `MINE` 动作匹配）。调用前实体须已有 `inspectable`、`inventoryitem`、`equippable`、`floater`（基础工厂已加 inspectable／inventoryitem／floater，`equippable` 由 `setup_components` 添加）；示例在 `master_postinit` 里先挂 `finiteuses` 并设 4 次上限/耗尽移除，`setup_components` 同时服务首次生成与修复回调。

## 常见陷阱与限制

- **heatonuse 是"对工具的温度增量"**：原版全部传负数（使用即降温、同时暖持有者）；误传正数会越用越热（[OnUsed](../../../raw/current/scripts/components/fumaroletool.lua#L121-L131)）。
- **不是无限修复**：`FUMAROLETOOL_NUMUSES = 4`，每次新破损扣 1 次（读档恢复不扣），第 4 次破损时 `finiteuses.current` 归零触发 `onfinished` → 整个实体 Remove（[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)）；锄的回调同样最终 Remove（[farm_hoe.lua:34-40](../../../raw/current/scripts/prefabs/farm_hoe.lua#L34-L40)）。
- **冰箱与海面会把工具"冻坏"**：fridge 分支不看 `heatrock` 标签，直接按 1°/s 压向 0（[inventoryitemtemperature.lua:225-227](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L225-L227)）；落水立即截到量程 1/3 以下（[floater.lua:146-163](../../../raw/current/scripts/components/floater.lua#L146-L163)）。`heatrock` 排除只作用于物品换热扫描——工具／暖石彼此不互热，但不妨碍火源加热与冰箱冷却。
- **修复门槛是 60 度**：`TEMPS[4]`；停在 1~59 度仍是破损态（[fumaroletool.lua:5-10](../../../raw/current/scripts/components/fumaroletool.lua#L5-L10)）。
- **nameoverride 只改检视字符串**：破损显示名仍是"炽热斧"等，检视才换成 BROKEN_FUMAROLETOOLITEM 描述（[inspectable.lua:7 注释](../../../raw/current/scripts/components/inspectable.lua#L7)、[取键](../../../raw/current/scripts/stringutil.lua#L375)）。
- **档位回调只在未破损时触发**：破损中 `onupdatetemperaturerange` 不被调用，工具组件也不存在，损坏的工具无法工作／装备／当武器（[fumaroletool.lua:12-13](../../../raw/current/scripts/components/fumaroletool.lua#L12-L19)）。
- **WX-78 扫描模块双倍降温**：该消费端两段 `if item.components.fumaroletool` 对同一工具连续调用两次 `OnUsed`，冷却翻倍（[SGwx78_common.lua:557-572](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L557-L572)）。
- **皮肤代码就绪但无皮肤**：所有回调含 `GetSkinBuild` 分支，但本 build `prefabskins.lua` 无 fumarole 条目；做皮肤时图标基础名注意锄的 `fumarole_farm_hoe` 与贴图 build `fumarolehoe` 不同名。
- **档 1 基础图标未在本文件声明**：`fumarole_assets` 只声明 `_2/_3/_4` 图，基础名图未验证（快照无资源目录），引用档 1 图标需自行确认图集。

## 验证记录

- 源码审阅：2026-11-06，steam-buildid 26335394（`raw/current/BUILD.txt`）。完整读取 `components/fumaroletool.lua`（172 行）与 `components/damagetypebonus.lua`；核验 `standardcomponents.lua` 两个工厂与 5 个本地 helper（L2010-2135）、五个工具文件的 FUMAROLE_DATA／四个 fumarole_* 回调／注册行及各自基础工厂全文、`components/inventoryitemtemperature.lua` 全文（换热扫描、fridge、浮动、速率限幅）、`heater`/`weapon`/`finiteuses`/`tool`/`equippable`/`inspectable`/`floater`/`trapfumaroleburning` 相关方法、`inventoryitem.lua` 温度接线、`temperature.lua` 装备／携带取暖、`actions.lua` DoToolWork、`bufferedaction.lua`＋`entityscript.lua` 的 OnUsedAsItem／CanDoAction 分发、SGwilson chop 状态与 SGwx78_common 扫描消费、itemtile show_broken_ui 三处消费、trap_fumarole 扫描与 trapfumaroleburning 升温链（componentutil `DoDeltaTemperatureToEntity`）、temperatureoverrider/fumarolelocaltemperature 环境温度层、tuning 全部相关键（含 `seg_time=30`、`wilson_attack=34` 推导）、recipes.lua:1077-1081、`chinese_s.po` 五个 `NAMES.*` 键（L343059-343082）、`BROKEN_FUMAROLETOOLITEM`（L9964-9967）与五组 DESCRIBE 状态键（L12929-13023）。未覆盖：quagmire 模式行为、皮肤实装、资源图集、锄地 SG 全流程（归 [[reference/prefabs/farm-hoe]]）。
- 选档说明（complex）：五注册共享同一工厂协议，温度档／破损／修复／消耗链与五份差异表需要同页对照才能完整表达；拆分会把工厂协议与变体差异分离，主题不可按对象拆分。
- 游戏运行：未运行。冷却速率、修复耗时、效率与增伤数值为源码推导值，未实测。
- 结构检查：validate_page.py 与 strict 均通过；本次修改的 Lua 示例经 parser_probe 语法通过。

相关：[[reference/components/fumaroletool|fumaroletool 组件]]、[[reference/components/inventoryitemtemperature]]、[[reference/components/heater]]、[[reference/components/damagetypebonus]]、[[reference/components/trapfumaroleburning]]、[[reference/components/fumarolelocaltemperature]]、[[reference/prefabs/axe|斧头族]]、[[reference/prefabs/pickaxe|鹤嘴锄]]、[[reference/prefabs/shovel|铲子]]、[[reference/prefabs/hammer|锤子]]、[[reference/prefabs/farm-hoe|园艺锄]]

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
