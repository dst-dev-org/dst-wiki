---
title: "teleportedoverride"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "向紫法杖传送流程提供目标实体或落点回调，不拦截所有传送方式。"
aliases: ["传送落点覆盖", "紫法杖目的地", "被传送目标", "teleportedoverride"]
tags: [component, callback, modding]
dst_build: "steam-buildid:26230496"
sources: ["components/teleportedoverride.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/wagstaff_npc.lua", "prefabs/pigman.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# teleportedoverride

## 快速判断

- 职责与适用：挂在被传送实体上，保存两个回调供消费者询问；组件自己不发起传送、不加标签、不做地图验证。[完整接口](../../../raw/current/scripts/components/teleportedoverride.lua#L3-L21)
- 优先使用：本页已核验的紫法杖路径用 `SetDestPositionFn` 返回位置，或 `SetDestTargetFn` 返回目的实体。[消费顺序](#关键接口)
- 重要副作用：位置回调优先于目的实体的坐标，但目的实体仍可能收到 `onteleto`；两个回调不是互斥开关。[坐标选择](../../../raw/current/scripts/prefabs/staff_common.lua#L348-L362)、[目标通知](../../../raw/current/scripts/prefabs/staff_common.lua#L279-L282)
- 修改边界：不覆盖虫洞 [[reference/components/teleporter]]、倒走表 [[reference/components/positionalwarp]] 或任意 Physics:Teleport；本页结论限已核验消费者。

## 关键接口

| 接口 | 回调约定及证据 |
|---|---|
| `SetDestTargetFn(fn)`／`GetDestTarget()` | 回调 `fn(inst)`，返回目的实体或 nil；无函数时 nil，无返回类型／有效性保护。[实现](../../../raw/current/scripts/components/teleportedoverride.lua#L7-L13) |
| `SetDestPositionFn(fn)`／`GetDestPosition()` | 回调 `fn(inst)`，返回具有 x/z 的位置对象或 nil；getter 的 `and/or` 也把 false 转 nil。[实现](../../../raw/current/scripts/components/teleportedoverride.lua#L15-L21) |

紫法杖先检查 `noteleport`／当前位置链接许可；允许时，目的实体优先小游戏，再 override，再拴住的对象，未选中且非水生目标才找紫宝石底座。随后位置优先 override，否则随机点／目的实体 teletopos／GetPosition。[目标选择](../../../raw/current/scripts/prefabs/staff_common.lua#L427-L445)、[位置选择](../../../raw/current/scripts/prefabs/staff_common.lua#L348-L362)

回调在这个路径中只收到被传送实体，不能假定带施法者、法杖或候选位置。返回 nil 表示交回后续默认选择，**不等于拒绝传送**。原版瓦格斯塔夫 NPC 返回附近可行走位置，找不到则返回自身位置。[原版回调](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L396-L405)

## 典型接入

### 猪人被紫法杖命中时保持当前位置

目标模组 `modmain.lua`，服务端为 `pigman` 增加目的地回调；不改美术／动作，不用于通用免疫。若其他模组已有组件，本例保留其策略，不覆盖。[组件](../../../raw/current/scripts/components/teleportedoverride.lua#L15-L21)、[猪人工厂](../../../raw/current/scripts/prefabs/pigman.lua#L858)、[Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("pigman", function(inst)
    if not GLOBAL.TheWorld.ismastersim
        or inst.components.teleportedoverride ~= nil then
        return
    end
    inst:AddComponent("teleportedoverride")
    inst.components.teleportedoverride:SetDestPositionFn(function(target)
        return target:GetPosition()
    end)
end)
```

这是“最终选择当前坐标”，不是取消整次法术：原版仍有法杖消耗、洞穴小地震／地表闪电、隐藏与恢复等流程。原实现最终使用返回位置的 x/z，y 固定 0；不适用于任意平台／海上垂直位置需求。[消耗与洞穴](../../../raw/current/scripts/prefabs/staff_common.lua#L364-L374)、[位置执行](../../../raw/current/scripts/prefabs/staff_common.lua#L331-L346)、[结束副作用](../../../raw/current/scripts/prefabs/staff_common.lua#L279-L329)

## 常见陷阱与限制

- **返回布尔值或三个数字：** getter 不组装向量；紫法杖按 `locpos.x/z` 消费，需返回位置对象。[消费](../../../raw/current/scripts/prefabs/staff_common.lua#L331-L346)
- **把 nil 当取消：** nil 会继续原版选点；需要门禁应核验具体法术资格／阻止路径，不能伪造无效位置触发错误。[回退链](../../../raw/current/scripts/prefabs/staff_common.lua#L348-L362)
- **目的目标也被回调删除：** 原版选择时未统一检查返回实体有效性，且可能访问 teletopos／GetPosition；回调应返回真实可用实体，不在选点中安排破坏性操作。[选择与访问](../../../raw/current/scripts/prefabs/staff_common.lua#L348-L362)
- **配置不自动保存：** 组件没有 OnSave／OnLoad，回调应在每次构造重新设置；闭包中的运行时目标需另做关系保存，可参考 [[reference/components/entitytracker]]。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 6 处并迁移证据文件：P-112 重构后紫法杖传送逻辑整体迁至 prefabs/staff_common.lua，六处锚点按同源代码重定位（teleport_end/continue/start/func，行号 279-445）；结论修订：无——teleportedoverride 目标／位置优先级链两版逐字一致，新增的 rechargeable 消耗与 notlightningteleport 分支为王杖（king_cane）共用路径，telestaff 仍走 finiteuses＋闪电；prefabs/staff_common.lua 已加入 sources。
- 依据：报告 P-112（detail-prefabs.md:174）＋变更清单命中 prefabs/staff.lua、prefabs/staff_common.lua、prefabs/pigman.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；完整审阅四个薄接口，核验紫法杖目标／位置优先级、消耗与结束副作用及原版 NPC 回调。未枚举所有传送实现或审校法术完整资格链。
- 结构与语法：本批执行格式、登记、链接／源码锚点检查及完整示例 Lua 5.1 语法检查；结果见本批交付报告。
- 游戏运行：未运行；猪人命中、洞穴／地表、海上平台、目标消失、多人及其他模组策略冲突待实测。
