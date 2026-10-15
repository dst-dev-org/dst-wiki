---
title: "remoteteleporter"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "薇诺娜传送伞的目的台选择、附近物品转移与回调；资格、隐藏恢复和耗能依赖动作／SG／宿主。"
aliases: ["传送伞周边物品半径改为三", "薇诺娜传送台选择", "remoteteleporter"]
tags: [component, item, callback, actions, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/remoteteleporter.lua", "prefabs/winona_telebrella.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# remoteteleporter

## 快速判断

- 职责与适用：从当前世界 winonateleportpadmanager 登记台中选目的地，移动人物和调用方提供的物品集合；不是任意出口配对或跨分片迁移 API，配对入口见 [[reference/components/teleporter]]。[选台](../../../raw/current/scripts/components/remoteteleporter.lua#L90-L137)
- 优先使用：正常交互走 REMOTE_TELEPORT／原 SG；CanActivate 与 Teleport 分开，后者不会自动执行前者，也不自动调用开始／停止回调。[动作](../../../raw/current/scripts/actions.lua#L6032-L6044) [方法](../../../raw/current/scripts/components/remoteteleporter.lua#L139-L149)
- 重要副作用：选台优先距离阈值外最近，否则阈值内最远；outofcamera／incamera 只是变量名，依据是物理半径＋TUNING 距离平方，不读取实际相机视野。[算法](../../../raw/current/scripts/components/remoteteleporter.lua#L90-L130)
- 修改边界：原伞 CanActivate 检查技能、CheckDestination 检查台供电；OnTeleported 即使收到 success=false 仍按燃料值扣能，不能假定 NODEST 免费。开始／停止管理燃料消耗状态。[原回调](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L274-L303)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetCanActivateFn(fn)`／`CanActivate(doer)` | fn(item,doer) 返回资格／原因；未配置默认 true。Teleport 不自行调用，普通动作及 SG 前置分别检查。[方法](../../../raw/current/scripts/components/remoteteleporter.lua#L10-L12) [资格](../../../raw/current/scripts/components/remoteteleporter.lua#L34-L39) [SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24687-L24705) |
| `SetCheckDestinationFn(fn)` | fn(item,target,doer) 过滤管理器中的台；此前检查起终点传送许可，但不等同通用安全落点查找。[消费](../../../raw/current/scripts/components/remoteteleporter.lua#L101-L121) |
| `SetItemTeleportRadius(r)`／`Teleport_GetNearbyItems(doer)` | 以人物 x/z、y=0 搜 _inventoryitem，排除 INLIMBO／FX／NOCLICK／DECOR，再排 canbepickedup=false。r 未配置返回 nil；此方法只返回集合，不自动安装待传物品。[收集](../../../raw/current/scripts/components/remoteteleporter.lua#L30-L60) |
| `SetNearbyItems(items)` | 直接存集合；成功移动时内部取出并清字段，失败分支不自动清。物品相对人物出发点的水平偏移被保留，未逐个搜索可行落点。[移动](../../../raw/current/scripts/components/remoteteleporter.lua#L58-L88) [失败](../../../raw/current/scripts/components/remoteteleporter.lua#L133-L137) |
| `Teleport(doer)` | 成功 true，失败 false,NODEST；成功对人物／物品发 teleported，并给目的台发 remoteteleportreceived。需要人物 Physics，不能当任意实体通用迁移。[移动及事件](../../../raw/current/scripts/components/remoteteleporter.lua#L61-L88) [结果](../../../raw/current/scripts/components/remoteteleporter.lua#L123-L137) |
| 三个阶段回调 setter | 开始 fn(item,doer)，已执行 fn(item,doer,success,target,items,from_x,from_z)，停止 fn(item,doer,success)。失败的已执行回调仅传到 target=nil，不提供有效集合／坐标。[成功](../../../raw/current/scripts/components/remoteteleporter.lua#L82-L88) [失败／阶段](../../../raw/current/scripts/components/remoteteleporter.lua#L133-L149) |

### 原动作与物品恢复协作

库存候选检查 engineering／handyperson 和装备状态，SG 前置还要求物品在手槽。SG 的 out 阶段收集并移出附近物品，执行动作前 SetNearbyItems；in 阶段及退出再 ReturnToScene，并恢复人物控制、显示与回调。单调 Teleport 不包含这些步骤，组件没有通用移除／存档恢复协议。[候选](../../../raw/current/scripts/componentactions.lua#L2864-L2871) [手槽](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24687-L24705) [收集](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24760-L24773) [安装](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24786-L24811) [恢复](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24881-L24917) [退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24928-L24963)

## 典型接入

### 原传送伞附近物品收集半径设为三

目标模组 `modmain.lua`，无新资源；原 winona_telebrella 已配置技能／供电过滤、三阶段回调和 1.84 半径，仅把下一次收集使用的半径改为 3。[原挂载](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L431-L437)

```lua
AddPrefabPostInit("winona_telebrella", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local teleporter = inst.components.remoteteleporter
    if teleporter ~= nil then
        teleporter:SetItemTeleportRadius(3)
    end
end)
```

3 是世界距离，不是物品数量或台选择阈值；只影响 SG 下一次 Teleport_GetNearbyItems 的候选，不改变已收集集合。保留原工程技能、目的台供电、能量回调与 SG 恢复，不直接调 Teleport 来绕过动画。半径不由组件存档，原工厂与本 Hook 重建；停用并重载恢复原配置。[收集参数](../../../raw/current/scripts/components/remoteteleporter.lua#L43-L57) [原业务](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L274-L303)

## 常见陷阱与限制

- **直接 Teleport 会查技能并带走周围物品**：资格和物品集合要由调用链准备；本组件只消费已存集合，未自动收集。普通模组改原功能应保留动作／SG。[动作](../../../raw/current/scripts/actions.lua#L6032-L6044) [集合](../../../raw/current/scripts/components/remoteteleporter.lua#L58-L88)
- **找不到目的台就不触发耗能**：失败仍调用 onteleportedfn，原伞回调不按 success 分支；更换该策略需核验完整耗能与停止链，不能靠动作失败推断。[失败](../../../raw/current/scripts/components/remoteteleporter.lua#L133-L137) [原耗能](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L289-L303)
- **物品移动成功就完整恢复显示**：内部只移动和发事件，原先 RemoveFromScene 的对象仍靠 SG ReturnToScene；新调用方还需处理中断／失败、残留集合和每个偏移落点。[内部](../../../raw/current/scripts/components/remoteteleporter.lua#L61-L88) [原恢复](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24881-L24892)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、SGwilson.lua（SB-17）逐 hunk 对照，锚点重定位 10 处（REMOTE_TELEPORT 6013→6022、EQUIPPED 候选 2807→2852、SG pre／out／in 状态段整体 +166），结论修订 0 处——REMOTE_TELEPORT 动作与候选、SG 各状态段内容均逐块比对未变；本页无 JUMPIN 链结论，R-04 的 JUMPIN 排除标签 `vault_teleporter`→`virtualroomteleporter` 只作用于 SCENE.teleporter 处理器（P-123），不影响本页；R-05 重写 IsTeleportingPermittedFromPointToPoint，组件 ：103 处"起终点传送许可"检查仍成立，实现细节归 componentutil；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、伞的服务端安装与技能／供电／燃料回调、REMOTE_TELEPORT／两类候选和完整 pre／out／in 服务端 SG。未审校世界管理器登记、目的台全部逻辑、客户端预测、所有电路／充能或新传送系统。
- 结构检查：本批格式、来源、源码引用／行号、大小和登记通过；完整半径配置例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；台选择、NODEST／中断、物品偏移／恢复、技能／供电、燃料变化和主机／远端待实测。
