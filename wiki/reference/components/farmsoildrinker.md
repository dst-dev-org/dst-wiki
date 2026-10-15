---
title: "farmsoildrinker"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "调整作物土壤耗水率并统计阶段湿润时间，区分土壤含水、饮水回调与人物潮湿。"
aliases: ["作物耗水减半", "土壤饮水", "湿润时间比例", "farmsoildrinker"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farmsoildrinker.lua", "components/farming_manager.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farmsoildrinker

## 快速判断

- 职责与适用：植物侧向世界登记饮水者，提供每秒土壤水分变化率，并累计干／湿时长；不是 [[reference/components/moisture|人物湿度]]。[组件](../../../raw/current/scripts/components/farmsoildrinker.lua#L2-L49)
- 优先使用：改耗水包装 `getdrinkratefn`；读阶段湿润比例用 `CalcPercentTimeHydrated()`。实际水量由世界 farming_manager 汇总，单加组件不会主动浇水。[读数](../../../raw/current/scripts/components/farmsoildrinker.lua#L32-L43) [消费者](../../../raw/current/scripts/components/farming_manager.lua#L299-L327)
- 重要副作用：原作物饮水率是负值，乘 0.5 才减半消耗；比例读取也会结算当前计时并重置计时起点。[参数](../../../raw/current/scripts/tuning.lua#L5802-L5806) [计时](../../../raw/current/scripts/components/farmsoildrinker.lua#L25-L38)
- 修改边界：本页服务端固定位置的原作物，依赖世界管理器及宿主回调；只改耗水不保证免除水分压力，压力看湿润时长比例。[原回调](../../../raw/current/scripts/prefabs/farm_plants.lua#L121-L127) [压力判断](../../../raw/current/scripts/prefabs/farm_plants.lua#L203-L209)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `getdrinkratefn`／`GetMoistureRate()` | `(plant) -> number`；无回调／失败值时 0 | 世界按 dt 乘总速率；原作物无 growable 或 IsGrowing 时返回 plant_def.moisture.drink_rate，否则 0。原数据为负值，不能把正数称耗水。[接口](../../../raw/current/scripts/components/farmsoildrinker.lua#L41-L43) [宿主](../../../raw/current/scripts/prefabs/farm_plants.lua#L121-L123) [汇总](../../../raw/current/scripts/components/farming_manager.lua#L317-L327) |
| `onsoilmoisturestatechangefn` | `(plant,cur_state,prev_state)` | 组件只转发；原宿主再调用 UpdateMoistureTime。世界只在 0 与非 0 之间切换时通知，并调整 wildfireprotected 标签。[转发](../../../raw/current/scripts/components/farmsoildrinker.lua#L45-L49) [宿主](../../../raw/current/scripts/prefabs/farm_plants.lua#L125-L127) [通知](../../../raw/current/scripts/components/farming_manager.lua#L109-L123) |
| `UpdateMoistureTime(is_soil_moist, was_soil_moist)` | 累计秒数，无显式返回 | 第一参数当前未使用；按第二参数把上一时段计入 wet 或 dry，随后更新 timer_start_time。参数顺序不能对调。[实现](../../../raw/current/scripts/components/farmsoildrinker.lua#L25-L30) |
| `CalcPercentTimeHydrated()` | 0～1；无湿润时长时 0 | 先向世界查询当前湿润状态并补记时间；wet/(dry+wet)，不是水量百分比。世界判断还考虑全局 wetness。[计算](../../../raw/current/scripts/components/farmsoildrinker.lua#L32-L38) [湿润判定](../../../raw/current/scripts/components/farming_manager.lua#L472-L475) |
| `Reset()` | 清 time.dry／wet，重置起点 | 原阶段结算先读取湿润比例，再 Reset，统计窗口不是一生累计。[实现](../../../raw/current/scripts/components/farmsoildrinker.lua#L19-L23) [阶段](../../../raw/current/scripts/prefabs/farm_plants.lua#L238-L253) |
| `OnSave()`／`OnLoad(data)`／`CopyFrom(rhs)` | 保存 time 与起点的已过秒数 | Load 重建起点，不把旧绝对时间直接搬入；data.time 须存在，未做任意缺字段保护。随机种子替换用 CopyFrom 继承统计。[存读](../../../raw/current/scripts/components/farmsoildrinker.lua#L51-L64) [复制](../../../raw/current/scripts/components/farmsoildrinker.lua#L15-L17) [替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L166) |

构造延迟 0 秒向 TheWorld 发 ms_registersoildrinker，移除组件发 ms_unregistersoildrinker；管理器按当时地块登记／移除，并监听实体 onremove。该实现不是移动实体的自动跨地块追踪器，不把原植物用法推广到可移动容器。[组件生命周期](../../../raw/current/scripts/components/farmsoildrinker.lua#L2-L13) [登记与移除](../../../raw/current/scripts/components/farming_manager.lua#L183-L215) [世界监听](../../../raw/current/scripts/components/farming_manager.lua#L613-L614)

## 典型接入

### 种植胡萝卜的原耗水率减半，保留停长时不饮水

放目标模组 `modmain.lua`，目标 `farm_plant_carrot`，无需新资源。包装已有回调而不改共享 plant_def.moisture，保留原 IsGrowing 门禁与原值符号；只改这一类植物。原胡萝卜用低耗水档，名称由定义和工厂注册。[参数](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L59-L65) [命名](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L258) [注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926-L937) [安装](../../../raw/current/scripts/prefabs/farm_plants.lua#L863-L865) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_plant_carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local drinker = inst.components.farmsoildrinker
    if drinker == nil or drinker.getdrinkratefn == nil then return end
    local original = drinker.getdrinkratefn
    drinker.getdrinkratefn = function(plant)
        return (original(plant) or 0) * 0.5
    end
end)
```

生效于后续世界水量刷新，以及原魔法催生读取 GetMoistureRate 的路径；原魔法按剩余成长时间乘饮水率一次调整土壤水分。其他植物及天气蒸发不变，停用后重新初始化恢复原回调，已变化土壤水量不回滚。[世界刷新](../../../raw/current/scripts/components/farming_manager.lua#L299-L327) [魔法路径](../../../raw/current/scripts/prefabs/farm_plants.lua#L595-L604)

阶段水分压力由 [[reference/components/farmplantstress]] 协作结算；照料只处理快乐，见 [[reference/components/farmplanttendable]]，不代替浇水。

## 常见陷阱与限制

- **把速率改成正数“吸更多水”**：世界直接相加，方向反了；沿用负率并乘倍率，0 只停植物该项消耗，不停止天气干燥。[汇总](../../../raw/current/scripts/components/farming_manager.lua#L313-L327)
- **只加组件便期待正确干湿统计**：世界通知只转发给回调，无回调不会结算切换前时段；原工厂同时配置饮水与状态变化回调。[转发](../../../raw/current/scripts/components/farmsoildrinker.lua#L45-L49) [配置](../../../raw/current/scripts/prefabs/farm_plants.lua#L863-L865)
- **Reset 后立刻读比例判断缺水**：湿润累计仍为 0 时结果就是 0；原作物先取比例再重置，不能交换顺序，也不能把频繁查看当无副作用读取。[比例](../../../raw/current/scripts/components/farmsoildrinker.lua#L32-L38) [阶段](../../../raw/current/scripts/prefabs/farm_plants.lua#L238-L243)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（tuning FARM_PLANT_DRINK 档位段 +14）；无结论修订，仅锚点迁移——各档耗水率数值未变（tuning-hunk-map 5 改值键不含），farmsoildrinker.lua、farming_manager.lua、farm_plants 系全部未变。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua。
- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖速率符号／每秒消费、干湿转移统计、读数副作用、世界登记与生命周期、存读／替换、阶段压力及魔法用量。未审校世界农作管理器全部逻辑或可移动饮水者适配。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only，不证明水量刷新时机或暂停效果。
- 游戏运行：未运行；干湿切换、降雨、离屏／存读、成长暂停、魔法催生、实体替换及其他模组倍率叠加待实测。
