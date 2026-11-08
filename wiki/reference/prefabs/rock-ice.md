---
title: "迷你冰川（rock_ice）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "原版世界里的采矿点 rock_ice：五阶段 dryup／empty／short／medium／tall 随季节与 seasonprogress 逐级升降，ICE_MINE 采掘按级掉冰，火烤约 4 秒降一级，abandon_ship 时按当前级沉没掉冰，贴地水洼子实体与名称在迷你冰川／融化的迷你冰川之间切换。"
aliases: ["rock_ice", "迷你冰川", "冰面巨石", "融化的迷你冰川"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/rock_ice.lua", "prefabs/ice_puddle.lua", "prefabs/ice_splash.lua", "prefabs/boat.lua", "componentutil.lua", "components/workable.lua", "components/lootdropper.lua", "components/timer.lua", "components/lunarhailbuildup.lua", "components/propagator.lua", "standardcomponents.lua", "tuning.lua", "components/seasons.lua", "components/penguinspawner.lua", "components/retrofitforestmap_anr.lua", "prefabs/rock_ice_temperature.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 迷你冰川（rock_ice）

## 快速判断

- 职责与适用：`rock_ice` 是原版世界里固定布点的采矿障碍物，不是物品、不是建筑、也不是生物；它的玩法职责是“按季节自我升降的可再生冰矿”。五阶段为 `dryup`／`empty`／`short`／`medium`／`tall`，阶段既决定名字、外观、碰撞与可采性，也决定采掘掉多少冰。[阶段表](../../../raw/current/scripts/prefabs/rock_ice.lua#L23-L58)
- 优先使用：查阶段、外观与显示名查[五阶段与显示名](#五阶段与显示名)；查四季映射与阈值查[季节映射与阈值](#季节映射与阈值)；查采掘与掉冰查[采掘与冰产量](#采掘与冰产量)；查火烤与沉没查[火烤与沉没](#火烤与沉没)。
- 重要副作用：每次阶段变化都会重设碰撞体（障碍↔无碰撞）、迷你地图图标、雪覆盖、月雹堆积与 `workable` 工作量；`dryup` 时再加 `CLASSIFIED` 标签并关掉采掘，`empty` 时查看状态返回 `MELTED`。[阶段同步段](../../../raw/current/scripts/prefabs/rock_ice.lua#L168-L198)
- 修改边界：本页只写冰川本体。冰物品的食用、灭火、水源与腐坏见 [[reference/prefabs/ice]]；贴地水洼与冰溅 FX 见 [[reference/prefabs/ice_puddle]]；月雹堆积组件见 [[reference/components/lunarhailbuildup]]（S6）。独立文件 `rock_ice_temperature.lua` 的温度变体只在导航里列名。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rock_ice` | 单工厂 `rock_ice_fn`：实体＋Transform／SoundEmitter／AnimState／MiniMapEntity／Network → bank／build `ice_boulder` → `MakeObstaclePhysics(inst, 1)`、图标 `iceboulder.png`、标签 `antlion_sinkhole_blocker`／`frozen`、`MakeSnowCoveredPristine` → 网络量 `_ismelt`／`_stage` → 非专用端生成 `ice_puddle` 子实体 → `SetPristine` → 客户端返回 → 主模拟挂 `lootdropper`／`workable`／`inspectable`／`timer` | 阶段公开量是 `_stage`（0 基）与 `_ismelt`，客户端靠 `stagedirty` 回调重播水洼动画并限条件生成冰溅；`scrapbook_anim` 固定为 `full`。[工厂前半](../../../raw/current/scripts/prefabs/rock_ice.lua#L322-L367)、[主模拟组件](../../../raw/current/scripts/prefabs/rock_ice.lua#L369-L398) |

- 依赖声明只证明加载关系：`prefabs` 表列出 `ice`、`ice_puddle`、`ice_splash`，实际生成点分别见掉冰与阶段同步段。[依赖](../../../raw/current/scripts/prefabs/rock_ice.lua#L7-L12)
- 主模拟在构造末尾还做四件事：按季节定点设初始阶段、随机三档阈值、注册 `firemelt`／`stopfiremelt`／`abandon_ship` 监听、监听世界状态 `cycles`。[构造末尾](../../../raw/current/scripts/prefabs/rock_ice.lua#L384-L410)

### 五阶段与显示名

| 阶段 | 动画 | 露出岩石 | 工作量 | 冰数 `icecount` | 其它 |
|---|---|---|---|---|---|
| `dryup` | `dryup` | 否 | −1（不可采） | 无 | `isdriedup=true`，加 `CLASSIFIED` |
| `empty` | `melted` | 否 | −1（不可采） | 无 | 查看状态返回 `MELTED` |
| `short` | `low` | 是 | `ICE_MINE`＝3 | 2 | — |
| `medium` | `med` | 是 | `ICE_MINE×0.67`＝2.01 | 2 | — |
| `tall` | `full` | 是 | `ICE_MINE×0.67`＝2.01 | 3 | 网络量初值按此级索引写入 |

`ICE_MINE`＝3（采矿工作量单位）。`medium`／`tall` 的 2.01 是浮点数：两次 1 点的工作后剩 0.01，不小于 `workable` 的 0.01 epsilon，所以第三次才归零——在默认每次 1 点工作量下，三档实际都是敲 3 下；单次工作量按工具效率或 worker 的 `workmultiplier` 计算（`actions.lua` 的 `numworks`）；目标侧的 `workable.workmultiplierfn` 是另一层，`rock_ice` 未设置它。[ICE_MINE](../../../raw/current/scripts/tuning.lua#L849)、[工作量结算](../../../raw/current/scripts/components/workable.lua#L130-L155)、[numworks 计算](../../../raw/current/scripts/actions.lua#L1703-L1717)

`showrock` 为真时名字取 `STRINGS.NAMES.ROCK_ICE`＝“迷你冰川”，为假时取 `STRINGS.NAMES.ROCK_ICE_MELTED`＝“融化的迷你冰川”，并同时切换 `no_wet_prefix`（融化态不再加“湿的”前缀）（S2）。[名字与动画](../../../raw/current/scripts/prefabs/rock_ice.lua#L69-L93)、[两个 NAMES 键](../../../raw/current/scripts/languages/chinese_s.po#L348745-L348752)

初始网络值有一处错位：构造时 `_stage:set(STAGE_INDICES["tall"])` 写的是 5，而序列化约定是“索引 − 1”，`DeserializeStage` 会读成 6，`STAGES[6]` 为 `nil`，所以构造里在 `SetPristine` 之前直接调用的那次 `OnStageDirty`（L361，`SetPristine` 在 L363）既不改名也不换水洼动画；真正的首帧阶段由 `DoTaskInTime(0)` 里的 `SetStage` 写入。[反序列化](../../../raw/current/scripts/prefabs/rock_ice.lua#L65-L67)、[初始网络值](../../../raw/current/scripts/prefabs/rock_ice.lua#L348-L363)

阶段变化时被一起改写的还有：碰撞体（`ChangeToObstaclePhysics`／`RemovePhysicsColliders`）、迷你地图开关、雪覆盖符号、`workable` 的可采性与工作量、`dryup` 的 `CLASSIFIED` 标签、月雹堆积量与是否移除。[同步细节](../../../raw/current/scripts/prefabs/rock_ice.lua#L138-L198)

## 季节映射与阈值

`TryStageChange` 读 `TheWorld.state.seasonprogress`（季节开始 0、结束 1），再按季节给出“目标阶段”，由 `SetStage` 落地。[progress 定义](../../../raw/current/scripts/components/seasons.lua#L244)、[世界态分发](../../../raw/current/scripts/components/worldstate.lua#L116-L125)、[四季分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L229-L257)

| 季节 | 目标阶段（按 `pct=seasonprogress` 与三档阈值） | 推进方向 |
|---|---|---|
| 春 | `pct<th1`→tall；`<th2`→medium；`<th3`→short；否则 empty | `"melt"`（只降） |
| 夏 | 恒 `dryup` | `"melt"`（只降） |
| 秋 | `pct<th1`→empty；`<th2`→short；`<th3`→medium；否则 tall | `"grow"`（只升） |
| 冬 | 恒 `tall` | `"grow"`（只升） |

三档阈值在构造时各掷一次：`threshold1`∈[0.4, 0.6)、`threshold2`∈[0.65, 0.85)、`threshold3`∈[0.9, 1.1)，用 `Lerp(.4,.6,math.random())` 这类区间插值生成，`math.random()` 取不到上界。[阈值](../../../raw/current/scripts/prefabs/rock_ice.lua#L397-L400)、[Lerp](../../../raw/current/scripts/mathutil.lua#L14)

`"melt"` 与 `"grow"` 都把一次回调限制成**只走一级**：`"melt"` 要求当前索引大于目标索引，否则直接返回，并在未要求整级快照时把目标改成“当前 − 1”；`"grow"` 对称地只 +1。所以“夏天一到立刻干涸”不成立，阶段是随计时器逐级变化的（推导）。[±1 夹取](../../../raw/current/scripts/prefabs/rock_ice.lua#L109-L132)

计时节奏（S1）：每个新的一天开始时，若没有 `rock_ice_change` 计时器就起一个 `math.random(TOTAL_DAY_TIME, TOTAL_DAY_TIME*3)`，即 480–1440 秒（1–3 个游戏日）后触发一次阶段判定；如果这 10 秒内刚被采过，则改为 30 秒后重排。[重排](../../../raw/current/scripts/prefabs/rock_ice.lua#L212-L216)、[每日启动](../../../raw/current/scripts/prefabs/rock_ice.lua#L259-L263)、[TOTAL_DAY_TIME](../../../raw/current/scripts/tuning.lua#L63)

构造瞬间不掷计时器，而是直接定点：已有 `inst.stage` 就保持不动，否则按季节设 `tall`（春／冬）、`dryup`（夏）或 `empty`（秋）。[初始阶段](../../../raw/current/scripts/prefabs/rock_ice.lua#L384-L395)

由此可得一条边界（源码推导）：`dryup` 只能靠 `"grow"` 上升，而春季只走 `"melt"`，所以干涸的迷你冰川在整个春季都不会恢复，要等秋季的 `"grow"` 才逐级长回来。[四季分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L229-L257)、[只降分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L109-L124)

## 采掘与冰产量

逐级产量、整级快照与沉没掉冰都在本节展开（S4）。`workable` 用 `ACTIONS.MINE`，初始工作量 `ICE_MINE`；每次阶段切换都会按新阶段的 `work` 重设（`short` 3、`medium`／`tall` 2.01、`dryup`／`empty` 直接 `SetWorkable(false)`）。[工作组件](../../../raw/current/scripts/prefabs/rock_ice.lua#L372-L376)、[阶段重设](../../../raw/current/scripts/prefabs/rock_ice.lua#L184-L198)

一次工作把当前级采空时进入 `OnWorked`：`crit = numworks >= 1000`，`snap_to_stage = crit or 不是 character／shadowminion`；再调 `SetStage(inst, "empty", "work", snap)`。不是整级快照时目标被夹到“当前 − 1”，于是只降一级并只结算这一级的冰；整级快照时一次降到 `empty`，把途中经过的每一级都结算一遍。[OnWorked](../../../raw/current/scripts/prefabs/rock_ice.lua#L201-L210)

掉冰循环是 `for i = currentstage, targetstage + 1, -1`，每级内 `for i = 1, math.random(STAGES[i].icecount)`（内层 `i` 只作用于内层，上界表达式取外层级号），因此**每级给 1..icecount 件**，不是固定值：[掉冰循环](../../../raw/current/scripts/prefabs/rock_ice.lua#L184-L192)

| 从某级采到下一级 | 本次掉冰 |
|---|---|
| tall → medium | 1–3 件（`icecount`＝3） |
| medium → short | 1–2 件（2） |
| short → empty | 1–2 件（2） |
| 一整座从 tall 挖到 empty（角色逐级） | 3–7 件（逐级各掷一次，推导） |

声音只在落到 `empty` 时播一次 `iceboulder_smash`，中间级没有换音效。[音效条件](../../../raw/current/scripts/prefabs/rock_ice.lua#L201-L210)

## 火烤与沉没

`firemelt` 到达时启动 4 秒任务，到点调 `SetStage(inst, "dryup", "melt")`；由于 `"melt"` 只降一级，这一次只把当前级降一档，而不是直接干涸。火还在烧时 `propagator` 以 `PROPAGATOR_DT`＝0.5 秒为周期 tick 反复推 `firemelt`；4 秒任务到期后要等下一次 tick 才会开新任务，所以实际约 4.0–4.5 秒降一级。`stopfiremelt` 取消未到点的任务。[4 秒任务](../../../raw/current/scripts/prefabs/rock_ice.lua#L265-L281)、[tick 周期](../../../raw/current/scripts/components/propagator.lua#L3)、[推事件](../../../raw/current/scripts/components/propagator.lua#L213-L216)

按此推导：从 `tall` 一路烤到 `dryup` 要 4 次任务、约 16 秒持续火烤，中途火灭只停在当时那一级。`ice` 物品侧的同一事件影响腐坏速率，见 [[reference/prefabs/ice#灭火与遇火]]。

`abandon_ship` 到达时执行 `Sink`：遍历阶段表，若当前级有 `icecount` 就按**恰好**该数量在冰川位置生成冰（`tall` 3、`medium` 2、`short` 2，`empty`／`dryup` 不掉），随后 `Remove` 自身，没有融化动画。该事件由船体下沉、临时地块被移除等平台消失路径向平台上的实体推送。[Sink](../../../raw/current/scripts/prefabs/rock_ice.lua#L308-L320)、[推给平台实体](../../../raw/current/scripts/prefabs/boat.lua#L463-L471)、[临时地块](../../../raw/current/scripts/componentutil.lua#L222-L237)

另一个移除开关是 `remove_on_dryup`：为真时进入 `isdriedup` 阶段会置 `persists=false`，休眠中立即移除、否则 2 秒后移除；构造本身把它写成 `nil`。原版有两处写入方，都与企鹅冰面有关：企鹅群生成时把新生成在冰面周围的冰川置真，旧存档改造（retrofit）也给企鹅冰 15.1 半径内的既有冰川置真，效果是这些“浮冰上的冰川”干涸后直接消失，而不是留一具空壳。[dryup 移除](../../../raw/current/scripts/prefabs/rock_ice.lua#L153-L166)、[企鹅生成](../../../raw/current/scripts/components/penguinspawner.lua#L306-L308)、[旧档改造](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1445-L1455)

## 存档恢复

`OnSave` 只写 `stage` 与 `remove_on_dryup` 两个字段；`OnLoad` 先恢复 `remove_on_dryup`，再用 `while inst.stage ~= data.stage do SetStage(inst, data.stage) end` 把阶段推到存档值。因为这里不传 `source`，`SetStage` 不做 ±1 夹取，直接跳到目标级。[存读](../../../raw/current/scripts/prefabs/rock_ice.lua#L283-L296)、[无 source 分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L102-L136)

两条边界（源码推导）：三档阈值不写存档，每次读档都在构造时重新随机；若存档阶段正是 `dryup` 且 `remove_on_dryup` 为真，读档推进阶段时会直接进入移除分支。[只存两项](../../../raw/current/scripts/prefabs/rock_ice.lua#L283-L286)、[阈值与开关](../../../raw/current/scripts/prefabs/rock_ice.lua#L397-L402)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：workable | `SetWorkAction(MINE)`、`SetWorkLeft(ICE_MINE)`、`SetOnWorkCallback(OnWorked)`；阶段变化时重设或关闭 | 仅主模拟；`dryup`／`empty` 时不可采 | [[reference/components/workable]]；[接线](../../../raw/current/scripts/prefabs/rock_ice.lua#L372-L376) |
| 组件：lootdropper | 构造时 `SetLoot("tall")` 设概率掉落表；掉冰主要走 `SpawnLootPrefab` | 概率表名按 `size` 选 `rock_ice_short`／`medium`／`tall`，但调用点只有一处且恒为 `"tall"` | [[reference/components/lootdropper]]；[SetLoot](../../../raw/current/scripts/prefabs/rock_ice.lua#L14-L21)、[唯一调用](../../../raw/current/scripts/prefabs/rock_ice.lua#L369-L370) |
| 组件：timer | `rock_ice_change` 计时器驱动 `TryStageChange` | 仅主模拟；每 1–3 日一次 | [[reference/components/timer]]；[timerdone](../../../raw/current/scripts/prefabs/rock_ice.lua#L298-L302) |
| 组件：inspectable | `getstatus` 在 `empty` 时返回 `MELTED` | 仅主模拟 | [[reference/components/inspectable]]；[状态](../../../raw/current/scripts/prefabs/rock_ice.lua#L304-L306) |
| 组件：lunarhailbuildup | 非干涸阶段挂载并按 `short`／`medium`／`tall` 设堆积量，干涸与融化态移除 | helper 在 `SetStage` 内调用 | [[reference/components/lunarhailbuildup]]；[堆积分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L138-L151) |
| 外部写入：`penguinspawner`／旧档改造 | 在企鹅冰面附近生成或发现 `rock_ice` 时写 `remove_on_dryup=true`，使其干涸后自删 | 只影响被写的实例 | [企鹅生成](../../../raw/current/scripts/components/penguinspawner.lua#L306-L308)、[旧档改造](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1445-L1455) |
| 网络：`_ismelt`／`_stage` | `stagedirty` 复制到客户端，客户端重播水洼动画、按 `ismelt` 生成冰溅 | 客户端只挂 `stagedirty` 监听 | [网络量](../../../raw/current/scripts/prefabs/rock_ice.lua#L348-L359)、[水洼与冰溅](../../../raw/current/scripts/prefabs/rock_ice.lua#L80-L91) |
| 关联 Prefab：`ice_puddle` | 非专用端生成并 `SetParent` 到冰川，阶段变化时换动画 | 专用端不生成 | [[reference/prefabs/ice_puddle]]；[生成](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358) |
| 关联 Prefab：`ice_splash` | `ismelt` 且未休眠且非干涸时在冰川位置生成并覆写为阶段动画 | 一次性 FX | [[reference/prefabs/ice_puddle]]；[冰溅](../../../raw/current/scripts/prefabs/rock_ice.lua#L86-L90) |
| 关联 Prefab：`ice` | 采掘与沉没的产物 | 主模拟生成 | [[reference/prefabs/ice]]；[掉冰](../../../raw/current/scripts/prefabs/rock_ice.lua#L184-L198) |
| 同族变体：`rock_ice_temperature` | 同构的温度玩法版本，本页只在导航列名 | 独立文件，Wiki 待建 | [名字覆写](../../../raw/current/scripts/prefabs/rock_ice_temperature.lua#L289-L295) |
| 标签与物理 | `frozen`、`antlion_sinkhole_blocker`；障碍碰撞半径 1，阶段变化时切换 | 仅构造与阶段变化时改 | [物理与标签](../../../raw/current/scripts/prefabs/rock_ice.lua#L336-L342) |
| 资源：`anim/ice_boulder.zip`、`iceboulder.png` | 本体动画与迷你地图图标 | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/rock_ice.lua#L1-L5) |

## 典型接入

### 让整座冰川更早进入下一档（改三档阈值）

- 前提与选择：服务端 `modmain.lua` 的 postinit。阈值在 `rock_ice_fn` 里随机生成，postinit 在其后执行，可直接覆盖三个字段；不要改季节长度或 `seasonprogress`，那会波及所有按进度判定的系统。[原阈值](../../../raw/current/scripts/prefabs/rock_ice.lua#L397-L400)
- 生效与撤销：只影响之后的阶段判定，已有阶段不回溯；停用模组后新生成的冰川恢复随机区间。阈值是 `[0,1]` 的季节进度比例，不是天数。
- 原版依据：春、秋共用同一组阈值：春季按 `pct < th1/2/3` 判 tall／medium／short／empty，秋季反向判 empty／short／medium／tall。阈值调小意味着两季都更早跨过第一档——春更早降档，秋也更早升档；夏恒 `dryup`、冬恒 `tall`，与阈值无关。例：`seasonprogress=0.35`、`th1=0.30` 时秋季目标是 `short`，而原版 `th1≥0.40` 仍以 `empty` 为目标。仍受“一次回调最多升降一级、且要等计时器”的限制。[四季分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L229-L257)、[逐级夹取](../../../raw/current/scripts/prefabs/rock_ice.lua#L102-L132)

```lua
-- modmain.lua：把三档阈值固定成 0.30／0.55／0.80
AddPrefabPostInit("rock_ice", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.threshold1 = 0.30
    inst.threshold2 = 0.55
    inst.threshold3 = 0.80
end)
```

- 调整与验证：验证时看春季第一次 `TryStageChange` 后是否按预期降级；因为阈值不入存档，读档后仍是本例的三个值（postinit 每次构造都会跑），而不是重新随机。

## 常见陷阱与限制

- **边界：阶段不会一次跳好几级。** `"melt"`／`"grow"` 都只允许走一级，只有 `source` 为 `nil`（读档）或 `snap_to_stage`（非角色采掘、单次工作量 ≥1000）时才直接跳到目标级。[夹取](../../../raw/current/scripts/prefabs/rock_ice.lua#L109-L132)
- **边界：春季救不回干涸的冰川。** `dryup` 需要 `"grow"` 才能上升，春季只走 `"melt"`；要等秋季。[季节分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L229-L257)
- **边界：概率掉落表恒为 `rock_ice_tall`。** `SetLoot(inst, size)` 的 `size` 有三档分支，但全文件只有构造处一次调用且实参是 `"tall"`，阶段变化不会重设；该表数据也不在本快照内（未核验表内容）。[三档分支](../../../raw/current/scripts/prefabs/rock_ice.lua#L14-L21)、[调用点](../../../raw/current/scripts/prefabs/rock_ice.lua#L369-L370)
- **边界：初始 `_stage` 与序列化错位。** 构造写 5、反序列化读 6，首帧 `OnStageDirty` 拿到 `nil` 因而不改名／不换水洼动画；读客户端的 `_stage` 不能当成阶段索引。[初始值](../../../raw/current/scripts/prefabs/rock_ice.lua#L348-L361)
- **边界：作祟不推进采掘。** `MakeHauntableWork` 的工作分支整段被注释，只保留空回调，所以幽灵对冰川作祟不会掉冰。[空回调](../../../raw/current/scripts/standardcomponents.lua#L1107-L1122)、[挂载](../../../raw/current/scripts/prefabs/rock_ice.lua#L418)
- **边界：阈值不入存档。** 每次读档都在构造时重新随机，只有 `stage` 与 `remove_on_dryup` 被保留。[存读字段](../../../raw/current/scripts/prefabs/rock_ice.lua#L283-L296)
- **未核验**：单次工作量 ≥1000 的来源工具或效果；四季阶段的游戏内观感与切换节奏（S7）。

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（`raw/current/BUILD.txt` 实读）。`rock_ice.lua` 全文核对阶段表、`SetStage` 的四条来源分支、四季映射与阈值、计时器节奏、掉冰循环与整级快照、4 秒火烤、`Sink`、存读与构造顺序；并核 `propagator` 的 `firemelt` 推送、`boat.lua`／`componentutil.lua` 的 `abandon_ship` 推送、`workable` 的工作量与完成规则、`lootdropper` 的概率表读取、`seasons`／`worldstate` 的 `seasonprogress`、`tuning` 的 `ICE_MINE`／`TOTAL_DAY_TIME`，以及两个 `NAMES` 键。
- 游戏运行：未运行；四季阶段、融化节奏与沉没均为源码审阅与推导（S7）。未覆盖：多人下阶段同步的实际表现、月雹堆积的视觉叠加。
- 检视修订（2026-11-09，同一 build）：①火烤节奏由“每帧”改为 `PROPAGATOR_DT`＝0.5 秒 tick、实际约 4.0–4.5 秒一级（`propagator.lua` L3、L94、L213-L216）；②`OnStageDirty` 调用点在 `SetPristine` 之前（`rock_ice.lua` L361、L363），锚点改为 L348-L363；③“敲 3 下”限定默认每次 1 点工作量；④温度变体写明为独立文件并补进 `sources`；⑤aliases 删去与 S2 冲突的“冰川”；⑥删除对 ice 页的反向互指（采掘细节统一留在本节）；⑦未核验项“实测观感”改为“游戏内观感”。
- 复验修订（2026-11-10）：①摘要“火烤每 4 秒降一级”改“约 4 秒”，与正文 4.0–4.5 秒一致；②单次工作量来源改写为工具效率／worker 的 `workmultiplier`（`actions.lua` L1703-L1717），并注明 `rock_ice` 未设置目标侧 `workmultiplierfn`。
- 外部检视修订（2026-11-10，W05 报告 R01）：示例结论由“春夏更早融、秋冬更晚长”改为“春秋共用阈值，调小后春更早降档、秋也更早升档；夏恒 dryup、冬恒 tall”，并补 `seasonprogress=0.35`／`th1=0.30` 的对照例与“一次最多一级、需等计时器”的条件（`rock_ice.lua` L229-L257、L102-L132）。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 error／0 warning。本页 Lua 示例为完整片段，未在游戏内执行。

## 相关链接

- 产物冰物品：[[reference/prefabs/ice]]
- 贴地水洼与冰溅：[[reference/prefabs/ice_puddle]]
- 月雹堆积：[[reference/components/lunarhailbuildup]]
- 采掘接口：[[reference/components/workable]]
- 掉落接口：[[reference/components/lootdropper]]

返回 [[reference/prefabs/index|Prefab 索引]]。
