---
title: "teleporter"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "管理虫洞类入口的目标、激活与接收流程，区分瞬移坐标、人物 SG 和跨分片迁移。"
aliases: ["虫洞传送", "传送门配对", "传送落点偏移", "teleporter"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/teleporter.lua", "prefabs/wormhole.lua", "stategraphs/SGwilson.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# teleporter

## 快速判断

- 职责与适用：同分片配对出口、人物／物品接收及可选分片迁移；原版虫洞在服务端挂载，动作与 SG 另行合作。[宿主](../../../raw/current/scripts/prefabs/wormhole.lua#L148-L168)
- 优先使用：`Target(exit)` 指定单向目标，双向需双方各设；业务走 `Activate(doer)`，不绕过它直接用底层 Teleport。[目标与启用](../../../raw/current/scripts/components/teleporter.lua#L349-L371)、[激活](../../../raw/current/scripts/components/teleporter.lua#L96-L166)
- 重要副作用：激活回调、跟随者转移、接收计时／屏幕渐变和人物 SG 存在协作；返回 true 不是所有对象都已成功安全落地。[移动限制](../../../raw/current/scripts/components/teleporter.lua#L174-L235)
- 修改边界：`IsActive` 不验证目标有效性或地图安全；`IsBusy` 不是 Activate 的自动门禁。新增传送门还需动作、标签／预测、SG 与资源，挂组件不等于可交互。[状态判断](../../../raw/current/scripts/components/teleporter.lua#L43-L53)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `Target(other)`／`GetTarget()` | 记录／返回普通目标；nil 清除，不自动建立反向关系。[实现](../../../raw/current/scripts/components/teleporter.lua#L349-L363) |
| `SetEnabled(bool)`／`IsActive()` | 默认 enabled=true；有普通／临时／迁移目标才 active。普通目标、migration_data、enabled 变化维护 teleporter 标签；直接写临时目标不走该观察器。[初始化](../../../raw/current/scripts/components/teleporter.lua#L9-L32)、[判断](../../../raw/current/scripts/components/teleporter.lua#L43-L45) |
| `Activate(doer)` | inactive 返回 false；先调用 `onActivate(inst, doer, migration_data)`，迁移分支再检查世界是否可用；同分片移动与接收后转移符合条件的随从并返回 true。[实现](../../../raw/current/scripts/components/teleporter.lua#L96-L166) |
| `offset`／`trynooffset` | 使用接收端 offset，默认 2；trynooffset 先试中心点；普通陆／水生落点有不同查找，失败不保证取消。[实现](../../../raw/current/scripts/components/teleporter.lua#L174-L233) |
| `onActivateByOther(inst, source, doer)` | 接收端通知，人物与物品入口都会触发；不是发送端回调。[物品](../../../raw/current/scripts/components/teleporter.lua#L289-L300)、[人物](../../../raw/current/scripts/components/teleporter.lua#L325-L347) |
| `travelcameratime`／`travelarrivetime` | 默认 3／4 秒；人物镜头与到达任务，需保证时序合理；物品接收固定 3.5 秒，不读取这两个值。[默认](../../../raw/current/scripts/components/teleporter.lua#L21-L22)、[物品](../../../raw/current/scripts/components/teleporter.lua#L289-L300) |
| `RegisterTeleportee`／`UnregisterTeleportee`／`IsBusy` | 预登记等待对象与接收中计数共同决定 busy；Activate 不自动拒绝 busy。[实现](../../../raw/current/scripts/components/teleporter.lua#L47-L68) |
| `MigrationTarget(worldid,x,y,z)` | nil 清迁移数据；非 nil 时 Activate 优先走迁移，需其他且可用分片；不把游戏分片 ID 当 Steam build。[配置](../../../raw/current/scripts/components/teleporter.lua#L353-L359)、[迁移](../../../raw/current/scripts/components/teleporter.lua#L105-L113) |
| `UseTemporaryExit(doer, exit)` | 临时覆盖出口、调用 Activate 后清除；不改变永久配对，自身出口另缩短到达时间／停用渐变。[实现](../../../raw/current/scripts/components/teleporter.lua#L70-L82) |

## 生命周期

人物原版 SG 先预登记，在跳入动画结束时取消登记再 Activate；成功后还负责隐藏、无敌、输入关闭，并依赖到达状态收尾。直接从 idle 调 Activate 不能取代这条链。[SG 激活与隐藏](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15728-L15761)、[到达处理](../../../raw/current/scripts/components/teleporter.lua#L311-L347)

物品接收会挂到出口、移出场景，3.5 秒后脱离并落地；出口计数减少并发送 `doneteleporting`，可触发 OnDoneTeleporting。原版虫洞投料先 DropItem，避免把仍在物品栏的实体直接当落地输入。[物品处理](../../../raw/current/scripts/components/teleporter.lua#L244-L300)、[投料](../../../raw/current/scripts/prefabs/wormhole.lua#L73-L76)、[完成通知](../../../raw/current/scripts/components/teleporter.lua#L237-L242)

## 典型接入

### 原版虫洞到达时尝试在出口周围散开

目标模组 `modmain.lua`，将每个普通 wormhole 的接收偏移由原版 0 改为 2；保留原配对、人物 SG、声音与精神消耗，无新增资源。[原配置](../../../raw/current/scripts/prefabs/wormhole.lua#L163-L168)、[落点读取](../../../raw/current/scripts/components/teleporter.lua#L174-L211)、[Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wormhole", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local teleporter = inst.components.teleporter
    if teleporter ~= nil then
        teleporter.offset = 2
    end
end)
```

`2` 是地图距离，查找可能退到半径一半或中心，不保证固定两单位。设置入口 A 的 offset 影响**到达 A**，不只影响从 A 出发；原版物品延迟吐出的散落还会再处理坐标，本例不保证物品最终半径也是 2。仅改构造参数，停用模组后按原版重建。[物品散落](../../../raw/current/scripts/components/teleporter.lua#L244-L287)

## 常见陷阱与限制

- **IsActive／true 被当落地保证：** Teleport 可因水陆不匹配提前 return，但 Activate 不读取移动结果，仍可能继续接收并返回 true；自定义对象需在上层核验目标及水陆资格。[移动分支](../../../raw/current/scripts/components/teleporter.lua#L215-L233)、[调用顺序](../../../raw/current/scripts/components/teleporter.lua#L115-L125)
- **认为停用可中止已接收对象：** SetEnabled 只影响后续 active 判断，未取消已有任务；IsTargetBusy 还假定普通目标具备 teleporter，不用于任意坐标实体。[忙碌](../../../raw/current/scripts/components/teleporter.lua#L47-L53)、[人物任务](../../../raw/current/scripts/components/teleporter.lua#L325-L347)
- **把 enabled 当已存档开关：** OnSave 不保存 enabled；saveenabled 主要控制目标保存，migration_data 仅在有普通 target 的条件内写出，组件加载也未恢复 migration_data。迁移出口需宿主重新配置。[保存加载](../../../raw/current/scripts/components/teleporter.lua#L373-L417)
- **接收中物品存档有风险：** 当前代码写 `data.items[#data.items]`，空表首条写索引 0；OnLoad 用 pairs 恢复，不能据此保证标准数组序列化完整保留中途物品。本页不提供该场景的可靠性承诺，需游戏存档链验证。[实现](../../../raw/current/scripts/components/teleporter.lua#L373-L407)
- **随从不是全部容器递归：** 跳过 noleashing／被持有随从；额外检查人物主栏与已装备容器内物品的 leader，未递归所有嵌套容器。[过滤](../../../raw/current/scripts/components/teleporter.lua#L84-L94)、[遍历](../../../raw/current/scripts/components/teleporter.lua#L127-L163)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（SGwilson jumpin 激活段 +91，区间文本比对一致）；结论修订：无。vault_teleport 事件泛化／charliearena_teleport 新状态（SB-17 域 5/6）与 componentactions JUMPIN 的 vault→virtualroomteleporter 标签迁移均不在本页虫洞配对／Activate／落点结论范围内（teleporter.lua 本版未变）。
- 依据：player-hubs hunk 地图（SB-17）＋actions-ca 地图域 7＋变更清单命中 stategraphs/SGwilson.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖目标／启用、落点／随从、人物物品接收、保存实现与虫洞／人物 SG 的必要协作。未完整审校迁移／世界生成配对／SG 到达恢复及实际存档序列化。
- 结构与语法：本批执行格式、登记、链接／源码锚点检查及完整示例 Lua 5.1 语法检查；结果见本批交付报告。
- 游戏运行：未运行；多人拥挤、水陆／平台、跳入中断、接收中读档、出口移除、跨分片及专服待实测。
