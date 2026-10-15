---
title: "occupiable"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "管理鸟笼式单实体占用、取回与嵌套存档，区分动作门禁、占用回调和删除前释放。"
aliases: ["鸟笼装鸟", "单实体占用", "取出笼中鸟", "occupiable"]
tags: [component, building, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/occupiable.lua", "components/occupier.lua", "prefabs/birdcage.lua", "componentactions.lua", "actions.lua", "constants.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# occupiable

## 快速判断

- 职责与适用：占用物件一侧保存一个真实实体；原版鸟笼仅在主模拟安装，与被收纳实体的 [[reference/components/occupier]] 配对。[宿主](../../../raw/current/scripts/prefabs/birdcage.lua#L596-L611)
- 优先使用：先 `CanOccupy`，从原 inventory owner 移出后 `Occupy`；释放用 `Harvest`，由调用方决定放地或交给玩家。[STORE](../../../raw/current/scripts/actions.lua#L2593-L2597) [HARVEST](../../../raw/current/scripts/actions.lua#L2669-L2673)
- 重要副作用：占用时设置 persists、owner、父子关系并隐藏实体；隐藏会停 Brain／SG 与物理，不等于停止所有组件任务。[占用](../../../raw/current/scripts/components/occupiable.lua#L59-L98) [隐藏](../../../raw/current/scripts/entityscript.lua#L348-L375)
- 修改边界：不是格子 [[reference/components/container]]，也不是把入巢生物化成数量的 [[reference/components/childspawner]]；鸟笼动画、喂食、饥饿死亡产物由宿主回调完成。[鸟笼回调](../../../raw/current/scripts/prefabs/birdcage.lua#L323-L370) [死亡产物](../../../raw/current/scripts/prefabs/birdcage.lua#L424-L440)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `occupanttype`／`occupant` | 初值均 nil | 空位发布 `<类型>_occupiable` 标签，占用后改为 occupied；当前常量只有 `OCCUPANTTYPE.BIRD="bird"`。不要直接赋 occupant 代替完整流程。[标签](../../../raw/current/scripts/components/occupiable.lua#L1-L35) [常量](../../../raw/current/scripts/constants.lua#L2009-L2012) |
| `IsOccupied()`／`GetOccupant()` | boolean／实体或 nil | 只读当前引用；IsOccupied 不额外检查实体有效性。[实现](../../../raw/current/scripts/components/occupiable.lua#L44-L50) |
| `CanOccupy(occupier)` | boolean；要求非 nil 实体 | 空位、已设类型、实体有对应标签且有 occupier；不检查 inventoryitem、距离或原 owner。[实现](../../../raw/current/scripts/components/occupiable.lua#L52-L57) |
| `Occupy(occupier)` | 成功 true，否则 nil | 自身只查空位、非 nil、occupier 组件，**不调用 CanOccupy**；先设 owner、调用占用者回调，再调用宿主回调，之后才挂子体／隐藏。[实现](../../../raw/current/scripts/components/occupiable.lua#L59-L100) |
| `onoccupied`／`onemptied`／`onperishfn` | `(host, occupant)`／`(host)`／`(host, occupant)` | 无 setter，直接配置；宿主 onemptied 实际只收到 host，不能按鸟笼函数声明臆造第二实参。perished 路径先 emptied、再 perish、最后清引用并删除占用者。[回调](../../../raw/current/scripts/components/occupiable.lua#L68-L95) |
| `Harvest()` | inventoryitem 实体或 nil | 清 owner／引用、取消占用监听、解除父子、调 emptied，再 ReturnToScene；不自动 GiveItem、定位或 OnDropped，也不调用占用者 onoccupied(nil)。[实现](../../../raw/current/scripts/components/occupiable.lua#L104-L118) |

### 保存、空位恢复与删除

OnSave 把有效占用者的完整 GetSaveRecord 嵌入 occupant；OnLoad 经 SpawnSaveRecord 重建后直接 Occupy，因此重新触发两侧占用回调，且绕过 CanOccupy。类型与回调由宿主构造恢复，不由组件这份数据保存。[存读](../../../raw/current/scripts/components/occupiable.lua#L120-L134)

占用者发 onremove 会清另一项 perished 监听、调用 emptied、清 occupant；perished 路径还会脱离父子、返回场景并删除原实体。`OnRemoveFromEntity` 仅移标签，不 Harvest、不解除 owner，也不主动清占用者事件；移组件前应先完成释放。[事件](../../../raw/current/scripts/components/occupiable.lua#L75-L98) [移组件](../../../raw/current/scripts/components/occupiable.lua#L37-L42)

直接删除宿主会按实体通用规则删除其子体，不能指望鸟自动落地。原版锤毁鸟笼先 Harvest、设地面坐标并 OnDropped，再删除宿主；定制拆除也应保留这条路径。[实体删除](../../../raw/current/scripts/entityscript.lua#L1779-L1783) [鸟笼拆除](../../../raw/current/scripts/prefabs/birdcage.lua#L372-L387)

## 典型接入

### 原版鸟笼只允许新装入金丝雀，保留已有占用者

完整目标模组 `modmain.lua`；复用 birdcage／canary，无新增资源。在组件构造后包装门禁，检查时才限定 prefab；鸟笼取走死亡产物后会重建 occupiable，因此选 ComponentPostInit，避免只包装初始组件。[Hook](../../../raw/current/scripts/modutil.lua#L567-L573) [调用形参](../../../raw/current/scripts/entityscript.lua#L655-L665) [重建](../../../raw/current/scripts/prefabs/birdcage.lua#L479-L507)

```lua
AddComponentPostInit("occupiable", function(component)
    if not GLOBAL.TheWorld.ismastersim then return end
    local previous = component.CanOccupy
    component.CanOccupy = function(self, bird)
        if self.inst.prefab == "birdcage"
            and (bird == nil or bird.prefab ~= "canary") then
            return false
        end
        return previous(self, bird)
    end
end)
```

仅收紧正常 STORE 的服务端门禁；原占用、喂食、取出、死亡、存档逻辑保持原路径。读档中的其他鸟仍可恢复，因为 OnLoad 直接 Occupy；不驱逐旧鸟。原版菜单只看类型标签，所以其他鸟仍可能显示装入动作而执行失败；这不是完整的客户端动作过滤。[动作提示](../../../raw/current/scripts/componentactions.lua#L1516-L1533) [执行门禁](../../../raw/current/scripts/actions.lua#L2593-L2597)

停用模组并重载恢复原门禁，没有额外存档字段；其他模组直接调用 Occupy 仍能绕过此限制。将 canary 替换为其他 prefab 前核对其 bird 标签、occupier、inventoryitem 以及鸟笼动画支持，不能凭名字添加收纳能力。

## 常见陷阱与限制

- **只加类型标签就能收纳／取出**：动作提示按标签选择，实际还需 occupier；标准 STORE 访问 inventoryitem，而 Harvest 也要求它。自定义非物品占用者可能能 Occupy 却无法原路取回，应设计专用释放流程。[候选动作](../../../raw/current/scripts/componentactions.lua#L1516-L1533) [STORE](../../../raw/current/scripts/actions.lua#L2593-L2597) [Harvest](../../../raw/current/scripts/components/occupiable.lua#L104-L118)
- **在 emptied 中总能读到旧鸟**：Harvest 先清 occupant，perished／onremove 则在回调后清，且不传旧鸟；需要死亡实体用 onperishfn 参数，不把 emptied 当统一的“取出实体”回调。[顺序](../../../raw/current/scripts/components/occupiable.lua#L75-L118)
- **直接 RemoveComponent 当放生**：只清标签，旧子体／引用并未按 Harvest 释放；鸟笼原版仅在转到 shelf 内容状态时移组件，再在 shelf 物品取走后恢复配置。[组件清理](../../../raw/current/scripts/components/occupiable.lua#L37-L42) [宿主状态转换](../../../raw/current/scripts/prefabs/birdcage.lua#L479-L507)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua、constants.lua、entityscript.lua（更新报告 R-01/R-04/R-06/R-08）——STORE／HARVEST 动作、OCCUPANTTYPE 常量与实体删除子体逻辑均不在本轮变更 hunk 内，结论无涉；锚点重定位 6 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；完整审阅组件与 occupier，核对 STORE／HARVEST、类型常量、鸟笼注册／主模拟构造、回调、shelf 重建、存读与删除前释放、实体移组件／删除行为。未穷举其他模组或所有销毁工具。
- 结构检查：本页格式、来源文件、源码锚点与大小通过；完整 modmain 经 Full Moon Lua 5.1 syntax-only。导航与审校表由本批次统一登记。
- 游戏运行：未运行；需验证新笼／旧档其他鸟、金丝雀装入、拒绝其他鸟、死亡产物取走后的组件重建、锤毁与停用重载。

相关：[[reference/components/occupier]]；[[reference/components/inventoryitem]]；[[reference/components/trader]]。
