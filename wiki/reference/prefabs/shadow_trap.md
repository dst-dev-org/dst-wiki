---
title: "暗影陷阱与减速表现（shadow_trap）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "暗影秘典“暗影陷阱”法术的两个注册：玩法陷阱 shadow_trap 与目标减速贴片 shadow_trap_debuff_fx；核验工厂、端、生成点、父挂与清理。"
aliases: ["shadow_trap", "shadow_trap_debuff_fx", "暗影陷阱", "影陷阱", "Shadow Sneak", "陷阱减速贴片"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/shadow_trap.lua", "stategraphs/SGshadow_trap.lua", "prefabs/waxwelljournal.lua", "prefabs/reticuleaoe.lua", "prefabs/shadow_pillar.lua", "prefabs/shadowwaxwell.lua", "prefabs/rabbit.lua", "prefabs/bunnyman.lua", "prefabs/monkey.lua", "prefablist.lua", "tuning.lua", "components/locomotor.lua", "components/hauntable.lua", "components/timer.lua", "components/map.lua", "scrapbook_prefabs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影陷阱与减速表现（shadow_trap）

## 快速判断

- 职责与适用：同文件两注册。`shadow_trap` 是法术落地的陷阱（无生命／战斗／物理，靠 SG 播动画，醒来每 0.1 秒探测 3 距离，爆炸在 6 距离结算）；`shadow_trap_debuff_fx` 是目标减速／恐慌贴片（`CLASSIFIED`、`FinalOffset(7)`）。[注册](../../../raw/current/scripts/prefabs/shadow_trap.lua#L415-L416)、[陷阱](../../../raw/current/scripts/prefabs/shadow_trap.lua#L299-L349)、[贴片](../../../raw/current/scripts/prefabs/shadow_trap.lua#L383-L413)
- 优先使用：查“暗影陷阱／影陷阱／减速贴片”选本页；柱底贴片归 [[reference/prefabs/shadow_pillar]]，施法圈与落点见 [[reference/prefabs/reticuleaoe]]。
- 重要副作用：命中目标各出 `shadow_despawn`、补 0 伤害 `attacked`（`noattack` 状态跳过）；可恐慌且非噩梦目标再 `Panic(12)`＋`"shadow_trap"` 键 2/3 外部移速 12 秒；带 `has_nightmare_state` 的兔子／兔人／猴子改推 `ms_forcenightmarestate`，不减速。[结算](../../../raw/current/scripts/prefabs/shadow_trap.lua#L140-L184)、[参数](../../../raw/current/scripts/tuning.lua#L2829-L2832)
- 修改边界：激活、爆炸与驱散由 SG 驱动，`TriggerTrap` 仅在 `activated` 标签下开始扫描；陷阱加 `NOBLOCK` 退出部署检查。施法成本归 waxwelljournal（待建）。动画包与专服表现未运行核验。[触发](../../../raw/current/scripts/prefabs/shadow_trap.lua#L190-L207)

## 注册与构造

两注册同文件末尾返回，均 `CreateEntity` 手工组装并联网：`shadow_trap` 仅主模拟建柱底贴片、计时器与 SG；`shadow_trap_debuff_fx` 的 `persists=false`，仅主模拟排队循环动画并暴露 `OnSetTarget`／`KillFX`。[返回](../../../raw/current/scripts/prefabs/shadow_trap.lua#L415-L416) `prefablist.lua` 登记文件，加载即注册两名。[prefablist](../../../raw/current/scripts/prefablist.lua#L1114-L1114)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_trap` | `fn`：Transform／AnimState／SoundEmitter／Network → 双平台忽略标签 → bank／build `shadow_trap` 播 `spawn` → `net_event("shadow_trap.shockwave")` → `SetPristine` → 非主模拟早退；主模拟建 `base`、`timer`（`lifetime`＝3 个 `total_day_time`）、监听 `timerdone`／`onsink`、`SetStateGraph`，暴露 `EnableGroundFX`／`EnableTargetFX`／`TriggerTrap` 与睡醒／读档回调。 | 联网玩法陷阱，默认持久化，触发或驱散后 `persists=false`；`OnShockwave` 在生成端本地建非联网冲击波（黑色半透明、缩放 1.9、animover 自删），专服不建。[工厂](../../../raw/current/scripts/prefabs/shadow_trap.lua#L299-L349)、[冲击波](../../../raw/current/scripts/prefabs/shadow_trap.lua#L273-L297) |
| `shadow_trap_debuff_fx` | `fxfn`：Transform／AnimState／Network → `NOCLICK`、`CLASSIFIED`（`FX` 被注释：父挂后仍会悬浮）→ bank／build `shadow_trap_debuff`、播 `debuff_pre_medium`、alpha 0.5、`SetFinalOffset(7)` → `SetPristine` → 非主模拟早退；主模拟 `PushAnimation("debuff_loop_medium")`、暴露 `OnSetTarget`／`KillFX`、`persists=false`。 | 联网贴附表现；`OnSetTarget` 按目标 `smallcreature`／`largecreature` 切 small／large，否则 medium；`KillFX` 播 `debuff_pst_*` 后 animover 自删。[工厂](../../../raw/current/scripts/prefabs/shadow_trap.lua#L383-L413)、[收尾](../../../raw/current/scripts/prefabs/shadow_trap.lua#L351-L381) |

名称核验：`languages/chinese_s.po` 只有 `STRINGS.SPELLS.SHADOW_TRAP`＝“暗影陷阱”（msgid “Shadow Sneak”），两注册名都无 `STRINGS.NAMES.*`；标题用官方法术名＋效果描述。[po](../../../raw/current/scripts/languages/chinese_s.po#L415359-L415362)

### 生成点与清理

- `shadow_trap`：唯一生成点是 `waxwelljournal.lua` 的 `TrapSpellFn`（平台落点移除 `ignorewalkableplatforms`）。`prefablist`／`prefabs` 表只是加载／预载声明；`scrapbook` 条目被注释，兔子／兔人／猴子只留注释，`magic_pumpkin` 只播同名音效。[施法](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L56-L68)、[预载](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L10-L20)、[注释](../../../raw/current/scripts/scrapbook_prefabs.lua#L2193-L2193)
- `shadow_trap_debuff_fx`：唯一生成点在同文件 `TryTrapTarget`，父挂目标后立即 `OnSetTarget`；同文件 `prefabs` 表预载。[生成](../../../raw/current/scripts/prefabs/shadow_trap.lua#L171-L179)
- 父挂：柱底贴片、目标圈、冲击波父挂陷阱（:327-328、:104-105、:296）；贴片父挂目标（:175）；`shadow_despawn` 命中时随目标平台、驱散时父挂陷阱（:150-158、SG :199）；`shadow_glob_fx` 平台父挂平台。[父挂](../../../raw/current/scripts/prefabs/shadow_trap.lua#L327-L330)、[平台脱钩](../../../raw/current/scripts/prefabs/shadow_trap.lua#L150-L158)
- 清理（陷阱）：`lifetime` 到期 → `DispellTrap`：睡着直接 Remove，否则 `GoToState("dispell")` 后 1 秒 Remove；触发态关目标圈、`base:KillFX()`、`TriggerTrap`，扫描 0.75 秒停、0.5 秒关地面痕、1.2 秒＋10 帧 Remove；触发后 `DispellTrap` 直接返回。[驱散](../../../raw/current/scripts/prefabs/shadow_trap.lua#L209-L224)、[扫描收尾](../../../raw/current/scripts/prefabs/shadow_trap.lua#L190-L207)、[SG 驱散](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L189-L231)
- 清理（贴片）：12 秒 `EndSpeedMult` 移除移速键并 `KillFX`；目标 `death` 只 `KillFX`，`onremove` 才先取世界坐标、`SetParent(nil)` 再 `KillFX`；睡着直接 Remove，否则 `pst` 后 animover Remove；陷阱自删不影响贴片。[恢复](../../../raw/current/scripts/prefabs/shadow_trap.lua#L131-L138)、[监听](../../../raw/current/scripts/prefabs/shadow_trap.lua#L372-L380)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：timer／health` | timer 计 `lifetime`（1440 秒），`timerdone` 转 `DispellTrap`；扫描用 `IsDead()` 过滤。 | 主模拟；只读。 | [[reference/components/timer]]；[计时](../../../raw/current/scripts/prefabs/shadow_trap.lua#L332-L335) |
| `组件：hauntable／locomotor（目标侧）` | 读 `panicable` 判可恐慌；`Panic(12)` 后设 2/3 外部移速，12 秒后移除。 | 主模拟；贴片与减速只在目标有 locomotor 时挂。 | [[reference/components/hauntable]]、[[reference/components/locomotor]]；[结算](../../../raw/current/scripts/prefabs/shadow_trap.lua#L168-L180) |
| `组件：rooted` | **未使用**：陷阱只减速＋恐慌，不定身；定身对照 [[reference/prefabs/shadow_pillar]]。 | — | [[reference/components/rooted]]；[对照](../../../raw/current/scripts/prefabs/shadow_trap.lua#L168-L180) |
| `SG／状态标签` | 工厂暴露 `base` 与 `TriggerTrap`／`EnableTargetFX`／`EnableGroundFX`；陷阱 Lua 读 `canactivate`／`candetect`／`near`／`activated` 并调 `GoToState`。 | 主模拟；`activate`／`activating_loop`／`trigger`／`dispell` 带 `activated`。 | [SG](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L97-L231) |
| `事件：ms_forcenightmarestate` | 对 `has_nightmare_state` 目标推该事件（时长 480 秒＋随机）；兔子、兔人、猴子监听并变形。 | 主模拟；与恐慌／减速互斥。 | [推送](../../../raw/current/scripts/prefabs/shadow_trap.lua#L162-L164)、[兔子](../../../raw/current/scripts/prefabs/rabbit.lua#L436-L437)、[兔人](../../../raw/current/scripts/prefabs/bunnyman.lua#L688-L689)、[猴子](../../../raw/current/scripts/prefabs/monkey.lua#L514-L515) |
| `关联 FX：shadow_pillar_base_fx` | 生成、父挂并缩放 0.7；`trigger` 态 `KillFX`，`collidewithboat` 时直接 `Remove`。 | 联网 FX；工厂与降下表现归影柱页。 | [[reference/prefabs/shadow_pillar]]；[生成](../../../raw/current/scripts/prefabs/shadow_trap.lua#L327-L330) |
| `关联 FX：reticuleaoeshadowtarget_6` | `EnableTargetFX(true)` 生成并父挂陷阱，关闭时 `KillFX`；黑色 `idle_target_6` 目标圈。 | 联网 target FX；`trigger` 态关闭。 | [[reference/prefabs/reticuleaoe]]；[开关](../../../raw/current/scripts/prefabs/shadow_trap.lua#L101-L111) |
| `关联 FX：shadow_glob_fx` | 激活期间每帧按环状偏移尝试（26 点），0.7 内无 `shadowtrail` 才落痕；触发／非撞船驱散时各一枚。 | 主模拟；平台父挂平台，海面无平台开涟漪。 | [[reference/prefabs/shadow_glob_fx]]；[地面痕](../../../raw/current/scripts/prefabs/shadow_trap.lua#L28-L97) |
| `关联 FX：shadow_despawn／ocean_splash_small1／2` | `shadow_despawn` 每个命中目标一枚、驱散时给陷阱本体一枚；小水花在海面时于 spawn、trigger 与非撞船驱散各落一枚。 | 主模拟；平台目标的 `shadow_despawn` 父挂平台，水花需 `IsOceanAtPoint`。 | [despawn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L632-L665)、[[reference/prefabs/shadow_despawn|暗影消散]]、[小水花](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L5-L10)、[[reference/prefabs/ocean_splash_med1]] |
| `消费者：waxwelljournal` | 条目 `STRINGS.SPELLS.SHADOW_TRAP`：图标 `shadow_trap.tex`、瞄准 `reticuleaoe_1_6`／`reticuleaoeping_1_6`，服务端 `SetSpellFn(TrapSpellFn)`。 | 服务端施法；平台落点移除 `ignorewalkableplatforms`。 | [条目](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L289-L308)、[施法](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L56-L68)（物品页待建） |
| `资源` | 三个 anim zip＋`maxwell_rework/shadow_trap/*` 与 `ghost_spawn` 音效。 | 只核验声明与播放处，包未解包。 | [资源表](../../../raw/current/scripts/prefabs/shadow_trap.lua#L1-L10) |

## 典型接入

### 给自定义效果挂同款目标减速贴片

- 前提与选择：宿主在主模拟、目标带 `locomotor`（带 `hauntable.panicable` 才会恐慌）；复用 `shadow_trap_debuff_fx` 可得三档贴片与目标死亡／移除自动收尾；减速键用独立 key，避免与原版 `"shadow_trap"` 覆盖。
- 生效与撤销：`SetParent(target.entity)` 后 `OnSetTarget` 选尺寸并监听 `death`／`onremove`；到时或手动 `KillFX` 播 `pst` 后 animover 自删；撤销用同一来源与 key。原版依据：[TryTrapTarget](../../../raw/current/scripts/prefabs/shadow_trap.lua#L168-L180)、[EndSpeedMult](../../../raw/current/scripts/prefabs/shadow_trap.lua#L131-L138)。

```lua
-- 服务端：给目标挂暗影陷阱同款减速贴片，12 秒后恢复
local PANIC_TIME = TUNING.SHADOW_TRAP_PANIC_TIME
local SPEED_MULT = TUNING.SHADOW_TRAP_SPEED_MULT
local SPEED_KEY = "my_trap_debuff"

local function ApplyTrapDebuff(target)
    if target.components.locomotor == nil then
        return
    end
    if target.components.hauntable ~= nil and target.components.hauntable.panicable then
        target.components.hauntable:Panic(PANIC_TIME)
    end
    local fx = SpawnPrefab("shadow_trap_debuff_fx")
    fx.entity:SetParent(target.entity)
    fx:OnSetTarget(target)
    target.components.locomotor:SetExternalSpeedMultiplier(target, SPEED_KEY, SPEED_MULT)
    target:DoTaskInTime(PANIC_TIME, function(inst)
        if inst.components.locomotor ~= nil then
            inst.components.locomotor:RemoveExternalSpeedMultiplier(inst, SPEED_KEY)
        end
        if fx:IsValid() then
            fx:KillFX()
        end
    end)
end
```

- 调整与验证：时长、倍率与 key 取自调参值；`KillFX`／`OnSetTarget` 只在主模拟副本存在。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`prefablist` 登记名字不等于会生成**：该表登记要加载的文件，实际生成点只有 waxwelljournal 的 `TrapSpellFn`；`scrapbook_prefabs` 条目被注释，其余命中只是注释或音效名。[生成](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L56-L68)、[注释](../../../raw/current/scripts/scrapbook_prefabs.lua#L2193-L2193)
- **误区：陷阱是纯视觉或“地雷”组件**：源码注释明确不用 `mine`，自建探测、激活与结算；本体无生命／战斗／物理，伤害为 0。[注释](../../../raw/current/scripts/prefabs/shadow_trap.lua#L23-L23)、[结算](../../../raw/current/scripts/prefabs/shadow_trap.lua#L140-L184)
- **边界：探测半径与命中半径不同，实体 `noattack` 标签与 SG 的 `noattack` 状态也不是同一条件**：3 距离激活、6 距离结算；实体 `noattack` 标签只在探测排除表中，爆炸命中表不排除，带该标签仍可被命中；命中后是否跳过 0 伤害 `attacked`，另由目标是否有 SG 且当前带 `noattack` 状态标签判定（无 SG 时不跳过）。[常量](../../../raw/current/scripts/prefabs/shadow_trap.lua#L115-L123)、[命中](../../../raw/current/scripts/prefabs/shadow_trap.lua#L162-L179)
- **边界：噩梦生物走另一条分支**：带 `has_nightmare_state` 的兔子／兔人／猴子不进入恐慌／减速分支，改为被推噩梦状态；`CanPanic` 也认该字段。[判定](../../../raw/current/scripts/prefabs/shadow_trap.lua#L125-L129)、[分支](../../../raw/current/scripts/prefabs/shadow_trap.lua#L162-L180)
- **边界：重复命中与贴片收尾**：目标已有 `_shadow_trap_task` 时只取消并重排 12 秒任务，不再生成第二枚贴片、不再调 `OnSetTarget`；陷阱约 1.2 秒＋10 帧自删，贴片随目标按 12 秒或目标死亡／移除收尾。[复用](../../../raw/current/scripts/prefabs/shadow_trap.lua#L171-L179)、[自删](../../../raw/current/scripts/prefabs/shadow_trap.lua#L201-L206)
- **边界：触发后不可再驱散**：`trigger` 已置 `persists=false`，`DispellTrap` 直接返回；`collidewithboat` 分支只在带 `ignorewalkableplatforms` 的陷阱脚下出现平台时走到，船上落点时标签已被移除，船沉由 `onsink` 驱散。[驱散](../../../raw/current/scripts/prefabs/shadow_trap.lua#L209-L224)、[平台检测](../../../raw/current/scripts/prefabs/shadow_trap.lua#L226-L245)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。核验 `shadow_trap.lua` 416 行（两注册、端分界、探测／命中、减速与噩梦分支、父挂、清理）、`SGshadow_trap.lua` 全文、waxwelljournal 的 `TrapSpellFn` 与条目接线、reticuleaoe 的 `reticuleaoeshadowtarget_6`、shadow_pillar 的 base 工厂、shadowwaxwell 的 `shadow_despawn` 工厂、`tuning.lua` 四参数、locomotor／hauntable／timer／map 接口、兔子／兔人／猴子监听与 `chinese_s.po`。
- 反查：全 `scripts` 按两注册名检索，`shadow_trap` 只命中自身、prefablist、waxwelljournal、SGshadow_trap、scrapbook（注释）与兔子／兔人／猴子／magic_pumpkin（注释或音效）；`shadow_trap_debuff_fx` 只命中自身；另两生成点逐名反查，无本页外消费者。
- 未运行；`validation` 保持 `source-reviewed`。`validate_page.py`、`validate_source_links.py --strict`、Lua 片段 `parser_probe.exe --syntax-only` 通过（仅语法）。
- 未覆盖：动画包内容与时序、客户端观感、专服／晚加入、读档中途、多目标实际帧序、秘典物品页 UI。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]；其他外围表现见 [[reference/prefabs/fx-index|FX 主题总索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
