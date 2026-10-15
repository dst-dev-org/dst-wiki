---
title: "暗影主教羽群、裂隙门粒子与墨荒替身／附体特效（shadow_bishop_fx 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "暗影 Boss 与裂隙生物系的 4 个联网 FX：暗影主教的攻击羽群（继承宿主缩放与透明度）、暗影裂隙门的分阶段粒子层（宿主单槽引用管生灭）、墨荒巨口的跳跃替身（带物理的攻城壳）、虚空面具附体死亡的复活演出（animover 后给宿主发面具）。"
aliases: ["shadow_bishop_fx", "shadowrift_portal_fx", "shadowthrall_mouth_dupe_fx", "shadowthrall_parasite_fx", "暗影主教羽群", "裂隙门粒子", "墨荒替身", "巨口跳跃特效", "虚空面具复活特效", "暗影棋子特效", "暗影裂隙特效"]
tags: [prefab, fx, creature, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadowchesspieces.lua", "stategraphs/SGshadow_bishop.lua", "prefabs/shadowrift_portal.lua", "prefabs/lunarrift_portal.lua", "components/riftspawner.lua", "prefabs/shadowthrall_mouth.lua", "stategraphs/SGshadowthrall_mouth.lua", "components/shadowthrallmanager.lua", "prefabs/shadowthrall_parasite.lua", "components/shadowparasitemanager.lua", "components/corruption.lua", "prefabs/clockwork_common.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影主教羽群、裂隙门粒子与墨荒替身／附体特效（shadow_bishop_fx 等 4 注册）

## 快速判断

- 职责与适用：4 个 FX 分四条宿主链——①`shadow_bishop_fx`：暗影主教攻击时在自身位置反复绽放的**羽群动画**（一次 `feather_loop` 播完自删，继承宿主缩放与半透明）；②`shadowrift_portal_fx`：暗影裂隙门的**分阶段粒子层**（`particle_N_pre/loop`，阶段升级换档、关门播消失），生灭完全由裂隙门单槽引用管理；③`shadowthrall_mouth_dupe_fx`：墨荒巨口的**跳跃替身**——带 50 质量物理的联网壳，巨口隐身时它替身起跳落地；④`shadowthrall_parasite_fx`：虚空面具宿主死亡后的**复活演出**——播 `atk` 一次，结束后给宿主装备新面具并满血复活。[主教 FX 注册](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L552-L555)、[裂隙 FX 注册](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L521-L523)、[巨口 FX 注册](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L390-L391)、[寄生 FX 注册](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L209-L210)
- 本体辨析：`shadow_bishop`（官方名"暗影主教"，[po](../../../raw/current/scripts/languages/chinese_s.po#L349255-L349257)）是三暗影棋子之一（带升级／掉落／Brain 的玩法怪）；`shadowrift_portal`（官方名"裂隙"，[po](../../../raw/current/scripts/languages/chinese_s.po#L349170-L349172)）是裂隙系统在洞穴投放的世界设施（`riftspawner` 按 rift 定义生成，[投放](../../../raw/current/scripts/components/riftspawner.lua#L114-L122)）；`shadowthrall_mouth`（官方名"墨荒"，[po](../../../raw/current/scripts/languages/chinese_s.po#L349210-L349212)，阴影阵营显示名"狞笑"，[po](../../../raw/current/scripts/languages/chinese_s.po#L349214-L349217)）是 shadowthrallmanager 召唤的三类憎恶之一（[点选](../../../raw/current/scripts/components/shadowthrallmanager.lua#L235-L248)）；`shadowthrall_parasite`（官方名"虚空面具"，[po](../../../raw/current/scripts/languages/chinese_s.po#L349220-L349222)，阴影显示名"暗域先驱"）是面具附体系统的浮游体。四者玩法本页只辨析链条。
- 重要副作用：羽群与粒子 FX 零玩法组件；**替身 FX 带真实物理**（CHARACTERS 碰撞组、GROUND 碰撞罩，质量 50）；寄生 FX 的 `animover` 回调会**改宿主玩法状态**（装备面具、满血、切 `parasite_revive` 状态）。羽群与寄生 FX 有 `animover` 自删；替身的 `animover` 只推事件给宿主，不删自己；粒子 FX 无自删。
- 修改边界：只覆盖 4 注册的工厂、生成点、父挂与清理；棋子升级数值、裂隙门阶段／瘴气／孵怪、墨荒战斗状态机全貌、面具附体系统全链不展开。
- 名称：4 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 键（检索 `NAMES.SHADOW_BISHOP_FX`／`SHADOWRIFT_PORTAL_FX`／`SHADOWTHRALL_MOUTH_DUPE_FX`／`SHADOWTHRALL_PARASITE_FX` 皆无命中），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_bishop_fx` | `bishopfxfn`：Transform／AnimState／Network → 标签 `FX` → bank=build=`shadow_bishop_fx`、播一次 `feather_loop`（无循环参数）、FinalOffset 1 → `SetPristine` → 客户端早退 → 服务端 `persists=false`＋`animover` 自删。 | 一次性羽群：宿主 SG 生成后补 `SetScale`／`SetMultColour` 同步主教当前体型与透明度；生命期＝一轮动画。[工厂](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L526-L549)、[注册](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L555) |
| `shadowrift_portal_fx` | `portalfxfn`：Transform／AnimState／Network（无 SoundEmitter）→ 标签 `FX`＋`NOCLICK`＋`NOBLOCK` → bank=build=`shadowrift_portal`、播 `particle_1_pre`＋续 `particle_1_loop` → LightOverride 1 → `SetPristine` → 客户端早退 → 服务端 `persists=false`、暴露 `PlayStage`／`Disappear`。 | 粒子层：`PlayStage(stage, load)` 换阶段动画（读档走 `load=true` 直播循环）；`Disappear` 只播 `particle_disappear`，**不自动删除**。[工厂](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L463-L492)、[PlayStage/Disappear](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L213-L223) |
| `shadowthrall_mouth_dupe_fx` | `dupefxfn`：Transform／AnimState／DynamicShadow／Network／Physics（质量 50、CHARACTERS 组、GROUND 罩、胶囊 0.75）→ 六面向、标签 `FX`＋`NOCLICK` → bank=build=`shadow_thrall_mouth`、`fin_red` 符号提亮 → `SetPristine` → 客户端早退 → 服务端 `persists=false`、自装 `colouradder`＋`bloomer`、`animover` 推 `dupe_animover` 给 `main`。 | 跳跃替身：生成即被宿主 `RemoveFromScene`（不进场景）；宿主的 colouradder／bloomer `AttachChild` 它，染色与泛光随巨口。回调不删自己，只通知宿主换状态。[工厂](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L347-L387)、[回调](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L341-L344)、[注册](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L391) |
| `shadowthrall_parasite_fx` | `fxfn`：Transform／AnimState／SoundEmitter／Network → 标签 **`fx`（小写异例）**＋`NOCLICK` → bank=build=`shadow_thrall_parasite`、播一次 `atk`、循环音效 `thrall_idle_LP` → 非专服另造本地 `CreateHairFx` 子体跟随 `follow_fx_swap` → `SetPristine` → 客户端早退 → 服务端 `animover` 走 `fx_onanimover`、`persists=false`。 | 复活演出：`target` 字段由生成方写入；animover 后删自己再给宿主发面具。工厂内音效与本地子体在 SetPristine 前（两端执行）。[工厂](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L173-L205)、[animover 回调](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L141-L170)、[注册](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L210) |

## 关联机制导航

### 生成点与宿主调用

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `SGshadow_bishop` `attack` 状态 8 帧 | `shadow_bishop_fx` | 主模拟；主教起手时先给一次羽群（同时进入无敌），位置＝主教位置、`SetScale`／`SetMultColour` 复制宿主（等级放大与 alpha 变淡随之带入） | [DoSwarmFX](../../../raw/current/scripts/stategraphs/SGshadow_bishop.lua#L13-L17)、[起手](../../../raw/current/scripts/stategraphs/SGshadow_bishop.lua#L48-L53) |
| `SGshadow_bishop` `attack_loop` 状态 | `shadow_bishop_fx` | 主模拟；同状态内 `DoPeriodicTask(1.2, DoSwarmFX, .5)` 反复绽放，主教向目标推进并按 `ATTACK_TICK`(0.5) 群体判定 | [循环任务](../../../raw/current/scripts/stategraphs/SGshadow_bishop.lua#L96-L99)、[调值](../../../raw/current/scripts/tuning.lua#L1607-L1608) |
| `shadowrift_portal` 工厂（`CreateParticleFx`） | `shadowrift_portal_fx` | **工厂内生点位于 SetPristine 之前的共享段——两端各执行一次**，并登记 `highlightchildren`；主模拟 `OnPortalWake` 另有补建（睡眠时已删） | [工厂生成](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L393-L395)、[唤醒补建](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L238-L247) |
| `shadowthrall_mouth` 工厂 | `shadowthrall_mouth_dupe_fx` | 主模拟；巨口组装时生成，随即 `RemoveFromScene` 收起、写 `dupe.main = inst` 回指宿主 | [生成与收起](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L273-L275) |
| `SGshadowthrall_mouth` `leap` 状态 | `shadowthrall_mouth_dupe_fx` | 主模拟；巨口 `Hide`＋关碰撞，替身 `ReturnToScene` 传送到宿主位、播 `leap`、按目标距离定速度冲出；`dupe_animover` 回调切 `land`，onexit 再 `RemoveFromScene` 收回 | [leap 使用](../../../raw/current/scripts/stategraphs/SGshadowthrall_mouth.lua#L558-L600)、[跳冷却](../../../raw/current/scripts/tuning.lua#L7725) |
| `shadowparasitemanager:ReviveHosted` | `shadowthrall_parasite_fx` | 主模拟；被面具附体的宿主死亡后，其死亡状态调本方法：在尸体位生成 FX 并写 `fx.target = 宿主` | [ReviveHosted](../../../raw/current/scripts/components/shadowparasitemanager.lua#L378-L399) |

`shadowrift_portal_fx` 的阶段驱动全部由裂隙门发起：`TryStageUp` 调 `PlayStage(next_stage)` 换档（1→3 级，每级约 4 天＋随机，[升级](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L127-L129)、[调值](../../../raw/current/scripts/tuning.lua#L7778-L7782)）；`ClosePortal`（15 天到期）先让粒子 `Disappear`、再按消失动画时长排 `be_removed` 计时删门（[关门](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L166-L181)）；读档按存档阶段直接 `PlayStage(stage, true)` 或播消失（[读档](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L271-L281)）。

### 宿主链辨析（各自怎么走到 FX）

- **暗影主教**：发条主教（`bishop` 标签棋子）被国王魔杖类腐蚀命中时，`corruption` 组件推 `ms_becomeshadowchess` 事件（[推送](../../../raw/current/scripts/components/corruption.lua#L160-L163)），发条侧 `OnBecomeShadowChess` 按标签映射 `SpawnPrefab("shadow_bishop")` 并继承位置朝向（[映射与生成](../../../raw/current/scripts/prefabs/clockwork_common.lua#L203-L220)；bishop/knight/rook 三 prefab 的监听已 grep 复核）。主教靠击杀其他棋子积累升级（等级改变缩放、alpha 与羽群观感，[升级](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L178-L218)）。
- **裂隙门**：`riftspawner` 在洞穴按 `CreateRiftPortalDefinition("shadowrift_portal", ...)` 注册的定义投放（阴影亲和、三类洞窟限定），门本体带障碍物理／震地／瘴气播种／孵怪（[定义](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L502-L516)、[门工厂](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L355-L460)）。
- **墨荒巨口**：`shadowthrallmanager` 召唤憎恶时按配额把 `shadowthrall_mouth` 作为三类之一生成（[召唤](../../../raw/current/scripts/components/shadowthrallmanager.lua#L235-L248)）；巨口被 shadow_pillar 类来源 root 时，事件经 `inst.dupe` 的监听把 `rooted` 组件同步装到替身上（[root 同步](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L69-L78)、[监听](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L323-L324)）。
- **虚空面具**：面具击杀宿主时给死者写 `shadowthrall_parasite_hosted_death` 标记（hats.lua:5593，grep 复核）；猪／蜘蛛／兔人／鱼人／石虾的死亡状态据此改走"被附体死亡"分支并调 `ReviveHosted`（五个 SG 的调用点已 grep 复核）；玩家的死亡走 SGwilson 专属 `death_hosted` 状态自行复活，**不经过本 FX**（SGwilson 全文无 `shadowthrall_parasite_fx` 引用）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `shadow_bishop_fx` | 工厂 `animover` 自删（一轮 `feather_loop` 播完即走） | 每次羽群都是新实体；宿主攻击被打断也不留残留。[自删](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L546-L547) |
| `shadowrift_portal_fx` | **无自删**：门睡眠 `OnPortalSleep` 显式 Remove；关门后由门的 `be_removed` 计时移除，粒子经 `AddChild` 登记进门的 Lua `children`（[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935)），门 `Remove` 时由 children 循环回收（[children 循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783)）；门被拆同理 | 生灭挂在门的 `_fx` 单槽引用上；`particle_disappear` 播完后到门真正删除之间有间隙（[睡眠移除](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L228-L235)、[关门计时](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L179-L180)、[AddChild](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L206-L210)） |
| `shadowthrall_mouth_dupe_fx` | 无自删：巨口 `OnRemoveEntity` 里 `dupe:Remove()`；`leap` 结束只是 `RemoveFromScene` 收起不删除 | 替身与巨口同生共死，反复出入场景复用同一实体（[收起](../../../raw/current/scripts/stategraphs/SGshadowthrall_mouth.lua#L598-L600)、[随宿主删除](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L154-L157)） |
| `shadowthrall_parasite_fx` | `animover` 自删（回调里先 `Remove` 再给宿主发面具；末尾还有一次冗余 `Remove`） | 宿主无 inventory/health 时只删 FX 不复活（[回调守卫](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L146-L170)） |

## 典型接入

### 复刻"隐身本体＋替身壳"的突进演出

- 前提与选择：本体需要消失期间仍有联网可碰撞的表现体时，用一个带 Physics 的 FX 做替身：工厂生成后立刻 `RemoveFromScene` 藏起，需要时 `ReturnToScene`＋Teleport＋动画＋速度；替身 `animover` 推事件回宿主控制状态切换。适合跳扑／冲锋类 Boss 技能。
- 生效与撤销：替身的出入场景必须成对（`ReturnToScene` ↔ `RemoveFromScene`），宿主删除时在 `OnRemoveEntity` 里补删替身；染色用宿主 colouradder/bloomer 的 `AttachChild` 传播（[巨口挂接](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L273-L275)、[染色](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L311-L315)）。

```lua
-- 服务端：leap 状态里的替身驱动（对应 SGshadowthrall_mouth）
inst.dupe.Physics:Teleport(x, y, z)
inst.dupe.AnimState:PlayAnimation("leap")
inst.dupe:ReturnToScene()
inst.dupe.Physics:SetMotorVelOverride(dist / inst.dupe.AnimState:GetCurrentAnimationLength(), 0, 0)
-- 替身 animover 推 dupe_animover 给 main，宿主据此切 land 状态
```

- 调整与验证：替换点是注册名与动画名；替身碰撞罩在工厂里配置，速度按动画时长反推保证落点对齐。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`shadowrift_portal_fx` 依赖宿主单槽引用，别指望它自己消失。** `Disappear` 只播动画不删实体；直接 `SpawnPrefab` 出来不给它父挂到门上，它会以 `particle_1_loop` 永久循环（persists=false 保证不跨存档，但当前会话常驻）。[工厂尾](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L487-L490)
- **边界：粒子 FX 的工厂生成点在共享段。** `portalfn` 在 SetPristine 之前调 `CreateParticleFx`——客户端工厂也会各自生成一份本地子体（对比 lunarrift_portal 把 `lunarrift_portal_shadow` 放在 master 段，[对照](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L636-L642)）；跨端实际渲染份数未运行核验。
- **边界：替身不是纯视觉。** `shadowthrall_mouth_dupe_fx` 带质量 50 的物理与 GROUND 碰撞罩，leap 期间它就是"巨口的身体"；把 `FX` 标签实体一律按可穿透处理会误判碰撞。[物理配置](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L355-L361)
- **边界：寄生 FX 的 Tag 是小写 `"fx"`。** 工厂写的是 `inst:AddTag("fx")`（[L181](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L181)），不是惯用的 `"FX"`——按大写 `"FX"` 标签检索会漏掉它（台账异例复核属实）。
- **边界：`shadow_bishop_fx` 的"feather_loop"是播完一轮动画即自删，不是一帧。** 工厂 `PlayAnimation("feather_loop")` 无循环参数，`animover` 在整轮动画结束后触发；同一次攻击内羽群靠 SG 的周期任务反复生成（[任务](../../../raw/current/scripts/stategraphs/SGshadow_bishop.lua#L96-L99)）。台账批 7"一帧后 animover"表述按源码实况修正为"一轮后"。
- **边界：寄生 FX 的 animover 会改宿主状态。** 它不是纯收尾：给宿主发 `shadow_thrall_parasitehat`、装备、满血、切 `parasite_revive`；target 无 inventory（如非物品栏实体）时宿主不会复活。[回调](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L146-L170)
- **边界：寄生体本体的 highlight 层是死赋值。** 本体工厂先把 hair 写进 `highlightchildren` 又立刻覆盖为空表（[L92-L100](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L92-L100)），colouradder 同步实际到不了那撮本地发焰；复刻时别照抄这组赋值。
- **边界：主教羽群的观感随等级变化。** 羽群复制宿主缩放与 alpha：主教 3 级的 scale 与 0.74 alpha 会原样带入 FX（[升级改缩放与透明](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L186-L194)、[复制](../../../raw/current/scripts/stategraphs/SGshadow_bishop.lua#L16-L17)）；调 FX 视觉时先看宿主等级。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/shadowchesspieces.lua`、`prefabs/shadowrift_portal.lua`、`prefabs/shadowthrall_mouth.lua`、`prefabs/shadowthrall_parasite.lua`；按需读取 `stategraphs/SGshadow_bishop.lua`（DoSwarmFX 与 attack/attack_loop）、`stategraphs/SGshadowthrall_mouth.lua`（leap 状态）、`components/shadowparasitemanager.lua`（ReviveHosted／SpawnFloater）、`components/shadowthrallmanager.lua`（憎恶点选）、`components/corruption.lua`（事件推送）、`prefabs/clockwork_common.lua`（映射与转化）、`components/riftspawner.lua`（投放）、`prefabs/lunarrift_portal.lua`（pristine 分界对照）、`tuning.lua` 相关调值。对 4 个注册名以完整名在 `raw/current/scripts` 全域反查：`shadow_bishop_fx` 命中注册（shadowchesspieces.lua:555）、资产（:28）、bank/build（:535/:536）、生成（SGshadow_bishop.lua:14，attack 与 attack_loop 两处调用同一函数）——SpawnPrefab 字面唯一；`shadowrift_portal_fx` 命中注册（shadowrift_portal.lua:523）、依赖（:19）、生成（:207，由工厂 L393 与 OnPortalWake L245 两次调用 CreateParticleFx）——生成点唯一；`shadowthrall_mouth_dupe_fx` 命中注册（shadowthrall_mouth.lua:391）、依赖（:11）、生成（:273）——生成点唯一；`shadowthrall_parasite_fx` 命中注册（shadowthrall_parasite.lua:210）、依赖（:10）、生成（shadowparasitemanager.lua:395）——生成点唯一，SGwilson 无引用。宿主侧反查：`ms_becomeshadowchess` 仅 corruption.lua:160 推送、bishop/knight/rook 三 prefab 监听；`shadowthrall_parasite_hosted_death` 写入点 hats.lua:5593，`ReviveHosted` 调用点 SGpig:188／SGspider:144／SGbunnyman:171／SGmerm:1080／SGrocky:771；`shadowparasitemanager` 组件挂在 cave.lua:527 的世界上。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.SHADOW_BISHOP`＝"暗影主教"、`SHADOWRIFT_PORTAL`＝"裂隙"、`SHADOWTHRALL_MOUTH`＝"墨荒"、`SHADOWTHRALL_MOUTH_ALLEGIANCE`＝"狞笑"、`SHADOWTHRALL_PARASITE`＝"虚空面具"、`SHADOWTHRALL_PARASITE_ALLEGIANCE`＝"暗域先驱"（锚点见"快速判断"）；4 个覆盖注册名均无 NAMES 键，标题使用效果描述。
- 分档说明：`standard`——4 注册四条宿主链，各链独立小节；未选 complex，无多变体状态机，链间无共享状态。
- 未覆盖：棋子升级数值表与 Brain、裂隙门的瘴气／孵怪／震地玩法、墨荒战斗状态机其余状态与 `shrouden_bite` 召唤形态、面具附体系统全链（面具物品、玩家 `death_hosted` 状态）、动画包内容、替身与粒子 FX 的跨端渲染份数；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：补 `AddChild`／`children` 回收锚点，区分 animover 自删与 dupe 事件；同步维护日期。

## 相关链接

- [[reference/prefabs/shadowthrall_projectile_fx|墨荒弹幕投射物]]（同一墨荒宿主系的攻击 FX，对照"伤害留在弹体"的写法）
- [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]]（巨口被 root 的来源方；rooted 组件见 [[reference/components/rooted]]）
- 返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]
