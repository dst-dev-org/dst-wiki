---
title: "moisture"
created: "2026-10-16"
updated: "2026-10-28"
type: "component"
summary: "服务端修改角色湿度、持续速率、防雨与强制干燥，并保留同步、存档和撤销边界。"
aliases: ["角色湿度", "加湿", "干燥", "持续湿度", "防雨", "强制干燥", "moisture"]
tags: ["component", "mechanism", "network-sync", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["components/moisture.lua", "components/moisture_replica.lua", "components/inventory.lua", "prefabs/player_classified.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "util/sourcemodifierlist.lua", "modutil.lua", "mainfunctions.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# moisture

## 快速判断

- 职责与适用：玩家服务端组件维护 `0..maxmoisture` 的湿度、湿润状态和每秒变化速率；默认上限 100、初值 0，原版玩家在 `player_common` 添加该组件。[components/moisture.lua:26](../../../raw/current/scripts/components/moisture.lua#L26 [prefabs/player_common.lua:2737](../../../raw/current/scripts/prefabs/player_common.lua#L2737
- 优先使用：一次性改值用 `DoDelta`；持续效果用带来源的 `AddRateBonus`，在效果撤销时用同一来源／key 移除。不要直接写 `moisture` 字段。[components/moisture.lua:317](../../../raw/current/scripts/components/moisture.lua#L317
- 重要副作用：`ForceDry(true, source)` 立即归零、停止更新；多个来源可并存，最后一个撤销后才恢复更新。[components/moisture.lua:74](../../../raw/current/scripts/components/moisture.lua#L74
- 修改边界：组件不在客户端副本上运行；防雨针对雨水速率，波浪等事件应按其自身入口的 `GetWaterproofness()` 规则判断。[[reference/components/waterproofer#常见陷阱与限制]]

## 关键接口

| 接口 | 约定 | 作用与边界 |
|---|---|---|
| `GetMoisture()`／`GetMaxMoisture()`／`GetMoisturePercent()` | 数值／上限／比例 | 读取服务端当前值；`GetRate()` 是每秒试图改变的湿度，不等于已受上下限钳制的实际变化。[components/moisture.lua:163](../../../raw/current/scripts/components/moisture.lua#L163 [components/moisture.lua:309](../../../raw/current/scripts/components/moisture.lua#L309 |
| `DoDelta(num, no_announce?)` | number, boolean? | 加减后钳制到 `0..maxmoisture`、更新湿润状态并发出 `moisturedelta`；强制干燥时无效。负数用于立即干燥，`true` 抑制角色湿度段位台词。[components/moisture.lua:132](../../../raw/current/scripts/components/moisture.lua#L132 |
| `SetMoistureLevel(num)`／`SetPercent(per)` | 绝对值／比例 | 同样受强制干燥门禁；前者会发事件但源码事件的 `old` 与 `new` 都是设定后的值，不能用它推导实际差值。[components/moisture.lua:149](../../../raw/current/scripts/components/moisture.lua#L149 [components/moisture.lua:188](../../../raw/current/scripts/components/moisture.lua#L188 |
| `AddRateBonus(src, bonus, key?)`／`RemoveRateBonus(src, key?)` | 稳定来源、湿度点／秒、可选键 | 常规路径加算，负值干燥、正值增湿；同 source／key 重设会覆盖，省 key 移除会清掉该来源全部键。EntityScript 来源移除时自动清理；table／字符串来源在功能关闭时显式移除。[components/moisture.lua:317](../../../raw/current/scripts/components/moisture.lua#L317 [util/sourcemodifierlist.lua:61](../../../raw/current/scripts/util/sourcemodifierlist.lua#L61 |
| `ForceDry(force, source?)` | boolean、来源实体? | 适用于暂时必须保持干燥的状态，不是普通防雨。非自身来源会监听其 `onremove` 自动撤销；自身来源仍须显式 `ForceDry(false, inst)`。[components/moisture.lua:74](../../../raw/current/scripts/components/moisture.lua#L74 |
| `GetWaterproofness()`／`waterproofnessmodifiers` | `0..1` 汇总／加算来源表 | 事件 getter 不含庇护；雨水另计庇护与 rainimmunity。角色来源防水用 `waterproofnessmodifiers:SetModifier(src, value, key)`／`RemoveModifier(src, key)`，装备贡献见 [[reference/components/waterproofer]]。旧 `inherentWaterproofness` 字段已标 deprecated，不作为新方案首选。[components/moisture.lua:44](../../../raw/current/scripts/components/moisture.lua#L44 [components/moisture.lua:212](../../../raw/current/scripts/components/moisture.lua#L212 [components/moisture.lua:239](../../../raw/current/scripts/components/moisture.lua#L239 |

### 持续速率的组成

常规更新以 `环境增湿 + 装备增湿 - 自然干燥 + 外部 bonus` 得到 `rate`（湿度点／秒），乘 `dt` 调 `DoDelta`。仅环境增湿率大于 0 时抑制自然干燥；完全防雨时即使下雨仍可自然干燥。持浮具／浸浴走独立增湿分支，不能把防雨当全部增湿免疫。[components/moisture.lua:253](../../../raw/current/scripts/components/moisture.lua#L253 [components/moisture.lua:287](../../../raw/current/scripts/components/moisture.lua#L287

存在睡袋干燥率时，直接用它替代上述常规公式，外部 bonus 也不参与；随后仍按条件扣除吸湿器贡献，再应用湿度上下限。因此 `AddRateBonus(-0.10)` 只增加每秒 0.10 点的干燥贡献，不保证净下降速度。[components/moisture.lua:304](../../../raw/current/scripts/components/moisture.lua#L304 [components/moisture.lua:338](../../../raw/current/scripts/components/moisture.lua#L338

## 典型接入

### 最小完整接入：角色常驻微量干燥

放入模组 `modmain.lua`。在服务端角色 PostInit 登记每秒 `-0.10` 湿度贡献；原版工厂已添加 moisture，读档重建角色时也重新登记，不依赖 modifier 自身存档。功能禁用时调用同一 helper 的 `false` 分支。独立示例显式读 `GLOBAL.TheWorld`；项目已有全局代理可沿用裸 `TheWorld`，见 [[cookbook/mod-starter#全局名称与加载环境]]。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586 [mainfunctions.lua:387](../../../raw/current/scripts/mainfunctions.lua#L387

```lua
local RATE_SOURCE = {}
local RATE_KEY = "example_mod_passive_drying"
local RATE_BONUS = -0.10

local function SetPassiveDrying(inst, enabled)
    local moisture = inst.components.moisture
    if moisture == nil then
        return
    end

    if enabled then
        moisture:AddRateBonus(RATE_SOURCE, RATE_BONUS, RATE_KEY)
    else
        moisture:RemoveRateBonus(RATE_SOURCE, RATE_KEY)
    end
end

AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    SetPassiveDrying(inst, true)
end)
```

`RATE_SOURCE` 与 key 是可替换的模组内稳定标识；持续效果不要每次创建新 table 登记。由物品／Buff 实体提供时可改用该实体作为 `src`：脱离／失效时显式移除，实体被删除时列表自动清理，无需再为相同删除事件加重复监听。一次性干燥在同样的服务端前提下用 `DoDelta(-15, true)`；不要手工改 `player_classified` 网络变量。

### 防雨还是强制干燥

- 要“挡住雨水增湿”：给角色自身登记防水 modifier，或装备带 [[reference/components/waterproofer]] 的物品；雨水路径在总防水值至少 1 时返回 0，rainimmunity 也令该路径返回 0，但不承诺拦截浸浴或所有事件加湿。[components/moisture.lua:212](../../../raw/current/scripts/components/moisture.lua#L212
- 要“某状态期间保持 0”：`moisture:ForceDry(true, source)`；结束时对同一来源 `ForceDry(false, source)`。非自身来源必须是可监听 `onremove` 的实体，不沿用速率 bonus 的任意 table 来源；省略来源默认为角色自身，结束时仍须显式撤销。原版死亡／重置流程可作参照。[prefabs/player_common_extensions.lua:644](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L644 [prefabs/player_common_extensions.lua:835](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L835

## 常见陷阱与限制

- **客户端也改一遍以同步 UI**：不要。服务端 `maxmoisture`、`moisture` 与 `ratescale` 的属性回调写入 `player_classified` 网络值；客户端由 dirty 事件转发 `moisturedelta`，副本只提供 `IsWet()`。[components/moisture.lua:4](../../../raw/current/scripts/components/moisture.lua#L4 [prefabs/player_classified.lua:514](../../../raw/current/scripts/prefabs/player_classified.lua#L514 [components/moisture_replica.lua:1](../../../raw/current/scripts/components/moisture_replica.lua#L1
- **存档会保留所有持续来源或强制干燥**：仅 `moisture` 数值写入组件存档；`externalbonuses`、`forcedrysources` 和当前 `rate` 未在该保存函数写入。可重建的模组状态应在加载／角色初始化后重登记，临时来源要有撤销路径。[components/moisture.lua:378](../../../raw/current/scripts/components/moisture.lua#L378
- **读档恢复值会立即重建湿润事件**：`OnLoad` 只直接恢复 moisture，不调用 `DoDelta`／`SetMoistureLevel`，也不立即重算 wet／发事件；不能把该恢复动作当作一次完整的改值通知。[components/moisture.lua:385](../../../raw/current/scripts/components/moisture.lua#L385
- **`ForceDry` 只清一次就会恢复**：它是按来源集合计数；多个来源中任一来源撤销不会解除其余来源的锁定。[components/moisture.lua:77](../../../raw/current/scripts/components/moisture.lua#L77
- **旧 `SetWaterproofInventory` 是常规方案**：源码标为 deprecated，改用 `ForceDry`；普通防雨则选防水汇总，不要用强制干燥掩盖其他湿度机制。[components/moisture.lua:183](../../../raw/current/scripts/components/moisture.lua#L183

### 待核验问题

- 本页未逐一核验全部湿度消费者、事件加湿入口、吸湿器内部算法或所有角色状态下的更新时序；涉及这些差异时再查实际调用点。免湿来源／标签与原版食物效果见 [[reference/components/moistureimmunity]]，不只用 ForceDry 概括其行为。
- 未进行游戏运行、多人联机、存档重载或与其他模组叠加测试。

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（player_common moisture 构造、mainfunctions PostInit 段）；无结论修订，仅锚点迁移——R-08 核验：本版降雨位置门控（`CanPointHaveRain`）加在 entityscript 通用湿润查询 `GetIsWet`／`GetWetMultiplier` 的无组件回退分支（entityscript.lua:765/:785）及物品初始赋湿入口，不在本组件的雨水速率路径；moisture.lua、moisture_replica.lua 本版未变，速率／防水／强制干燥结论逐访问条件保留。依据：全局枢纽 diff 审阅＋变更清单命中 player_classified.lua（仅新增网络字段段）、player_common.lua、player_common_extensions.lua、mainfunctions.lua。
- 源码审阅：2026-10-16，Steam buildid 25643519；覆盖修改、速率分支、防水汇总、强制干燥、来源替换／清理、玩家网络桥接、保存恢复和最小接入的工厂／PostInit 依据。
- 结构检查：元数据、来源文件、大小、链接及唯一索引登记通过；示例经 Full Moon 3.0.0 按 Lua 5.1 解析与原字节往返检查。上述检查不代表运行通过。
- 未运行游戏；示例体感、多人联机、读档与跨模组来源协作仍需运行验证。

相关：[[reference/components/waterproofer]]；[[reference/components/moistureimmunity]]；[[reference/components/equippable]]；[[reference/components/inventory]]。
