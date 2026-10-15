---
title: "heater"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置附近、携带或装备热源的输出与热／冷方向，明确暖石、携带范围和人物体温的区别。"
aliases: ["随身热源", "携带供热", "暖石供热", "装备供热", "热源输出", "heater"]
tags: [component, item, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/heater.lua", "components/temperature.lua", "components/inventory.lua", "prefabs/heatrock.lua", "modutil.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# heater

## 快速判断

- 职责与适用：提供热／冷输出供 temperature 查询；热源不必有自身 temperature。附近、携带、装备是三个独立读取通道。[heater.lua:53](../../../raw/current/scripts/components/heater.lua#L53)、[:64](../../../raw/current/scripts/components/heater.lua#L64)、[:72](../../../raw/current/scripts/components/heater.lua#L72)
- 优先使用：先选 heat／carriedheat／equippedheat 或其 fn，再设 SetThermics；固定自身温度用 [[reference/components/temperature]]，绝热用 [[reference/components/insulator]]。
- 重要副作用：构造添加 HASHEATER，移组件删除标签；附近消费者先 GetHeat，再看方向，因此 heatfn 可动态改变热／冷状态。[heater.lua:16](../../../raw/current/scripts/components/heater.lua#L16)、[temperature.lua:437](../../../raw/current/scripts/components/temperature.lua#L437
- 修改边界：服务端计算。输出是换热参考温度，环境和其他热源仍参与，不保证人物恒温；携带范围按消费者枚举，不等同任意嵌套持有者。[temperature.lua:370](../../../raw/current/scripts/components/temperature.lua#L370

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `heat`／`heatfn` → `GetHeat(observer)` | 温度数值或 nil；fn(inst, observer) | fn 返回 nil／false 才回退 heat；0 是有效数值。仅附近通道。[heater.lua:53](../../../raw/current/scripts/components/heater.lua#L53 |
| `carriedheat`／`carriedheatfn` → `GetCarriedHeat(observer?)` | 温度, multiplier；fn(inst, observer) | fn 存在就直接使用其结果，不回退 carriedheat；temperature 调用时不传 observer。[heater.lua:72](../../../raw/current/scripts/components/heater.lua#L72)、[temperature.lua:383](../../../raw/current/scripts/components/temperature.lua#L383 |
| `equippedheat`／`equippedheatfn` → `GetEquippedHeat(observer?)` | 温度, multiplier | 与携带同样的 fn 优先规则，无 heat／carriedheat 自动回退；消费者不传 observer。[heater.lua:64](../../../raw/current/scripts/components/heater.lua#L64)、[temperature.lua:373](../../../raw/current/scripts/components/temperature.lua#L373 |
| `carriedheatmultiplier` | 默认 1 | 携带及装备都乘这个值；作用于 (heat-current) 的贡献，不直接乘最终温度或速率。[heater.lua:9](../../../raw/current/scripts/components/heater.lua#L9)、[temperature.lua:377](../../../raw/current/scripts/components/temperature.lua#L377 |
| `SetThermics(exo, endo)` | 热／冷 boolean；默认 true,false | 携带通道可两者 true，低于热源时升温、高于时降温；附近通道两者 true 时优先 exo 分支。[heater.lua:24](../../../raw/current/scripts/components/heater.lua#L24)、[temperature.lua:385](../../../raw/current/scripts/components/temperature.lua#L385)、[:465](../../../raw/current/scripts/components/temperature.lua#L465) |
| `SetShouldFalloff(should_falloff)`／`SetHeatRadiusCutoff(radius)` | boolean／距离或 nil | 只控制附近通道；默认距离平方衰减，cutoff 额外裁剪，不扩大查找范围。[heater.lua:37](../../../raw/current/scripts/components/heater.lua#L37)、[temperature.lua:441](../../../raw/current/scripts/components/temperature.lua#L441 |

## 典型接入

### 让暖石只在携带时按35度双向换热

目标模组 `modmain.lua`；复用原版 heatrock 资源／工厂和现有 heater，不新增任务。主动替换原版按环境分档的输出，关闭附近输出；原版分别绑定 heatfn 和 carriedheatfn，所以只改数值字段会被函数覆盖。[heatrock.lua:54](../../../raw/current/scripts/prefabs/heatrock.lua#L54)、[:243](../../../raw/current/scripts/prefabs/heatrock.lua#L243)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("heatrock", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local heater = inst.components.heater
    if heater == nil then return end
    heater.heatfn = nil
    heater.heat = nil
    heater.carriedheatfn = nil
    heater.carriedheat = 35
    heater.carriedheatmultiplier = 1
    heater:SetThermics(true, true)
end)
```

携带时 temperature 根据自身温度选择加热或冷却，贡献为 (35-current)；自身已是 35 时该项为零，环境仍能改变人物温度。这不是人物 35 的锁定；仅需要取暖时改 SetThermics(true,false)。石头自身温度仍由原版 temperature 更新，需要锁定再读 [[reference/components/temperature#让已有暖石自身保持35度并在读档后重新赋温]]。

入对应持有范围即参与下一次计算，离开即不再汇总，无需手动撤销人物任务／数值。原版 heatrock 没有 equippedheat，不能因为装备了这个物品就假设携带配置生效。停用后新实例恢复原版输出；图标、灯光与 fueled 耐久仍走原版，不承诺它们与固定输出一致。[temperature.lua:370](../../../raw/current/scripts/components/temperature.lua#L370)、[heatrock.lua:115](../../../raw/current/scripts/prefabs/heatrock.lua#L115

### 携带范围与地面热源的分工

| 使用模式 | 消费路径与边界 |
|---|---|
| 人物主栏 | 遍历 inventory.itemslots，调用 GetCarriedHeat；不靠世界查找，所以入栏 INLIMBO 不阻止此路径。[temperature.lua:381](../../../raw/current/scripts/components/temperature.lua#L381 |
| 装备槽 | 遍历 equipslots，只调用 GetEquippedHeat；想兼顾两种状态须分别配置。[temperature.lua:370](../../../raw/current/scripts/components/temperature.lua#L370 |
| 原版背包／鼠标手持 | GetOverflowContainer().slots 和 GetActiveItem 同走携带通道；不递归所有包内容器，特殊附加槽／多装备栏需另核验。[temperature.lua:391](../../../raw/current/scripts/components/temperature.lua#L391)、[inventory.lua:1459](../../../raw/current/scripts/components/inventory.lua#L1459 |
| 附近地面热源 | 10 单位内查 HASHEATER，排除指定标签与 INLIMBO；默认因子 1-distsq/100，再按湿度与热／冷分支处理。半径裁剪仅缩小作用范围。[temperature.lua:4](../../../raw/current/scripts/components/temperature.lua#L4)、[:350](../../../raw/current/scripts/components/temperature.lua#L350)、[:434](../../../raw/current/scripts/components/temperature.lua#L434) |

原版暖石的两种输出调用同一 HeatFn，依据当地温差切换热／冷／中性，携带倍率为 2.1；这是动态石头模式，不是通用“热源必须装 temperature”的要求。[heatrock.lua:54](../../../raw/current/scripts/prefabs/heatrock.lua#L54)、[:246](../../../raw/current/scripts/prefabs/heatrock.lua#L246)、[tuning.lua:2292](../../../raw/current/scripts/tuning.lua#L2292

## 常见陷阱与限制

- **只写 heat=35 就能随身供热**：携带 getter 不回退 heat；按目标通道配置数值／fn，并清理该通道旧 fn。[heater.lua:72](../../../raw/current/scripts/components/heater.lua#L72
- **数值越大加热越快且最终恒温**：多个槽位热源分别累加贡献，最终 rate 还受 temperature 限制；多份携带可能叠加，不是固定上限或恒温保证。[temperature.lua:381](../../../raw/current/scripts/components/temperature.lua#L381)、[:487](../../../raw/current/scripts/components/temperature.lua#L487)
- **heatfn 的 observer 一定是持有者**：附近调用传接收者，携带／装备的原版调用不传 observer；可选参数要判 nil，不能直接访问 observer.components。[temperature.lua:373](../../../raw/current/scripts/components/temperature.lua#L373)、[:437](../../../raw/current/scripts/components/temperature.lua#L437)
- **新 Prefab 服务端 AddComponent 就足够维护双端标签**：原版建议 HASHEATER 在 SetPristine 前显式添加；新工厂按此公共初始化，PostInit 案例复用已有暖石标签。[heater.lua:16](../../../raw/current/scripts/components/heater.lua#L16)、[heatrock.lua:215](../../../raw/current/scripts/prefabs/heatrock.lua#L215

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（tuning.lua:2282→2292，暖石携带倍率键值未变，显示标签同步）；结论修订：无，仅锚点迁移。未引用 FUMAROLEHOE 系键。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖三个输出通道、回退差异、方向／距离／倍率、实际携带枚举与暖石动态参照；未审校所有热源或特殊容器／装备模组。
- 结构检查：页面、导航、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；不同温度下双向换热、主栏／背包／手持转移、多份效果、图标／耐久与其他模组协作待实测。

相关：[[reference/components/temperature]]；[[reference/components/inventory#携带范围的选择]]；[[reference/components/inventoryitem#生命周期]]。
