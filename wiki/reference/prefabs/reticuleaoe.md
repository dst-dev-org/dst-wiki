---
title: "瞄准圈、准星与 AoE 目标指示特效（reticule 家族）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "八个 reticule 系 prefab 文件注册的 48 个瞄准圈、准星与落点提示：37 个非联网本地 FX 与 11 个联网 target FX 的工厂、变体、生成点和清理边界。"
aliases: ["reticule", "reticuleaoe", "reticuleaoesmall", "reticuleaoesummon", "reticuleaoe_1_6", "reticuleaoe_1d2_12", "reticuleaoe5line", "reticuleaoecatapultvolley", "reticuleaoecatapultelementalvolley", "reticuleaoecatapultwakeup", "reticuleaoeping", "reticuleaoesmallping", "reticuleaoesummonping", "reticuleaoeping_1_6", "reticuleaoeping_1d2_12", "reticuleaoeping5line", "reticuleaoefiretarget_1ping", "reticuleaoeghosttarget_ping", "reticuleaoecatapultvolleyping", "reticuleaoecatapultwakeupping", "reticuleaoewinonaengineeringping", "reticuleaoehostiletarget", "reticuleaoefriendlytarget", "reticuleaoecctarget", "reticuleaoesmallhostiletarget", "reticuleaoesummontarget", "reticuleaoesummontarget_1", "reticuleaoesummontarget_1d2", "reticuleaoeshadowtarget_6", "reticuleaoefiretarget_1", "reticuleaoeghosttarget", "reticuleaoehostiletarget_1d25", "reticulearc", "reticulearcping", "reticulecharging", "reticuleline", "reticuleline2", "reticulelineping", "reticulelong", "reticulelongping", "reticulelongmulti", "reticulelongmultiping", "reticulemultitarget", "reticulemultitargetsub", "reticulemultitargetping", "golfclub_reticule", "golfclub_reticule_fx", "golfclub_reticulecharging", "golfclub_reticuleping", "reticule 家族", "瞄准圈", "准星", "AoE 目标指示", "落点提示", "蓄力瞄准环"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/reticule.lua", "prefabs/reticuleaoe.lua", "prefabs/reticulearc.lua", "prefabs/reticulecharging.lua", "prefabs/reticuleline.lua", "prefabs/reticulelong.lua", "prefabs/reticulemultitarget.lua", "prefabs/golfclub_reticule.lua", "components/reticule.lua", "components/aoetargeting.lua", "components/aoecharging.lua", "components/chargingreticule.lua", "components/golfclub_reticule.lua", "components/playercontroller.lua", "stategraphs/SGwilson.lua", "prefabs/books_lavaarena.lua", "prefabs/winona_remote.lua", "prefabs/waxwelljournal.lua", "prefabs/willow_ember.lua", "prefabs/shadow_trap.lua", "prefabs/slingshot.lua", "prefabs/fireballstaff.lua", "prefabs/ghostcommand_defs.lua", "prefabs/cannon_aoe_range_fx.lua", "components/deployhelper.lua", "prefabs/hammer_mjolnir.lua", "prefabs/sleepbomb.lua", "prefabs/healingstaff.lua", "prefabs/lavaarena_firebomb.lua", "prefabs/spear_lance.lua", "prefabs/lavaarena_heavyblade.lua", "prefabs/wathgrithr_shield.lua", "prefabs/spear_gungnir.lua", "prefabs/spear_wathgrithr.lua", "prefabs/blowdart_lava.lua", "prefabs/blowdart_lava2.lua", "prefabs/lavaarena_lucy.lua", "prefabs/woodie.lua", "prefabs/walter.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 瞄准圈、准星与 AoE 目标指示特效（reticule 家族）

## 快速判断

- 职责与适用：覆盖 `prefabs/` 下 8 个文件的 **48 个实际注册名**，分三类：持续瞄准环（`reticule`／`reticuleaoe`／`reticulearc`／`reticuleline`／`reticulelong` 及变体）、一次性扩散 `ping`、联网落点 `target` FX；另含蓄力方向环与高尔夫专用环。[基础工厂](../../../raw/current/scripts/prefabs/reticule.lua#L6-L23)、[AoE 三工厂](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L129-L269)、[注册块](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L295-L359)
- 优先使用：新物品的持续环填 `aoetargeting.reticule.reticuleprefab`、点击提示填 `pingprefab`，由本地 `reticule` 组件创建；只有需要全服看见的落点才用 MakeTarget 联网名并接 `SetTargetFX`／`SpawnTargetFXAt`。[组件配置](../../../raw/current/scripts/components/aoetargeting.lua#L19-L30)、[本地创建](../../../raw/current/scripts/components/reticule.lua#L35-L69)、[联网生成](../../../raw/current/scripts/components/aoetargeting.lua#L123-L141)
- 重要副作用：48 名中 **37 个非联网**（`FX`＋`NOCLICK`＋`persists=false`，只在生成端可见），**11 个 MakeTarget 联网**（AddNetwork＋`net_smallbyte` 淡入；服务端 `KillFX` 才反向淡出）。ping 按各工厂 0.4–0.5 秒自删；target 淡入后驻留，不 `KillFX` 不会自行消失，不能当免清理的持久标记。[非联网构造](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L140-L158)、[联网构造](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L232-L266)、[淡出](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L185-L201)
- 修改边界：这些 prefab 只提供视觉与跟随形态，不负责施法授权、命中或范围伤害；换 prefab 只换外观，半径仍由技能组件决定。bank/build 按原版 anim 声明，zip 未解包；改名或复用须自备同名 zip。[AoE 注册](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L295-L300)、[资源声明](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L5-L7)

## 注册与构造

8 个文件共 48 个注册名，全部由 `CreateEntity` 手工组装，只加 `FX`／`NOCLICK` 标签。非联网型先 `SetCanSleep(false)`＋`persists=false` 再 `AddTransform`／`AddAnimState`；联网型只在 `MakeTarget` 出现，额外 `AddNetwork`、`net_smallbyte` 与 `SetPristine`。非联网环用 `LAYER_WORLD_BACKGROUND`，target 用 `LAYER_BACKGROUND`，均 SortOrder 3；多数环 1.5 倍（`reticule` 无缩放、`reticulemultitarget` 1.15）；ping 另挂 bloom。[通用非联网构造](../../../raw/current/scripts/prefabs/reticule.lua#L6-L23)、[MakeReticule](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L140-L158)、[MakeTarget](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L232-L266)

三个工厂分工：`MakeReticule` 常驻环、`MakePing` 0.5 秒自删、`MakeTarget` 联网淡入后驻留（服务端 `KillFX` 才反向删除）；其余文件各自内建工厂。下表逐名列出注册行、bank/build/anim、端与主要生成点。

### 基础准星与 AoE 底盘（10 个）

| 注册名 | 工厂与初始形态 | 生成点／用途（全 scripts 反查） | 证据 |
|---|---|---|---|
| `reticule` | `Prefab` 直连：bank/build `reticule`、anim `idle`、OnGround、无缩放、非联网 | `components/reticule.lua` 的默认 `reticuleprefab`；无专属配置时作通用准星，由 playercontroller 按装备创建 | [L28](../../../raw/current/scripts/prefabs/reticule.lua#L28) |
| `reticuleaoe` | `MakeReticule`：bank/build `reticuleaoe`、anim `idle`、OnGroundFixed、1.5、非联网 | 默认 AoE 盘：fireballstaff:68-69／hammer_mjolnir:73-74／sleepbomb:97-98／waxwelljournal:316-317／books_lavaarena:89／healingstaff:66-67 | [L295](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L295) |
| `reticuleaoesmall` | 同上，anim `idle_small`、OnGroundFixed | 小型盘：lavaarena_firebomb:71-72／spear_lance:62-63 | [L296](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L296) |
| `reticuleaoesummon` | 同上，anim `idle_summon`、OnGroundFixed | 召唤盘：books_lavaarena:90 的 `book_elemental`（`MakeBook` 参数） | [L297](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L297) |
| `reticuleaoe_1_6` | 同上，anim `idle_1_6`、OnGroundFixed | 暗影陷阱范围：waxwelljournal:295-296 | [L298](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L298) |
| `reticuleaoe_1d2_12` | 同上，anim `idle_1d2_12`、OnGroundFixed | 暗影工人／守卫范围：waxwelljournal:253-254、274-275 | [L299](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L299) |
| `reticuleaoe5line` | 同上，anim `idle_5live`、OnGround（非 Fixed） | 五线型暗影火：willow_ember:818-819 | [L300](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L300) |
| `reticuleaoecatapultvolley` | 覆写 bank/build `winona_catapult_placement`、anim `idle_1d25_4d4_16d6`、OnGroundFixed；postinit 置 `deployhelper_key="catapult_volley"`＋LightOverride 1 | 薇诺娜投石机齐射：winona_remote:291-292；作为 `placerinst` 供 `TriggerDeployHelpers` 选 helper | [L301-L305](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L301-L305) |
| `reticuleaoecatapultelementalvolley` | 同上 bank/build 与 anim；`deployhelper_key="catapult_elementalvolley"`＋LightOverride 1 | 元素齐射：winona_remote:388-389 | [L306-L310](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L306-L310) |
| `reticuleaoecatapultwakeup` | 同上 bank/build，anim `idle_16d6`；`deployhelper_key="catapult_wakeup"`＋LightOverride 1 | 投石机唤醒／加速：winona_remote:325-326、359-360 | [L311-L315](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L311-L315) |

`deployhelper_key` 由 [[reference/components/deployhelper|deployhelper]] 的 `TriggerDeployHelpers` 读取；这三个变体不是普通伤害范围圈，而是给部署辅助器用的“假 placer”。[消费](../../../raw/current/scripts/components/deployhelper.lua#L3-L12)

### 一次性扩散 ping（11 个）

`MakePing` 统一 0.5 秒生命周期：创建时记录 `t0`，每帧按 `1→scaleup` 放大并淡出 mult/add colour，`DoTaskInTime(0.5, Remove)`。[工厂](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L35-L78)

| 注册名 | bank/build、anim、朝向、放大倍率 | 生成点／用途 | 证据 |
|---|---|---|---|
| `reticuleaoeping` | `reticuleaoe`／`idle`／Fixed／1.05 | fireballstaff:69、hammer_mjolnir:74、sleepbomb:98、waxwelljournal:317、healingstaff:67 | [L318](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L318) |
| `reticuleaoesmallping` | `reticuleaoe`／`idle_small`／Fixed／1.1 | lavaarena_firebomb:72、spear_lance:63 | [L319](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L319) |
| `reticuleaoesummonping` | `reticuleaoe`／`idle_summon`／Fixed／1.0667 | books_lavaarena:67 动态拼名 `reticule.."ping"`；`book_elemental` 的 `reticuleaoesummon` 对应本行，`book_fossil` 走 `reticuleaoeping` | [L320](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L320) |
| `reticuleaoeping_1_6` | `reticuleaoe`／`idle_1_6`／Fixed／1.1 | waxwelljournal:296 | [L321](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L321) |
| `reticuleaoeping_1d2_12` | `reticuleaoe`／`idle_1d2_12`／Fixed／1.08333 | waxwelljournal:254、275；walter:451 直接生成 | [L322](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L322) |
| `reticuleaoeping5line` | `reticuleaoe`／`idle_5live`／OnGround／1.08333 | willow_ember:819 | [L323](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L323) |
| `reticuleaoefiretarget_1ping` | `reticuleaoe`／`idle_summon_target`／Fixed／1.08333 | willow_ember:638、705、738 | [L325](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L325) |
| `reticuleaoeghosttarget_ping` | `reticuleaoe`／`idle_summon_target`／Fixed／1.08333 | ghostcommand_defs:222、309 | [L327](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L327) |
| `reticuleaoecatapultvolleyping` | `winona_catapult_placement`／`idle_1d25_4d4_16d6`／Fixed／1.08；postinit LightOverride 1 | winona_remote:292、389 | [L329-L333](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L329-L333) |
| `reticuleaoecatapultwakeupping` | `winona_catapult_placement`／`idle_16d6`／Fixed／1.036；postinit LightOverride 1 | winona_remote:326、360 | [L334-L338](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L334-L338) |
| `reticuleaoewinonaengineeringping` | `winona_battery_placement`／`idle_1d6`／OnGround／1.125；postinit LightOverride 1 | winona_remote 的 `PingCatapult` 直接生成于投石机位置（:43） | [L339-L343](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L339-L343) |

### 联网落点 target（11 个）

`MakeTarget` 是家族中唯一联网的子类：`AddNetwork`＋`net_smallbyte(GUID, name.."._fade", "fadedirty")`，客户端监听 `fadedirty` 插值 alpha；`FADE_FRAMES=10`：先 10 帧淡入后取消周期任务并驻留；服务端 `KillFX` 把 fade 值推到反向，再用约 11 帧淡出，超过 20 后 `Remove`。不调用 `KillFX` 时 FX 保持显示。颜色由注册时传入的 RGBA 固定，实例再 `OverrideMultColour(0..)` 从透明淡入。[联网构造](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L232-L266)、[淡出状态机](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L185-L201)

| 注册名 | anim、朝向、固定颜色（RGBA） | 生成点／用途 | 证据 |
|---|---|---|---|
| `reticuleaoehostiletarget` | `idle_target`／OnGround／{1,.25,0,1} 橙 | 仅预载声明：fireballstaff:20、hammer_mjolnir:18；scripts 内未见 `SetTargetFX`，快照外 event_server_data 未核验 | [L345](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L345) |
| `reticuleaoefriendlytarget` | `idle_target`／OnGround／{0,1,.25,1} 绿 | 仅预载声明：healingstaff:19；scripts 内未见 `SetTargetFX`，快照外 event_server_data 未核验 | [L346](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L346) |
| `reticuleaoecctarget` | `idle_target`／OnGround／{.3,.5,.2,1} 暗绿 | waxwelljournal 暗影柱 `SetTargetFX`（:319）；books_lavaarena:6 仅预载 | [L347](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L347) |
| `reticuleaoesmallhostiletarget` | `idle_small_target`／OnGround／{1,.25,0,1} 橙 | 仅预载声明：lavaarena_firebomb:24、spear_lance:11；scripts 内未见 `SetTargetFX`，快照外 event_server_data 未核验 | [L348](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L348) |
| `reticuleaoesummontarget` | `idle_summon_target`／OnGround／{.3,.5,.2,1} 暗绿 | 仅预载声明：books_lavaarena:14；快照外 `event_server_data` 可能配置，未核验 | [L349](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L349) |
| `reticuleaoesummontarget_1` | `idle_target_1`／OnGround／{.3,.5,.2,1} | waxwelljournal 暗影陷阱 `SetTargetFX`（:298） | [L350](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L350) |
| `reticuleaoesummontarget_1d2` | `idle_target_1d2`／OnGround／{.3,.5,.2,1} | waxwelljournal 工人／守卫 `SetTargetFX`（:256、277） | [L351](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L351) |
| `reticuleaoeshadowtarget_6` | `idle_target_6`／OnGround／{.1,.1,.1,1} 黑 | shadow_trap 的 `EnableTargetFX` 生成并 `SetParent(inst.entity)`（:104-105），关闭时 `KillFX` | [L352](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L352) |
| `reticuleaoefiretarget_1` | `idle_summon_target`／OnGround／{255/255,161/255,61/255,1} 橙 | 双用途：willow_ember 持续环（:637-638、704-705、737-738）＋服务端落点 `SetTargetFX`（:645、712） | [L353](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L353) |
| `reticuleaoeghosttarget` | `idle_summon_target`／OnGround／{255/255,161/255,61/255,1} 橙 | 双用途：ghostcommand_defs 持续环（:221、308）＋服务端落点（:230、317） | [L354](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L354) |
| `reticuleaoehostiletarget_1d25` | `winona_catapult_placement`／`idle_target_1d25`／OnGround／{1,.25,0,1} | winona_remote 齐射／元素齐射落点（:295、392） | [L355-L359](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L355-L359) |

### 弧形、蓄力、线形与长条（10 个）

| 注册名 | 工厂与初始形态 | 生成点／用途 | 证据 |
|---|---|---|---|
| `reticulearc` | 文件内 `MakeReticule`：bank/build `reticulearc`、anim `idle`、OnGround、1.5、非联网 | 弧形攻击：lavaarena_heavyblade:73-74、wathgrithr_shield:170-171 | [L65](../../../raw/current/scripts/prefabs/reticulearc.lua#L65) |
| `reticulearcping` | 同文件 `ping` 分支：同 bank/build、bloom、1→1.075、0.5 秒自删 | 同上（:74、:171） | [L66](../../../raw/current/scripts/prefabs/reticulearc.lua#L66) |
| `reticulecharging` | `Prefab` 直连；assets 用 `reticulelong.zip`、bank/build `reticulelong`、anim `idle`、OnGround、1.5；加 `chargingreticule`（`ease=true`、`pingprefab="reticulelongping"`） | 弹弓蓄力：slingshot:651-652 的 `aoecharging.reticuleprefab` | [L49](../../../raw/current/scripts/prefabs/reticulecharging.lua#L49) |
| `reticuleline` | `MakeReticule`：bank/build `reticuleline`、anim `idle`、OnGround、1.5 | 直线型：spear_gungnir:79-80、spear_wathgrithr:430-431、willow_ember:770-771 | [L89](../../../raw/current/scripts/prefabs/reticuleline.lua#L89) |
| `reticuleline2` | 同一 `MakeReticule`，资产按 `anim/"..name..".zip"` 动态取 `reticuleline2.zip`、bank/build `reticuleline2` | woodie 变身瞄准（reticule 组件，:349） | [L90](../../../raw/current/scripts/prefabs/reticuleline.lua#L90) |
| `reticulelineping` | 独立 `pingfn`：固定 bank/build `reticuleline`、bloom、1→{1.04,1.25}、0.4＋0.1 秒自删 | 直线型点击提示：spear_gungnir:80、spear_wathgrithr:431、willow_ember:771 | [L91](../../../raw/current/scripts/prefabs/reticuleline.lua#L91) |
| `reticulelong` | `MakeReticule`：bank/build `reticulelong`、anim `idle`、OnGround、1.5 | 长条型：blowdart_lava2:89-90、lavaarena_lucy:91-92、slingshot:772-773 | [L72](../../../raw/current/scripts/prefabs/reticulelong.lua#L72) |
| `reticulelongping` | `ping` 分支：同 bank/build、1→{1.04,1.3}、0.4 秒自删；带 `chargescale`＋`SetChargeScale` | 同上；另被 `aoecharging` 松手时硬编码生成（:164） | [L73](../../../raw/current/scripts/prefabs/reticulelong.lua#L73) |
| `reticulelongmulti` | `MakeReticule`：bank/build `reticulelong`、anim `idle_multi`、OnGround、1.5 | 多线长条：blowdart_lava:85-86 | [L74](../../../raw/current/scripts/prefabs/reticulelong.lua#L74) |
| `reticulelongmultiping` | `ping` 分支：anim `idle_multi`、同放大与自删 | 同上（:86） | [L75](../../../raw/current/scripts/prefabs/reticulelong.lua#L75) |

### 多目标与高尔夫专用（6 个）

| 注册名 | 工厂与初始形态 | 生成点／用途 | 证据 |
|---|---|---|---|
| `reticulemultitarget` | `main`：bank/build `reticuleaoe`、anim `idle_1d2_12`、OnGround、1.15、`Hide()`，挂 `updatelooper` 每帧取 `willow_ember_common.GetBurstTargets(ThePlayer)` | willow_ember 火焰爆发持续环（:670-671） | [L158](../../../raw/current/scripts/prefabs/reticulemultitarget.lua#L158) |
| `reticulemultitargetsub` | `sub`：bank/build `reticuleaoe`、anim `idle_target_1`、OnGround、不缩放 | 由 main 对每个目标生成并 `AddChild`（:40-42），随 main 的 `onremove` 清理（:17-23） | [L159](../../../raw/current/scripts/prefabs/reticulemultitarget.lua#L159) |
| `reticulemultitargetping` | `pingfn`：bank/build `reticuleaoe`、anim `idle_1d2_12`、初始 1.15、bloom、1→{1.04,1.25}、0.4＋0.1 秒自删 | 同上（:671） | [L160](../../../raw/current/scripts/prefabs/reticulemultitarget.lua#L160) |
| `golfclub_reticule_fx` | `MakeReticule`：bank/build `golfclub_reticule`、anim `idle`、OnGround、1.5 | 球杆瞄准：`golfclub_reticule` 组件动态设置 `reticuleprefab`（:227） | [L102](../../../raw/current/scripts/prefabs/golfclub_reticule.lua#L102) |
| `golfclub_reticulecharging` | 同上加 `chargingreticule`（`ease`、`pingprefab="golfclub_reticuleping"`）＋`SetChargeScale`＋`chargingsfx`；删除时杀 `golfclub_reticule_windup` 音效 | 球杆蓄力：同组件 `StartCharging`（:272） | [L103](../../../raw/current/scripts/prefabs/golfclub_reticule.lua#L103) |
| `golfclub_reticuleping` | `ping` 分支：同 bank/build、bloom、1→{1.04,1.3}、0.4 秒自删、`SetChargeScale` | 球杆命中提示：组件 `OnTargetDirty` 生成（:192） | [L104](../../../raw/current/scripts/prefabs/golfclub_reticule.lua#L104) |

名称核验：`STRINGS.NAMES` 中没有任何 reticule／golfclub_reticule 键，`chinese_s.po` 唯一命中是 `STRINGS.UI.CONTROLSSCREEN.OVERRIDE_CONTROL_NAMES.AIMING`（“Aiming Reticule”＝“瞄准光圈”，按键说明而非 prefab 名），故标题与别名用已核验中文效果描述＋注册名，不是官方译名；该 `.po` 未计入 `sources`。

## 关联机制导航

| 类型／对象 | 本族如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：reticule` | 管理持续 `reticuleprefab` 与 `pingprefab`；创建时 `SpawnPrefab`，销毁时清输入／相机监听；颜色随地图资格切换，多目标子实体会一起染色 | 本地组件，依赖 TheInput／TheCamera；不负责施法授权 | [[reference/components/reticule]]、[创建](../../../raw/current/scripts/components/reticule.lua#L35-L69)、[ping](../../../raw/current/scripts/components/reticule.lua#L87-L101)、[销毁](../../../raw/current/scripts/components/reticule.lua#L72-L84) |
| `组件：aoetargeting` | 持有 `reticule` 配置表；StartTargeting 时浅拷贝到新挂的 `reticule` 组件；`SetTargetFX` 只登记，`SpawnTargetFXAt` 才生成联网落点 | 组件本体双端；生成落点要求服务端上下文与施法 SG 调用 | [[reference/components/aoetargeting]]、[配置表](../../../raw/current/scripts/components/aoetargeting.lua#L19-L30)、[拷贝](../../../raw/current/scripts/components/aoetargeting.lua#L98-L108)、[生成](../../../raw/current/scripts/components/aoetargeting.lua#L123-L141) |
| `组件：aoecharging` | `owner.HUD` 时按 `reticuleprefab` 生成蓄力环并 `LinkToEntity(owner)`；松手时生成 ping；更新驱动 `Snap`／旋转 | 只在实际渲染 HUD 的本地端创建；松手 ping 名当前硬编码为 `reticulelongping` | [[reference/components/aoecharging]]、[生成环](../../../raw/current/scripts/components/aoecharging.lua#L107-L109)、[硬编码 ping](../../../raw/current/scripts/components/aoecharging.lua#L163-L170) |
| `组件：chargingreticule` | 跟随 owner 位置，按鼠标／手柄更新朝向；`SetChargeScale` 定义在工厂侧、由外部刷新回调调用；`Snap` 收束；删除时清 move handler 与相机监听 | 本地 FX 组件；`pingprefab` 字段本组件不读 | [[reference/components/chargingreticule]]、[关联](../../../raw/current/scripts/components/chargingreticule.lua#L41-L71)、[清理](../../../raw/current/scripts/components/chargingreticule.lua#L15-L20) |
| `组件：golfclub_reticule` | 客户端动态增删 `reticule`，在 `golfclub_reticule_fx`／`golfclub_reticulecharging` 间切换；另生成 `cannon_aoe_range_fx` 范围圈与 `golfclub_reticuleping` | 服务端只同步 target／chargeticks；视觉环只在有 HUD 的客户端存在 | [[reference/components/golfclub_reticule]]、[切环](../../../raw/current/scripts/components/golfclub_reticule.lua#L220-L227)、[蓄力环](../../../raw/current/scripts/components/golfclub_reticule.lua#L272-L275)、[ping](../../../raw/current/scripts/components/golfclub_reticule.lua#L191-L197) |
| `API：PlayerController` | `RefreshReticule` 选当前手持或玩家 `reticule` 组件并创建；`EchoReticuleAt` 先生成同 prefab，若 `reticule.Flash` 非空则调用 `Flash()`，否则排 1 秒后 `Remove`（当前 scripts 无调用点） | 本地输入层；`EchoReticuleAt` 仅用于回显 | [[reference/components/playercontroller]]、[回显](../../../raw/current/scripts/components/playercontroller.lua#L1711-L1718)、[刷新](../../../raw/current/scripts/components/playercontroller.lua#L1723-L1735) |
| `SG：SGwilson` | 服务端施法状态调用 `SpawnTargetFXAt`，给返回 FX 挂 `onremove` 清理；收尾时优先 `KillFX`，否则 `Remove` | 仅服务端生成；具体状态覆盖法杖、书、武器等多条路径 | [生成示例](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16201-L16203)、[清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L529-L534) |
| `关联 Prefab：cannon_aoe_range_fx` | 另一套非联网地面圈／船炮准星，由 boatcannonuser 与 golfclub_reticule 复用；与本族构造同型但不同 bank/build | 本地；不联网 | [[reference/prefabs/cannon_aoe_range_fx|船炮瞄准范围圈特效]]、[工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L11-L30) |
| `资源：bank/build` | `reticule.zip`、`reticuleaoe.zip`、`reticulearc.zip`、`reticulelong.zip`、`reticuleline.zip`、`reticuleline2.zip`、`golfclub_reticule.zip`，以及薇诺娜的 `winona_catapult_placement.zip`／`winona_battery_placement.zip` | 原版复用；`reticuleline2.zip` 由动态拼名声明，`reticulecharging` 复用 `reticulelong.zip`；zip 内容未解包核验 | [AoE 资产](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L5-L7)、[线形动态名](../../../raw/current/scripts/prefabs/reticuleline.lua#L57-L60)、[蓄力资产](../../../raw/current/scripts/prefabs/reticulecharging.lua#L1-L3) |

生成通道（全 scripts 反查）：

- 持续环：`PlayerController:RefreshReticule` → `Reticule:CreateReticule` → `SpawnPrefab(reticuleprefab)`；换装备／移除组件时 `DestroyReticule`。[components/playercontroller.lua:1723-1735](../../../raw/current/scripts/components/playercontroller.lua#L1723-L1735)、[components/reticule.lua:35-36](../../../raw/current/scripts/components/reticule.lua#L35-L36)
- 点击 ping：`Reticule:PingReticuleAt(pos)` → `SpawnPrefab(pingprefab)`，再设位置／颜色；平台位置会临时 `SetParent` 并在 `onremove` 还原。[components/reticule.lua:87-124](../../../raw/current/scripts/components/reticule.lua#L87-L124)
- 联网落点：`aoetargeting:SetTargetFX(name)` 只存 `targetprefab`；施法 SG 调 `SpawnTargetFXAt(pos)` 才生成，之后必须由服务端再调 `KillFX` 才反向淡出，不调用则永久驻留。[components/aoetargeting.lua:58-60](../../../raw/current/scripts/components/aoetargeting.lua#L58-L59)、[components/aoetargeting.lua:123-141](../../../raw/current/scripts/components/aoetargeting.lua#L123-L141)
- 蓄力方向：`aoecharging:SetChargingOwner` 按 `reticuleprefab` 生成；高尔夫另走 `golfclub_reticule` 组件的动态 `reticule`。[components/aoecharging.lua:107-109](../../../raw/current/scripts/components/aoecharging.lua#L107-L109)
- 多目标：`reticulemultitarget` 的 `updatelooper` 每次重建 `_targets`，为每个目标 `SpawnPrefab("reticulemultitargetsub")` 并 `AddChild`。[prefabs/reticulemultitarget.lua:26-46](../../../raw/current/scripts/prefabs/reticulemultitarget.lua#L26-L46)

## 典型接入

### 为自定义 AoE 物品接上本族指示器

- 前提与选择：目标模组 `modmain.lua`；自定义物品须已有 `aoetargeting`，且施法状态已按原版调用 `SpawnTargetFXAt`。只做视觉时选 `reticuleaoe`＋`reticuleaoeping`；要让全服看到落点再选一个 MakeTarget 名。[原版接线](../../../raw/current/scripts/prefabs/fireballstaff.lua#L68-L69)、[落点消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16201-L16203)
- 生效与撤销：在 `SetPristine` 之后改 `aoetargeting.reticule` 表；下一次 `StartTargeting` 才会浅拷贝到新 `reticule` 组件。`SetTargetFX` 只在服务端有意义，且真正生成仍由施法 SG 触发。移除物品／停止瞄准后由 `DestroyReticule` 清理本地环；联网 target 不调 `KillFX` 不会消失。[拷贝时机](../../../raw/current/scripts/components/aoetargeting.lua#L98-L108)、[清理](../../../raw/current/scripts/components/reticule.lua#L72-L84)
- 原版依据：`waxwelljournal` 在服务端 `SetTargetFX("reticuleaoecctarget")`，`ghostcommand_defs` 同时用 `reticuleaoeghosttarget` 作环和落点，均为已验证模式。[waxwelljournal:316-319](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L316-L319)、[ghostcommand_defs:221-230](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L221-L230)

```lua
AddPrefabPostInit("mystaff", function(inst)
    local targeting = inst.components.aoetargeting
    if targeting == nil then
        return
    end

    targeting.reticule.reticuleprefab = "reticuleaoe"
    targeting.reticule.pingprefab = "reticuleaoeping"

    if TheWorld.ismastersim then
        targeting:SetTargetFX("reticuleaoehostiletarget")
    end
end)
```

- 调整与验证：把 `mystaff` 换成自定义注册名；视觉半径与 `reticuleaoe_1_6`／`_1d2_12` 等变体只改贴图，不改技能范围。`reticuleaoeping` 0.5 秒自删，`reticuleaoehostiletarget` 需服务端 `KillFX` 才淡出（约 11 帧），不能当免清理的持久标记；若自定义施法不走原版 SG 的 `SpawnTargetFXAt`，`SetTargetFX` 不会产生任何实体。[ping 自删](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L67-L69)、[target 淡出](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L185-L201)

## 常见陷阱与限制

- **误区：reticule 系全是本地 FX**：`reticuleaoe` 的 11 个 MakeTarget 注册名带 `AddNetwork`、`SetPristine` 与 `net_smallbyte`，会在服务端生成并复制给客户端；而 `reticuleaoe`／`reticuleaoeping` 等同文件变体没有 Network，只在生成端可见。[联网构造](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L232-L266)
- **误区：`prefabs` 依赖声明就等于实际生成**：fireballstaff／healingstaff／books_lavaarena 的目标 FX 仅在 prefabs 数组里预载，当前 scripts 没有对应 `SetTargetFX`；只有 waxwelljournal、winona_remote、ghostcommand_defs、willow_ember、shadow_trap 等实际接线。快照外的 `event_server_data("lavaarena", ...)` 不在 `scripts/`，本页未核验。[声明反例](../../../raw/current/scripts/prefabs/fireballstaff.lua#L13-L20)
- **边界：`aoecharging.pingprefab` 不是通用配置**：松手分支硬编码 `SpawnPrefab("reticulelongping")`，把字段改成别的 ping 不会生效；当前只有 slingshot 的配置恰好等于硬编码值。[components/aoecharging.lua:163-170](../../../raw/current/scripts/components/aoecharging.lua#L163-L170)
- **边界：`reticulecharging` 上的 `chargingreticule.pingprefab` 无人读取**：该组件只处理跟随、方向与 Snap，ping 由 aoecharging 生成；字段名不能当自动提示保证。[工厂赋值](../../../raw/current/scripts/prefabs/reticulecharging.lua#L39-L41)、[组件全文](../../../raw/current/scripts/components/chargingreticule.lua#L1-L114)
- **边界：target FX 不会自行消失**：`MakeTarget` 只在前 10 帧淡入，随后取消周期任务并驻留；必须由服务端 `KillFX` 触发反向淡出，超过 20 后 `Remove`。不清理会一直显示，`persists=false` 只表示不存档。[淡出](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L185-L201)、[KillFX](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L223-L227)
- **边界：换 prefab 只换外观**：`reticuleaoe` 系列半径由动画贴图与 1.5 缩放表达，真实施法范围来自 `aoespell`／技能数据；`SetDeployRadius` 只是地图净空检查，不是伤害半径。[注册块](../../../raw/current/scripts/prefabs/reticuleaoe.lua#L295-L300)、[[reference/components/aoetargeting#关键接口]]
- **边界：资源名与 prefab 名不完全一致**：`reticulecharging` 用 `reticulelong.zip` 与 `reticulelong` bank；`reticuleline2` 的 zip 名由 `name` 动态拼出；改名或复用到自定义 prefab 时必须自备匹配的 anim zip，不能只改注册名。[蓄力资产](../../../raw/current/scripts/prefabs/reticulecharging.lua#L1-L3)、[动态资产](../../../raw/current/scripts/prefabs/reticuleline.lua#L57-L60)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。完整读取 `prefabs/reticule.lua`、`reticuleaoe.lua`、`reticulearc.lua`、`reticulecharging.lua`、`reticuleline.lua`、`reticulelong.lua`、`reticulemultitarget.lua`、`golfclub_reticule.lua`，逐名核对 48 个注册名、工厂、bank/build/anim、朝向、缩放、标签与端；反查全 `scripts/` 的 `SpawnPrefab`／`reticuleprefab`／`pingprefab`／`SetTargetFX` 与动态拼名。`STRINGS.NAMES` 检索无本族键，未使用官方译名。未覆盖：anim zip 内容与游戏内外观、快照外 `event_server_data` 的 Lava Arena 配置、各平台客户端显示差异。
- 游戏运行：未运行。
- 分档说明：8 文件 48 注册、两套端模型，按 `complex` 记录；实测 178 行／约 31.8 KiB，在 400 行／32 KiB 内。
- 结构检查：2026-11-04 实测通过；`validate_page.py`（complex，178 行／约 31.8 KiB）、`validate_source_links.py --strict`（112 引用／90 目标，0 错 0 警）、Lua 示例 `parser_probe --syntax-only`（374 B，exit 0）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。
