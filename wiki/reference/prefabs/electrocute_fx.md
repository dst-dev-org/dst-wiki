---
title: "电击表现与连锁闪电（electrocute_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "电击反应的表现叠加层：挂在被电目标上按体型播 shock 动画并闪黄白，主模拟延迟生成 shock_arc_fx 连锁附近的可电击单位；触发链来自 electric stimuli 攻击与闪电，本实体不结算伤害。"
aliases: ["electrocute_fx", "电击特效", "电击表现", "shock_fx 电击", "连锁闪电", "电击连锁"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/electrocute_fx.lua", "stategraphs/commonstates.lua", "componentutil.lua", "stategraphs/SGwilson.lua", "components/playerlightningtarget.lua", "prefabs/lightninggoat.lua", "stategraphs/SGwx78_possessedbody.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 电击表现与连锁闪电（electrocute_fx）

## 快速判断

- 职责与适用：生物被电（武器 electric stimuli、electricattacks 攻击者、闪电）后的表现层＋连锁：`SetFxTarget` 把本特效挂到目标上按体型播 `shock_{size}[_{height}]` 动画，并延迟 11 帧向附近可电击单位劈 `shock_arc_fx`、推 `electrocute` 事件形成连锁（[SetFxTarget](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L135-L171)、[DoFork](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L29-L92)）。
- 优先使用：查"怎么被电／怎么连锁"看触发链小节；生物要接入电击反应在状态图里用 `CommonStates.AddElectrocuteStates`＋`CommonHandlers.OnElectrocute`，不要直接生成表现层。
- 重要副作用：主模拟经目标 colouradder 闪黄白（r=g=c、b=c/2）；调用 SetFxTarget 必须传数值 duration（选音效直接比较该参数）；带 `attackdata` 时才 fork；连锁计数只认进入 `electrocute` 状态的目标。[闪色](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L96-L110)、[计数](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L81-L87)
- 修改边界：伤害由攻击方／闪电另行结算，本实体无 combat；fork 过滤、targets 共享表均主模拟；玩家的电击走 SGwilson 专属状态而非本实体（见触发链）。[玩家路径](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3417-L3428)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `electrocute_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `shock_fx`、播 `shock_small`（显式循环 `true`）、bloom、FinalOffset 1、FX＋NOCLICK → SetPristine → 主模拟监听 `animover`、挂 [[reference/components/updatelooper|updatelooper]]、`persists=false`、暴露 SetFxTarget／CancelFlash | 单一注册；动画资源按体型动态切换，`_pst` 由时长任务或 animover 补播。[工厂](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L188-L221)、[注册](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L223) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 体型与动画：GetCombatFxSize／GetElectrocuteFxAnim | `SetFxTarget` 取目标的物理半径＋小／大生物标签得 size（tiny/small/med/large）与 height（low/high/nil），拼 `shock_{size}[_{height}]` 并显式循环播放；50% 概率镜像缩放，并按目标缩放取倒数抵消父级缩放 | size 可被 `override_combat_fx_size` 等字段覆写；height 依水生／飞行／有无 electrocute 状态推断 | [GetCombatFxSize](../../../raw/current/scripts/componentutil.lua#L959-L990)、[GetElectrocuteFxAnim](../../../raw/current/scripts/componentutil.lua#L992-L994)、[应用](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L140-L152) |
| 触发链：attacked（electric） | 攻击 stimuli 为 electric（weapon stimuli 或 [[reference/components/electricattacks|electricattacks]] 攻击者）→ `CommonHandlers.OnAttacked` → `try_electrocute_onattacked`（绝缘、noelectrocute/nointerrupt 态、恢复延迟过滤）→ `try_goto_electrocute_state`：有 electrocute 态进状态（状态内 spawn）；无则直接 spawn＋进 hit 态，`sg.mem.burn_on_electrocute` 时点燃 | 免疫标签 electricdamageimmune 不在此过滤（注释说明会破坏玩家行为）；`CalcEntityElectrocuteDuration` 默认 0.8 秒、burn_on_electrocute 缩到 0.4 秒 | [attack_can_electrocute](../../../raw/current/scripts/stategraphs/commonstates.lua#L195-L213)、[try_goto_electrocute_state](../../../raw/current/scripts/stategraphs/commonstates.lua#L238-L278)、[时长](../../../raw/current/scripts/componentutil.lua#L1003-L1018) |
| 触发链：electrocute 事件 | 挂 `CommonHandlers.OnElectrocute` 的生物 SG 监听 `electrocute` 事件 → `try_electrocute_onevent` → 同上进入电击；本实体 DoFork 对连锁目标 `PushEventImmediate("electrocute", data)` 即走此路 | 事件带 attackdata 才继续 fork；`lightninggoat` 借同一事件直接 `setcharged`（电击＝充能） | [OnElectrocute](../../../raw/current/scripts/stategraphs/commonstates.lua#L344-L346)、[onevent](../../../raw/current/scripts/stategraphs/commonstates.lua#L327-L333)、[闪电羊充能](../../../raw/current/scripts/prefabs/lightninggoat.lua#L161-L163)、[监听注册](../../../raw/current/scripts/prefabs/lightninggoat.lua#L285) |
| 触发链：闪电 | 地面落雷 AOE：`StrikeLightningAtPoint` 对可电击（CanEntityBeElectrocuted）且非玩家的实体调 `LightningStrikeAttack`——直接扣血并推无 attackdata 的 `electrocute`（不 fork）；玩家接雷走 [[reference/components/playerlightningtarget|playerlightningtarget]] → 同函数，但 SGwilson 无 `electrocute` 事件监听，只掉血不进电击态 | `CanEntityBeElectrocuted`＝SG 有 electrocute 态或 sg.mem.burn_on_electrocute，且非 sg.mem.noelectrocute | [LightningStrikeAttack](../../../raw/current/scripts/componentutil.lua#L1032-L1052)、[AOE 门控](../../../raw/current/scripts/componentutil.lua#L1065-L1090)、[玩家接雷](../../../raw/current/scripts/components/playerlightningtarget.lua#L4-L9) |
| 连锁 fork | DoFork：范围 `ELECTROCUTE_FORK_RANGE(3)+r+PADDING(3)` 查注册标签实体，过滤后生成 `shock_arc_fx` 居中指向、`PushEventImmediate("electrocute", data)`；玩家攻击者且非 PVP 的目标组排除 player，怪物组可电玩家 | numforks 默认 2，逐级递减；`ELECTROCUTE_FORK_DELAY=11` 帧；每轮最多 `FORK_TARGETS=2` 个新目标（进 electrocute 态才计数） | [DoFork](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L29-L92)、[StartFork](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L126-L133)、[数值](../../../raw/current/scripts/tuning.lua#L8654-L8661) |
| 玩家专属路径 | SGwilson 的 `electrocute` 状态播 legacy `shock` 动画并在脚下挂旧 `shock_fx` 实体（另一预制，[[reference/prefabs/shock_fx|shock_fx 族页]]），再 `StartElectrocuteForkOnTarget` 复用本文件的 fork 逻辑；SGwx78_possessedbody 同构 | 玩家不生成本实体；fork 需 attackdata | [玩家状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3405-L3428)、[fork 调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3457)、[同构状态](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L3026) |
| 音效 | 按 size 选 4 档，时长 ≤0.4 秒用短版（*_sml/med/big），否则加长版 | 由 SetFxTarget 依 duration 选择 | [音效表](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L8-L22)、[选择](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L154-L155) |
| 组件：colouradder | 闪白黄 Push/Pop（b=c/2）；目标无该组件时补挂 | 主模拟；客户端可见性同 electricchargedfx 的边界（颜色更新可走 colouraddersync 或 AnimState，实际客户端效果未运行验证） | [OnUpdate](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L159-L165)、[[reference/components/colouradder]] |
| API／数据：StartElectrocuteForkOnTarget | 全局函数，legacy 玩家 shock 表现复用 fork；参数 target＋data（需 attackdata） | 主模拟调用 | [定义](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L174-L178) |

## 典型接入

### 给生物接入"可被电击＋连锁"

- 前提与选择：SG 注册 `CommonStates.AddElectrocuteStates(states, timelines, anims)`（有专属电击动画时传 anims），并在 SG 里加 `CommonHandlers.OnElectrocute()` 与 `CommonHandlers.OnAttacked()`；是否被电由 `CanEntityBeElectrocuted`（有 electrocute 态或 `sg.mem.burn_on_electrocute`）决定（[AddElectrocuteStates](../../../raw/current/scripts/stategraphs/commonstates.lua#L1509-L1553)）。
- 生效与撤销：电击态 onenter 即生成 electrocute_fx 并按时长 timeout，`electrocute_pst` 收尾；onexit 依 `sg.mem.burn_on_electrocute` 补点燃；表现层随目标移除自动清理（[状态](../../../raw/current/scripts/stategraphs/commonstates.lua#L1509-L1568)）。
- 原版依据：牛／蚁狮／蝙蝠等直接注册三件套（如 SGBeefalo）即获得连锁与表现。

```lua
-- SG 文件中，states/events 为该文件已有的状态与事件数组。
require("stategraphs/commonstates")
CommonStates.AddElectrocuteStates(states)
-- 仅在尚未注册这两类处理器时添加；已有 OnAttacked 时保留它。
table.insert(events, CommonHandlers.OnAttacked())
table.insert(events, CommonHandlers.OnElectrocute())
```

- 调整与验证：不想被电可直接 `sg.mem.noelectrocute = true`（CanEntityBeElectrocuted 与事件路径都会拦）；怪物想让电击能波及玩家，攻击时不要以玩家身份施加 electric stimuli（目标组会换掉 player，见陷阱）。

## 常见陷阱与限制

- **误区：生成本实体会造成伤害**：本实体只管表现与事件转发，扣血/点燃在攻击结算与 LightningStrikeAttack；给目标挂本特效不会让它"掉血抽搐"。[职责分界](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L96-L110)
- **边界：DoFork 的死亡过滤是空转**：`IsEntityDead(inst)` 传入的是表现层自身而非候选实体（fx 无 health replica，恒 false），死亡目标主要靠 `noelectrocute` 态标签／limbo／ally 过滤兜底（[过滤行](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L62-L70)、[IsEntityDead 定义](../../../raw/current/scripts/componentutil.lua#L7-L13)）。
- **边界：PVP 决定连锁能否电玩家**：攻击者是玩家（或玩家随从）且 PVP 未开时用排除 player 的标签组；怪物电击、或 PVP 开启时，连锁可以点玩家（[两组标签](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L37-L57)）。
- **边界：`_pst` 依赖 animover**：从未 SetFxTarget 的裸实体初始动画显式循环，不能保证收到 animover 或自行退场；OnAnimOver 仅在实际收到事件时补播 `_pst` 并等后续事件移除；SetFxTarget 过的目标改由时长任务在 `duration-0.1+rand*0.2` 秒切 `_pst`（[OnAnimOver](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L112-L119)、[调度](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L157)）。
- **边界：玩家的电击不走本实体**：SGwilson 有专属 `electrocute` 状态与 legacy shock_fx；给玩家 SpawnPrefab("electrocute_fx") 只会得到表现层，没有玩家的眩晕暂停／bloom 逻辑（[玩家状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3405-L3428)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；工厂全文（SetFxTarget／DoFork／闪色／音效表）、commonstates 的攻击与事件两条触发链、AddElectrocuteStates、componentutil 的体型／时长／闪电函数、SGwilson 与 SGwx78_possessedbody 的玩家路径、闪电羊监听、tuning 数值已核验；shock_arc_fx 工厂、shock_fx（legacy）工厂、各生物 SG 具体电击动画差异未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
