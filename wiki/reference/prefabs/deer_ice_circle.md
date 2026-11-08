---
title: "无眼鹿冰火法阵（deer_ice_circle）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "宝石无眼鹿施法落地的持续法阵：冰圈冻结锁定＋减速＋降温，火圈积热点燃＋升温＋propagator 真实传播火焰；同文件另有充能角饰与子效共 9 个注册。"
aliases: ["deer_ice_circle", "deer_fire_circle", "deer_ice_fx", "deer_ice_burst", "deer_ice_flakes", "deer_ice_charge", "deer_fire_burst", "deer_fire_flakes", "deer_fire_charge", "deer_fx", "无眼鹿法阵", "冰火法阵", "克劳斯鹿法阵"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/deer_fx.lua", "prefabs/deer.lua", "stategraphs/SGdeer.lua", "prefabs/deerclops_mutated_fx.lua", "prefabs/deerclopseyeball_sentryward.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 无眼鹿冰火法阵（deer_ice_circle）

## 快速判断

- 职责与适用：宝石无眼鹿（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L341475-L341477)；红／蓝 gem 变体）施法时在目标位置落下的持续法阵：冰圈逐帧冻结锁定／减速／降温／灭火，火圈逐帧积热点燃／升温并用 propagator 真实传播火焰——都是有玩法作用的实体，不是纯视觉。同文件另有施法角饰与子效，共 9 个注册。[注册表](../../../raw/current/scripts/prefabs/deer_fx.lua#L499-L547)、[冰圈效果](../../../raw/current/scripts/prefabs/deer_fx.lua#L129-L192)、[火圈效果](../../../raw/current/scripts/prefabs/deer_fx.lua#L255-L315)
- 优先使用：查"地面冰圈／火圈／冻结减速圈／火焰传播圈"选本页；施法节奏与目标选择见 deer.lua 的 DoCast 链。
- 重要副作用：冰圈内 locomotor 临时减速系数 0.4、温度高于 -2 时降到 -2、已冻目标锁定 30 帧、未昏睡且 grog<0.1 时增加 0.1（可越过 0.1）；火圈内 30 帧积热后点燃、温度抬到 72，propagator 随半径传播伤害。[冰参数](../../../raw/current/scripts/tuning.lua#L697-L701)、[火参数](../../../raw/current/scripts/tuning.lua#L704-L706)
- 修改边界：法阵效果全部在主模拟逐帧（period 0 任务）；客户端另有 ThePlayer 减速预测与 light 渐变 netvar；带 `deergemresistance` 标签的宝石鹿自身免减速。[客户端预测](../../../raw/current/scripts/prefabs/deer_fx.lua#L194-L205)、[主模拟入口](../../../raw/current/scripts/prefabs/deer_fx.lua#L213-L218)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deer_ice_circle` | MakeFX 数据驱动：light(冰色)＋onground＋ice_circle_LP 循环音 → common 加 `deer_ice_circle` 标签与 `_rad` net_float → SetPristine → 客户端早退（起本地减速预测）→ 主模拟 OnInitIceCircle 每帧扩张结算、animover 停音、KillFX 清标签归零 | 冻结／减速主逻辑 OnUpdateIceCircle；TriggerFX 后才有子效与强冷路径。[构造](../../../raw/current/scripts/prefabs/deer_fx.lua#L499-L508)、[common](../../../raw/current/scripts/prefabs/deer_fx.lua#L224-L232) |
| `deer_fire_circle` | 同工厂：light(火色)＋bloom＋onground＋fire_circle_LP → 主模拟加 propagator（damages=true）StartSpreading、OnInitFireCircle 每帧扩张结算 | 点燃与传播逻辑 OnUpdateFireCircle；KillFX 停传播。[构造](../../../raw/current/scripts/prefabs/deer_fx.lua#L526-L536)、[propagator](../../../raw/current/scripts/prefabs/deer_fx.lua#L328-L336) |
| `deer_ice_charge`／`deer_fire_charge` | light＋bloom＋looping＋charge_LP；master 延 1 帧播 gemstaff＋beam 音 | 施法前奏父挂到鹿角 `swap_antler_red`，收招 KillFX("blast")。[注册](../../../raw/current/scripts/prefabs/deer_fx.lua#L516-L524)、[音效](../../../raw/current/scripts/prefabs/deer_fx.lua#L354-L361) |
| `deer_ice_fx`／`deer_ice_flakes`／`deer_fire_flakes` | looping 子效（ice_fx 带 steam_LP 循环音）；TriggerFX 时父挂到法阵实体 | onkillfx 停 loop 音；随 KillFX 一起退场。[注册](../../../raw/current/scripts/prefabs/deer_fx.lua#L509-L515)、[火片](../../../raw/current/scripts/prefabs/deer_fx.lua#L537-L538) |
| `deer_ice_burst`／`deer_fire_burst` | oneshotanim 单次动画（`loop`／`idle`），按动画长度 +0.25 秒后自删 | 法阵边缘随机角度爆裂。[oneshot 路径](../../../raw/current/scripts/prefabs/deer_fx.lua#L470-L472)、[注册](../../../raw/current/scripts/prefabs/deer_fx.lua#L514) |

法阵与角饰均无 NAMES 中文条目；标题"法阵"为已核验效果描述，非官方译名。

## 关联机制导航

法阵由主模拟生成方安排 KillFX 收场；角饰由施法状态收招时 KillFX("blast")，循环子效随法阵 KillFX 退场，burst 则按动画长度＋0.25 秒自删。下列小节区分玩法结算与表现子效。[法阵定时](../../../raw/current/scripts/prefabs/deer.lua#L235-L239)、[收招](../../../raw/current/scripts/stategraphs/SGdeer.lua#L382-L398)、[子效退场](../../../raw/current/scripts/prefabs/deer_fx.lua#L68-L89)、[单次退场](../../../raw/current/scripts/prefabs/deer_fx.lua#L470-L472)

### 冰圈结算（deer_ice_circle）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 冰圈结算：OnUpdateIceCircle | 半径每帧 `×0.98+0.06` 收敛到 3；找 oneof {locomotor, freezable, fire, smolder, inventoryitemtemperature}；减速 PushTempGroundSpeedMultiplier(0.4)（deergemresistance 免）；burnable 且非 fueled 的灭火；冻结锁定 30 帧＋缓冷 0.1，未冻且无 track 时补冷 max(1, resistance-coldness+1) | NOTAGS 含全部 FUELTYPE 的 `*_fueled` 与 playerghost／INLIMBO／flight／invisible；补冷/锁定仅在 TriggerFX 后，之前只有 70% 抗性以下缓冷 | [扩张与目标](../../../raw/current/scripts/prefabs/deer_fx.lua#L129-L146)、[冻结链](../../../raw/current/scripts/prefabs/deer_fx.lua#L156-L177)、[温度与grog](../../../raw/current/scripts/prefabs/deer_fx.lua#L179-L189)、[豁免表](../../../raw/current/scripts/prefabs/deer_fx.lua#L122-L128) |
| 客户端减速预测 | OnInitIceCircleClient 每帧对 ThePlayer 应用同款减速系数（非 ghost 且在半径内） | 仅本地玩家表现层；服务端仍对实体 locomotor 结算 | [客户端](../../../raw/current/scripts/prefabs/deer_fx.lua#L194-L211) |

### 火圈结算（deer_fire_circle）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 火圈结算：OnUpdateFireCircle | 半径每帧 `×0.9+0.36` 收敛到 3.6 并同步 propagator 范围；对 `_health`／canlight／freezable／inventoryitemtemperature 之一的目标：处于冰圈 3 距离内则整条跳过；冻结的解冻／冷度 -0.1；非 fueled 且有 health 的可燃物累计计数大于 30 后 `Ignite(true, inst)`、已燃 ExtendBurning | 温度低于 72 的抬到 72（高于 OVERHEAT_TEMP）；propagator damages=true 是真实火焰传播伤害 | [跳过冰圈目标](../../../raw/current/scripts/prefabs/deer_fx.lua#L274-L281)、[点燃链](../../../raw/current/scripts/prefabs/deer_fx.lua#L283-L305)、[传播组件](../../../raw/current/scripts/prefabs/deer_fx.lua#L328-L336) |

### 充能角饰与爆裂子效

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SGdeer 施法态 | magic_pre 第 268 行起生成 charge 角饰父挂鹿角；蓝鹿 19.5 帧 DoCast、红鹿在后续态 25 帧帧事件 DoCast；25 帧对 spells 逐个 TriggerFX 并 KillFX(charge, "blast") | `deergemresistance` 标签持有者（宝石鹿）进入冰/火圈时只置 shouldavoidmagic | [角饰父挂](../../../raw/current/scripts/stategraphs/SGdeer.lua#L265-L271)、[蓝鹿施法](../../../raw/current/scripts/stategraphs/SGdeer.lua#L285-L289)、[触发与收招](../../../raw/current/scripts/stategraphs/SGdeer.lua#L382-L398) |
| 爆裂与子效 | 冰圈起手 6 帧后每 5-6 帧在 `rad-0.7~rad-0.2` 环上落 deer_ice_burst；火圈 rad>1 后每 1-2 帧落 deer_fire_burst；TriggerFX 父挂 ice_fx／flakes／fire_flakes | DoBurst 随机角度（8 扇区抽取后移到表尾，允许后续再次抽中） | [TriggerFX](../../../raw/current/scripts/prefabs/deer_fx.lua#L50-L66)、[随机角](../../../raw/current/scripts/prefabs/deer_fx.lua#L93-L109)、[冰爆裂](../../../raw/current/scripts/prefabs/deer_fx.lua#L132-L138)、[火爆裂](../../../raw/current/scripts/prefabs/deer_fx.lua#L260-L266) |

### 共享施法与资源

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 施法入口：gemmed deer | castfx 按宝石选火圈／冰圈，castduration 4／6 秒后 KillFX，CD 8／10；DoCast 启动 deercast_cd（首次 4 秒） | SpawnSpell 落在目标位置；SpawnSpells 单轮最多 6 个、先 6 距离防重叠再 5→3 递减重试 | [castfx 配置](../../../raw/current/scripts/prefabs/deer.lua#L571-L577)、[SpawnSpell](../../../raw/current/scripts/prefabs/deer.lua#L235-L239)、[DoCast](../../../raw/current/scripts/prefabs/deer.lua#L279-L283)、[防重叠](../../../raw/current/scripts/prefabs/deer.lua#L191-L194) |
| 光照渐变 | `_fade` net_smallbyte（fadedirty）驱动 light 强度／半径／falloff：onground 档 30 帧、半径 3；非 ground 档 15 帧、半径 2；KillFX 时 FadeOut | 主模拟每帧任务推 netvar，客户端监听 fadedirty 复算 | [渐变](../../../raw/current/scripts/prefabs/deer_fx.lua#L4-L40)、[配置](../../../raw/current/scripts/prefabs/deer_fx.lua#L418-L445)、[客户端监听](../../../raw/current/scripts/prefabs/deer_fx.lua#L456-L462) |
| 资源复用：deerclops 系 | `deer_ice_circle` 的 bank/build 被突变独眼巨鹿 FX 与独眼巨鹿哨兵塔复用（只借动画资源，不是生成法阵） | 资源声明复用，与法阵玩法无关 | [突变巨鹿](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L266-L267)、[哨兵塔](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L599-L600) |
| 数值：tuning | 首施 CD 4、施法距离 15、单轮上限 6、冰减速 0.4、冰温 -2、grog 0.1、冻结锁 30 帧、冰 CD 10、火温 72、点燃 30 帧、火 CD 8 | 单位：秒／帧／温度值／速度系数 | [数值](../../../raw/current/scripts/tuning.lua#L691-L706) |

## 典型接入

### 复用冰／火法阵

- 前提与选择：主模拟生成后设位置，需要子效与强冻结链时调用 `TriggerFX()`，到时调 `KillFX()` 收场；法阵效果从生成下帧开始。
- 生效与撤销：`persists=false` 不入存档；KillFX 清标签／停传播／渐隐光照，子效随 onremove 计数回收。
- 原版依据：deer.lua SpawnSpell 即"落点→定时 KillFX"的完整调用。

```lua
local fx = SpawnPrefab("deer_ice_circle")   -- 或 "deer_fire_circle"
fx.Transform:SetPosition(x, 0, z)
fx:TriggerFX()                              -- 可选：生成冰雾子效并启用强冻结链
fx:DoTaskInTime(6, fx.KillFX)               -- 原版冰圈 castduration=6
```

- 调整与验证：不调 TriggerFX 时冰圈走弱冷路径（仅 70% 抗性以下缓冷），与原版完整表现不同。[弱冷路径](../../../raw/current/scripts/prefabs/deer_fx.lua#L173-L175)

## 常见陷阱与限制

- **误区：只是冰火特效**：火圈 propagator `damages=true` 会真实传播火焰伤害；冻结／减速／温度／grog 都是持续玩法作用。[传播](../../../raw/current/scripts/prefabs/deer_fx.lua#L331-L335)
- **误区：火圈碰到就点燃**：非 fueled 且有 health 的可燃物需连续在圈内累计计数大于 30 才 Ignite，已燃的才 ExtendBurning；帧计数按目标分表跟踪。[积热点燃](../../../raw/current/scripts/prefabs/deer_fx.lua#L297-L305)
- **边界：fueled 豁免**：NOTAGS 含全部 FUELTYPE 的 `*_fueled`，带燃料槽的设施不受灭火／点燃影响。[豁免表](../../../raw/current/scripts/prefabs/deer_fx.lua#L122-L126)
- **边界：冰火互斥**：火圈对 3 距离内有未 killed 冰圈的目标整条跳过。[互斥](../../../raw/current/scripts/prefabs/deer_fx.lua#L274-L281)
- **边界：减速双端各自结算**：主模拟对每个 locomotor 实体推临时系数（[服务端](../../../raw/current/scripts/prefabs/deer_fx.lua#L150)），客户端只对 ThePlayer 做预测（[客户端](../../../raw/current/scripts/prefabs/deer_fx.lua#L202)）；无 locomotor 的实体不受减速。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；548 行工厂全文（9 注册、扩张／冻结／点燃／传播／渐变／爆裂／随机角）、deer.lua 施法链（castfx／DoCast／SpawnSpells／防重叠）、SGdeer 施法态与角饰、deerclops 两处资源复用、tuning 数值、译名检索已核验；freezable 抗性与冻结阈值内部实现、brain_gemmed 决策全流程、晚加入客户端的 `_rad` 初始显示未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
