---
title: "vaultorbteleporter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "精炼宝库球的地图传送器：登记缓冲地图动作、地图选点后经 Activate 校验目的地组件并回调执行传送。"
aliases: ["vaultorbteleporter", "宝库球传送器", "宝库球地图传送"]
tags: [component, item, actions]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/vaultorbteleporter.lua", "prefabs/vault_orb_refined.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vaultorbteleporter

## 快速判断

- 职责与适用：挂在物品（精炼宝库球 `vault_orb_refined`）上的两段式传送器：先 `StartMapAction` 生成 `bufferedmapaction` 进入地图选点，地图确认后动作链调用 `Activate` 执行。[启动选点](../../../raw/current/scripts/components/vaultorbteleporter.lua#L43-L58)
- 优先使用：动作生成与执行全部走 `ACTIONS.STARTVAULTORBTELEPORT` / `ACTIONS.VAULTORBTELEPORT_MAP`（右键 "Focus On" → 地图 "Portate"），由 `componentactions.lua:3092-3100` 按组件有无 bufferedmapaction 自动二选一；模组只需配置三个回调。[动作分派](../../../raw/current/scripts/componentactions.lua#L3092-L3100)
- 重要副作用：传送目的地必须是带 [[reference/components/vaultorbteleportdestination]] 的实体，缺失直接失败 NOTARGET；选点阶段给玩家挂 `vaultorbteleportdestinationtracker` 标签（决定地图图标可见性），取消时移除。[目的地门禁](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L70)
- 修改边界：组件自身不移动玩家——`Activate` 只是回调 `onactivatefn`，实际传送在宿主回调里（`vault_orb_refined.lua:40-47` 经 `vault_teleport` 事件进入虚拟房间）；实体/组件移除时清理缓冲动作。[回调宿主](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L40-L47)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnActivateFn(fn)` | `fn(inst, doer, target) -> success, reason` | Activate 时的最终校验与执行；返回非真则失败并透传 reason。[调用点](../../../raw/current/scripts/components/vaultorbteleporter.lua#L72-L77) |
| `SetOnStartMapActionFn(fn)` | `fn(inst, doer) -> success, reason` | 进入地图选点前的许可检查；失败则不生成 bufferedmapaction。[调用点](../../../raw/current/scripts/components/vaultorbteleporter.lua#L47-L52) |
| `SetOnCancelMapActionFn(fn)` | `fn(inst, doer)` | 缓冲动作被移除（取消/实体移除）时回调；注意 `OnRemoveEntity` 里先 Remove 再回调的顺序。[实现](../../../raw/current/scripts/components/vaultorbteleporter.lua#L10-L25) |
| `StartMapAction(doer)` | `-> success, reason` | 已有缓冲动作则拒绝；成功时生成 `bufferedmapaction` 并 `SetupMapAction(ACTIONS.VAULTORBTELEPORT_MAP, inst, doer)`。[实现](../../../raw/current/scripts/components/vaultorbteleporter.lua#L43-L58) |
| `CancelMapAction()` | `-> nil` | 移除缓冲动作但不触发取消回调（区别于 OnRemoveEntity 路径）。[实现](../../../raw/current/scripts/components/vaultorbteleporter.lua#L60-L65) |
| `Activate(doer, target)` | `-> success, reason` | 目的地无 vaultorbteleportdestination 即 `false, "NOTARGET"`；通过后调 onactivatefn。[实现](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L80) |
| `bufferedmapaction` | entity 或 nil | 选点期间的缓冲动作实体；componentactions 据此在地图上生成 VAULTORBTELEPORT_MAP。[读取点](../../../raw/current/scripts/componentactions.lua#L3094-L3097) |

## 典型接入

### 地图选点传送全链（原版参照）

- 前提与选择：宿主物品需 `inventoryitem`（地图动作要求）并配齐三个回调；目的地块由 telebase 按宝石挂 vaultorbteleportdestination（见该组件页）。[宿主配置](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L97-L100)
- 生效与撤销：右键物品 → StartMapAction（玩家获 tracker 标签）→ 地图点选有效目的地 → VAULTORBTELEPORT_MAP.fn 调 Activate → 宿主回调推 `vault_teleport`；选点中停手/打断（`stopcontinuousaction`）会调 CancelMapAction，取消回调摘标签。[取消链](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L61-L63)

```lua
-- 服务端：动作校验与执行的关键约定（vault_orb_refined.lua 精简参照）
inst:AddComponent("vaultorbteleporter")
inst.components.vaultorbteleporter:SetOnStartMapActionFn(function(inst, doer)
    doer:AddTag("vaultorbteleportdestinationtracker")
    return true
end)
inst.components.vaultorbteleporter:SetOnCancelMapActionFn(function(inst, doer)
    if doer then doer:RemoveTag("vaultorbteleportdestinationtracker") end
end)
inst.components.vaultorbteleporter:SetOnActivateFn(function(inst, doer, target)
    DoTeleport(inst, doer, target)  -- 实际位移在这里
    return true
end)
```

- 调整与验证：地图端有效落点由 `ACTIONS.VAULTORBTELEPORT_MAP.maponly_checkvalidpos_fn` 决定（16 距离内找 `vaultorbteleportdestinationtrackericon` 标签实体，`actions.lua:7008-7033`）；改探测半径应动 `TUNING.VAULT_ORB_REFINED_DETECTION_RADIUS`。

## 常见陷阱与限制

- **目的地忘挂 vaultorbteleportdestination：** 选点阶段地图图标可查（按图标标签找实体），但 Activate 门禁查的是目的地组件本身，缺组件时传送失败 NOTARGET。[门禁](../../../raw/current/scripts/components/vaultorbteleporter.lua#L68-L70)
- **CancelMapAction 与取消回调不等价：** 直接调 `CancelMapAction` 只删缓冲动作，`oncancelmapactionfn` 不会触发（tracker 标签会残留）；需要完整取消走实体移除路径或自行补回调。[差异](../../../raw/current/scripts/components/vaultorbteleporter.lua#L60-L65)
- **把组件当位移工具：** 组件不含任何传送实现；没有 `onactivatefn` 时 Activate 恒返回 true 但什么也不发生。[回调可选](../../../raw/current/scripts/components/vaultorbteleporter.lua#L72-L79)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行），核对 vault_orb_refined 三回调与取消链、actions.lua:7002-7060 两动作实现与选点校验、componentactions.lua:3092-3100 分派；目的地侧证据另见 [[reference/components/vaultorbteleportdestination]]。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
