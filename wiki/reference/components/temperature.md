---
title: "temperature"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "控制人物或物品自身温度、自然换热与温度锁定，区分暖石温度、热源输出和人物体温。"
aliases: ["恒温35度", "物品恒温", "人物体温", "温度锁定", "暖石温度", "temperature"]
tags: [component, lifecycle, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/temperature.lua", "components/heater.lua", "components/inventory.lua", "components/temperatureoverrider.lua", "prefabs/heatrock.lua", "entityscript.lua", "modutil.lua", "mainfunctions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# temperature

## 快速判断

- 职责与适用：维护宿主自身温度，按环境、热源、绝热与直接容器状态更新；人物过冷／过热还调用 health。物品温度不等于持有者体温。[temperature.lua:311](../../../raw/current/scripts/components/temperature.lua#L311)、[:517](../../../raw/current/scripts/components/temperature.lua#L517)
- 优先使用：瞬时赋温 SetTemperature，增减 DoDelta，暂停自然更新 SetTemp；暖石供热另用 [[reference/components/heater]]，装备绝热用 [[reference/components/insulator]]。[temperature.lua:71](../../../raw/current/scripts/components/temperature.lua#L71)、[:159](../../../raw/current/scripts/components/temperature.lua#L159)
- 重要副作用：SetTemperature 限制在 mintemp／maxtemp 内，跨过冻结／过热阈值发状态事件，每次调用均发 temperaturedelta；人物 current 通过 player_classified 同步。[temperature.lua:7](../../../raw/current/scripts/components/temperature.lua#L7)、[:166](../../../raw/current/scripts/components/temperature.lua#L166)
- 修改边界：服务端配置。SetTemp 非 nil 会跳过自然换热及本组件温度伤害，但不阻止直接赋温，也不在每帧重写锁定值；保存恢复另见下文。[temperature.lua:316](../../../raw/current/scripts/components/temperature.lua#L316

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `GetCurrent()`／`SetTemperature(value)` | 温度数值 | 后者一次赋值并发事件，不锁定后续变化；冻结为 <0，过热为 >overheattemp。[temperature.lua:111](../../../raw/current/scripts/components/temperature.lua#L111)、[:166](../../../raw/current/scripts/components/temperature.lua#L166) |
| `DoDelta(delta, skipinsulation?)` | 相对温度；省略第二项会应用绝热 | 正 delta 用夏季绝热，负 delta 用冬季绝热，然后 SetTemperature；true 跳过缩减。[temperature.lua:71](../../../raw/current/scripts/components/temperature.lua#L71 |
| `SetTemp(temp)` | number 或 nil | 非 nil 存锁并立即 SetTemperature；nil 解除锁，不还原旧 current；不同来源不叠加。[temperature.lua:159](../../../raw/current/scripts/components/temperature.lua#L159 |
| `SetModifier(name, value)`／`RemoveModifier(name)` | 独立名称＋温度偏移 | 加到自然换热的目标差；0／nil 删除。普通 keyed 表，不是实体来源自动清理的 SourceModifierList。[temperature.lua:198](../../../raw/current/scripts/components/temperature.lua#L198)、[:367](../../../raw/current/scripts/components/temperature.lua#L367) |
| `SetInsulationModifier(type, src, insulation, key?)`／`RemoveInsulationModifier(type, src, key?)` | SEASONS.WINTER／SUMMER；加法来源列表 | 增删绝热贡献，不是恒温；最终冬夏值分别非负封底。[temperature.lua:231](../../../raw/current/scripts/components/temperature.lua#L231)、[:302](../../../raw/current/scripts/components/temperature.lua#L302) |
| `SetTemperatureInBelly(delta, duration)` | 温度贡献、持续秒数 | 覆盖唯一腹内槽并重设到期任务，不立即 DoDelta；更新时还检查热／冷食物阈值。[temperature.lua:94](../../../raw/current/scripts/components/temperature.lua#L94)、[:418](../../../raw/current/scripts/components/temperature.lua#L418) |
| `IgnoreTags(...)` | 排除的热源标签 | 替换整个名单并保留 INLIMBO；暖石传 heatrock，影响附近热源查询，不是携带槽位过滤。[temperature.lua:155](../../../raw/current/scripts/components/temperature.lua#L155)、[heatrock.lua:241](../../../raw/current/scripts/prefabs/heatrock.lua#L241 |
| `SetFreezingHurtRate(rate)`／`SetOverheatHurtRate(rate)` | 伤害／秒 | 自然更新乘 dt；过热未单设时回退冻结率，不控制其他伤害来源。[temperature.lua:63](../../../raw/current/scripts/components/temperature.lua#L63)、[:517](../../../raw/current/scripts/components/temperature.lua#L517) |

## 典型接入

### 让已有暖石自身保持35度并在读档后重新赋温

目标模组 `modmain.lua`；复用原版 heatrock 工厂、资源、组件和灯光，不新增文件。只修改石头自身；原版注册名和温度初始化见 [heatrock.lua:237](../../../raw/current/scripts/prefabs/heatrock.lua#L237)、[:301](../../../raw/current/scripts/prefabs/heatrock.lua#L301)，PostInit 在工厂后执行。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:388](../../../raw/current/scripts/mainfunctions.lua#L388

```lua
AddPrefabPostInit("heatrock", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local function ApplyTemperature(item)
        if item.components.temperature ~= nil then
            item.components.temperature:SetTemp(35)
        end
    end
    ApplyTemperature(inst)
    local oldload = inst.OnLoad
    inst.OnLoad = function(item, data, newents)
        if oldload ~= nil then oldload(item, data, newents) end
        ApplyTemperature(item)
    end
end)
```

组件 OnLoad 能直接改 current，且先于实体 OnLoad；因此保留原实体回调，再重设锁和值，避免只在 PostInit 设置却读回旧温度。[temperature.lua:128](../../../raw/current/scripts/components/temperature.lua#L128)、[entityscript.lua:2005](../../../raw/current/scripts/entityscript.lua#L2005

35 是游戏温度数值。本例不挂周期任务；取消效果用 SetTemp(nil)，重新启用自然更新但保留当前温度。停用模组后新实例不再带锁，已存 current 仍按原版读档规则处理；其他模组再次 SetTemperature／DoDelta／OnLoad 包装仍可干扰，本例不是任意来源无法打破的恒温。[temperature.lua:119](../../../raw/current/scripts/components/temperature.lua#L119)、[:159](../../../raw/current/scripts/components/temperature.lua#L159)

原版 HeatFn 按自身与当地环境的温差分档，输出 -10／10／25／40／60，且可能关闭换热；自身 35 不保证供热 35、人物 35 或一直发光。灯光／图标靠 temperaturedelta 更新，而锁定会停止日常更新事件；环境改变后不能据此保证显示刷新。需要固定携带输出时读 [[reference/components/heater#让暖石只在携带时按35度双向换热]]，持续照明另核验独立灯光及归属。[heatrock.lua:41](../../../raw/current/scripts/prefabs/heatrock.lua#L41)、[:54](../../../raw/current/scripts/prefabs/heatrock.lua#L54)、[:85](../../../raw/current/scripts/prefabs/heatrock.lua#L85)、[:115](../../../raw/current/scripts/prefabs/heatrock.lua#L115)

### 自然温度、绝热与容器如何选择

- 附近／装备／携带热源：heater 提供换热值；自然更新先汇总环境和热源，再按绝热限制 rate，最后乘直接持有者 preserver 的倍率与 dt。不是立即把人物赋值到热源温度。[temperature.lua:367](../../../raw/current/scripts/components/temperature.lua#L367)、[:370](../../../raw/current/scripts/components/temperature.lua#L370)、[:487](../../../raw/current/scripts/components/temperature.lua#L487)、[:512](../../../raw/current/scripts/components/temperature.lua#L512)
- 降低变化速率：装备用 [[reference/components/insulator]]；容器内物品用 [[reference/components/preserver]]。fridge 有独立冷却分支；倍率取直接 owner，不沿任意嵌套链继承。[temperature.lua:328](../../../raw/current/scripts/components/temperature.lua#L328
- 改当地环境：GetTemperatureAtXZ 会选择有效范围内最近的 temperatureoverrider，再回退局部火山温度／世界温度；本页不提供修改世界环境的完整接入。[temperatureoverrider.lua:14](../../../raw/current/scripts/components/temperatureoverrider.lua#L14

## 生命周期

- 构造先 OnUpdate(0)，再启动组件更新；SetTemp、传送、health 无敌提前返回，所以不是只暂停温度值而继续结算本组件伤害。[temperature.lua:55](../../../raw/current/scripts/components/temperature.lua#L55)、[:316](../../../raw/current/scripts/components/temperature.lua#L316)
- OnSave 只存 current 与腹内剩余效果，不存锁、偏移和绝热来源；人物 OnLoad 还按世界温度调整，不保证精确还原旧体温。常驻配置在初始化重建，瞬时锁定案例另处理恢复顺序。[temperature.lua:119](../../../raw/current/scripts/components/temperature.lua#L119
- OnRemoveFromEntity 取消腹内任务、撤 sheltered 监听并清人物 isfreezing；不是所有温度 UI／来源的完整撤销函数。宿主 RemoveComponent 停止组件更新。[temperature.lua:103](../../../raw/current/scripts/components/temperature.lua#L103)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 常见陷阱与限制

- **SetTemperature(35) 当作恒温**：只是一次赋温，下一轮自然换热仍发生；按本页案例用锁并处理读档，外部直接赋温另协调。[temperature.lua:159](../../../raw/current/scripts/components/temperature.lua#L159
- **给人物 SetTemp 来模拟持有暖石**：会跳过环境与温度伤害，改变人物规则；仅供热优先 heater，持有者体温仍由人物 temperature 算。[temperature.lua:316](../../../raw/current/scripts/components/temperature.lua#L316
- **温度不变就无事件／无耐久变化**：SetTemperature 每次都发 temperaturedelta，hasrate 使用当前缓存 rate；暖石监听还可能按温区历史扣 fueled。本例未禁用该监听或耐久。[temperature.lua:178](../../../raw/current/scripts/components/temperature.lua#L178)、[heatrock.lua:140](../../../raw/current/scripts/prefabs/heatrock.lua#L140

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（entityscript／mainfunctions 行号迁移）；结论修订：无（R-08 潮湿度门控不触及本页换热／绝热结论）。
- 依据：全局枢纽 hunk 地图（R-08/R-09）＋变更清单命中 entityscript.lua、mainfunctions.lua。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖赋温／锁、偏移／绝热、自然汇总、容器、伤害、保存／清理及暖石调用与读档顺序；未审校所有特殊容器／人物机制或全组件类型。
- 结构检查：页面、导航、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；恒温与读档、环境变更的显示／耐久、多人及其他模组协作待实测。

相关：[[reference/components/heater]]；[[reference/components/health]]；[[reference/components/inventoryitem#生命周期]]；[[reference/components/edible#关键接口]]。
