---
title: "独眼巨鹿激光族：焦痕、烟尾与命中闪光（deerclops_laserscorch 等 5 注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "独眼巨鹿冬季盛宴激光攻击的五个注册：三个纯 FX（地面焦痕、命中点红色烟团、目标加色闪光）与两个带 combat／Trigger 的混合载体，按生成、端、淡出与伤害归属核验边界。"
aliases: ["deerclops_laserscorch", "deerclops_lasertrail", "deerclops_laserhit", "deerclops_laser", "deerclops_laserempty", "独眼巨鹿激光", "独眼巨鹿激光族", "巨鹿激光", "激光焦痕", "冬季盛宴巨鹿激光"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/deerclops_laser.lua", "prefabs/deerclops.lua", "stategraphs/SGdeerclops.lua", "prefabs/willow_ember.lua", "prefabs/willow_flame_fx.lua", "components/combat.lua", "componentutil.lua", "tuning.lua", "netvars.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 独眼巨鹿激光族：焦痕、烟尾与命中闪光（deerclops_laserscorch 等 5 注册）

## 快速判断

- 职责与适用：独眼巨鹿（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L342075-L342077)）冬季盛宴形态的激光攻击配套五个注册：`deerclops_laser`／`deerclops_laserempty` 是带 combat、由 `Trigger` 结算伤害的混合载体，`deerclops_laserscorch`／`deerclops_lasertrail`／`deerclops_laserhit` 是三个纯 FX。[注册](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L400-L404)、[冬季盛宴开关](../../../raw/current/scripts/prefabs/deerclops.lua#L522-L529)
- 优先使用：复用“地面焦痕淡出／命中点红色烟团／目标加色闪光”看三个纯 FX 行；要理解激光如何分段生成、何时结算伤害及归属，看“关联机制导航”的 SG、Trigger 与伤害归属条目。[SpawnLaser](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L177-L224)、[DoDamage](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L39-L147)
- 重要副作用：载体不是纯视觉：SG 把 `caster` 设为巨鹿后逐段 `Trigger`，`DoDamage` 会破坏可工作物、采摘、对 `_combat` 目标结算伤害、抛射物品；三个纯 FX 自身没有 combat／health。[伤害与附带效果](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L101-L125)、[物品](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L131-L146)
- 修改边界：激光只在冬季盛宴巨鹿的 `laserbeam` 状态生成；载体的 `Trigger`、烟团的 `FastForward` 都在主模拟早退之后才赋值，客户端没有这些方法；`laserhit` 是非联网、无位置的目标附加闪光。[端分界](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L192-L204)、[状态](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L933-L948)

## 注册与构造

同一文件五个独立工厂；两个载体共用 `common_fn`，三个 FX 各自独立。下表按“载体 → 纯 FX”顺序。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deerclops_laser` | `fn → common_fn(false)`：Transform＋Network；非空分支加 AnimState（bank `deerclops_laser_hits_sparks`／build `deerclops_laser_hit_sparks_fx`，命中时播 `hit_1..5`）＋Light（强度 .6、半径 1、衰减 .7、色 1/.2/.3，初始关闭）→ Hide、`notarget`／`hostile`、`SetPrefabNameOverride("deerclops")` → SetPristine → 客户端早退 → combat（默认 150）＋0 帧自删兜底＋`Trigger`＋`persists=false`。 | 混合载体：`DoDamage` 时才 `Show()`、播 `hit_N`、开灯（第 4 帧半径 .5、第 5 帧关）并在动画长＋2 帧后自删，同时生成焦痕／烟团。[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L164-L207)、[命中表现](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L43-L58)、[注册](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L400) |
| `deerclops_laserempty` | `emptyfn → common_fn(true)`：保留 Transform＋Network、Hide、标签、名称覆盖、combat／`Trigger`／0 帧自删；**不建 AnimState／Light**。 | 隐形玩法变体：`DoDamage` 不播动画、不生成焦痕／烟团，2 帧后自删，但仍按 `Trigger` 结算伤害；SG 只在激光起点 i=0 生成它，并把它当作相机震动源。[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L164-L207)、[空变体收尾](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L57-L58)、[起点生成](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L200-L206) |
| `deerclops_laserscorch` | `scorchfn`：Transform＋AnimState＋Network → bank/build `burntground` 播 `idle`，OnGround／背景层／SortOrder 3 → NOCLICK／FX → net_byte `deerclops_laserscorch._fade` 初值 75（脏事件 `fadedirty`）→ `DoPeriodicTask(0, Scorch_OnUpdateFade)`＋首次着色 → SetPristine → 客户端监听 fadedirty 并早退 → 主模拟随机旋转、`persists=false`。 | 纯 FX：20 帧红光＋40 帧保持＋15 帧淡出（75 tick）；服务端减到 1 时 Remove，客户端把本地值置 0 并转透明；无 combat／health／音效。[淡出](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L217-L249)、[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L251-L286)、[netvar](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L268-L272) |
| `deerclops_lasertrail` | `trailfn`：Transform＋AnimState＋Network → FX／NOCLICK → bank/build `lavaarena_staff_smoke_fx` 播 `idle`，`SetAddColour(1,0,0,0)`＋`SetMultColour(1,0,0,1)`＋bloom → SetPristine → 客户端早退 → 主模拟 `persists=false`、动画长＋2 帧后 Remove，并暴露 `FastForward`。 | 纯 FX：红色烟团，实际生成在每个命中点而不是沿激光路径；`FastForward(pct)` 取 0-1、取消旧任务、`SetTime(len*pct)` 后按剩余时长重排；原版以 0.3-0.7 随机相位错开。[FastForward](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L288-L296)、[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L298-L327)、[命中点调用](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L52-L55) |
| `deerclops_laserhit` | `hitfn`：只 CreateEntity → `CLASSIFIED` 标签、注释“Non-networked entity”、`persists=false` → 暴露 `SetTarget`、0 帧兜底 Remove；无 Transform／AnimState／Network。 | 非联网的目标加色闪光，不能摆位；`SetTarget` 取消兜底任务、设 `OnRemoveEntity`，有 `bloomer` 走 `PushBloom` 否则直写 AnimState，`flash` 0.8-1.2 每帧减 0.075，低于 0.3 弹泛光，归零 Remove；移除时还原直写效果。[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L387-L398)、[绑定](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L368-L385)、[逐帧](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L344-L366)、[还原](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L329-L342) |

五个注册在 `languages/chinese_s.po` 全文件都没有 `STRINGS.NAMES.*` 键；标题中的“独眼巨鹿”只是宿主官方名，焦痕／烟尾／命中闪光等均为效果描述，不是 FX 的官方译名。[po](../../../raw/current/scripts/languages/chinese_s.po#L342075-L342077)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SG／状态：`SGdeerclops` `"laserbeam"` | onenter 播 `atk2`、`SwitchToEightFaced`、面向目标、停止移动、播 charge；timeline 第 19 帧播 laser 音并 `SpawnLaser`；`animover` 回 idle。 | `haslaserbeam` 仅冬季盛宴巨鹿为真；ChooseAttack 在“目标已冻结，或目标不处于将冻未冻且冷却已过”时进入。[状态](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L933-L948)、[第 19 帧](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L993-L997)、[结束](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1038-L1041)、[条件](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L302-L315)、[宿主开关](../../../raw/current/scripts/prefabs/deerclops.lua#L522-L529) |
| 生成：`SpawnLaser` | 以宿主朝向 +90° 为前向，i=0..10 共 11 段（步长 .75、起点 1.25）；i=0 生成 `laserempty`，其余 `laser`；每段设 `caster=宿主`、落点、`Trigger(delay*FRAMES)`；i=0 额外触发相机震动；循环后在末点再补 2 个 laser（延迟 +1／+2 帧，未设 caster）。 | 主模拟；前方不可通行且 i<=0 时直接返回，否则在受阻段后停止；共享 `targets`／`skiptoss` 防重复。[SpawnLaser](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L177-L224)、[循环与起点](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L188-L210)、[末点补射](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L217-L223) |
| 伤害：`Trigger → DoDamage` | `Trigger` 取消 0 帧自删，延迟为 0 时立即 `DoDamage`，否则排程；`DoDamage` 设 `ignorehitrange`，按 workable／pickable／combat 分支处理：破坏／采摘并抛出掉落物，或用 caster 的 combat（无 caster 时用自身）`DoAttack`；战斗分支命中后生成 `laserhit`，再解冻／减冷／把温度拉到 10。最后抛射附近物品。 | 主模拟；`targets`／`skiptoss` 在整条激光间共享，同一目标不会重复受伤；载体 combat 默认伤害 150。[Trigger](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L149-L158)、[DoDamage](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L39-L147)、[伤害与附带效果](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L101-L125) |
| 伤害归属：caster 与自身 combat | caster 有效时走宿主 combat：宿主 `playerdamagepercent=.5`（对玩家 75）且命中触发宿主 `OnHitOther`（加冷 2、温度 >0 时降温 5、冰裂表现）；最后两段未设 caster，走载体自身 combat（默认 150、无玩家减伤修正、无宿主 OnHitOther）。 | 仅主模拟。[宿主 combat](../../../raw/current/scripts/prefabs/deerclops.lua#L438-L439)、[OnHitOther](../../../raw/current/scripts/prefabs/deerclops.lua#L205-L221)、[CalcDamage](../../../raw/current/scripts/components/combat.lua#L871-L905)、[玩家修正条件](../../../raw/current/scripts/componentutil.lua#L1607-L1609)、[数值](../../../raw/current/scripts/tuning.lua#L1070-L1071) |
| 其他消费者：Willow 火焰投掷 | `willow_throw_flame` 在落点火 10/30 秒后生成同一 `deerclops_laserscorch`；`firethrow` 施法链在 `willow_ember`，相关页见 [[reference/prefabs/willow_shadow_flame|薇洛暗焰]]。 | 主模拟；与巨鹿无关，仅复用焦痕资源与淡出协议。[落点](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L269-L272)、[施法](../../../raw/current/scripts/prefabs/willow_ember.lua#L199-L202)、[入口](../../../raw/current/scripts/prefabs/willow_ember.lua#L412-L415) |
| 资源 | 激光用 `anim/deerclops_laser_hit_sparks_fx.zip`（bank `deerclops_laser_hits_sparks` 与 build 名不完全一致）；焦痕用 `anim/burntground.zip`；烟团用 `anim/lavaarena_staff_smoke_fx.zip`。 | 三个纯 FX 都没有音效；包内帧数与视觉未解包。[assets](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L1-L14) |

## 典型接入

### 命中点焦痕＋烟团

- 前提与选择：主模拟、已有落点；只想补表现时用这两个，不要生成带 combat 的 `deerclops_laser` 载体。烟团不是拖尾。
- 生效与撤销：焦痕 75 tick 自行淡出；烟团按动画长度自删，`FastForward` 只改播放进度与剩余寿命；两者都不造成伤害。
- 原版依据：`DoDamage` 在命中点先生成焦痕，再生成烟团并以 0.3-0.7 随机相位快进（[命中点调用](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L52-L55)）。

```lua
-- 主模拟：在落点 (x, 0, z) 只放表现，不生成带 combat 的激光载体
local x, z = 0, 0 -- 换成实际落点

local scorch = SpawnPrefab("deerclops_laserscorch")
scorch.Transform:SetPosition(x, 0, z)

local trail = SpawnPrefab("deerclops_lasertrail")
trail.Transform:SetPosition(x, 0, z)
trail:FastForward(GetRandomMinMax(.3, .7)) -- 仅主模拟有该方法
```

- 调整与验证：`FastForward` 参数夹在 0-1；焦痕没有缩放参数（原版直接生成、未设 Scale）；未运行验证实际视觉。

## 常见陷阱与限制

- **误区：三个 FX 是伤害来源**：`laserscorch`／`lasertrail`／`laserhit` 都没有 combat／health，激光伤害只在 `DoDamage` 里由载体结算。[载体 combat](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L198-L200)、[纯 FX 工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L251-L398)
- **误区：烟团沿光束拖尾**：`deerclops_lasertrail` 在每个命中点单独生成，是红色烟团；`FastForward` 只把动画随机快进，不改变位置或伤害。[生成位置](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L52-L55)
- **边界：载体必须在 Spawn 同帧 `Trigger`**：工厂用 0 帧任务兜底自删，`Trigger` 只在 `inst.task` 非空时生效；进入 `DoDamage` 后 `task` 置 nil，第二次 `Trigger` 无效，客户端则根本没有该方法。[兜底](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L202-L203)、[Trigger](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L149-L158)
- **边界：`deerclops_laserempty` 不是“无伤害”版本**：它保留 combat／`Trigger`，只缺 AnimState／Light；SG 在起点生成并让它立即结算伤害与相机震动，不能把它当纯空壳。[空变体](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L57-L58)、[起点生成](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L200-L206)
- **边界：末两段没有 caster**：它们走载体自身 combat，默认伤害同为 150，但没有宿主 `playerdamagepercent=.5` 与 `OnHitOther`；共享目标表通常已挡住重复命中，归属差异只在极端情况下显现。[末点补射](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L217-L223)、[自身 combat 分支](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L103-L109)
- **边界：`laserhit` 不能摆位，且依赖目标 AnimState**：无 Transform／Network，只能 `SetTarget`；未绑定会被 0 帧任务删除。有 `colouradder`／`bloomer` 时走组件通道，否则直写 AnimState；移除时 `OnRemoveHit` 解绑并还原直写部分。[工厂](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L387-L398)、[还原](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L329-L342)
- **边界：焦痕 netvar 不逐帧同步**：服务端只在创建时 `set` 初值，之后两端各自 `set_local` 递减；晚加入表现未核验。[net_byte](../../../raw/current/scripts/netvars.lua#L9)、[set_local](../../../raw/current/scripts/netvars.lua#L52-L57)、[焦痕递减](../../../raw/current/scripts/prefabs/deerclops_laser.lua#L268-L272)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。核验五个工厂、`deerclops.lua` 的冬季盛宴开关／`OnHitOther`、`SGdeerclops.lua` 的 `laserbeam`／ChooseAttack／SpawnLaser、Willow 侧焦痕复用，以及 combat／componentutil／tuning／netvars 的伤害与同步依据；`chinese_s.po` 全文件检索无五个注册的 `STRINGS.NAMES.*` 键。全 `scripts` 反查：`laserempty` 生成仅 SG 起点（依赖数组不计）；`laserscorch` 仅 DoDamage 与 Willow；`lasertrail`／`laserhit` 仅 DoDamage。各动画包内容、客户端同步与晚加入未核验。
- 结构检查：`validate_page.py` 通过（standard 档）；`validate_source_links.py --strict` 通过（61 条引用，0 error／0 warning）；Lua 示例通过 `parser_probe.exe --syntax-only`。
- 未覆盖：各动画包的实际帧数与视觉、客户端动画同步细节、Willow 火焰投掷的完整玩法链、巨鹿激光攻击的实际运行表现。

同族冰系技能见 [[reference/prefabs/deerclops_icespike_fx|独眼巨鹿冰刺]] 与 [[reference/prefabs/deerclops_icelance_ping_fx|晶体独眼巨鹿冰圈特效组]]。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
