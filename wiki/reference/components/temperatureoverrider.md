---
title: "temperatureoverrider"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "用已同步的范围实体覆盖当地环境温度，理解最近来源选择、眼球塔启停及禁止卸载的生命周期。"
aliases: ["局部环境温度", "范围温度覆盖", "眼球塔温度", "temperatureoverrider"]
tags: [component, lifecycle, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/temperatureoverrider.lua", "prefabs/deerclopseyeball_sentryward.lua", "components/inventoryitemholder.lua", "components/fumarolelocaltemperature.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# temperatureoverrider

## 快速判断

- 职责与适用：将活动实体注册为圆形局部环境温度来源；查询范围内最近实体的温度，未命中才回退火山局部温度／世界温度。它不直接锁住人物体温。[temperatureoverrider.lua:14-37](../../../raw/current/scripts/components/temperatureoverrider.lua#L14-L37)；体温消费者见 [[reference/components/temperature#自然温度绝热与容器如何选择]]。
- 优先使用：调整已有眼球塔组件的 SetRadius／SetTemperature，保留眼球插拔控制 Enable／Disable；新 prefab 应在两端共同构造阶段安装，参照下文原版工厂。
- 重要副作用：活动半径和温度是 net_float；客户端收到半径 dirty 后增删本地注册项，不同来源不相加、不按高温／低温择优。[temperatureoverrider.lua:47-78](../../../raw/current/scripts/components/temperatureoverrider.lua#L47-L78)
- 修改边界：两端安装、服务端配置；`OnRemoveFromEntity` 直接 assert(false)，不能 RemoveComponent，停用用 Disable，整实体删除有独立注销钩子。[temperatureoverrider.lua:83-127](../../../raw/current/scripts/components/temperatureoverrider.lua#L83-L127)

## 关键接口

| 接口／字段 | 签名或已核验约定 | 条件、作用与证据 |
|---|---|---|
| `SetTemperature(value)`／`GetTemperature()` | 游戏温度数值 | setter 仅主机写 netvar；初始温度 25。读值不检查启用状态，不代表当前位置命中该来源。[temperatureoverrider.lua:66-70](../../../raw/current/scripts/components/temperatureoverrider.lua#L66-L70)、[temperatureoverrider.lua:97-108](../../../raw/current/scripts/components/temperatureoverrider.lua#L97-L108) |
| `SetRadius(radius)`／`GetActiveRadius()` | 世界距离；有效半径来自 netvar | 主机默认配置 radius=16、enabled=false；启用时改 radius 立即更新活动半径，停用时只改配置供下次 Enable 使用。[temperatureoverrider.lua:41-45](../../../raw/current/scripts/components/temperatureoverrider.lua#L41-L45)、[temperatureoverrider.lua:66-78](../../../raw/current/scripts/components/temperatureoverrider.lua#L66-L78)、[temperatureoverrider.lua:110-120](../../../raw/current/scripts/components/temperatureoverrider.lua#L110-L120) |
| `Enable()`／`Disable()` | 无参数、无返回值 | 主机幂等启停；活动半径在配置值／0 间切换，0 对应注销，客户端由 dirty 同步注册。[temperatureoverrider.lua:116-138](../../../raw/current/scripts/components/temperatureoverrider.lua#L116-L138) |
| `GetTemperatureAtXZ(x, z)`／`GetLocalTemperature(inst)` | 全局函数 → 温度数值 | 前者按平面距离平方选范围内最近来源，边界包含；后者取实体坐标后转调。不经本组件实例调用。[temperatureoverrider.lua:14-37](../../../raw/current/scripts/components/temperatureoverrider.lua#L14-L37) |

## 典型接入

### 调整眼球塔的环境温度并保留原版插拔启停

目标模组 `modmain.lua`，不新增资源。注册名是 `deerclopseyeball_sentryward`；原工厂在 SetPristine 和客户端返回前添加组件，随后服务端设置半径／温度并初始化为停用状态。PostInit 只配置已经存在的组件，不在此时增挂 netvar 组件。[deerclopseyeball_sentryward.lua:440-462](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L440-L462)、[:498-501](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L498-L501)、[:645](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L645)

```lua
AddPrefabPostInit("deerclopseyeball_sentryward", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local overrider = inst.components.temperatureoverrider
    if overrider == nil then return end
    overrider:SetRadius(20)
    overrider:SetTemperature(5)
end)
```

20 是世界距离，5 是环境温度。只改变本组件的覆盖范围和值；塔仍由 `OnEyeballGiven` Enable、`OnEyeballTaken` Disable，保留插拔、地图、生成与视觉等原版回调。[deerclopseyeball_sentryward.lua:153-188](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L153-L188)、[:196-203](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L196-L203)

这不会将塔的地面冰冻、直接实体赋温、生成范围或放置预览一并改为 20／5；这些是宿主独立逻辑，不能从环境覆盖推断人物立即到 5 度。[deerclopseyeball_sentryward.lua:517-523](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L517-L523)、[:546-568](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L546-L568)；体温自然换热复用 [[reference/components/temperature]]。

本组件无 OnSave／OnLoad。配置在每次工厂初始化后重建；原塔的眼球由 inventoryitemholder 保存，读档经 GiveItem 再触发原 given 回调恢复启用，不需本例再包装 OnLoad。[inventoryitemholder.lua:157-162](../../../raw/current/scripts/components/inventoryitemholder.lua#L157-L162)、[:214-232](../../../raw/current/scripts/components/inventoryitemholder.lua#L214-L232)

临时撤销覆盖用 Disable；其后再次插眼仍由原版 Enable 恢复。撤销本例数值修改需恢复所需原配置，或停用模组后重新载入实例；不要 RemoveComponent。实体移除时 OnRemoveEntity 注销活动来源。[temperatureoverrider.lua:83-91](../../../raw/current/scripts/components/temperatureoverrider.lua#L83-L91)

### 自定义 prefab 的构造边界

沿用原塔的两端共用阶段：创建 Network 后、SetPristine 前 AddComponent；客户端保留组件并返回，主机之后 SetRadius／SetTemperature 并按自身状态 Enable。它不是 replica 配对组件，服务端单挂无法让客户端维护对应来源。本页不提供缺少工厂／资源上下文的临时增挂示例。[deerclopseyeball_sentryward.lua:440-462](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L440-L462)

## 常见陷阱与限制

- **多个覆盖器自动叠加／取最低温**：查询只选范围内最近实体，同距离没有稳定优先级保证；温度为 0 也会有效返回（Lua 中 0 为真）。[temperatureoverrider.lua:14-31](../../../raw/current/scripts/components/temperatureoverrider.lua#L14-L31)
- **禁用后恢复旧覆盖温度**：Disable 仅移出活动集合；查询重新选其他有效来源，再回退世界 net 的 fumarolelocaltemperature（其查询可返回 nil），最后世界温度，不保存／恢复当前位置旧值。[temperatureoverrider.lua:29-31](../../../raw/current/scripts/components/temperatureoverrider.lua#L29-L31)、[fumarolelocaltemperature.lua:140-165](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L140-L165)
- **任意半径都有校验**：SetRadius 不夹取范围或验证正值；0 用于非活动，负半径会参与平方比较，实际接入用明确正数，不依赖非法输入行为。[temperatureoverrider.lua:18-21](../../../raw/current/scripts/components/temperatureoverrider.lua#L18-L21)、[:110-138](../../../raw/current/scripts/components/temperatureoverrider.lua#L110-L138)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖组件全文、两端 netvar／注册／最近选择／停用／移除、真实塔注册及工厂顺序、插拔与容器保存恢复；体温消费复用同 build temperature 页。未审校其他所有环境温度消费者。
- 结构检查：页面格式、链接、大小及源码行锚点检查通过；完整 modmain 示例经现有 Full Moon 解析器 Lua 5.1 语法检查。
- 游戏运行：未运行；联机同步、重连、交叠来源、插拔／读档及独立冰冻效果待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。