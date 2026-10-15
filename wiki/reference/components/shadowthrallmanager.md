---
title: "shadowthrallmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级影牢组件：月之暗裂开启后占用梦魇裂缝，在玩家附近布置瘴气环并按裂缝类型生成三随从或巨口影怪，战斗脱战或全灭后释放裂缝进入冷却。"
aliases: ["影怪三随从管理", "梦魇裂缝控制", "shadowthrallmanager"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shadowthrallmanager.lua", "prefabs/nightmarefissure.lua", "components/corruption.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowthrallmanager

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于洞穴世界（[cave](../../../raw/current/scripts/prefabs/cave.lua#L570)）；影裂（shadow rift）加入裂缝池后，挑选一条梦魇裂缝"接管"为竞技场，布置瘴气与影怪，直至战斗分出结果或玩家走远（[L143-L168](../../../raw/current/scripts/components/shadowthrallmanager.lua#L143-L168)）。
- 优先使用：模组通常不直接调用；强制腐化裂缝走 [corruption（王之杖）回调 `ControlFissure`](../../../raw/current/scripts/components/corruption.lua#L142-L143)；查询状态用 `GetControlledFissure`／`GetThrallCount`／`IsRegisteredFissure`（[L203-L205](../../../raw/current/scripts/components/shadowthrallmanager.lua#L203-L205)、[L545-L551](../../../raw/current/scripts/components/shadowthrallmanager.lua#L545-L551)）。
- 重要副作用：控制中的裂缝被强制进 `controlled` 梦魇相位且不再受昼夜切换影响（[fissure 钩子](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L274-L280)）；释放时在场影怪被无掉落秒杀、瘴气移除（[L405-L443](../../../raw/current/scripts/components/shadowthrallmanager.lua#L405-L443)）。
- 修改边界：只在 `riftspawner:IsShadowPortalActive()` 时工作，裂缝池事件 `ms_riftaddedtopool`／`ms_riftremovedfrompool` 驱动启停（[L143-L158](../../../raw/current/scripts/components/shadowthrallmanager.lua#L143-L158)、[L350-L351](../../../raw/current/scripts/components/shadowthrallmanager.lua#L350-L351)）；裂缝本体相位与可挖行为归 nightmarefissure prefab。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ControlFissure(fissure)` | → bool | 无控制中裂缝才成功：推 `OnNightmarePhaseChanged("controlled", false, true)` 后持有它（[L533-L543](../../../raw/current/scripts/components/shadowthrallmanager.lua#L533-L543)）。 |
| `OnFissureAnimationsFinished(fissure)` | 由 prefab `animover` 回调 | 竞技场就绪：重置挖冷、绕裂缝半径 4 放 8 团 `miasma_cloud`、启动 1.13s 周期刷怪任务（[L498-L531](../../../raw/current/scripts/components/shadowthrallmanager.lua#L498-L531)、[prefab 回调](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L88-L96)）。 |
| `OnSpawnThralls()` | 内部周期回调 | 需 14 距离内有玩家；按 `self.thralltype` 生成 TRIO（hands/horns/wings）或 MOUTH（3×`shadowthrall_mouth`），出生角背向玩家 ±50 度、延迟 0/0.6/1.2s 进 `spawndelay` 状态（[L218-L264](../../../raw/current/scripts/components/shadowthrallmanager.lua#L218-L264)）。 |
| `thralltype` | `THRALL_TYPES.SHADOW.TRIO or .MOUTH` | 来自裂缝池事件的 `riftthralltype`；王之杖强制腐化时为 nil，刷怪时随机五五开（[L160-L168](../../../raw/current/scripts/components/shadowthrallmanager.lua#L160-L168)、[L228-L231](../../../raw/current/scripts/components/shadowthrallmanager.lua#L228-L231)、[constants](../../../raw/current/scripts/constants.lua#L2798)）。 |
| `RegisterFissure(inst)` / `UnregisterFissure(inst)` / `IsRegisteredFissure(inst)` | — | 裂缝实体唤醒／睡眠时由 prefab 维护候选池（带 onremove 监听）；只有 `fissure`/`fissure_lower` prefab 参与（[L189-L205](../../../raw/current/scripts/components/shadowthrallmanager.lua#L189-L205)、[注册钩子](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L311-L325)、[允许表](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L299-L302)）。 |
| `ReleaseFissure(cooldown)` | 秒 | 释放控制：清刷怪任务、设内部冷却（下限 5s）、`OnReleasedFromControl`、清瘴气、无掉落击杀残存影怪（[L405-L443](../../../raw/current/scripts/components/shadowthrallmanager.lua#L405-L443)、[KillThrall](../../../raw/current/scripts/components/shadowthrallmanager.lua#L390-L403)）。 |
| `SafeToReleaseFissure()` | → bool | 加载宽限 5s 内不许释放；任一影怪在战斗（有目标或 15s 内受过攻击，[TUNING](../../../raw/current/scripts/tuning.lua#L7771)）则不放（[L284-L306](../../../raw/current/scripts/components/shadowthrallmanager.lua#L284-L306)、[L270-L282](../../../raw/current/scripts/components/shadowthrallmanager.lua#L270-L282)）。 |
| `OnDreadstoneMineCooldown(fromload)` / `OnFissureMinedFinished(fissure)` | — | 挖恐石冷却转发给裂缝；挖完后 1 日再恢复可挖（[L470-L490](../../../raw/current/scripts/components/shadowthrallmanager.lua#L470-L490)、[TUNING](../../../raw/current/scripts/tuning.lua#L7773)）。 |
| `FindGoodFissureLocation()` | → x, y, z 或 nil | 世界设置 `fissure ~= "never"` 时复用现有裂缝，否则在处于 `Nightmare` 区域的玩家 30 距离外找无建筑点生成临时 `fissure`（[L87-L124](../../../raw/current/scripts/components/shadowthrallmanager.lua#L87-L124)、[L42-L58](../../../raw/current/scripts/components/shadowthrallmanager.lua#L42-L58)）。 |

## 关键流程

### 竞技场生命周期

1. 触发：影裂裂缝入池 → 记录 thralltype，1.13s 周期 `TickFindingGoodFissures`（内部冷却期间跳过）选最近裂缝或生成临时裂缝并 `ControlFissure`（[L87-L141](../../../raw/current/scripts/components/shadowthrallmanager.lua#L87-L141)）。
2. 展开：裂缝动画完成 → 瘴气环 + 刷怪任务；玩家靠近到 14 内才实际生成三只影怪并互相用 entitytracker 结伴（[L207-L250](../../../raw/current/scripts/components/shadowthrallmanager.lua#L207-L250)）。
3. 释放路径：裂缝被移除（战斗打掉）或所有玩家远离（`UnregisterFissure`）→ 若不安全则起战斗检查任务轮询；任一随从被灭走 `FISSURE_COOLDOWN_DEFEATED_ANY_THRALLS`（3 日），否则 `FISSURE_COOLDOWN_WALKED_AWAY`（1/8 日）（[L170-L187](../../../raw/current/scripts/components/shadowthrallmanager.lua#L170-L187)、[L318-L332](../../../raw/current/scripts/components/shadowthrallmanager.lua#L318-L332)、[tuning](../../../raw/current/scripts/tuning.lua#L7769-L7770)）。
4. 全灭兜底：影怪 onremove 计数到 0 也触发释放（[L353-L388](../../../raw/current/scripts/components/shadowthrallmanager.lua#L353-L388)）。
5. 存档：保存冷却、恐石/刷怪计时余量与裂缝／瘴气／影怪 GUID；未完成展开动画的裂缝不存（[L557-L605](../../../raw/current/scripts/components/shadowthrallmanager.lua#L557-L605)）；LoadPostPass 重绑并按需补 `CheckForNoThralls`（[L625-L659](../../../raw/current/scripts/components/shadowthrallmanager.lua#L625-L659)）。

## 常见陷阱与限制

- **候选池靠实体唤醒**：`RegisterFissure` 在 `OnEntityWake` 才发生，远处休眠裂缝不在池内；选择范围因此被玩家位置约束（[钩子](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L311-L325)）。
- **临时裂缝自生自灭**：世界设置关闭裂缝时生成的 temp fissure 由 `MakeTempFissure` 管理并在释放路径解除注册（[L111-L114](../../../raw/current/scripts/components/shadowthrallmanager.lua#L111-L114)、[DisableTempFissure](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L104-L106)）。
- **释放即无掉落击杀**：残存影怪被 `KillThrall` 清空掉落表后扣血至 0，不会留尸或掉战利品（[L390-L403](../../../raw/current/scripts/components/shadowthrallmanager.lua#L390-L403)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、nightmarefissure 相位/唤醒/动画钩子、corruption 王之杖段、tuning 裂缝段、cave 装配行；未覆盖影怪状态图与掉落表细节（另有 [[reference/components/shadowthrall_mimics]] 主题页）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
