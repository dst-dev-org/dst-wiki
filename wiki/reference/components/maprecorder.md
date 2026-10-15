---
title: "maprecorder"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "记录玩家已探索地图到卷轴，检查世界会话并让另一玩家学习，管理读取消耗与地图元数据。"
aliases: ["复制玩家地图", "地图卷轴", "分享已探索地图", "maprecorder"]
tags: [component, item, actions, server-client, data]
dst_build: "steam-buildid:26230496"
sources: ["components/maprecorder.lua", "prefabs/mapscroll.lua", "prefabs/player_classified.lua", "actions.lua", "componentactions.lua", "prefabs/wagstaff_npc.lua", "prefabs/papyrus.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# maprecorder

## 快速判断

- 职责与适用：挂在承载地图的物品上，调用玩家 `player_classified.MapExplorer` 录制／学习探索数据；不是按坐标揭图或投递物品的组件。[对象与录制](../../../raw/current/scripts/components/maprecorder.lua#L13-L65)
- 优先使用：`RecordMap(player)` 录制，`TeachMap(player)` 学习；原版 `mapscroll` 在建造时录制，读取走 `TEACH`。[卷轴回调](../../../raw/current/scripts/prefabs/mapscroll.lua#L8-L14) [动作](../../../raw/current/scripts/actions.lua#L3260-L3272)
- 重要副作用：成功学习先执行 `onteachfn(inst, learner)`，再删除整件地图物品；空地图也会删除。不能靠教学回调返回 false 保留卷轴。[学习与消耗](../../../raw/current/scripts/components/maprecorder.lua#L68-L89)
- 修改边界：原版服务端挂载；地图属于 `session_identifier`，不能仅以地表／洞穴名称判定能否学习。`MapExplorer` 是实体引擎能力，本页不推断二进制地图格式或引擎合并细节。[端边界](../../../raw/current/scripts/prefabs/mapscroll.lua#L64-L100) [会话判断](../../../raw/current/scripts/components/maprecorder.lua#L26-L31) [引擎对象](../../../raw/current/scripts/prefabs/player_classified.lua#L1349-L1356)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `RecordMap(target)` | true；或 false, `"NOEXPLORER"`／`"BLANK"` | 没有 MapExplorer 时直接失败、不清旧数据；录制后保存会话、世界名、`target.name`、`cycles + 1`，空结果走 `ClearMap()`。[实现](../../../raw/current/scripts/components/maprecorder.lua#L45-L65) |
| `HasData()`／`IsCurrentWorld()` | boolean | 分别检查 mapdata 非空、mapsession 与当前世界会话相同；有数据不代表当前世界可读。[实现](../../../raw/current/scripts/components/maprecorder.lua#L26-L31) |
| `TeachMap(target)` | true；或 false, reason? | 空数据删除并返回 `BLANK`；异会话返回 `WRONGWORLD`；无探索器返回 `NOEXPLORER`；引擎拒绝学习只返回 false。这三种非空失败分支不自行删除物品。[实现](../../../raw/current/scripts/components/maprecorder.lua#L68-L89) |
| `ClearMap()` | 无返回 | 清数据、会话、作者、日期，保留 `maplocation`，总会调用已设置的数据变化回调。[实现](../../../raw/current/scripts/components/maprecorder.lua#L34-L41) |
| `SetOnDataChangedFn(fn)`／`SetOnTeachFn(fn)` | `fn(inst)`／`fn(inst, learner)` | setter 替换单一回调。数据回调供卷轴更新描述与图片；教学回调在物品删除前执行。[setter](../../../raw/current/scripts/components/maprecorder.lua#L18-L24) [原版呈现](../../../raw/current/scripts/prefabs/mapscroll.lua#L16-L38) |
| `TransferComponent(newinst)` | 目标须已有 maprecorder | 复制五个地图字段，不移除／清空来源，不复制回调；仅目标 `HasData()` 时通知目标回调。[实现](../../../raw/current/scripts/components/maprecorder.lua#L106-L115) |
| `IsTileSeeableInRecordedMap(target, tx, ty)` | boolean／引擎返回 | 空数据或无探索器为 false；有数据直接委托引擎，不另做当前会话检查。源码注明调用昂贵，多点需求不应机械逐点循环。[实现与注释](../../../raw/current/scripts/components/maprecorder.lua#L92-L103) |

保存这五个地图字段；加载时 `mapdata`、`mapsession` 同时存在才恢复完整记录，否则只在世界名变化时更新 `maplocation`。满足上述任一更新才通知数据回调；加载不会自动学习、删除或校正旧会话。直接给字段赋值也不会自动通知。[存读](../../../raw/current/scripts/components/maprecorder.lua#L120-L145)

## 典型接入

### 成功读取普通地图后返还一张莎草纸

目标模组 `modmain.lua` 的完整入口如下。沿用原版 `mapscroll` 与 `papyrus` 资源，只扩展成功教学回调并保留原版 `learnmap` 事件；不改变录制、世界验证、学习结果或卷轴消耗。[原版教学](../../../raw/current/scripts/prefabs/mapscroll.lua#L8-L14) [挂载](../../../raw/current/scripts/prefabs/mapscroll.lua#L64-L88) [莎草纸注册](../../../raw/current/scripts/prefabs/papyrus.lua#L32-L51)

```lua
AddPrefabPostInit("mapscroll", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    local recorder = inst.components.maprecorder
    if recorder == nil then
        return
    end
    local oldteach = recorder.onteachfn
    recorder:SetOnTeachFn(function(item, learner)
        if oldteach ~= nil then
            oldteach(item, learner)
        end
        local inventory = learner.components.inventory
        if inventory ~= nil then
            local paper = GLOBAL.SpawnPrefab("papyrus")
            if paper ~= nil then
                inventory:GiveItem(paper, nil, learner:GetPosition())
            end
        end
    end)
end)
```

只有 `LearnRecordedMap` 返回成功才返纸；空地图虽然被删除，也不会调用教学回调。`GiveItem` 的满栏、合并与掉落边界复用 [[reference/components/inventory#关键接口]]，不保证物品最终留在物品栏。该代码不依赖 GLOBAL 代理，显式使用 `GLOBAL.TheWorld`／`GLOBAL.SpawnPrefab`。

只命中注册名 `mapscroll`；另一个 `mapscroll_tricker` 共用组件工厂，但需独立 PostInit 才会应用。两种地图的注册与服务端分支见 [工厂](../../../raw/current/scripts/prefabs/mapscroll.lua#L91-L118)。去掉本段代码后，新建／重新加载的实例恢复原版回调；返纸结果已进入世界，不自动收回。

原版另一模式是瓦格斯塔夫将卷轴转成 `mapscroll_tricker`：先生成目标、`TransferComponent`，再显式移除旧卷轴并交付目标。组件名虽叫 Transfer，也不替调用者完成实体替换。[完整调用顺序](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1224-L1241) 擦除回收物品走 [[reference/components/erasablepaper]]，与成功教学返纸的触发不同。

## 常见陷阱与限制

- **把空白读取失败当作保留物品：** `TeachMap` 已删除空地图，而 `ACTIONS.TEACH` 将 `BLANK` 转为动作成功；直接调用组件与读取动作的返回语义不同。不要失败后再操作已删除卷轴。[组件](../../../raw/current/scripts/components/maprecorder.lua#L68-L73) [动作转换](../../../raw/current/scripts/actions.lua#L3267-L3272)
- **同为地表就认为能学习：** `maplocation` 用于描述／图片，实际门禁是会话 ID；从其他世界取得的非空地图仍可返回 `WRONGWORLD`。[会话判断](../../../raw/current/scripts/components/maprecorder.lua#L30-L31) [卷轴呈现](../../../raw/current/scripts/prefabs/mapscroll.lua#L19-L37)
- **在普通生物上调用录制：** 获取探索器只认 `target.player_classified.MapExplorer`；给生物加同名 Lua 组件不能满足这个条件。[获取路径](../../../raw/current/scripts/components/maprecorder.lua#L13-L15)
- **期待数据变化自动同步或触发：** 组件没有独立 netvar，也不监听任意字段写入；原版靠回调更新可检查描述与物品图片。修改元数据时要明确刷新入口，不能把 Lua 字段视为客户端数据。[变化回调](../../../raw/current/scripts/components/maprecorder.lua#L22-L24) [消费者](../../../raw/current/scripts/prefabs/mapscroll.lua#L16-L38)
- **客户端完整挂载组件：** 原版只在服务端挂组件；动作发现通过组件动作登记，使用物品时目标必须是操作者本人且为玩家，库存直接读也检查玩家标签。[动作发现](../../../raw/current/scripts/componentactions.lua#L1486-L1489) [库存读取](../../../raw/current/scripts/componentactions.lua#L2790-L2793) 不需在客户端复制整个地图记录，网络物品构造复用 [[reference/components/inventoryitem]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua、prefabs/player_classified.lua，更新报告 R-01/R-04）逐条核验，TEACH 动作与 maprecorder 动作采集无语义变化（player_classified 变更仅为虫怪吞咽音效 netvar），锚点重定位 4 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整组件、玩家 MapExplorer 挂载、两种卷轴工厂及回调、动作发现／执行、转移实例与返纸 Prefab；物品交付复用同版本 inventory 页。覆盖上述返回、存读、清空／转移通知和示例作用域。
- 游戏运行：未运行；引擎录制／学习数据结构、跨会话实际表现、满栏返纸及其他模组回调覆盖未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；完整 `modmain.lua` 示例通过 Lua 5.1 语法检查，未执行。
