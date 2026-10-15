---
title: "暗影囚牢·影柱法术体系（shadow_pillar）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "麦斯威尔暗影秘典“暗影囚牢”法术的四个注册：玩法柱、柱底地面贴片、非联网定身辅助与 CLASSIFIED 扫描实体的工厂、端、父挂、生成点和清理。"
aliases: ["shadow_pillar", "shadow_pillar_base_fx", "shadow_pillar_target", "shadow_pillar_spell", "暗影囚牢", "Shadow Prison", "影柱", "暗影柱", "影柱法术", "暗影柱法术", "影柱地面贴片"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadow_pillar.lua", "prefabs/waxwelljournal.lua", "prefabs/shadow_trap.lua", "prefabs/trap_vines.lua", "prefablist.lua", "components/rooted.lua", "stategraphs/SGshadow_trap.lua", "stategraphs/SGshrouden.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影囚牢·影柱法术体系（shadow_pillar）

## 快速判断

- 职责与适用：`prefabs/shadow_pillar.lua` 末尾一次返回四个注册，构成暗影秘典“暗影囚牢”法术的整条实体链——`shadow_pillar_spell` 在 4 距离内扫描目标并布柱，`shadow_pillar` 是围住目标的玩法柱，`shadow_pillar_base_fx` 是柱底地面贴片（海面另加客户端涟漪），`shadow_pillar_target` 是隐藏的非联网辅助，给目标临时装 `rooted` 做零移速／零物理质量。[注册](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L777-L780)、[扫描](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L719-L742)、[定身](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482-L486)
- 优先使用：查“影柱／暗影柱／暗影囚牢／影柱定身／影柱地面贴片”选本页。影陷阱与 `shadow_trap_debuff_fx` 归 `shadow_trap.lua` 的另行建页，本页只记共用关系；`rooted` 组件的接口与多来源回收见 [[reference/components/rooted]]；秘典施法时的瞄准圈与联网落点见 [[reference/prefabs/reticuleaoe]]。
- 重要副作用：柱体**故意不加 `FX` 标签**（源码注释：否则 walkableplatform 检测不到），默认带 `ignorewalkableplatforms`；目标在平台上时 `SetTarget` 会移除该标签，让柱子随船移动。扫描时对每个首次命中的目标，布柱前会先推 `dispell_shadow_pillars`，并补一次 0 伤害 `attacked`。[标签](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L293-L298)、[平台](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L227-L231)、[驱散](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L671-L673)
- 修改边界：本页只写实体工厂、端、父挂、生成与清理。施法成本与秘典条目归 waxwelljournal 物品页（Wiki 待建）与 [[reference/components/aoespell|aoespell]]；定身物理归 [[reference/components/rooted|rooted]]；陷阱玩法归 shadow_trap 页。动画包内容、客户端观感、专服实测未核验。

## 注册与构造

四个注册同在一个文件末尾返回，工厂全部是 `CreateEntity` 手工组装，没有 `MakeX` 包装：`shadow_pillar` 与 `shadow_pillar_base_fx` 联网（`AddNetwork`＋`SetPristine`），`shadow_pillar_target` 是唯一不调用 `AddNetwork` 的实体（无动画、无 `SetPristine`），`shadow_pillar_spell` 加 `CLASSIFIED` 标签（同文件注释称其为 hidden entity）。只有 `shadow_pillar` 进 `prefablist.lua`（:1113），另外三名由预制依赖表与 `SpawnPrefab` 按需加载。[返回](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L777-L780)、[依赖表](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L13-L27)、[prefablist](../../../raw/current/scripts/prefablist.lua#L1113-L1113)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_pillar` | `pillar_fn`：Transform＋AnimState＋SoundEmitter＋Network → 标签 `NOCLICK`／`shadow_pillar`／`ignorewalkableplatforms`／`ignorewalkableplatformdrowning`（**故意不加 FX**）→ bank／build `shadow_pillar`、两个符号透明度 0.75 → `SetPristine` → 非主模拟早退；主模拟加 `entitytracker`＋`timer`（`delay` 0）、监听 `timerdone`／`onsink`，暴露 `SetDelay`／`SetTarget` 与存档三回调。 | 联网玩法柱，默认持久化；升起时从 6 个变体里带偏随机、65% 概率翻转，两者都进存档；降下时才置 `persists=false`。[工厂](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L285-L326)、[变体](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L43-L63)、[存档](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L235-L273) |
| `shadow_pillar_base_fx` | `base_fn`：Transform＋AnimState＋Network → FX／NOCLICK → bank／build `shadow_pillar_fx`、播 `pre`、alpha .6、点采样、朝地面、背景层、SortOrder 3 → **非专服** 1 秒后 `TryRipples` → `SetPristine` → 主模拟 `PushAnimation("idle")`、暴露 `KillFX`、`persists=false`。 | 联网地面贴片；客户端也保留实体并播动画。涟漪是客户端本地非联网实体（父挂贴片，背景层、海面混合），专服不生成。[工厂](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L377-L413)、[涟漪](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L330-L370) |
| `shadow_pillar_target` | `target_fn`：只有 Transform＋`ignorewalkableplatforms`（**无网络、无 AnimState、无 SetPristine**）→ `entitytracker`＋`timer`（`delay` 0）→ `timerdone` 监听，暴露 `SetDelay`／`SetTarget` 与存档三回调。 | 非联网隐藏辅助。`SetTarget` 时给目标补 `rooted` 并 `AddSource(inst)`，按 1 秒周期结算攻击者；目标死亡／传送／进 limbo／被驱散／切到带 `flight`／`invisible` 的状态都会 Remove。[工厂](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L568-L592)、[监听与结算](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482-L522) |
| `shadow_pillar_spell` | `spell_fn`：Transform＋SoundEmitter＋Network＋`CLASSIFIED` → `SetPristine` → 非主模拟早退；主模拟播 `shadow_goop_ground` 循环、`StartFX` 每 2 帧试一个环状落点、每 .25 秒 `DoPillars` 扫描、1.25 秒停任务并杀 loop、1.5 秒 Remove、`persists=false`。 | 施法扫描实体（半径 4、只取 `locomotor`；非 PVP 排除 player，PVP 允许玩家目标）；对每个首次命中的目标先推 `dispell_shadow_pillars` 再围一圈柱子；同一施法内每个目标只布一次（`targets` 表跨周期保留）。[工厂](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L749-L773)、[布柱](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L671-L717)、[排除标签](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L720-L723) |

名称核验：简中 `.po` 没有 `STRINGS.NAMES.SHADOW_PILLAR*`（四个注册名均无），唯一命中是法术条目 `STRINGS.SPELLS.SHADOW_PILLARS`＝“暗影囚牢”（msgid “Shadow Prison”）。故标题取官方法术名＋已核验效果描述；正文“影柱”是效果描述，不是官方 prefab 名，`_base_fx`／`_target`／`_spell` 三个内部名也没有官方译名。[po](../../../raw/current/scripts/languages/chinese_s.po#L415349-L415352)

### 生成点与清理

- `shadow_pillar`：唯一生成点在同一文件的 `DoPillarsTarget`——以目标为圆心、按 `GetPhysicsRadius` 加 padding 算半径与柱数，逐点 `SpawnPrefab("shadow_pillar")` 后写入本轮的 `newpillars`；此外只在 `prefablist.lua` 登记、在 `prefabs_spell` 表中预载。[生成](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L699-L712)、[预载](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L22-L27)
- `shadow_pillar_base_fx`：三个生成点——柱体 `PreRaise`（升起前建、父挂柱体、随机旋转）、柱体 `Pillar_OnLoad`（读档补建并按柱动画帧对齐）、`shadow_trap.lua` 陷阱工厂（父挂陷阱并缩放到 0.7）。陷阱页另行维护玩法，本页只共用工厂。[PreRaise](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L120-L127)、[读档](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L255-L261)、[陷阱](../../../raw/current/scripts/prefabs/shadow_trap.lua#L327-L330)
- `shadow_pillar_target`：唯一生成点 `DoPillarsTarget`，位置取目标坐标；`SetDelay` 把延迟拉到最后一个柱子，`SetTarget` 同时传半径与“是否在平台上”。[生成](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L692-L695)
- `shadow_pillar_spell`：唯一生成点 `waxwelljournal` 的 `PillarsSpellFn`，由服务端施法回调生成；在平台上时由秘典把法术实体父挂到平台。[法术](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L37-L54)
- 父挂（逐注册）：柱体与辅助目标都不 `SetParent`，平台上随船靠 `ignorewalkableplatforms` 标签切换；贴片父挂柱体（`PreRaise`／读档）、涟漪父挂贴片；法术实体在平台施法时由秘典父挂平台；`shadow_glob_fx` 在海面且有平台时父挂平台。[柱标签](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L227-L231)、[贴片父挂](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L122-L126)、[秘典](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L44-L50)
- 反查补记：全 `raw/current/scripts` 以四个字面名定向检索，未发现其它生成／消费者；`trap_vines.lua` 只复用同名 `shadow_pillar_fx.zip`／bank，`scrapbook_prefabs.lua` 的相关行被注释。[同名资源](../../../raw/current/scripts/prefabs/trap_vines.lua#L8-L8)
- 清理链（柱）：`delay` 到期 → 8 帧后 `PreRaise` → 再 7 帧 `DoRaise` → `warningtime`（震动＋rumble）→ `lifetime`（`sanity_lower`＋`DoLower`）。`DoLower` 停 rumble、清目标引用、置 `persists=false`、`animover` 自删；2 帧后让贴片 `KillFX`（播 `pst` 后 animover 自删），10 帧后补一次海面水花。被驱散或目标消失时，若仍在延迟升起阶段（`_delayraisetask` 未清或 `delay` 计时仍在）直接 Remove，已升起则按降下处理。[计时](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L130-L163)、[降下](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L84-L118)
- 清理链（辅助目标）：`lifetime` 到期或目标删除时辅助目标直接 Remove，柱子通过目标的 `onremove` 监听收尾；目标移动超过 0.5、或加速扣减时剩余时间小于本次扣减量（`t < dt`）时，先向目标推 `remove_shadow_pillars` 再 Remove（`t == dt` 只缩短并推 `reduce_shadow_pillars_time`，不会立即删除）。柱子收到后停 warning，把 lifetime 随机缩到 1 秒内，或走降下兜底。`rooted` 的来源回收由组件自带的 `onremove` 监听完成，辅助目标删除即解绑。[目标清理](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L444-L480)、[来源监听](../../../raw/current/scripts/components/rooted.lua#L51-L58)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：timer` | 柱体三键计时 `delay`／`warningtime`／`lifetime`；辅助目标两键 `delay`／`lifetime`。所有计时都在主模拟。 | 柱体 `warningtime` 取 `max(0, lifetime - 2.5)`；辅助目标用目标类型算 lifetime。 | [[reference/components/timer]]；[柱计时](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L145-L147) |
| `组件：entitytracker` | 柱体与辅助目标都用 `TrackEntity("target", target)` 记目标引用，供计时、读档与回调清理读取。 | 主模拟；读档由 `OnLoadPostPass` 恢复。 | [[reference/components/entitytracker]]；[记录](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L224-L226) |
| `组件：rooted` | 辅助目标是施加者：目标没有 `rooted` 时补组件，再 `AddSource(inst)`；辅助目标删除时靠组件的来源 `onremove` 监听回收，不需要显式 `RemoveSource`。 | 主模拟；来源是辅助目标实体本身。 | [[reference/components/rooted]]；[安装](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482-L486) |
| `事件契约` | `dispell_shadow_pillars`：法术每轮布柱前推给目标，柱与辅助目标都监听并清理；`ms_escape_rooted`：同作驱散信号；`remove_shadow_pillars`／`reduce_shadow_pillars_time`：辅助目标推给目标，分别结束或缩短柱计时。 | 主模拟；监听对象是目标实体，不是玩家控制器。 | [驱散监听](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L193-L198)、[收缩](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L469-L478) |
| `SG／状态标签` | 辅助目标监听目标的 `newstate`，状态标签含 `flight` 或 `invisible` 时 Remove；`ms_escape_rooted` 在本快照由 `SGshrouden` 的飞行／潜地切换推送。 | 目标 `sg` 为 nil 时不装 `newstate` 监听。 | [newstate](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L496-L502)、[推送点](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L198-L216) |
| `消费者：waxwelljournal` | 秘典条目 `STRINGS.SPELLS.SHADOW_PILLARS`；选中时设 `reticuleaoe`／`reticuleaoeping`、服务端 `SetTargetFX("reticuleaoecctarget")` 与 `PillarsSpellFn`；施法耗燃料与理智。 | 服务端施法；平台父挂。玩法归秘典物品页（Wiki 待建）。 | [[reference/components/aoespell]]；[条目](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L310-L328)、[施法](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L44-L52) |
| `关联 Prefab：shadow_trap` | 只共用 `shadow_pillar_base_fx`：陷阱工厂生成贴片并缩放到 0.7；`shadow_trap_debuff_fx` 是陷阱自己的减速／恐慌贴附表现，与本页四注册无关。施法、触发、驱散流程归该页。 | 陷阱页另行维护；`SGshadow_trap` 全文无 `shadow_pillar` 引用。 | [陷阱工厂](../../../raw/current/scripts/prefabs/shadow_trap.lua#L327-L330)、[陷阱自己的 FX](../../../raw/current/scripts/prefabs/shadow_trap.lua#L383-L416)、[[reference/prefabs/shadow_trap|暗影陷阱与减速表现]] |
| `关联 FX：shadow_glob_fx` | 法术 `TryFX` 沿 12 个环状偏移（1／3／8）每 2 帧试一个点，0.7 内没有 `shadowtrail` 才生成地面痕；海面有平台则父挂平台，否则开涟漪。 | 主模拟生成；实体自行收尾，详见其页面。 | [[reference/prefabs/shadow_glob_fx]]；[布痕](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L599-L644) |
| `关联 FX：sanity_raise／sanity_lower` | `delay` 到期在柱位落 `sanity_raise`，`lifetime` 到期落 `sanity_lower`；两名同页维护。 | 服务端落点，不父挂。 | [[reference/prefabs/sanity_raise]]；[升起](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L134-L135)、[降下](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L157-L159) |
| `关联 FX：ocean_splash_med1／2` | `DoSplash` 在升起与降下（延 10 帧）时检查海面，随机落 med1／2。 | 服务端；`IsOceanAtPoint` 门禁。 | [[reference/prefabs/ocean_splash_med1|ocean_splash_med1／2]]；[水花](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L65-L69) |
| `关联机制：reticuleaoe` | 秘典施法的持续瞄准环／点击提示／联网落点由该族提供，不是影柱实体。 | 客户端环＋服务端落点。 | [[reference/prefabs/reticuleaoe]]；[配置](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L316-L319) |
| `标签／端` | `shadow_pillar` 故意不加 `FX`、`shadow_pillar_target` 故意不加 `CLASSIFIED`，以免被 walkableplatform 过滤；`shadow_pillar_base_fx` 反而带 `FX`。辅助目标没有网络副本，客户端只能看到柱与贴片。 | 柱／贴片联网，辅助目标非联网，法术 `CLASSIFIED`。 | [柱注释](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L293-L298)、[目标注释](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L571-L574) |
| `资源` | 动画：`shadow_pillar.zip`（柱，6 个 pre／idle 变体＋pst＋shake）、`shadow_pillar_fx.zip`（贴片，`ALT_RENDERPATH`）、`splash_weregoose_fx.zip`＋`splash_water_drop.zip`（涟漪）；音效：`maxwell_rework/shadow_pillar/pre`、`rumble`、`dontstarve/sanity/shadowrock_down`、`maxwell_rework/shadow_magic/shadow_goop_ground`。 | 声明与播放已核验；包内容与时长未解包。 | [资源表](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L1-L11) |

## 典型接入

### 给自定义柱体配同款地面贴片

- 前提与选择：宿主是服务端实体且带 Transform；直接复用 `shadow_pillar_base_fx` 可得到原版 pre→idle→pst 的节奏与海面涟漪，不必自建 FX。只应在柱体升起时生成、降下时让贴片收尾。
- 生效与撤销：`SetParent(宿主)` 后贴片随宿主移动／删除；`KillFX` 播 `pst`，animover 自删。涟漪在非专服由客户端按“父实体带 `ignorewalkableplatforms` 且脚下是海”补建。
- 原版依据：[影柱 PreRaise](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L120-L127) 与 [shadow_trap 工厂](../../../raw/current/scripts/prefabs/shadow_trap.lua#L327-L330)。

```lua
-- 服务端：升起前挂贴片，降下时收尾（对应原版 PreRaise / DoLower）
local base = SpawnPrefab("shadow_pillar_base_fx")
base.entity:SetParent(inst.entity)
base.Transform:SetRotation(math.random() * 360)
-- 陷阱那档小尺寸：base.Transform:SetScale(.7, .7, .7)
inst.base = base

-- 降下：与柱体 pst 对齐，2 帧后让贴片播 pst 并自删
inst.base:DoTaskInTime(2 * FRAMES, inst.base.KillFX)
```

- 调整与验证：缩放用 `Transform:SetScale`（原版陷阱为 0.7）；随机旋转只影响贴片朝向；`KillFX` 只在主模拟副本存在，客户端靠网络移除。示例仅做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`shadow_pillar` 只是视觉**：源码注释写的是“purely visual FX”，但它故意不加 `FX` 标签、参与平台检测；实际定身由非联网 `shadow_pillar_target` 给目标挂 `rooted` 完成，柱体负责位置、计时与表现。[注释](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L41-L41)、[定身](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482-L486)
- **误区：1.5 秒后柱子消失**：`shadow_pillar_spell` 在 1.25 秒停 `DoPillars` 扫描任务，1.5 秒自删只是移除施法实体（`TryFX` 也随之结束）；每根柱按 `lifetime` 独立计时（玩家 6／Boss 12／默认 24 秒），可被驱散或缩短，但不会随施法实体一起消失。[计时](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L145-L147)、[自删](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L766-L768)
- **边界：时长按目标类型分档，攻击者会加速**：`CalcTargetDuration` 先看 `epic`（12 秒）再看 `player`（6 秒），其余 24 秒；`warningtime = 剩余 - 2.5`。辅助目标每 1 秒按不重复的玩家攻击者数结算一次（非玩家攻击者记为 `other`，玩家数为 0 时按 1 计）；`SHADOW_PILLAR_BREAK_MULT` 当前 `MIN=MAX=1.5`，计数 ≥1 时每 1 秒额外扣 0.5 秒，人数上限按 4 计。[tuning](../../../raw/current/scripts/tuning.lua#L2834-L2841)、[结算](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L441-L479)
- **边界：环绕点位过滤的运算符优先级**：`not IsNearOther(...) and IsPassableAtPoint(...) and flying or (平台一致) and not IsGroundTargetBlocked(...)` 实际是 `(A and B and flying) or (C and D)`；对地面／平台目标，1.0 的 `newpillars` 去重与可通行检查只在 `flying` 分支参与，另一支路只看平台一致与 `IsGroundTargetBlocked`。[点位](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L699-L711)
- **边界：重叠柱子会互驱**：新柱 `delay` 到期时以 0.9 半径找带 `shadow_pillar` 标签且 `persists` 的其它柱，逐根 `Pillar_OnDispell`；这是跨目标／跨施法者的兜底路径，原版重施替换旧柱的主路径是 `DoPillarsTarget` 推 `dispell_shadow_pillars` 事件，与同一轮点位去重不是一层。[互驱](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L137-L143)、[重施驱散](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L673-L680)
- **边界：辅助目标在客户端不存在**：它没有 `AddNetwork`，并故意不加 `CLASSIFIED`（源码注释：加了 walkableplatform 就检测不到）。模组在客户端按标签找它必然落空；服务端它也不带 `FX`，按纯视觉标签筛选不会命中。需要引用时只能在主模拟按实体句柄追踪。[注释](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L571-L574)
- **边界：涟漪由父实体标签门控**：`TryRipples` 要求父实体带 `ignorewalkableplatforms`（即柱子不在船上）且脚下是海；在船上时柱体会移除该标签，贴片不再补涟漪。任务只在非专服起。[涟漪](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L356-L370)、[平台](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L227-L231)
- **边界：布柱会对命中目标推 0 伤害 `attacked`**：用于触发受击反应／仇恨；目标处于 `noattack` 状态标签时跳过。施法前先 `dispell_shadow_pillars`，所以重复施法是重置而不是叠柱。[attacked](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L714-L716)
- **边界：读档依赖实体引用与计时器状态**：柱与辅助目标都用 `entitytracker` 记目标 GUID，读档在 `OnLoadPostPass` 重建监听；辅助目标在目标丢失或计时器既无 `lifetime` 也无 `delay` 的坏数据时会直接删除实体，柱体读档在目标为 nil 时不删除，只是不再重建监听并继续自身计时。[柱读档](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L275-L283)、[目标读档](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L553-L566)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。逐行核验 `prefabs/shadow_pillar.lua` 780 行（四注册、三类端形态、计时链、父挂、读档与清理）、`waxwelljournal.lua` 的法术入口与平台父挂、`shadow_trap.lua` 的贴片复用、`SGshadow_trap.lua` 全文无 `shadow_pillar` 引用、`SGshrouden.lua` 的 `ms_escape_rooted` 推送、`rooted.lua` 的来源回收、`tuning.lua` 的时长参数与 `prefablist.lua` 的注册落点。全 `raw/current/scripts` 以四个注册名字面反查：`shadow_pillar` 命中自身、`prefablist`、`shadow_trap`（仅贴片）、`trap_vines`（仅同名动画）、`waxwelljournal`（仅 `_spell`）与 `scrapbook_prefabs`（注释）；`shadow_pillar_base_fx` 命中自身与 `shadow_trap`；`shadow_pillar_target`／`shadow_pillar_spell` 分别只命中自身（后者另在 `waxwelljournal` 生成）。`.po` 定向检索：无 `STRINGS.NAMES.SHADOW_PILLAR*`，仅 `STRINGS.SPELLS.SHADOW_PILLARS`＝“暗影囚牢”。本页一文件四注册，跨联网／非联网／`CLASSIFIED` 三种形态，端、父挂、计时与清理互不相同，需整体对照，按 `complex` 档。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：`validate_page.py` 通过（0 issue；109 行、约 23 KiB，`complex` 档内）；`validate_source_links.py --strict` 通过（70 处行引用、0 error／0 warning）；Lua 片段 `parser_probe.exe --syntax-only` 通过（只查语法、未执行，临时文件已删）。
- 未覆盖：动画 zip 内容与时长、客户端实际观感、专服与晚加入、读档中途表现、法术周期内实际扫描次数与客户端涟漪帧对齐、秘典 UI 与图标。

返回 [[reference/prefabs/index|Prefab 索引]]；外围 FX 分组见 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]。
