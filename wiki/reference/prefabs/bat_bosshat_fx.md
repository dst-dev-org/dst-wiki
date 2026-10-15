---
title: "猎犬蝙蝠帽层、影分身演出、蝙蝠棒皮肤扇翅与惊树蝠群（bat_bosshat_fx 等 5 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "蝙蝠系 5 个外围 FX：猎犬蝙蝠被戴到头上时的联网咀嚼层（酸浸/暗影换装、三只本地栖立小蝠）、枯萎猎犬蝙蝠血量告急的分身影演出、蝙蝠棒两款皮肤的命中扇翅（net_tinybyte 代理＋本地动画）与万圣惊夜砍树惊出的单玩家可见蝠群（SetClassifiedTarget）。"
aliases: ["bat_bosshat_fx", "bat_boss_shadow_split_fx", "batbat_bats", "batbat_fantasy_fx", "battreefx", "猎犬蝙蝠帽特效", "蝙蝠帽咀嚼特效", "蝙蝠影分身特效", "蝙蝠棒皮肤特效", "蝙蝠棒命中扇翅", "惊树蝠群", "万圣惊夜蝙蝠"]
tags: [prefab, fx, creature, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bat_boss.lua", "prefabs/batbat.lua", "prefabs/battreefx.lua", "stategraphs/SGbat.lua", "brains/bat_bossbrain.lua", "components/spooked.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/batbosscave.lua", "widgets/batover.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "actions.lua", "prefabs/hats.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 猎犬蝙蝠帽层、影分身演出、蝙蝠棒皮肤扇翅与惊树蝠群（bat_bosshat_fx 等 5 注册）

## 快速判断

- 职责与适用：5 个 FX 分四条宿主链。①`bat_bosshat_fx`——活体猎犬蝙蝠（bat_boss）"戴头"后挂在玩家头部的**联网咀嚼层**：SoundEmitter＋AnimState＋Follower，由宿主的 0.6 秒周期咀嚼任务联动发声播动画，非专服另建三只本地"栖立小蝠"层（[注册](../../../raw/current/scripts/prefabs/bat_boss.lua#L921)）；②`bat_boss_shadow_split_fx`——枯萎猎犬蝙蝠（bat_boss_shadow）血量告急时的**分身影演出**，播一次 `split_fx` 自删（[注册](../../../raw/current/scripts/prefabs/bat_boss.lua#L923)）；③`batbat_bats`＋`batbat_fantasy_fx`——蝙蝠棒（batbat）两款皮肤的**命中扇翅**：无 AnimState 的 `net_tinybyte` 变体代理＋各端本地播放（[注册](../../../raw/current/scripts/prefabs/batbat.lua#L269-L270)）；④`battreefx`——万圣惊夜砍树惊出的**蝠群演出**，`SetClassifiedTarget` 只让伐木玩家收到实体（[注册](../../../raw/current/scripts/prefabs/battreefx.lua#L90)）。
- 本体辨析：`bat_boss`（官方名"猎犬蝙蝠"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339195-L339197)）是巨蝠洞 Boss 级生物，**自身就是可装备 HEAD 槽的活体"帽"**——SGbat 的 chomp 状态会把它 equip 到玩家头上；`bat_boss_shadow`（官方名"枯萎猎犬蝙蝠"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339205-L339207)）经分身机制增殖；`batbat`（官方名"蝙蝠棒"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339090-L339092)）是吸血武器本体。死亡掉落的合成帽 `bat_bosscorpsehat`（官方名"蝙蝠帽"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339200-L339202)）有独立 FX `bat_bosscorpsehat_fx`（[hats.lua:6566](../../../raw/current/scripts/prefabs/hats.lua#L6566)），不属本页。
- 重要副作用：帽层自带声音（联网吮血声＋本地 growl 循环，仅戴帽者是玩家时）；惊树蝠群连带 HUD 全屏蝙蝠覆盖层与掉理智（理智惩罚在宿主 `spooked` 事件消费侧）；皮肤扇翅纯表现。
- 修改边界：只覆盖 5 注册的工厂、生成点、端与清理；SGbat 战斗状态机全貌、皮肤系统全链、batover widget 内部不展开。bat_boss 本体辨析见[本体辨析](#batboss-本体辨析)。
- 名称：5 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 键（检索 `NAMES.BAT_BOSSHAT_FX`／`BAT_BOSS_SHADOW_SPLIT_FX`／`BATBAT_BATS`／`BATBAT_FANTASY_FX`／`BATTREEFX` 皆无命中），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bat_bosshat_fx` | `fxfn`：Transform／AnimState／**SoundEmitter**／Follower／Network → bank=build=`bat_boss`、播 `chomp_fx` → 标签 **DECOR＋NOCLICK**（不用 FX 标签，源码注释"don't use FX, so that we can use NOCLICK instead"）→ 共享段挂 `colouraddersync` 与 `OnRemoveEntity` → SetPristine → 客户端 `updatelooper` PostUpdate 轮询父挂与 build 同步 → 主模拟 `persists=false`、暴露 `AttachToOwner`／`TriggerChompFx` 协议方法 | 网络化本体层：build 由宿主 OnEquip 覆写（酸浸 `bat_boss_acid_build`／暗影 `bat_boss_shadow_actions`）；[工厂](../../../raw/current/scripts/prefabs/bat_boss.lua#L847-L879)、[标签与回调](../../../raw/current/scripts/prefabs/bat_boss.lua#L856-L865) |
| `bat_boss_shadow_split_fx` | `splitfxfn`：Transform／AnimState／SoundEmitter／Network → 八面向 → bank=`bat_boss`、build=`bat_boss_shadow_actions`、播一次 `split_fx` → 标签 `FX`＋`NOCLICK` → SetPristine 客户端早退 → 主模拟 `animover` 自删、`persists=false`、`OnEntityWake` 一次性播 clone 叫声 | 一次性演出：唤醒即播 `dontstarve/creatures/bat/clone`（用后即焚），[工厂](../../../raw/current/scripts/prefabs/bat_boss.lua#L889-L918)、[唤醒声](../../../raw/current/scripts/prefabs/bat_boss.lua#L884-L886) |
| `batbat_bats` | `batsfn`：**Transform＋Network 代理壳（无 AnimState）** → 标签 `FX`＋`nointerpolate` → `net_tinybyte` 变体（`batbat_bats.variation`）→ 非专服 `DoTaskInTime(0)` 本地播放 → SetPristine → 主模拟 0.5 秒自删、`persists=false`、`OnFXSpawned` 钩子 | 代理模式：本地播放体非联网，bank=`batbat_scythe`、build=`bat_tree_fx`、播 `bat1-4`；[代理工厂](../../../raw/current/scripts/prefabs/batbat.lua#L166-L192)、[本地播放](../../../raw/current/scripts/prefabs/batbat.lua#L122-L146) |
| `batbat_fantasy_fx` | `fantasyfn`：与 `batsfn` 同构，net 名 `batbat_fantasy.variation` | 本地播放体 bank=`batbat_fantasy`、**`SetSkin("batbat_fantasy")`**（走皮肤系统而非 SetBuild）；[工厂](../../../raw/current/scripts/prefabs/batbat.lua#L239-L266)、[SetSkin](../../../raw/current/scripts/prefabs/batbat.lua#L210-L212) |
| `battreefx` | `fn`：Transform／AnimState／Network → bank=build=`bat_tree_fx`（工厂内**不播动画**，等 SetViewerAndAnim 选型）→ 标签 `FX`（无 NOCLICK）→ SetPristine → 客户端直接播本地声效＋排覆盖层任务 → 主模拟 30 帧提层、`animover` 自删、`persists=false`、0 帧保险自删 | 单玩家演出：动画名由 `SetViewerAndAnim(viewer, anim)` 写入并设 classified 目标；[工厂](../../../raw/current/scripts/prefabs/battreefx.lua#L59-L85)、[协议方法](../../../raw/current/scripts/prefabs/battreefx.lua#L41-L55) |

## 关联机制导航

### bat_bosshat_fx 的戴帽链

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成 | `bat_boss` 的 `OnEquip`（唯一生成点）：SpawnPrefab 后按状态换 build（酸浸／暗影），再 `AttachToOwner(owner)` | [OnEquip](../../../raw/current/scripts/prefabs/bat_boss.lua#L205-L214) |
| 挂接 | `AttachToOwner`：SetParent＋`FollowSymbol(owner, "swap_hat")`＋owner 有 `colouradder` 时 `AttachChild`（染色随宿主）；非专服补三只本地小蝠与本地 growl 声 | [fx_AttachToOwner](../../../raw/current/scripts/prefabs/bat_boss.lua#L815-L825) |
| 本地小蝠层 | `SpawnFollowFxForOwner`：3 × `CreateFxFollowFrame`——`idle1/2/3` 动画、**共享同一随机起始帧**（同步扇翅）、`swap_hat` 符号上错位挂点（第 8 参 i-1）、`highlightchild` 认主、加色由 fx 的 `colouraddersync` 回调统一写入 | [SpawnFollowFxForOwner](../../../raw/current/scripts/prefabs/bat_boss.lua#L782-L795)、[本地工厂](../../../raw/current/scripts/prefabs/bat_boss.lua#L755-L779) |
| 咀嚼联动 | SGbat 的脑在冷却好后走 `chomp_attack`：`CanHatTarget`（目标有物品栏且头部帽无 `monsterhat` 标签）→ `TryHat` 把 bat_boss equip 上头并立刻 `TriggerChompFx`＋首咬 | [CanHatTarget](../../../raw/current/scripts/stategraphs/SGbat.lua#L41-L49)、[TryHat](../../../raw/current/scripts/stategraphs/SGbat.lua#L237-L245) |
| 周期咀嚼 | OnEquip 起每 0.6 秒 `DoChomp`：先 `fx:TriggerChompFx()`（播 `headbite_bloodsuck`＋重播 `chomp_fx`＋随机镜像）再 `DoAttack(owner)` 结算伤害 | [DoChomp](../../../raw/current/scripts/prefabs/bat_boss.lua#L175-L179)、[周期任务](../../../raw/current/scripts/prefabs/bat_boss.lua#L239-L240)、[TriggerChompFx](../../../raw/current/scripts/prefabs/bat_boss.lua#L830-L833) |
| 客户端补建 | 客户端分支用 `updatelooper` PostUpdate 轮询：父挂就绪（等待 build 同步）后同调 `SpawnFollowFxForOwner`＋玩家 HUD 加 growl 声 | [fx_PostUpdate](../../../raw/current/scripts/prefabs/bat_boss.lua#L803-L811)、[客户端分支](../../../raw/current/scripts/prefabs/bat_boss.lua#L869-L874) |
| 收尾 | OnUnequip 显式 `fx:Remove()`；fx 的 `OnRemoveEntity` 再清三只小蝠与 focal 声道 | [OnUnequip](../../../raw/current/scripts/prefabs/bat_boss.lua#L264-L266)、[OnRemoveEntity](../../../raw/current/scripts/prefabs/bat_boss.lua#L836-L843) |

#### bat_boss 本体辨析

`bat_boss` 由巨蝠洞 `batbosscave` 的 `childspawner` 产出（玩家靠近即一口气放出，[SpawnChild](../../../raw/current/scripts/prefabs/batbosscave.lua#L13-L21)、[childname](../../../raw/current/scripts/prefabs/batbosscave.lua#L121)），是"活体帽"：被 chomp 状态戴到玩家头上后以头部装备身份每 0.6 秒咬一口（伤害走本体 combat），卸下时本体落到玩家脚下并按佩戴时长给眩晕/冷却（OnEquip/OnUnequip 同时负责换 FX build 与酸浸/暗影分支）。`bat_boss_shadow`（血量低于阈值时由脑推 `doclone` 增殖，[推送](../../../raw/current/scripts/brains/bat_bossbrain.lua#L149-L156)）共享同一套 OnEquip，即暗影蝙蝠戴头时帽层换 `bat_boss_shadow_actions` build——但它的双眉本地火焰（`CreateFlameLoop`，[生成](../../../raw/current/scripts/prefabs/bat_boss.lua#L651-L669)）是本体工厂内非注册本地实体，不属本页覆盖。

### bat_boss_shadow_split_fx 的分身链

脑在"有目标＋血量低于 `BAT_BOSS_SHADOW_CLONE_THRESHOLD`＋无既有分身"时推 `doclone`（[推送](../../../raw/current/scripts/brains/bat_bossbrain.lua#L149-L156)）；SG 事件只记时间戳（[记录](../../../raw/current/scripts/stategraphs/SGbat.lua#L176-L178)），`TryClone` 过 `CLONE_DELAY` 后进 `clone_pre` → `clone` 状态：先在原位同向生成 FX 播 `split_fx`，再生成真正的 `bat_boss_shadow` 本体（180° 背向、继承血量与目标、互设追踪）（[FX 生成](../../../raw/current/scripts/stategraphs/SGbat.lua#L1587-L1589)、[本体生成](../../../raw/current/scripts/stategraphs/SGbat.lua#L1591-L1593)）。FX 的 clone 叫声绑定 `OnEntityWake` 一次性播放（[唤醒声](../../../raw/current/scripts/prefabs/bat_boss.lua#L884-L886)），动画播完自删。

### batbat_bats／batbat_fantasy_fx 的皮肤代理链

- **皮肤映射**：两款皮肤在 skinprefabs 定义 `fx_prefab`——`batbat_fantasy`（官方皮肤名"魔幻蝙蝠棒"，[po](../../../raw/current/scripts/languages/chinese_s.po#L391315-L391317)）→ `batbat_fantasy_fx`；`batbat_scythe`（"蝙蝠镰刀"，[po](../../../raw/current/scripts/languages/chinese_s.po#L391320-L391322)）→ `batbat_bats`（[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1704-L1715)、[镰刀](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1717-L1728)）；`prefabskin` 统一装载进全局 `SKIN_FX_PREFAB` 表（[装载](../../../raw/current/scripts/prefabskin.lua#L4094-L4095)）。
- **生成**：蝙蝠棒 `onattack` 命中有效目标时查 `SKIN_FX_PREFAB[GetSkinName()]`，SpawnPrefab 后父挂**目标实体**并在其 `hiteffectsymbol` 上挂 Follower，再调 `OnFXSpawned(武器)`（[onattack](../../../raw/current/scripts/prefabs/batbat.lua#L35-L44)）。
- **变体洗牌**：`OnBatFXSpawned` 在武器上维护 `_batfxvariations` 洗牌序列（1-4 全排列），每次从队首两个候选取用、用后回填队尾，避免连击重复；无宿主武器时纯随机 4 选 1（[洗牌](../../../raw/current/scripts/prefabs/batbat.lua#L150-L160)）。
- **本地播放**：非专服在生成后一帧读 `variation>0` 建非联网播放体（位置复制自代理，V2C 注明有意不用 `SetFromProxy`）；扇翅声 `DoFlutterSound` 音量 outQuad 递减、间隔 9-10 帧连播 5 次后停；动画 `bat1-4` 播完自删（[PlayBatFX](../../../raw/current/scripts/prefabs/batbat.lua#L122-L146)、[递减声](../../../raw/current/scripts/prefabs/batbat.lua#L115-L118)、[animover](../../../raw/current/scripts/prefabs/batbat.lua#L146)）。代理本体 0.5 秒后自删（[主模拟收尾](../../../raw/current/scripts/prefabs/batbat.lua#L188)）。

### battreefx 的惊树链（万圣惊夜）

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 组件门禁 | 仅 `HALLOWED_NIGHTS` 活动期给玩家挂 `spooked` 组件并监听 `spooked` 事件 | [挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2746-L2748) |
| 触发 | `ACTIONS.CHOP` 工作成功后调 `spooked:Spook(树)` | [CHOP](../../../raw/current/scripts/actions.lua#L1748-L1754) |
| 选型门禁 | `FX_MAP` 按树 prefab/stage 选动画名（evergreen/lumpy/twiggy/leaf/marsh/mush 各型）；仅 stage 4（老树）且过 `ShouldSpook` 门槛（惊吓度>70 后掷运）才生成 | [FX_MAP](../../../raw/current/scripts/components/spooked.lua#L1-L30)、[门槛](../../../raw/current/scripts/components/spooked.lua#L79-L83)、[惊吓度](../../../raw/current/scripts/components/spooked.lua#L37-L44) |
| 生成 | FX 落在树位（y=-0.1）、继承树的缩放，`SetViewerAndAnim(伐木者, 动画)` 写入动画并设 classified 目标 | [生成](../../../raw/current/scripts/components/spooked.lua#L89-L98) |
| 可见性 | `Network:SetClassifiedTarget(viewer)`——只有该玩家的客户端收到实体；主模拟侧若 viewer 不是本机玩家（聆听主机），把自份 `OverrideMultColour` alpha 0 隐藏 | [SetViewerAndAnim](../../../raw/current/scripts/prefabs/battreefx.lua#L49-L55) |
| 本地声效 | `PlaySounds` 建声效子体（Transform＋SoundEmitter）：`bat_explode` 一次＋循环 flap，至父动画约 1.25 秒（减 8 帧）后子体自删 | [PlaySounds](../../../raw/current/scripts/prefabs/battreefx.lua#L21-L30)、[循环](../../../raw/current/scripts/prefabs/battreefx.lua#L12-L17) |
| 覆盖层 | `QueueBatOverlay` 让 ThePlayer 在动画约 1.25 秒时推 `batspooked`；HUD `batover` widget 收到后播全屏 `overlay`/`overlay_tall` | [排队](../../../raw/current/scripts/prefabs/battreefx.lua#L33-L37)、[widget 监听](../../../raw/current/scripts/widgets/batover.lua#L27)、[触发](../../../raw/current/scripts/widgets/batover.lua#L31-L37) |
| 演出收尾 | 主模拟 30 帧 `SetSortOrder(1)` 提层、`animover` 自删；0 帧保险自删（未被 `SetViewerAndAnim` 配置即删） | [主模拟段](../../../raw/current/scripts/prefabs/battreefx.lua#L80-L85) |

真正的惊吓代价在宿主侧：`spooked` 事件消费后延迟 1.35 秒扣理智（[OnSpooked](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L899-L904)），与覆盖层时机对齐。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `bat_bosshat_fx` | 卸下/死亡掉落触发 OnUnequip 显式 Remove；`OnRemoveEntity` 补清三只本地小蝠与 focal growl 声道；本体（bat_boss）移除走引擎级联 | 三只小蝠父挂在**owner**上而非 fx 上，靠 OnRemoveEntity 显式清（[清理](../../../raw/current/scripts/prefabs/bat_boss.lua#L836-L843)） |
| `bat_boss_shadow_split_fx` | `animover` 自删（唯一入口） | 分身本体被打死不影响已生成的 FX（[自删](../../../raw/current/scripts/prefabs/bat_boss.lua#L912)） |
| `batbat_bats`／`batbat_fantasy_fx` | 代理 0.5 秒自删；本地播放体 animover 自删——两条生命周期独立 | 播放体不是代理的子体，代理先删不级联（[播放体独立](../../../raw/current/scripts/prefabs/batbat.lua#L134-L146)） |
| `battreefx` | `animover` 自删＋0 帧保险自删；声效子体由父动画计时自删 | 玩家侧覆盖层任务挂在 ThePlayer 上，FX 先删不撤任务（[保险自删](../../../raw/current/scripts/prefabs/battreefx.lua#L84)） |

## 典型接入

### 复刻"变体代理＋本地播放"的命中特效

- 前提与选择：命中表现想让每端各自渲染、且各端看到不同变体时，用一个只带 Network 的代理传 `net_tinybyte` 变体号：master 生成代理后立刻设变体，各端下帧读值建本地动画体。
- 生效与撤销：master 侧代理 0.5 秒自删即可；播放体 animover 自删；变体在宿主上洗牌可避免连续命中重复。

```lua
local function OnFXSpawned(inst, weapon) -- master：SpawnPrefab 后立刻被调
    inst.variation:set(math.random(4))
end

local function PlayLocalFx(proxy) -- 各非专服端下帧执行
    if proxy.variation:value() > 0 then
        local fx = CreateEntity()
        fx.persists = false
        fx.entity:AddTransform()
        fx.entity:AddAnimState()
        fx.AnimState:SetBank("mybank")
        fx.AnimState:PlayAnimation("hit"..tostring(proxy.variation:value()))
        fx:ListenForEvent("animover", fx.Remove)
    end
end
-- master 分支：inst:DoTaskInTime(.5, inst.Remove)；非专服分支：inst:DoTaskInTime(0, PlayLocalFx)
```

- 调整与验证：替换点是 bank/动画名与变体位数（`net_tinybyte` 上限 255）；变体必须生成同帧设置，晚于播放任务会整次跳播。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`bat_bosshat_fx` 的标签是 DECOR 而不是 FX。** 源码注释写明"don't use FX, so that we can use NOCLICK instead"（[L860](../../../raw/current/scripts/prefabs/bat_boss.lua#L860)）；按 `FX` 标签检索这类网络 FX 会漏掉它。
- **边界：三只小蝠与 growl 声只在非专服存在。** 它们由 `SpawnFollowFxForOwner` 创建且父挂在玩家身上（[L789](../../../raw/current/scripts/prefabs/bat_boss.lua#L789)）；专服只有网络化的咀嚼层本体，删 fx 本体不级联小蝠——必须走 `OnRemoveEntity`。
- **边界：代理与播放体是两个实体。** `batbat_bats` 的本地动画体有意不 `SetParent`/不 `SetFromProxy`（[V2C 注释](../../../raw/current/scripts/prefabs/batbat.lua#L134)），只复制位置；给代理挂子体、或指望代理删除带走播放体，都会落空。
- **边界：`variation` 为 0 时两端都不播。** 本地播放以 `proxy.variation:value() > 0` 为门禁（[L123](../../../raw/current/scripts/prefabs/batbat.lua#L123)）；复刻时变体必须在生成同帧由 `OnFXSpawned` 设置，否则整次命中无声无画。
- **边界：`battreefx` 只让一个人看见。** `SetClassifiedTarget` 使其余客户端根本没有该实体；聆听主机上"viewer 不是本机玩家"时还会把自己的副本 alpha 隐藏（[L55](../../../raw/current/scripts/prefabs/battreefx.lua#L55)）——不能把它当"全端同步"的 FX 参考。
- **边界：惊树链是万圣惊夜限定。** `spooked` 组件只在 `HALLOWED_NIGHTS` 活动期挂载（[门禁](../../../raw/current/scripts/prefabs/player_common.lua#L2746-L2748)），平时砍树不会出蝠群；且只有 stage 4（老树）参与判定。
- **边界：`bat_bosshat_fx` ≠ 蝙蝠帽的 FX。** 死亡掉落的 `bat_bosscorpsehat`（合成帽）用的是 hats.lua 的 `bat_bosscorpsehat_fx`（[对照](../../../raw/current/scripts/prefabs/hats.lua#L6566)）；本页 FX 只服务活体蝙蝠被戴头的场景。
- **边界：帽层的 build 跟随宿主状态。** 酸浸蝙蝠戴头时帽层换 `bat_boss_acid_build`、暗影蝙蝠换 `bat_boss_shadow_actions`（[换装](../../../raw/current/scripts/prefabs/bat_boss.lua#L209-L213)），客户端的三只小蝠以同步后的 build 生成（[等待注释](../../../raw/current/scripts/prefabs/bat_boss.lua#L870)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/bat_boss.lua`、`prefabs/batbat.lua`、`prefabs/battreefx.lua`、`components/spooked.lua`；按需读取 `stategraphs/SGbat.lua`（CanHatTarget/TryHat/chomp_attack/clone/TryClone/doclone）、`brains/bat_bossbrain.lua`（doclone 推送）、`prefabs/skinprefabs.lua` 与 `prefabskin.lua`（皮肤 fx_prefab 映射与装载）、`prefabs/batbosscave.lua`（本体生成）、`widgets/batover.lua`（覆盖层）、`prefabs/player_common.lua`（spooked 挂载与依赖表）、`prefabs/player_common_extensions.lua`（OnSpooked）、`actions.lua`（CHOP）、`prefabs/hats.lua`（蝙蝠帽 FX 边界）。对 5 个覆盖注册名以完整名在 `raw/current/scripts` 全域反查：`bat_bosshat_fx` 命中注册（bat_boss.lua:921）、依赖（:28/:38）、生成（:208）——**生成点唯一**；`bat_boss_shadow_split_fx` 命中注册（:923）、依赖（:39）、生成（SGbat.lua:1587）——**生成点唯一**；`batbat_bats` 命中注册（batbat.lua:269）、net 名（:175）、皮肤映射（skinprefabs.lua:1723/1726）——生成路径唯一（onattack 经 `SKIN_FX_PREFAB` 间接 SpawnPrefab）；`batbat_fantasy_fx` 命中注册（batbat.lua:270）、net 名（:248）、皮肤映射（skinprefabs.lua:1710/1713）——同上；`battreefx` 命中注册（battreefx.lua:90）、生成（components/spooked.lua:91）、依赖（player_common.lua:2220、prefablist.lua:105）——**生成点唯一**。宿主侧反查：`bat_boss` 由 batbosscave.lua:121 的 childspawner 产出、`bat_boss_shadow` 由 SGbat.lua:1591 分身生成；`doclone` 仅 bat_bossbrain.lua:156 推送、SGbat.lua:176 监听；`batspooked` 仅 battreefx.lua:37 推送、batover.lua:27 监听。台账批 2 结论复核属实（代理＋本地播放、皮肤映射在 skinprefabs.lua）；三个源文件内未发现清单外 FX 注册（CreateFlameLoop／CreateFxFollowFrame／PlayBatFX／PlaySounds 均为本地工厂，非注册 Prefab）。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.BAT_BOSS`＝"猎犬蝙蝠"、`BAT_BOSS_SHADOW`＝"枯萎猎犬蝙蝠"、`BAT_BOSSCORPSEHAT`＝"蝙蝠帽"、`BATBAT`＝"蝙蝠棒"、`BAT`＝"洞穴蝙蝠"（锚点见"快速判断"）；皮肤名 `SKIN_NAMES.batbat_fantasy`＝"魔幻蝙蝠棒"、`batbat_scythe`＝"蝙蝠镰刀"；5 个覆盖注册名均无 NAMES 键，标题使用效果描述。
- 分档说明：`size_class: standard`——5 注册四条宿主链，链间无共享状态，各链独立小节承载；未选 complex，无多变体状态机、无需整体保留的协作链。
- 未覆盖：SGbat 攻击/眩晕/嚎叫状态机全貌与 chomp_attack 帧表、bat_boss 数值与掉落、皮肤系统全链（`batbat_init_fn` 与 woven 换装）、batover widget 的 OnUpdate 细节、`bat_bosscorpsehat_fx` 工厂、FX_MAP 六类树的资源对应、动画包帧内容、皮肤 FX 命中位置的跨端表现；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/shadow_bishop_fx|暗影主教羽群与暗影裂隙散件]]（同为 Boss 战表现 FX 的网络壳写法对照）
- [[reference/prefabs/spooked_spider_rock_fx|惊吓蜘蛛虫群特效]]（同一 spooked 组件的 `TryCustomSpook` 分支——挖堆/敲石惊吓）
- [[reference/prefabs/woby_dash_shadow_fx|沃比暗影冲刺特效]]（本地残影代理的另一次实现）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（蝙蝠 Boss 系）
