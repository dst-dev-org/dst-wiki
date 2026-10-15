---
title: "moistureimmunity"
created: "2026-10-16"
updated: "2026-10-28"
type: component
summary: "按来源授予目标免湿，配合强制干燥、湿润查询与原版限时食物 Buff，并成对撤销。"
aliases: ["免湿", "免疫潮湿", "防水Buff", "食物免湿", "保持干燥", "moistureimmunity"]
tags: [component, food, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/moistureimmunity.lua", "components/moisture.lua", "entityscript.lua", "prefabs/foodbuffs.lua", "preparedfoods_warly.lua", "prefablist.lua", "prefabs/veggies.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moistureimmunity

## 快速判断

- 职责与适用：挂在**获得免湿的目标**上，保存来源实体集合；首个来源添加 `moistureimmunity` 标签，并对目标已有的 moisture 执行 `ForceDry(true, src)`。[moistureimmunity.lua:21](../../../raw/current/scripts/components/moistureimmunity.lua#L21
- 优先使用：限时食物效果可复用原版 `buff_moistureimmunity`；自定义装备／状态由来源实体成对调用 `AddSource(src)`／`RemoveSource(src)`。只需防雨贡献时选 [[reference/components/waterproofer]] 或 [[reference/components/moisture#防雨还是强制干燥]]。
- 重要副作用：已有 moisture 会立即清零并暂停更新；多个来源同时存在，撤销一个不解除其余来源，最后一个离开会**自动移除本组件**。[moistureimmunity.lua:5](../../../raw/current/scripts/components/moistureimmunity.lua#L5 [moisture.lua:74](../../../raw/current/scripts/components/moisture.lua#L74
- 修改边界：授予／撤销在服务端执行；仅 AddComponent 不添加标签或效果。`wet` 永久湿润标签在通用湿润查询中优先，不能承诺对此类对象生效。[entityscript.lua:770](../../../raw/current/scripts/entityscript.lua#L770 [entityscript.lua:794](../../../raw/current/scripts/entityscript.lua#L794

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `target:AddComponent("moistureimmunity")` | 仅在缺少时添加 | 构造时只建立空集合与删除回调；实际效果由 AddSource 开启。[moistureimmunity.lua:1](../../../raw/current/scripts/components/moistureimmunity.lua#L1 |
| `AddSource(src)` | 稳定的来源实体，无返回值 | 按实体去重，不按调用次数计数；非目标自身的来源监听 `onremove`。非自身来源须能作为 EntityScript 事件源，不用任意字符串／table。[moistureimmunity.lua:21](../../../raw/current/scripts/components/moistureimmunity.lua#L21 |
| `RemoveSource(src)` | 与添加时相同的实体 | 仅撤销该来源的 ForceDry 与监听；最后一个来源删除组件，后续重新授予前重新检查组件是否存在。[moistureimmunity.lua:37](../../../raw/current/scripts/components/moistureimmunity.lua#L37 |
| `OnRemoveFromEntity()` | 由宿主 RemoveComponent 调用 | 移除标签，撤销所有本组件来源及监听；只关闭某效果用 RemoveSource，不删除其他来源共享的组件。[moistureimmunity.lua:13](../../../raw/current/scripts/components/moistureimmunity.lua#L13 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 |
| `target:GetIsWet()`／`GetWetMultiplier()` | boolean／湿润倍率 | 无 `wet` 标签时，免湿标签使其分别返回 false／0；前者可在客户端查询，后者源码注明不可客户端使用。26148036 起两查询对无效实体早退 false／0（不再报错），其无组件回退分支的"下雨"判定另需该点 `CanPointHaveRain`——免湿分支先于该判定，逐访问条件（无效实体／无雨区／免湿标签）下旧结论均保留。[entityscript.lua:770](../../../raw/current/scripts/entityscript.lua#L770 [早退与门控](../../../raw/current/scripts/entityscript.lua#L771-L776) |

### 与直接 ForceDry 的区别

`ForceDry` 管理 moisture 自身的归零与更新门禁，细节由 [[reference/components/moisture#关键接口]] 维护。本组件额外管理免湿标签、湿润查询与来源生命周期；没有 moisture 的目标也可获得标签效果，但不会因此新增湿度数值组件。它不会禁用 inventoryitem 的自身湿度更新，不替代物品防潮的 [[reference/components/waterproofer]]。

若先 AddSource、后添加 moisture，本组件不会自动把旧来源补登记到新组件；同一来源再次 AddSource 也会因去重跳过。需要这类动态组装时，先准备目标 moisture，再授予效果。[moistureimmunity.lua:22](../../../raw/current/scripts/components/moistureimmunity.lua#L22

## 典型接入

### 吃原胡萝卜获得原版限时免湿效果

追加到目标模组 `modmain.lua`，各端使用同一模组版本；不新增美术、效果工厂或 PrefabFiles。原版 `foodbuffs` 已在 prefablist 中加载，注册 `buff_moistureimmunity`；原蛙鱼料理也通过实体 AddDebuff 施加它。[prefablist.lua:458](../../../raw/current/scripts/prefablist.lua#L458 [foodbuffs.lua:213](../../../raw/current/scripts/prefabs/foodbuffs.lua#L213 [preparedfoods_warly.lua:103](../../../raw/current/scripts/preparedfoods_warly.lua#L103

```lua
local BUFF_KEY = "my_mod_carrot_dry"

AddPrefabPostInit("carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local edible = inst.components.edible
    if edible == nil then return end

    local old_oneaten = edible.oneaten
    edible:SetOnEatenFn(function(food, eater)
        if old_oneaten ~= nil then old_oneaten(food, eater) end
        if eater ~= nil and eater:IsValid() and eater:HasTag("player") then
            eater:AddDebuff(BUFF_KEY, "buff_moistureimmunity")
        end
    end)
end)
```

替换 `my_mod_` 为模组前缀；只扩展 carrot，不改 carrot_cooked、三维收益或消耗。独立入口显式读 `GLOBAL.TheWorld`，已有代理项目沿用项目约定。原食用者回调与注册依据见 [[reference/components/edible#关键接口]]、[veggies.lua:1096](../../../raw/current/scripts/prefabs/veggies.lua#L1096 和 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

效果时长沿用原版工厂的 `TUNING.BUFF_MOISTUREIMMUNITY_DURATION`，本例不自定秒数；重复吃同键只刷新原效果计时，不新增来源。到期／目标死亡时工厂停止效果，脱附撤销该效果实体来源。主动关闭用 `eater:RemoveDebuff(BUFF_KEY)`；原版其他键的免湿可继续生效，撤销不会恢复施加前的湿度。[foodbuffs.lua:134](../../../raw/current/scripts/prefabs/foodbuffs.lua#L134 [foodbuffs.lua:150](../../../raw/current/scripts/prefabs/foodbuffs.lua#L150 [foodbuffs.lua:163](../../../raw/current/scripts/prefabs/foodbuffs.lua#L163 [foodbuffs.lua:222](../../../raw/current/scripts/prefabs/foodbuffs.lua#L222

### 自定义装备或状态授予

原版效果的附着回调在**目标**上按需加组件，再以**效果实体**为来源；脱附时先检查组件仍在，再 RemoveSource。[foodbuffs.lua:46](../../../raw/current/scripts/prefabs/foodbuffs.lua#L46 可在已有装备／Buff 的生效与撤销入口沿用此顺序，使用装备或效果实体作独立来源，不把组件挂到授予者就期待持有者受益。

物品仍存在但卸装／移出范围时必须显式撤销，不能只等删除事件。来源实体删除时，本组件与 moisture 各自处理 onremove，前者的删除回调不重复解除 ForceDry；避免另外串联内部回调。[moistureimmunity.lua:37](../../../raw/current/scripts/components/moistureimmunity.lua#L37 生效范围与回调选择见 [[reference/components/equippable#关键接口]]，完整效果工厂见 [[reference/components/debuff#三十秒加速效果并成对撤销]]。

## 常见陷阱与限制

- **同一来源被两个功能共用**：集合没有计数和 key，一个 RemoveSource 就撤销该来源；需要独立撤销的效果应使用不同来源实体。[moistureimmunity.lua:22](../../../raw/current/scripts/components/moistureimmunity.lua#L22
- **删组件只撤销自己的效果**：RemoveComponent 会清理全部本组件来源；按原来源 RemoveSource，并容忍最后一个来源移除后组件为 nil。[moistureimmunity.lua:13](../../../raw/current/scripts/components/moistureimmunity.lua#L13
- **给 `wet` 对象加 Buff 就会变干**：原版 moisture_attach 对永久湿润对象直接返回；Buff 实体存在或 AddDebuff 返回 true 都不证明已取得免湿效果。通用湿润查询也优先检查 `wet`。[foodbuffs.lua:46](../../../raw/current/scripts/prefabs/foodbuffs.lua#L46 [[reference/components/debuffable#关键接口]]
- **免湿来源会自动存档**：本组件没有 OnSave／OnLoad，moisture 只保存湿度数值；来源应由其装备／状态／效果恢复流程重新授予。原版 Buff 的目标集合保存恢复见 [[reference/components/debuffable#保存恢复与移除]]，不能推广为任意动态来源都能恢复。[moisture.lua:378](../../../raw/current/scripts/components/moisture.lua#L378
- **只手工加标签即可完整免湿**：标签影响通用湿润查询，但不会执行 ForceDry 清零／停更新，也不具备来源撤销；正常效果走组件接口。本页未核验所有伤害、酸雨或事件消费者，不把免湿概括为全环境伤害免疫。

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（entityscript GetIsWet 定义行、prefablist foodbuffs 登记）；无结论级修订，按 R-08 补充通用湿润查询行为：`GetWetMultiplier`／`GetIsWet` 26148036 起对无效实体早退 0／false，无组件回退分支的下雨判定追加 `CanPointHaveRain` 位置门控（entityscript.lua:749-750/:765/:773-774/:785）；`wet` 与 `moistureimmunity` 标签分支均先于该回退分支，本页免湿结论逐访问条件（室内／无雨区／无效实体）保留。依据：全局枢纽 diff 审阅＋变更清单命中 entityscript.lua、prefablist.lua（仅新增条目导致行移），moistureimmunity.lua、moisture.lua、foodbuffs.lua 未变。
- 源码审阅：2026-10-16，Steam buildid 25643519；核验来源去重／移除、标签与湿度门禁、宿主移除回调、原版食物 Buff 注册／附着／刷新／撤销，以及 carrot 接入位置；已有 edible／debuffable 证据按所链接章节复用。
- 结构检查：元数据、来源路径、大小、链接及唯一叶级登记通过；发布 Lua 片段经 Full Moon 3.0.0／Lua 5.1 解析与原字节往返检查。所选接入与陷阱章节可完整读取，检查不代表游戏运行通过。
- 游戏运行：未运行；食用、叠加来源、死亡／脱附、存档重载、客户端显示及其他模组协作待实测。未审阅全部湿度消费者或把本组件当独立存档系统。

相关：[[reference/components/moisture]]；[[reference/components/waterproofer]]；[[reference/components/debuffable]]。
