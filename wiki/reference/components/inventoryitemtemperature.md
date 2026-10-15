---
title: "inventoryitemtemperature"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "启用物品专用温度、目标偏移与绝热，明确容器冷却、湿度惩罚、事件编码和人物体温的区别。"
aliases: ["物品专用温度", "物品温度偏移", "物品温度事件", "吸湿器温度", "inventoryitemtemperature"]
tags: [component, item, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/inventoryitemtemperature.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "components/stackable.lua", "prefabs/inventoryitem_classified.lua", "prefabs/desiccant.lua", "tuning.lua", "entityscript.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inventoryitemtemperature

## 快速判断

- 职责与适用：`inventoryitem` 的服务端可选温度扩展，默认不启用，区别于人物／暖石的 [[reference/components/temperature]]。[宿主装配](#关键接口)
- 优先使用：宿主 SetTemperature 一次赋温、SetTemperatureModifier 调整目标；常见暖石需求先查 temperature／[[reference/components/heater]]，不叠两套自身温度。
- 重要副作用：`temperaturedelta` 有端与 payload 差异；原版吸湿器据此切换湿度控制，修改温度会改变实际机制。见[网络读取](#更新保存与网络读取)和[案例](#典型接入)。
- 修改边界：本组件不提供人物热源输出、温度锁定或伤害；放入口袋不等于供热。供热与携带范围见 [[reference/components/heater#携带范围与地面热源的分工]]。

## 关键接口

| 接口／字段 | 参数／约定 | 条件、副作用与证据 |
|---|---|---|
| `inventoryitem:EnableTemperature(enable)` | false 移除；其余值在缺失时启用并绑定 replica | 初值35、min=-20、max=90、冬夏绝热0；构造加同名标签。直接 AddComponent 未绑定 replica 不能安全写温度。[inventoryitem.lua:191](../../../raw/current/scripts/components/inventoryitem.lua#L191)、[inventoryitemtemperature.lua:37](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L37)、[:61](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L61)、[tuning.lua:2300](../../../raw/current/scripts/tuning.lua#L2300 |
| 宿主 `GetTemperature()`／`SetTemperature(temp)` | 数值；未启用 getter 返回 nil | setter 钳制 min/max 后发物品事件 `{last,new,mintemp,maxtemp}`，即使值不变也发；不锁定自然更新。[inventoryitem.lua:202](../../../raw/current/scripts/components/inventoryitem.lua#L202)、[inventoryitemtemperature.lua:132](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L132 |
| 宿主 `AddTemperature(delta)` | 相对温度变化，转交扩展 DoDelta | 正值受夏季、非正值受冬季绝热缩减后赋温；无 skipinsulation 参数。[inventoryitem.lua:260](../../../raw/current/scripts/components/inventoryitem.lua#L260)、[inventoryitemtemperature.lua:120](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L120 |
| 宿主 `SetTemperatureModifier(name, value)`／`RemoveTemperatureModifier(name)` | 独立键与目标温度偏移；0／nil删除 | 普通 keyed 表，无来源实体自动清理；撤销见[案例](#典型接入)。[inventoryitemtemperature.lua:168](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L168 |
| 扩展 `inherentinsulation`／`inherentsummerinsulation` | 冬／夏绝热，参与 `SEG_TIME/(SEG_TIME+绝热)` | 各封底0，分别减缓降温／升温；冰箱独立限速。[inventoryitemtemperature.lua:201](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L201)、[:297](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L297) |
| 宿主 `SetMinTemperature`／`SetMaxTemperature`／`SetSaveMinAndMaxTemperature` | 边界数值／保存开关 | 改边界只发事件，不重钳当前值；须保证 min≤max。存档边界见下节。[inventoryitem.lua:224](../../../raw/current/scripts/components/inventoryitem.lua#L224)、[inventoryitemtemperature.lua:139](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L139 |

### 自然换热与容器分支

`GetTargetDeltaTemperature()` 由**当地温度 + modifier + 湿度惩罚 - 当前温度**起算，再加漂浮修正和附近热源。绝热控制速率，默认每秒变化绝对值上限1；差值绝对值>0.25才更新，目标为整数时仍补完小差值。[inventoryitemtemperature.lua:217](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L217)、[:297](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L297)

| 条件 | 本扩展的实际处理 |
|---|---|
| 直接 owner 有 fridge 且无 nocool | 提前返回以0为目标的差值与负冷却率，lowcool 减半；跳过附近热源与通常绝热限速。 |
| 直接 owner 有 pocketdimension_container | 用世界温度并跳过附近热源；普通容器仍查当地温度。只查直接 owner，不递归嵌套链。 |
| 自身 floater 正漂浮 | 从目标差再扣 `当前温度 + OCEAN_AMBIENT_TEMPERATURE_PENALTY`，不是赋固定海水温度。 |
| 附近热源 | 10单位内查 HASHEATER，排除 heatrock／INLIMBO；调用 `heater:GetHeat(self.inst)` 并处理方向／衰减／半径，无人物装备／携带热源枚举。 |

宿主 `SetTemperatureMaxMoisturePenalty(0)` 去掉湿度比例造成的目标降温（默认最大30）；`SetNoWetTemperaturePenalty(true)` 只去掉已湿时附近热源因子的惩罚，**不关闭前一项**。desiccant 同时设置两项。[inventoryitemtemperature.lua:149](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L149)、[:207](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L207)、[:255](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L255)、[desiccant.lua:147](../../../raw/current/scripts/prefabs/desiccant.lua#L147

### 更新、保存与网络读取

- OnEntityWake 启动周期任务：变化时1秒、稳定时3秒，首次随机延迟；睡眠记录剩余任务时间并取消，醒来按当前环境补算。运行中给已醒实体启用时不会自动补 wake；本例复用构造期已启用的对象。[inventoryitemtemperature.lua:22](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L22)、[:85](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L85)、[entityscript.lua:632](../../../raw/current/scripts/entityscript.lua#L632)、[mainfunctions.lua:751](../../../raw/current/scripts/mainfunctions.lua#L751
- OnSave 存 temperature，save_min_and_max_temp 为真才存边界；OnLoad 先恢复边界再 SetTemperature 并发事件。modifier、绝热、湿度策略与任务不保存，须初始化重建。移组件写温度0、删标签、取消任务；移除 inventoryitem 连带移除扩展。[inventoryitemtemperature.lua:327](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L327)、[:75](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L75)、[inventoryitem.lua:106](../../../raw/current/scripts/components/inventoryitem.lua#L106)、[entityscript.lua:2005](../../../raw/current/scripts/entityscript.lua#L2005
- 服务端真实温度经 replica 序列化到 classified net_byte：-11至6附近步长0.25，外侧步长1，属于有损编码。客户端 DeserializeTemperature 解码后只发 `{new=温度}` 的 `temperaturedelta`，不含服务端的 last／边界。[inventoryitem_classified.lua:131](../../../raw/current/scripts/prefabs/inventoryitem_classified.lua#L131)、[:150](../../../raw/current/scripts/prefabs/inventoryitem_classified.lua#L150)
- **该 build 客户端 `replica.inventoryitem:GetTemperature()` 无本地扩展时直接返回 net_byte 原码**，不能与服务端温度阈值比较。无 classified 时返回0，不证明启用或真实0度。远端显示须使用解码值并处理初始同步、owner／容器打开者限定的可见性。[inventoryitem_replica.lua:498](../../../raw/current/scripts/components/inventoryitem_replica.lua#L498)、[:516](../../../raw/current/scripts/components/inventoryitem_replica.lua#L516)、[:171](../../../raw/current/scripts/components/inventoryitem_replica.lua#L171)

## 典型接入

### 给原版吸湿器的自然目标增加10度偏移

完整代码放已加载模组 `modmain.lua`，服务端只改普通 desiccant；复用其组件／资源，无新增任务。原工厂、注册名与加载约定见 [desiccant.lua:147](../../../raw/current/scripts/prefabs/desiccant.lua#L147)、[:181](../../../raw/current/scripts/prefabs/desiccant.lua#L181)、[[reference/components/inventoryitem#典型接入]]。

```lua
local MODIFIER_KEY = "example_mod_desiccant_warm_bias"

AddPrefabPostInit("desiccant", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local inventoryitem = inst.components.inventoryitem
    if inventoryitem ~= nil
        and inst.components.inventoryitemtemperature ~= nil then
        inventoryitem:SetTemperatureModifier(MODIFIER_KEY, 10)
    end
end)
```

10为下次自然计算的目标温度偏移，并非每秒升10度，仍受边界和[容器／换热分支](#自然换热与容器分支)影响。读档重建实例重登 modifier，OnLoad 不清它，无须包装 OnLoad。换成模组独有 key；撤销调用同宿主 `RemoveTemperatureModifier(MODIFIER_KEY)`，不还原旧温度。停用并重载后不再登记，已存温度继续自然变化。

原版监听依据自身／目标温度切换湿度控制，持有者仍潮湿时还会限制物品温度；本例保留这些规则，会改变切换时机，不保证固定吸湿速度或人物升温。[desiccant.lua:53](../../../raw/current/scripts/prefabs/desiccant.lua#L53)、[:165](../../../raw/current/scripts/prefabs/desiccant.lua#L165)

## 常见陷阱与限制

- **照搬 temperature:SetTemp／SetInsulationModifier**：本扩展没有这些接口；赋温、偏移和边界的区别见[接口](#关键接口)，不构成恒温方案。
- **跨组件／端假定相同 temperaturedelta**：根据[同步约定](#更新保存与网络读取)检查 payload，不能要求客户端也有 last／边界。
- **堆叠精确保留每件温度**：拆堆调用宿主 SetTemperature，合堆按件数加权；未启用时 setter 无效。新堆叠宿主须保证两侧温度均为数值。[stackable.lua:141](../../../raw/current/scripts/components/stackable.lua#L141)、[:175](../../../raw/current/scripts/components/stackable.lua#L175)、[inventoryitemtemperature.lua:113](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L113

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 8 处（inventoryitem 四个宿主 setter、tuning STARTING_TEMP、mainfunctions 睡醒分发、entityscript OnLoad 分发）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 inventoryitem.lua（本版改动在湿润继承／拾取阴燃段，温度 setter 仅行移）、tuning.lua、entityscript.lua、mainfunctions.lua，inventoryitemtemperature.lua 与 desiccant 未变。
- 源码审阅：2026-10-21，steam-buildid:25643519；扩展全文及本页宿主、换热／容器、更新／存读／清理、堆叠、网络原码／解码及 desiccant 消费者。人物／暖石和 PostInit 复用同版本 Wiki。
- 检查：本页结构、来源存在性、链接／源码行号及完整 modmain 示例 Lua 5.1 语法通过，未检查导航登记。
- 游戏运行：未运行；编码边界／初始同步、多人可见性、吸湿器切换、睡醒／读档和模组冲突待实测；未枚举全部温度物品／容器。

相关：[[reference/components/inventoryitem]]；[[reference/components/inventoryitemmoisture]]；[[reference/components/temperature]]；[[reference/components/heater]]。
