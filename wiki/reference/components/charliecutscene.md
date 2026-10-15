---
title: "charliecutscene"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "中庭之门过场编排器：按柱位定向镜头锁定，排布 Charlie 修门/嵌钥匙两段演出，锁梦魇相位、禁交易并驱动 ritual 仪式。"
aliases: ["中庭过场", "Charlie 修门演出", "charliecutscene 组件"]
tags: [component, network-sync, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/charliecutscene.lua", "prefabs/atrium_gate.lua", "prefabs/charlie_npc.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# charliecutscene

## 快速判断

- 职责与适用：仅挂在 atrium_gate 上（prefabs/atrium_gate.lua:1361），客户端只做相机联动（:205-207 早退），服务端排布整段演出。
- 优先使用：修门过场 → `Start()`/`Finish()`（:219-255）；嵌钥匙仪式 → `StartKeySocket()`/`FinishKeySocket()`（:454-490），由 atrium_gate/charlie_npc 流程调用。
- 重要副作用：`Start` 推世界事件 `charliecutscene`(true) 与 `ms_locknightmarephase("wild")`（:222-223），`Finish` 推 `shadowrift_opened`（:254，riftspawner.lua:37 消费开启暗影裂隙）；演出期间 `trader:Disable()`。
- 修改边界：镜头锁经 `net_bool _iscameralocked` 与 `net_ushortint _cameraangle` 同步（:166-167），客户端 `ClientLockCamera` 接管 TheCamera（:184-191）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Start`／`Finish` | `() -> nil` | 修门段：锁相、禁交易、生成 `charlie_npc` 演出、修门；`Finish` 由 charlie_npc.lua 调。:219-255 |
| `StartKeySocket`／`FinishKeySocket` | `() -> nil` | 嵌钥匙段：加 `intense` 标签、排 `SocketVaultKey` 与 ritual FX/EnableRitual 时序（:448-452）；结束清标签。:454-490 |
| `CollectAtriumPillarsData` | `() -> nil` | 50 范围找 `pillar_atrium`，按互相直角关系分 `back` 与两侧 `side`（:1-2、:100-126）；缓存 gate_pos/atrium_pillars（:259-266）。 |
| `FindSceneCameraAngle`／`FindRitualAngle` | `() -> number` | 由 back 柱方位角 +180 再 ±90/45 度偏移，角度归一（:268-284）；ritual 角也被 atrium_gate.lua:700 复用。 |
| `SpawnCharlieHand`／`SpawnCharlieHandKeyStone` | `() -> nil` | 生成 `charlie_hand`/`charlie_hand_keystone` 于三柱反演点（`side1+side2-back`，:389-423）；双向 entitytracker 跟踪。 |
| `RepairGate` | `() -> boolean` | 换 build 播 "fixed"、震屏、按激活/冷却态切小地图图标（:293-308）；debugcommands.lua:5267-5268 亦调用。 |
| `IsGateRepaired` | `() -> boolean` | atrium_gate.lua:203/:585/:649 消费（可刷性、生成判定）。:494-496 |
| `OnSave`/`OnLoad`/`LoadPostPass` | 常规存取 | 载入即跳过过场：running→`Finish`，runningkeysocket→直接嵌钥匙落定态（:498-544）。 |

## 关键流程：演出时序（写死常量）

- 修门段：`CHARLIE_SPAWN_DELAY`=2s 出 Charlie → `START_REPAIRING_GATE_DELAY`（3s+80帧）播 "fixing" → `REPAIR_GATE_DELAY` 完成修复；结束前 `TWEEN_TO_BLACK` 渐黑再 3.5s 还原（:14-32、:134-156）。
- 嵌钥匙段：`START_SOCKETING_GATE_DELAY` 触发 `SocketVaultKey`，`PLACE_RITUAL_CIRCLE_DELAY` 前 71 帧按 ritual 角放 3 处圆阵 FX（`NUM_RITUAL_MARKINGS`=3，与 atrium_gate.lua:753 注释同步），到点 `EnableRitual`+震屏（:427-452）。
- 客户端：`iscameralockeddirty` 后由焦点源（`StartFocusSource`，dist 12~25）驱动 `CharlieCam_UpdateFn`，进入范围锁相机、离开解锁（:39-76）；中途加入的客户端在 DoTaskInTime(0) 补判（:178-180）。

## 典型接入

### 查询/触发过场状态（mod 侧只读为宜）

- 前提与选择：仅中庭门实体有组件；判断演出用世界事件 `charliecutscene` 监听比轮询组件更稳。
- 生效与撤销：`Start`/`StartKeySocket` 依赖 pillar 实体存在（`back` 为 nil 会报错），不建议 mod 主动触发；加载中过场自动跳过并落定态。
- 原版依据：atrium_gate.lua:1361、:585-590（世界生成 rifts_enabled_cave="always" 时直接 RepairGate）。

```lua
AddPrefabPostInit("atrium_gate", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("charliecutscene", function(world, running)
        -- running: 过场开始/结束（修门与嵌钥匙两段共用同一事件）
    end, TheWorld)
end)
```

- 调整与验证：`charliecutscene` 事件数据为布尔；`ms_locknightmarephase` 由 nightmareclock.lua:215 消费锁 "wild"。

## 常见陷阱与限制

- 时序常量与动画帧强绑定：`REPAIR_GATE_ANIM_LENGTH`=114帧、`SOCKET_ANIM_LENGTH`=132帧（:14-15），改演出节奏需整体重排。
- 柱位数据缓存不刷新：`CollectAtriumPillarsData` 首次收集后不再重算（:259-262），柱子被动过再触发会取旧数据。
- 交易禁用不落存档：`trader.enabled` 恢复仅在 `Finish`/`FinishKeySocket` 执行（:246-249 注释），演出中停服由 OnLoad 的跳过路径兜底。
- `LoadPostPass` 直接操作 `inst._runningkeysocket`（:534），atrium_gate.lua:589 依赖该字段——两文件强耦合。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 547 行；挂载 atrium_gate.lua:1361，消费方 atrium_gate.lua:203/:585-590/:649/:700/:753、charlie_npc.lua:43-45、debugcommands.lua:5267-5268、nightmareclock.lua:215、riftspawner.lua:37 逐行核对。
- 游戏运行：未运行。
