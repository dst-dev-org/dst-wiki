---
title: "一次性碎裂与爆裂表现族（shatter、splash_spiderweb 等 8 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "8 个源文件里的 8 个一次性表现注册：冻结碎裂（shatter 的等级壳＋本地动画双端代理）、蛛网溅落（脚本层无生成点）、贝壳小生物、木牌碎裂、南瓜炸弹爆裂（含确定性 PRNG 本地碎块）、风滚草破碎、工具损坏代理与亮茄炸弹爆裂；宿主破坏／爆炸回调生成、animover 自删为主。"
aliases: ["shatter", "splash_spiderweb", "singingshell_critterfx", "propsignshatterfx", "pumpkin_bomb_explode_fx", "tumbleweedbreakfx", "brokentool", "bomb_lunarplant_explode_fx", "碎裂特效", "冻结碎裂", "爆裂表现", "蛛网溅落", "工具损坏", "风滚草破碎", "南瓜炸弹", "亮茄炸弹"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/shatter.lua", "components/shatterfx.lua", "components/freezable.lua", "standardcomponents.lua", "prefabs/slingshotammo.lua", "prefabs/staff_projectile.lua", "prefabs/staff_common.lua", "prefabs/splash_spiderweb.lua", "prefabs/spider_web_spit.lua", "prefabs/singingshell.lua", "fx.lua", "prefablist.lua", "prefabs/propsign.lua", "prefabs/pigking.lua", "prefabs/pumpkin_bomb.lua", "prefabs/tumbleweed.lua", "prefabs/tumbleweedspawner.lua", "prefabs/brokentool.lua", "components/pocketwatch_dismantler.lua", "prefabs/pocketwatch.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhermitcrab.lua", "stategraphs/SGwx78_possessedbody.lua", "prefabs/bomb_lunarplant.lua", "components/explosive.lua", "stategraphs/SGworm_boss_head.lua", "prefabs/merm.lua", "prefabs/amulet.lua", "prefabs/hound.lua", "prefabs/deerclops.lua", "prefabs/crabking.lua", "prefabs/daywalker2.lua", "components/pinnable.lua", "widgets/werebadge.lua", "simutil.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 一次性碎裂与爆裂表现族（shatter、splash_spiderweb 等 8 注册）

## 快速判断

- 职责与适用：8 个源文件里的 **8 个一次性表现注册**，共性是"宿主生命周期回调生成、播完即删、零玩法组件"：`shatter`（冻结碎裂，等级壳＋本地动画双端代理）、`splash_spiderweb`（蛛网溅落，**脚本层无生成点**）、`singingshell_critterfx`（贝壳敲碎后的小生物表演）、`propsignshatterfx`（猪王竞技场木牌碎裂）、`pumpkin_bomb_explode_fx`（南瓜炸弹爆裂＋本地碎块雨）、`tumbleweedbreakfx`（风滚草采集／烧尽破碎）、`brokentool`（工具损坏代理）、`bomb_lunarplant_explode_fx`（亮茄炸弹平面伤害爆裂）。注册行：[shatter](../../../raw/current/scripts/prefabs/shatter.lua#L93)、[splash_spiderweb](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L54)、[singingshell_critterfx](../../../raw/current/scripts/prefabs/singingshell.lua#L333)、[propsignshatterfx](../../../raw/current/scripts/prefabs/propsign.lua#L223)、[pumpkin_bomb_explode_fx](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L330)、[tumbleweedbreakfx](../../../raw/current/scripts/prefabs/tumbleweed.lua#L518)、[brokentool](../../../raw/current/scripts/prefabs/brokentool.lua#L49)、[bomb_lunarplant_explode_fx](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L297)
- 三种载体模式：①**无 AnimState 壳＋本地动画代理**（shatter、splash_spiderweb、brokentool——壳只有 Transform／Network，各端延 1 帧自建非联网动画）；②**壳自播动画**（其余 5 注册——AnimState／SoundEmitter 在联网壳上，主模拟驱动随网络复制）；③②的变体：`pumpkin_bomb_explode_fx` 额外在各端用**确定性 PRNG** 自建本地碎块（无 netvar 同步）。
- 重要副作用：壳的清理以 `animover` 自删为主，壳另带 1 秒定时兜底（shatter／splash_spiderweb／brokentool）或动画长度兜底（tumbleweedbreakfx）；南瓜碎块走 `ErodeAway` 侵蚀链。全部 8 注册零伤害：伤害都在宿主侧结算（爆炸走 explosive 组件、冻结走 freezable 的寒冷值）。
- 修改边界：只覆盖 8 个注册的工厂、生成点、端模型与清理；宿主玩法（炸弹配方与投掷、贝壳演奏、木牌竞技场规则、风滚草掉落表）只在条目内辨析不展开。`float_fx_front`／`float_fx_back`、`splash_green_small`、`erode_ash`、`ash`、`splash_sink` 等被连带生成的通用 FX 不属于本页覆盖。
- 名称：`PUMPKIN_BOMB`＝"南瓜炸弹"、`BOMB_LUNARPLANT`＝"亮茄炸弹"、`TUMBLEWEED`＝"风滚草"、`SINGINGSHELL_OCTAVE3/4/5`＝"低音／中音／高音贝壳钟"、`HOMESIGN`＝"木牌"（`propsign` 的显示名 override 目标）均有官方名（锚点见[验证记录](#验证记录)）；8 个覆盖注册名（`SHATTER`、`SPLASH_SPIDERWEB`、`SINGINGSHELL_CRITTERFX`、`PROPSIGNSHATTERFX`、`PUMPKIN_BOMB_EXPLODE_FX`、`TUMBLEWEEDBREAKFX`、`BROKENTOOL`、`BOMB_LUNARPLANT_EXPLODE_FX`）无 `STRINGS.NAMES.*` 键，标题与正文中的效果描述**非官方译名**。

## 注册与构造

### 无动画壳＋本地动画代理（shatter、splash_spiderweb、brokentool）

| 注册名 | 壳构造 | 本地动画（每台非专服机器自建） | 差异与证据 |
|---|---|---|---|
| `shatter` | Transform／Network、`SetTwoFaced`、`FX` 标签、`net_tinybyte _level`（`leveldirty`）；非专服监听 dirty；主模拟把 shatterfx 组件的 `SetLevel` 重载为写 netvar；`persists=false`＋1 秒自删 | 延 1 帧 `PlayShatterAnim`：非联网、`SetCanSleep(false)`、**父挂到壳的父实体**、bank／build `frozen_shatter`、FinalOffset 3、播 `break_iceblock` 音；shatterfx 组件按 `proxy._level` 取 tiny→huge 五档动画；`animover` 自删 | 唯一带"等级门控"的注册：level≤0 或已放过（`_complete`）就不播。[壳](../../../raw/current/scripts/prefabs/shatter.lua#L57-L91)、[netvar](../../../raw/current/scripts/prefabs/shatter.lua#L67)、[本地](../../../raw/current/scripts/prefabs/shatter.lua#L15-L45)、[等级表](../../../raw/current/scripts/prefabs/shatter.lua#L6-L13) |
| `splash_spiderweb` | Transform／Network、`FX` 标签；非专服延 1 帧；主模拟 `persists=false`＋1 秒自删 | 非联网、`SetCanSleep(false)`、**不父挂**、bank／build `splash_spiderweb`、播 `idle`、FinalOffset 3、无声音；`animover` 自删 | 三者中唯一无 netvar、无声音的；生成点缺口见[生成点表](#生成点与宿主调用)。[壳](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L27-L52)、[本地](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L6-L25) |
| `brokentool` | Transform／Network、**无 `FX` 标签、无 IsDedicated 门控**（延迟建本地动画的代码在所有机器执行，含专用服务器）；主模拟 `persists=false`＋1 秒自删 | 非联网、`SetCanSleep(false)`、**带 `MakeInventoryPhysics` 物理体**、bank `broketool`／build `broken_tool`、播 `used`；`animover` 自删 | 三者中端分界最松的一个；名称像"损坏的工具物品"，实际是纯表现实体（无 inventoryitem）。[壳](../../../raw/current/scripts/prefabs/brokentool.lua#L28-L47)、[本地](../../../raw/current/scripts/prefabs/brokentool.lua#L6-L26) |

### 壳自播动画（singingshell_critterfx、propsignshatterfx、pumpkin_bomb_explode_fx、tumbleweedbreakfx、bomb_lunarplant_explode_fx)

| 注册名 | 工厂壳 | 差异与证据 |
|---|---|---|
| `singingshell_critterfx` | Transform／AnimState／SoundEmitter／Network；bank `singingshell_creature`、build `singingshell_creature_basic`；随机播 `break_1/2/3`；标签 `NOCLICK`＋`FX`；主模拟 `persists=false` | 动画状态链最长：break→`jump_pre`→`jump_lag`→`jump_post`（挖掘帧生成 wood／rock 变体）→`jump_post` 结束自删；按随机排 1–2 条"受惊"音。[工厂](../../../raw/current/scripts/prefabs/singingshell.lua#L236-L328)、[状态链](../../../raw/current/scripts/prefabs/singingshell.lua#L274-L308) |
| `propsignshatterfx` | Transform／AnimState／SoundEmitter／Network；bank／build `sign_elite`、播 `shatter` 一次；标签 `FX`＋`NOCLICK`；主模拟 `animover` 自删＋`persists=false` | 最简注册：无变体、无本地子实体、无兜底。[工厂](../../../raw/current/scripts/prefabs/propsign.lua#L196-L220) |
| `pumpkin_bomb_explode_fx` | Transform／AnimState／SoundEmitter／Network；bank／build `pumpkin_bomb`、播 `used`；标签 `FX`＋`NOCLICK`；主模拟 0 秒播双爆炸声、`animover` 自删、`persists=false`；`OnEntityWake` 各端触发一次碎块雨 | 碎块雨见下方小节；两声爆炸叠加（黑火药＋蛋壳裂）。[工厂](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L296-L325)、[唤醒抛块](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L291-L294) |
| `tumbleweedbreakfx` | Transform／AnimState／Network、`SetFourFaced`；bank／build `tumbleweed`、播 `break`；标签 `FX`（**无 NOCLICK**）；主模拟 `persists=false`＋`animover` 自删＋**动画长度＋1 帧定时兜底** | 唯一写明兜底原因的注册（注释：离屏时动画睡眠不触发 animover）。[工厂](../../../raw/current/scripts/prefabs/tumbleweed.lua#L394-L421)、[兜底](../../../raw/current/scripts/prefabs/tumbleweed.lua#L416-L418) |
| `bomb_lunarplant_explode_fx` | Transform／AnimState／SoundEmitter／Network；bank／build `bomb_lunarplant`、播 `used`、`light_beam`／`pb_energy_loop` 双符号 Bloom＋LightOverride、`sleepcloud_pre` 符号借 sleepcloud build；标签 `FX`＋`NOCLICK`；主模拟 0 秒播爆炸声、`animover` 自删、`persists=false` | 无本地子实体；本体投掷期的白化与本地旋转核心是**本体**（`bomb_lunarplant`）的 netvar＋子实体系统，不属于本 FX。[工厂](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L260-L292)、[本体核心](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L123-L136) |

### pumpkin_bomb_explode_fx 的确定性本地碎块子系统

`fx_TossChunks`（[实现](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L262-L289)）以 `PRNG_Uniform(⌊x+0.5⌋·⌊z+0.5⌋·网络ID)` 播种——坐标先加 0.5 再取整；同位置同一实体在每台机器算出**同一串**块数（6–7）、角度、速度与贴图变体，不依赖 netvar：各端各自 `CreateEntity` 建 6–7 个非联网碎块（[CreatePumpkinChunk](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L194-L216)，`debris_01`–`debris_06` 随机镜像抛出），12–14 帧后查落水（[fx_TestChunkFloat](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L236-L250)：落水则自建 `CreateSplash` 本地水花＋在壳上播鸟落水声，并给碎块父挂联网的 `float_fx_front`／`float_fx_back` 浮层），1.5–1.75 秒后 `fx_DoErode` 对碎块与浮层各自 `ErodeAway`（[链](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L252-L260)）。`CreatePumpkinChunk` 与 `CreateSplash` 是 `CreateEntity` 直构的**未注册内部实体**，不是 Prefab。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `shatter` | `SpawnPrefab(v.prefab)`（v.prefab＝"shatter"）：[freezable.SpawnShatterFX](../../../raw/current/scripts/components/freezable.lua#L96-L113)；直接 `SpawnPrefab("shatter")`：[slingshotammo.lua#L287](../../../raw/current/scripts/prefabs/slingshotammo.lua#L283-L290)、[staff_projectile.lua#L18](../../../raw/current/scripts/prefabs/staff_projectile.lua#L16-L22) | freezable 链见[下节](#shatter-的等级系统与-freezable-消费链)；两条直接路径都是"目标不可冻结"时的兜底（冰弹／冰弹投射物直落 `SetLevel(2)`） |
| `splash_spiderweb` | **无**（诚实缺口） | 全 `raw/current/scripts` 反查仅 3 处：本文件、[prefablist.lua#L1209](../../../raw/current/scripts/prefablist.lua#L1209)（文件清单）、[spider_web_spit.lua#L10](../../../raw/current/scripts/prefabs/spider_web_spit.lua#L6-L11)（依赖声明，旁边挂着源码作者自己的疑问注释）。检索模式：`grep -rn "splash_spiderweb" --include=*.lua`；消费代码可能在快照外，不判死代码 |
| `singingshell_critterfx` | [singingshell.lua#L76](../../../raw/current/scripts/prefabs/singingshell.lua#L74-L79) | 唯一生成点：贝壳钟本体被锤 1 击的 `onfinishwork`（掉落壳片→生成本 FX→`Remove` 本体） |
| `propsignshatterfx` | [propsign.lua#L39](../../../raw/current/scripts/prefabs/propsign.lua#L30-L43) | 唯一生成点：本体收 `propsmashed` 事件（猪王竞技场／玩家投掷命中，[推送侧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11267-L11267)）→ 生成 FX＋木牌碎裂声＋本体 Remove。本体进场：猪王竞技场补道具（[pigking.lua#L354](../../../raw/current/scripts/prefabs/pigking.lua#L349-L356)） |
| `pumpkin_bomb_explode_fx` | [pumpkin_bomb.lua#L56](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L48-L57) | 唯一生成点：本体 `complexprojectile` 命中回调 `OnHit`（先让 explosive 组件爆炸并自删本体，再在原位生成 FX） |
| `tumbleweedbreakfx` | [tumbleweed.lua#L142](../../../raw/current/scripts/prefabs/tumbleweed.lua#L108-L145)、[tumbleweed.lua#L274](../../../raw/current/scripts/prefabs/tumbleweed.lua#L263-L276) | 两条都在本体文件内：①玩家采集风滚草（`pickable.onpickupfn`，本体当场撒掉落并 Remove）；②烧尽收尾 `spawnash`（本体 onburnt 1.2 秒后生成 `ash`＋本 FX 再 Remove）。本体由 [tumbleweedspawner.lua#L15](../../../raw/current/scripts/prefabs/tumbleweedspawner.lua#L15) 的 childspawner 持续补充 |
| `brokentool` | [pocketwatch_dismantler.lua#L29](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L21-L36)、[pocketwatch.lua#L139](../../../raw/current/scripts/prefabs/pocketwatch.lua#L135-L143)、[SGhermitcrab.lua#L3703](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L3691-L3703)、[SGwilson.lua#L15057](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15045-L15057)、[SGwx78_possessedbody.lua#L2361](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L2349-L2361) | 五处、三类场景：①怀表拆解（返还配方材料＋坏工具代理）；②作祟复活：`Revive_OnHaunt` 仅在作祟者带 `pocketwatchcaster` 且 `CastSpell` **成功**时掉落、生成 brokentool 并删除怀表，失败只 `Launch`；③SG 的 `toolbroke` 状态（播动画 `hit`）——工具用坏瞬间在玩家位置放代理；后一处三个 SG 共用同一 `tool.nobrokentoolfx` 门控（[propsign 设真](../../../raw/current/scripts/prefabs/propsign.lua#L185-L185)以免木牌双重出 FX） |
| `bomb_lunarplant_explode_fx` | [bomb_lunarplant.lua#L49](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L41-L50) | 唯一生成点：本体 `OnHit`（同南瓜炸弹结构）。本体进场：月石锻造台配方（[recipes.lua#L850](../../../raw/current/scripts/recipes.lua#L850)）；另可被蠕虫 Boss 咀嚼引爆（[SGworm_boss_head.lua#L230-L238](../../../raw/current/scripts/stategraphs/SGworm_boss_head.lua#L230-L238)） |

### shatter 的等级系统与 freezable 消费链

注册方在 `standardcomponents.lua` 的 `shatterfx.character = "shatter"`（[表](../../../raw/current/scripts/standardcomponents.lua#L323-L326)）被 5 个 `Make*FreezableCharacter` helper（tiny／small／medium／large／huge，对应 FX 等级 1–5）写入 freezable 组件的 `fxdata`（[helper 区](../../../raw/current/scripts/standardcomponents.lua#L328-L369)、[登记](../../../raw/current/scripts/components/freezable.lua#L81-L83)）。`SpawnShatterFX` 逐条生成 `shatter`：带跟随符号的走 `Follower:FollowSymbol`，否则 `AddChild` 挂为宿主 Lua 子实体再定位，最后调壳的 shatterfx 组件 `SetLevel(宿主等级)`（[分发](../../../raw/current/scripts/components/freezable.lua#L96-L113)）。调用点（消费文件）：解冻（[freezable.lua#L290](../../../raw/current/scripts/components/freezable.lua#L290-L290)）、被钉住时受击与拔钉（[pinnable.lua#L40](../../../raw/current/scripts/components/pinnable.lua#L35-L43)、[pinnable.lua#L166](../../../raw/current/scripts/components/pinnable.lua#L162-L168)）、蓝护符受击／格挡 proc 与作祟群冻（[amulet.lua#L68](../../../raw/current/scripts/prefabs/amulet.lua#L65-L71)、[amulet.lua#L461](../../../raw/current/scripts/prefabs/amulet.lua#L456-L464)）、冰杖命中（[staff_common.lua#L202](../../../raw/current/scripts/prefabs/staff_common.lua#L199-L204)）、冰弹命中可冻结目标（[slingshotammo.lua#L285](../../../raw/current/scripts/prefabs/slingshotammo.lua#L283-L290)）、蟹王冰弹命中（[crabking.lua#L1941](../../../raw/current/scripts/prefabs/crabking.lua#L1940-L1942)）、冰猎犬死亡寒爆（[hound.lua#L678](../../../raw/current/scripts/prefabs/hound.lua#L669-L679)，death 事件驱动 [L708](../../../raw/current/scripts/prefabs/hound.lua#L708-L708)）、独眼巨鹿寒息波及（[deerclops.lua#L218](../../../raw/current/scripts/prefabs/deerclops.lua#L217-L219)）、暗影中分体移除状态组件时的余寒（[daywalker2.lua#L735](../../../raw/current/scripts/prefabs/daywalker2.lua#L733-L737)）。

### 层、父子与端模型

| 对象 | 本族如何使用 | 边界 | 证据 |
|---|---|---|---|
| 本地代理父挂 | `shatter` 的本地动画父挂到**壳的父实体**（壳被 AddChild 到宿主时，动画随之挂到宿主）；splash_spiderweb／brokentool 的本地动画不父挂，位置一次性 | shatter 跟随符号路径（Follower）下壳无父，本地动画落世界位置、不跟符号 | [shatter 本地父挂](../../../raw/current/scripts/prefabs/shatter.lua#L27-L32)、[freezable 两条挂法](../../../raw/current/scripts/components/freezable.lua#L100-L106) |
| singingshell_critterfx 的平台父挂 | 生成后 0 秒自检：站在船（平台）上则 `entity:SetParent(platform.entity)`＋本地坐标（随船移动）；不在视觉地面则当场自删并放 `splash_green_small` | 父挂对象是平台不是本体（本体此时已删）；联网壳层级随船复制 | [平台判定](../../../raw/current/scripts/prefabs/singingshell.lua#L310-L323) |
| pumpkin 碎块浮层 | 碎块是非联网实体，落水时父挂联网 `float_fx_front/back` 到碎块上（方向反过来：网络体挂本地体） | 浮层随碎块移动；碎块被 ErodeAway 时浮层各自侵蚀 | [fx_TestChunkFloat](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L242-L249) |
| 端模型 | 模式①壳联网复制位置、动画全本地（含专服也建 brokentool 本地体）；模式②动画在主模拟播、随联网壳复制；pumpkin 碎块雨各端确定性自建（含专服，专服无渲染意义） | 模式②注册都没有客户端本地子实体（对照 bomb_lunarplant **本体**的 spin core 属本体页范围） | [非专服分支①](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L36-L40)、[无门控](../../../raw/current/scripts/prefabs/brokentool.lua#L34-L36)、[唤醒抛块](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L291-L294) |

### 清理路径

| 注册 | 壳 | 本地子实体 | 证据 |
|---|---|---|---|
| `shatter` | 主模拟 1 秒自删 | 本地动画 `animover` 自删；等级门控一次（`_complete`） | [壳自删](../../../raw/current/scripts/prefabs/shatter.lua#L87-L88)、[本地](../../../raw/current/scripts/prefabs/shatter.lua#L44-L44) |
| `splash_spiderweb` | 主模拟 1 秒自删 | 本地动画 `animover` 自删 | [壳](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L48-L49)、[本地](../../../raw/current/scripts/prefabs/splash_spiderweb.lua#L24-L24) |
| `brokentool` | 主模拟 1 秒自删 | 本地动画 `animover` 自删 | [壳](../../../raw/current/scripts/prefabs/brokentool.lua#L43-L44)、[本地](../../../raw/current/scripts/prefabs/brokentool.lua#L25-L25) |
| `singingshell_critterfx` | `jump_post` 的 `animover` 自删；非视觉地面当场自删 | 无本地子实体（wood／rock 变体是它生成的独立注册 FX，各自走通用 FX 工厂的自删） | [状态链收尾](../../../raw/current/scripts/prefabs/singingshell.lua#L304-L308)、[地面分支](../../../raw/current/scripts/prefabs/singingshell.lua#L318-L321) |
| `propsignshatterfx` | 主模拟 `animover` 自删 | 无 | [工厂尾](../../../raw/current/scripts/prefabs/propsign.lua#L216-L217) |
| `pumpkin_bomb_explode_fx` | 主模拟 `animover` 自删 | 碎块 1.5–1.75 秒后 `ErodeAway`（含浮层各自侵蚀） | [壳](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L321-L322)、[侵蚀链](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L287-L288) |
| `tumbleweedbreakfx` | 主模拟 `animover` 自删＋动画长度＋1 帧兜底 | 无本地子实体 | [双保险](../../../raw/current/scripts/prefabs/tumbleweed.lua#L416-L418) |
| `bomb_lunarplant_explode_fx` | 主模拟 `animover` 自删 | 无 | [工厂尾](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L288-L289) |

### 伤害与玩法归属辨析

8 个注册自身都没有 combat／explosive／weapon 等玩法组件；表现背后的结算位置：

- **南瓜／亮茄炸弹**：爆炸伤害在宿主临时装配的 explosive 组件里结算——`Explosive:OnBurnt` 做屏幕闪光＋震屏、逐目标 `GetAttacked`／击退、按 stack 结算建筑伤害，最后 `self.inst:Remove()`（[explosive.lua#L31-L157](../../../raw/current/scripts/components/explosive.lua#L31-L157)）。南瓜炸弹：范围伤害 50、建筑 4、10% 掉南瓜种子（[调参](../../../raw/current/scripts/tuning.lua#L9806-L9810)、[装配](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L23-L42)）；亮茄炸弹：`explosivedamage = 0`，伤害全走临时 planardamage 200＋对 `shadow_aligned` 1.1 倍增伤（[装配](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L15-L35)、[调参](../../../raw/current/scripts/tuning.lua#L398-L399)）。FX 都在爆炸结算并自删本体**之后**才生成。
- **shatter**：冻结数值由冰源直接调 `AddColdness`（freezable 系统），碎裂只是寒冷值跨过阈值后的视觉余韵；FX 壳与本地动画零结算。
- **propsign（本体）**：1 点武器伤害在木牌自己的 weapon 组件（[装备武器](../../../raw/current/scripts/prefabs/propsign.lua#L168-L170)）；碎裂 FX 与伤害无关。
- **brokentool**：出现的场合（工具用坏／怀表拆解／作祟成功）伴随的工具损耗由宿主侧 inventoryitem／配方逻辑处理，代理只是"坏掉的样子"。
- **singingshell_critterfx**：本体贝壳钟被锤碎才生成；演奏（cyclable 12 音阶、给范围内农田植物 `TendTo`）是本体 singingshell_octave3/4/5 的玩法（[演奏侧](../../../raw/current/scripts/prefabs/singingshell.lua#L52-L63)、[音阶实体](../../../raw/current/scripts/prefabs/singingshell.lua#L129-L233)）。

## 典型接入

### 手动播放一次碎裂／爆裂表现

- 前提与选择：模式②注册（propsignshatterfx、tumbleweedbreakfx、bomb_lunarplant_explode_fx 等）位置定位＋自删，直接 `SpawnPrefab` 即用；模式①需要喂"壳位置＋等级"（shatter）或接受 1 秒壳（splash_spiderweb／brokentool）。伤害、掉落与音效外的结算都由调用方自理。
- 生效与撤销：主模拟生成；shatter 要在 1 秒自删窗口内写 `components.shatterfx:SetLevel(1–5)`（netvar 触发本地动画）；模式②不用管收尾。
- 原版依据：[冰弹兜底写法](../../../raw/current/scripts/prefabs/slingshotammo.lua#L287-L290)、[免费注册写法](../../../raw/current/scripts/prefabs/propsign.lua#L39-L41)。

```lua
local function PlayIceShatter(x, y, z)
    local fx = SpawnPrefab("shatter")
    if fx ~= nil then
        fx.Transform:SetPosition(x, y, z)
        if fx.components.shatterfx ~= nil then
            fx.components.shatterfx:SetLevel(3) -- medium
        end
    end
end
```

- 调整与验证：替换点是注册名与等级（1–5）；`shatter` 的等级写入要在壳存活期间完成，晚于 1 秒自删会静默失效。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`brokentool` 是可拾取的"损坏工具"。** 它没有 inventoryitem／workable，是带本地动画代理的纯表现实体；真正可修复的坏工具走别的系统。名称来自"工具坏掉时出现的样子"。[壳工厂](../../../raw/current/scripts/prefabs/brokentool.lua#L28-L47)
- **误区：`splash_spiderweb` 有原版生成点。** 脚本层零生成：唯一外部引用是 `spider_web_spit` 的依赖声明（旁边就是源码作者的疑问注释）；依赖消费代码可能在快照外的事件数据里。不要把"被依赖"写成"被生成"。[缺口](#生成点与宿主调用)
- **边界：`brokentool` 的本地动画没有专服门控。** 与 shatter／splash_spiderweb 不同，延迟建本地体的代码不带 `IsDedicated` 分支、壳也没有 `FX` 标签——专用服务器上会真的建出带物理的非联网实体（1 秒后自删）。[对照](../../../raw/current/scripts/prefabs/brokentool.lua#L34-L36)
- **边界：shatter 的本地动画有单次门控。** `OnLevelDirty` 里 `_complete` 置位后不再响应新等级；壳 1 秒自删窗口内多次 `SetLevel` 只有第一次生效（等级为 0 或已放过则永不播放）。[门控](../../../raw/current/scripts/prefabs/shatter.lua#L47-L55)
- **边界：shatter 的两个同名陷阱。** `widgets/werebadge.lua` 与 `widgets/statusdisplays.lua` 上也有 `SpawnShatterFX` 方法，但那是 UI widget 自己的动画（生成 UIAnim），与 `shatter` prefab 无关；freezable 链之外同名不等于同物。[UI 同名](../../../raw/current/scripts/widgets/werebadge.lua#L47-L56)
- **边界：南瓜碎块是"各端各自算"的同步。** 确定性 PRNG 只保证同位置同实体各端一致；实体被移动后种子变化、浮层是网络体挂本地体——不保证与服务器逐帧一致，也不经 netvar。改这类表现时不要加随机数。[播种](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L264-L270)
- **边界：tumbleweedbreakfx 无 NOCLICK、singingshell_critterfx 自带平台父挂。** 前者可被点击命中（无碰撞体，仅标签层面）；后者上船会随船走、落水当场自删——直接复用时注意这两个行为差异。[标签](../../../raw/current/scripts/prefabs/tumbleweed.lua#L407-L407)、[平台](../../../raw/current/scripts/prefabs/singingshell.lua#L310-L323)
- **边界：清单外连带注册。** `singingshell_critterfx` 在 `jump_post` 挖掘帧会按平台材质生成 `singingshell_creature_woodfx`／`singingshell_creature_rockfx`（注册于根目录通用 FX 数据表 `fx.lua`，[条目](../../../raw/current/scripts/fx.lua#L2227-L2235)；生成于 [singingshell.lua#L283-L288](../../../raw/current/scripts/prefabs/singingshell.lua#L283-L288)）——这两个注册不属原定清单，按发现并入记录，本页不展开其工厂。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 8 个工厂文件（shatter／splash_spiderweb／brokentool／singingshell／propsign／pumpkin_bomb／tumbleweed／bomb_lunarplant 的全部工厂与生命周期函数）；按需读取 shatter 消费链（shatterfx／freezable 组件、standardcomponents 五个 helper、slingshotammo／staff_projectile／staff_common／amulet／pinnable／crabking／hound／deerclops／daywalker2 的调用点）、brokentool 五个生成点上下文、explosive 组件 OnBurnt、SGworm_boss_head 咀嚼引爆、merm 掉落池、recipes 两条配方、simutil `ErodeAway`、tuning 相关段。全 `raw/current/scripts` 反查（`grep -rn "注册名" --include=*.lua`）：`shatter` 字面命中 10 文件——生成 2 处（slingshotammo.lua:287、staff_projectile.lua:18）＋freezable 动态路径（standardcomponents.lua:325 注册名来源）＋依赖表（slingshotammo.lua:1410、world.lua:212、staff_projectile.lua:8、prefablist.lua:1142）＋本文件注册；`splash_spiderweb` 命中 3 处（本文件、prefablist.lua:1209、spider_web_spit.lua:10）——无生成点；`singingshell_critterfx` 命中本文件 3 处（依赖表、生成 L76、注册 L333）——唯一生成点；`propsignshatterfx` 命中 propsign.lua 3 处（依赖表 L15、生成 L39、注册 L223）——唯一生成点；`pumpkin_bomb_explode_fx`／`bomb_lunarplant_explode_fx` 各命中本文件 3 处（依赖表、OnHit 生成、注册）——各唯一生成点；`tumbleweedbreakfx` 命中 tumbleweed.lua 4 处（依赖表 L16、生成 L142／L274、注册 L518）；`brokentool` 命中 5 生成点（pocketwatch_dismantler.lua:29、pocketwatch.lua:139、SGhermitcrab.lua:3703、SGwilson.lua:15057、SGwx78_possessedbody.lua:2361）＋player_common.lua:2196 依赖表＋propsign.lua:185 门控字段。`singingshell_creature_woodfx`／`rockfx` 反查发现注册于根目录 `fx.lua`（通用 FX 数据表），按清单外发现并入"常见陷阱"末条。
- 名称核验：`languages/chinese_s.po` 定向检索——`PUMPKIN_BOMB`＝"南瓜炸弹"（[L347340-L347342](../../../raw/current/scripts/languages/chinese_s.po#L347340-L347342)）、`BOMB_LUNARPLANT`＝"亮茄炸弹"（[L339825-L339827](../../../raw/current/scripts/languages/chinese_s.po#L339825-L339827)）、`TUMBLEWEED`＝"风滚草"（[L350860-L350862](../../../raw/current/scripts/languages/chinese_s.po#L350860-L350862)）、`SINGINGSHELL_OCTAVE3/4/5`＝"低音／中音／高音贝壳钟"（[L349390-L349402](../../../raw/current/scripts/languages/chinese_s.po#L349390-L349402)）、`HOMESIGN`＝"木牌"（[L343935-L343937](../../../raw/current/scripts/languages/chinese_s.po#L343935-L343937)，propsign 显示名 override 目标）；`SHATTER`／`SPLASH_SPIDERWEB`／`SINGINGSHELL` 基名／`SINGINGSHELL_CRITTERFX`／`PROPSIGN`／`PROPSIGNSHATTERFX`／`PUMPKIN_BOMB_EXPLODE_FX`／`TUMBLEWEEDBREAKFX`／`BROKENTOOL`／`BOMB_LUNARPLANT_EXPLODE_FX` 均无 `STRINGS.NAMES.*` 键，正文使用已核验效果描述。
- 分档说明：`size_class: complex`——8 个注册分属 8 个文件、三种载体模式；shatter 有横跨 10+ 文件的 freezable 消费链与两级（壳 netvar→本地动画）端模型；pumpkin_bomb_explode_fx 有确定性 PRNG 本地碎块子系统与两个未注册内部实体；清理路径四种（animover 自删、1 秒壳兜底、动画长度兜底、ErodeAway 链）且与端模型强耦合，需要整体保留对照说明，超出 standard 单页容量。
- 未覆盖：`float_fx_front`／`float_fx_back`／`splash_green_small`／`splash_sink`／`ash`／`erode_ash`／`round_puff_fx_sm` 等连带通用 FX；singingshell 演奏系统（singingshellmanager／singingshelltrigger）与本体获得途径；猪王竞技场投掷玩法（SGpigelite）与木牌装备持握细节；天蛾／蠕虫 Boss 玩法全貌；炸弹皮肤与 scrapbook；动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/explode_small|小爆裂、换装烟与火药爆裂]]（同为爆裂表现族，对照 explosive 侧 FX 分工）
- [[reference/prefabs/groundpoundring_fx|冲击环、爆点与碎块表现特效]]（同族碎块与爆点）
- [[reference/prefabs/grasspartfx|小型一次性外围 FX 族]]（同为一次性短命 FX 的宿主回调生成模式）
- 返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
