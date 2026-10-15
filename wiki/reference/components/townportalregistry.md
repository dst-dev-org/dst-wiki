---
title: "townportalregistry"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级传送门登记器：跟踪所有城镇传送门与当前激活者，广播 linktownportals 配对事件，并按 WagPunk 竞技场屏障启停。"
aliases: ["传送门登记器", "townportalregistry", "城镇传送门", "linktownportals"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/townportalregistry.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/townportal.lua", "prefabs/townportaltalisman.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# townportalregistry

## 快速判断

- 职责与适用：森林与洞穴世界都安装的主模拟组件（构造即断言）；自己不传送，只维护 `_townportals` 列表、当前激活者 `_activetownportal` 与配对集合，向传送门实体广播 `linktownportals` 事件。[双世界安装](../../../raw/current/scripts/prefabs/cave.lua#L533) [断言](../../../raw/current/scripts/components/townportalregistry.lua#L7)
- 优先使用：传送门实体监听 `linktownportals`（参数为配对目标或 nil）；注册走 `ms_registertownportal` 世界事件（原版 townportal 与 townportaltalisman 都推）。[结构注册](../../../raw/current/scripts/prefabs/townportal.lua#L233) [护符注册](../../../raw/current/scripts/prefabs/townportaltalisman.lua#L188)
- 重要副作用：激活／停用事件（`townportalactivated`/`townportaldeactivated`，由 townportal 推）触发全表重新配对；WagPunk 屏障激活或玩家进出时复查，并对场内传送门关停 `channelable`。[激活源](../../../raw/current/scripts/prefabs/townportal.lua#L50) [复查](../../../raw/current/scripts/components/townportalregistry.lua#L99-L135)
- 修改边界：无公开方法、无存读钩子（配对关系每次激活即时重建）；链接许可委托 `IsTeleportLinkingPermittedFromPoint` 等全局判定。[许可](../../../raw/current/scripts/componentutil.lua#L1450)

## 关键流程

激活：`townportalactivated` 且当前无激活者、激活点允许链接 → 记为 `_activetownportal`；对每个其他传送门按"持有实体（inventoryitem 大归主）位置"双向许可判定，允许则标记 `linkedportals` 并推 `linktownportals, target`。停用：仅处理当前激活者，向所有已链接者推 `linktownportals`（无参）；移除已激活传送门等价停用。[激活](../../../raw/current/scripts/components/townportalregistry.lua#L30-L48) [停用](../../../raw/current/scripts/components/townportalregistry.lua#L50-L62) [移除](../../../raw/current/scripts/components/townportalregistry.lua#L64-L77)

WagPunk 竞技场：屏障激活事件或玩家进出（0 帧去抖）触发 `RecheckPortals`——激活者入场地则解除自身链接；场内其他传送门解除链接并 `channelable:SetEnabled(false)`，场外未链接者补链并启用；屏障关闭侧按同样规则恢复。[复查](../../../raw/current/scripts/components/townportalregistry.lua#L99-L135) [去抖](../../../raw/current/scripts/components/townportalregistry.lua#L147-L152)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `linktownportals` 实体事件 | 参数 `配对目标实体或 nil` | 组件广播的唯一协议；结构传送门与护符各自监听并改外观/行为。[结构监听](../../../raw/current/scripts/prefabs/townportal.lua#L231) [护符监听](../../../raw/current/scripts/prefabs/townportaltalisman.lua#L185) |
| `ms_registertownportal` 世界事件 | 参数 `传送门实体` | 登记去重后监听其 onremove；激活者已存在时对新登记者立即判定补链。[处理](../../../raw/current/scripts/components/townportalregistry.lua#L79-L97) |
| `townportalactivated`／`townportaldeactivated` 世界事件 | 参数 `传送门实体` | 由 townportal 激活/熄火时推；登记器据此建对/拆对。[推送](../../../raw/current/scripts/prefabs/townportal.lua#L50) |
| `GetDebugString()` | 调试串 | 显示登记数与激活态。[实现](../../../raw/current/scripts/components/townportalregistry.lua#L190-L196) |

## 典型接入

### 自定义传送门参与配对

- 前提与选择：主模拟端注册实体并监听 `linktownportals` 即可被纳入激活/停用广播；可传送性由全局许可函数决定，不在本组件配置。[许可判定](../../../raw/current/scripts/componentutil.lua#L1450)
- 生效与撤销：登记在 onremove 时自动清理；配对只在"激活者存在"期间有效。[清理](../../../raw/current/scripts/components/townportalregistry.lua#L64-L77)

```lua
-- 服务端，自定义传送门
AddPrefabPostInit("my_portal", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("linktownportals", function(inst, target)
        inst._linked_portal = target
    end)
    inst:DoTaskInTime(0, function()
        TheWorld:PushEvent("ms_registertownportal", inst)
    end)
end)
```

- 调整与验证：注册用 0 帧延迟避免世界组件未就绪；要参与"激活者"竞争还需自己推 `townportalactivated`（原版结构传送门的激活协议见 townportal.lua）。[激活推送](../../../raw/current/scripts/prefabs/townportal.lua#L50)

## 常见陷阱与限制

- **激活者是单槽：** 已有激活者时第二个 `townportalactivated` 被直接忽略（不会重配对）；要换激活者必须先停用前者。[门禁](../../../raw/current/scripts/components/townportalregistry.lua#L32-L34)
- **可携带传送门按持有者位置判定：** `inventoryitem:GetGrandOwner()` 决定参与判定的坐标，装在包里 vs 放在地上结果不同。[取位](../../../raw/current/scripts/components/townportalregistry.lua#L37-L38)
- **竞技场规则写死在复查里：** `IsPointInWagPunkArena`/`IsWagPunkArenaBarrierUp` 是 Map 方法，非竞技场世界这些分支恒不触发，但 `ms_wagpunk_barrier_*` 监听仍存在。[监听](../../../raw/current/scripts/components/townportalregistry.lua#L162-L164)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（202 行），核验双世界安装点、townportal/townportaltalisman 注册与广播协议、componentutil 许可函数。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/channelable]]
