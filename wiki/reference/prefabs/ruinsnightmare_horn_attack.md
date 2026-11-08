---
title: "潜伏梦魇穿地犄角突袭（ruinsnightmare_horn_attack）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "潜伏梦魇受击后隐去本体、从目标两侧各生成一只贴地犄角投射物；犄角以宿主 combat 真实结算伤害，命中点转为碰撞碎影并互删，宿主 1 秒后重新现身。"
aliases: ["ruinsnightmare_horn_attack", "穿地犄角", "梦魇犄角突袭", "潜伏梦魇犄角", "双角夹击"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ruinsnightmare_horn_attack.lua", "stategraphs/SGruinsnightmare.lua", "prefabs/nightmarecreature.lua", "components/combat.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 潜伏梦魇穿地犄角突袭（ruinsnightmare_horn_attack）

## 快速判断

- 职责与适用：`prefabs/ruinsnightmare_horn_attack.lua` 只注册 1 个联网实体 `ruinsnightmare_horn_attack`，是潜伏梦魇（`ruinsnightmare`，官方名"潜伏梦魇"）受击反击的**双角投射物**。它没有 health／combat／SG，伤害全部借宿主 `combat:DoAttack` 结算；角本身负责移动、朝向、命中判定与碰撞碎影。[注册](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L206)、[宿主数据](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L301-L315)
- 优先使用：给自定义 Boss 复刻"受击后隐去本体、从两侧穿刺"的表现时，照搬 `SetUp` 的双角协议；单角也能用，只是少了互删与夹击的收尾。[SG 双发](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L114-L121)
- 重要副作用：伤害以**宿主**为攻击者，不是角实体；每只角对同一目标只命中一次（各自 `targets` 表）；角在命中点转为带 `FX` 标签的碎影，1 秒后把宿主叫回；宿主处于裂隙（`planarentity`）时角更快、带击退且伤害含位面部分。[伤害循环](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L78-L122)、[重现](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L58-L68)
- 修改边界：只覆盖这 1 个注册的工厂、双角布置、伤害循环与宿主 SG 的生成／重现链；潜伏梦魇本体 AI、`SGruinsnightmare` 的其它状态、裂隙刷怪与掉落不展开。
- 名称：`STRINGS.NAMES.RUINSNIGHTMARE`＝"潜伏梦魇"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348819-L348822)）；`ruinsnightmare_horn_attack` 在简中 `.po` 无 `STRINGS.NAMES` 键，标题中的"穿地犄角突袭"是已核验效果描述，不是官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ruinsnightmare_horn_attack` | `fn()`：Transform／AnimState／Physics／SoundEmitter／Network → `MakeCharacterPhysics(10, 1.5)`＋`RemovePhysicsColliders`、初速 5.5、八向、bank `shadowcreature3`／build `shadow_insanity3_basic`、`horn_atk_pre` 接 `horn_atk`、半透明＋点采样＋隐藏 `red` 符号 → `SetPristine` → 非主模拟早退；主模拟设初／末速、`targets` 表、导出 `SetUp`／`_OnUpdateFn`、挂 `updatelooper`、`persists=false`。 | 单一注册；工厂阶段**不加 `FX` 标签**，命中后才加；无 health／combat／SG，物理只用于移动，`RemovePhysicsColliders` 后不挡路。[工厂](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L160-L204) |

工厂常量集中在文件头，全部是本地值、不经 TUNING 覆盖（[常量](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L15-L33)）：

| 常量 | 值 | 实际作用 |
|---|---:|---|
| `INITIAL_SPEED`／`FINAL_SPEED` | 5.5／13.5 | 普通态的起速与末速（单位／秒，`SetMotorVelOverride` 尺度） |
| `INITIAL_SPEED_RIFTS`／`FINAL_SPEED_RIFTS` | 7／15 | 宿主有 `planarentity` 时各 +1.5 |
| `FINAL_SPEED_TIME` | .7 | `easing.inCubic` 从初速加到末速的时长（秒） |
| `INITIAL_DIST_FROM_TARGET` | 10 | 双角相对目标的生成距离 |
| `COLLIDE_POINT_DIST_SQ` | 3 | 与目标落点距离平方小于 3（约 1.73）即转碰撞碎影 |
| `AOE_DAMAGE_RADIUS`＋`AOE_DAMAGE_RADIUS_PADDING` | 1.25＋3 | 先按 4.25 搜索，再用 1.25＋目标物理半径做精确命中 |
| `DAMAGE_OFFSET_DIST` | .4 | 伤害判定点沿角朝向前移 0.4，避免贴身漏判 |
| `OWNER_REAPPEAR_TIME` | 1 | 命中后宿主重新现身的延迟（秒） |

## 双角布置与宿主状态机

`SGruinsnightmare` 在 `horn_attack` 状态的 `animover` 里生成一对角（[生成](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L114-L121)、[状态](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L302-L329)）：

```lua
local left = SpawnPrefab("ruinsnightmare_horn_attack")
local right = SpawnPrefab("ruinsnightmare_horn_attack")

left:SetUp(inst, target)
right:SetUp(inst, target, left)
```

- **触发**：宿主收到 `attacked` 事件，且当前不在 `attack`／`hit`／`noattack` 状态、未死亡时，按 `TUNING.RUINSNIGHTMARE_HORNATTACK_CHANCE`（0.75）进入 `horn_attack`，否则走普通 `hit`。[事件](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L10-L16)、[调值](../../../raw/current/scripts/tuning.lua#L8383-L8383)
- **先传送再隐身**：`horn_attack` 播 `disappear`；`animover` 时先 `TryReappearingTeleport` 把宿主随机传送到附近可通行点，再生成双角并 `Hide()`＋加 `invisible` 标签。[状态](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L302-L329)、[传送](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L97-L112)
- **重现**：角命中后对宿主 `DoTaskInTime(1, PushEvent, "reappear")`；宿主的 `reappear` 处理器要求自己处于 `invisible` 且未死，才 `Show()` 并进入 `appear`。[重现](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L24-L29)
- **双角对称**：`SetUp` 以目标为圆心、在 10 距离处按 `45° × math.random(8)` 选一个方向落位并 `FacePoint` 目标；右角把左角的朝向再传回 `SetUp`，因此落在目标另一侧，形成夹击。[布置](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L127-L158)
- **互删**：两角的 `_pair` 互指；任一角到达碰撞点就 `inst._pair:Remove()`，另一角不再继续。[互删](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L58-L68)

`SetUp(inst, owner, target, other)` 的职责（[实现](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L127-L158)）：

| 参数／分支 | 行为 |
|---|---|
| `owner` | 写入 `inst.owner`，后续伤害用它的 `combat`；`planarentity` 分支也读它 |
| `target` | 取目标当前位置作为 `collision_x`／`collision_z`，并在 10 距离外生成、面向目标 |
| `other` | 非空时建立双向 `_pair`；空时用随机八向 |
| 声响 | 生成即播 `dontstarve/sanity/creature3/horn_slice` |
| 裂隙态 | 宿主有 `planarentity` 时 `ShowSymbol("red")`、`SetLightOverride(1)`、alpha 0.65，初／末速改用 7／15 |

## 命中与伤害循环

`OnUpdate` 每帧做两件事：先判断是否到达目标落点，再对前方做一次范围攻击（[实现](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L55-L123)）。

| 阶段 | 条件与动作 | 证据 |
|---|---|---|
| 到达落点 | 与 `collision_x/z` 距离平方 < 3 → 给宿主排 1 秒后的 `reappear` → `TurnIntoCollisionFx` → 删除 `_pair` → 本帧返回 | [L58-L72](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L58-L72) |
| 加速 | `easing.inCubic(存活时间, 初速, 末速−初速, .7)`，封顶末速，写入 `Physics:SetMotorVelOverride` | [L74-L76](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L74-L76) |
| 宿主 combat 缺失 | 直接返回，不做伤害（角继续飞） | [L78-L82](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L78-L82) |
| 忽略攻击距离 | 临时置 `combat.ignorehitrange = true`，让 `DoAttack` 不受宿主近战距离限制；循环结束恢复 `false` | [L84-L84](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L84-L84)、[L122-L122](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L122-L122) |
| 搜索 | 以角当前位置沿朝向前移 0.4 为圆心，半径 4.25，`must`＝`_combat`＋`player`，`cant`＝`INLIMBO`／`notarget`／`noattack`／`flight`／`invisible`／`playerghost` | [L86-L100](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L86-L100)、[L15-L16](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L15-L16) |
| 精确命中 | 目标未在 `targets`、有效、非 limbo、未死，且 `distsq < (1.25 + 目标物理半径)²`、`combat:CanTarget(v)` | [L95-L106](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L95-L106) |
| 裂隙态结算 | 先给骑乘者的坐骑也记 `targets`，再 `DoAttack(v)`，并推 `knockback`（`strengthmult=.6`、`forcelanded=true`） | [L107-L113](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L107-L113) |
| 普通态结算 | 只 `DoAttack(v)`，无击退 | [L111-L117](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L111-L117) |
| 去重 | 命中后 `inst.targets[v] = true`，同角不再打同一目标 | [L117-L117](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L117-L117) |

伤害归属是这段的核心：`DoAttack` 属于**宿主的 combat**，最终调用 `targ.components.combat:GetAttacked_Internal(self.inst, dmg, ...)`，`self.inst` 是潜伏梦魇而不是角。[结算](../../../raw/current/scripts/components/combat.lua#L1218-L1218) 因此角的伤害采用宿主的默认伤害（`TUNING.RUINSNIGHTMARE_DAMAGE`＝50，[调值](../../../raw/current/scripts/tuning.lua#L8379-L8380)）；裂隙态宿主另挂 `planardamage`（25），`DoAttack` 会把位面伤害一并送入结算。[宿主裂隙态](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L141-L172)

两个容易读错的点：

- **`targets` 是每只角各一份**：`targets` 在 `fn` 的主模拟段初始化，两只角互不共享；同一目标可以被左右两角各打一次（理论最多两次 50＋位面部分）。
- **落点是"目标生成时"的位置**：`collision_x/z` 在 `SetUp` 时固定为目标当时的坐标；目标跑开后角仍会飞到旧点转碎影并叫回宿主，伤害循环只按角沿途的当前位置判定。

## 收尾与清理

`TurnIntoCollisionFx` 是角的"第二种形态"（[实现](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L37-L53)）：

- `Physics:Teleport` 到碰撞点、播 `horn_atk_pst`、`SetFinalOffset(1)`、播 `horn_collide`；
- 从 `updatelooper` 移除自己的 `OnUpdate`，电机速度清零；
- **这时才 `AddTag("FX")`**，并监听 `animover` 与 `entitysleep` 自删。

角本身 `persists=false`，不入存档；宿主 SG 的 `horn_attack` 状态没有 `onexit` 清理角的逻辑，角的正常退场由碎影的两条监听或另一只角的互删决定。正常路径下，命中 → 碎影 → 播完 `pst` 自删；宿主随后 1 秒重现。

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `SG：SGruinsnightmare` | 受击反击状态机；生成双角、隐身、重现 | 仅 `ruinsnightmare` 数据项用该 SG；其它梦魇生物用 `SGshadowcreature` | [数据项](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L301-L315)、[注册表](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L301-L330) |
| `组件：combat`（宿主） | `DoAttack` 的真实伤害、`CanTarget`、`ignorehitrange` 临时开关 | 角只在主模拟跑 `OnUpdate`；宿主无 combat 时角不结算 | [宿主挂载](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L249-L251)、[角的调用](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L78-L122) |
| `组件：planarentity`／`planardamage`（宿主） | 裂隙态的速度加成、红色符号、击退分支与 25 位面伤害 | 由 `RuinsNightmare_CheckRift` 在裂隙池变化时增删 | [裂隙检查](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L141-L172) |
| `常量：RUINSNIGHTMARE_HORNATTACK_CHANCE` | 受击时进入角击的概率 | 0.75；与 `RUINSNIGHTMARE_DAMAGE`／`_PLANAR_DAMAGE` 同段 | [tuning](../../../raw/current/scripts/tuning.lua#L8376-L8386) |
| 关联 Prefab：`ruinsnightmare` | 宿主本体；角是它的攻击表现，不是独立生物 | 角由 SG 生成，不自然刷新 | [数据项](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L301-L315) |
| 关联页面：`stalker_shield1` | 同为"FX 实体借宿主／自身结算真实伤害"的外围表现，可对照伤害归属写法 | 骨盾是 FX 直接作 attacker；本页是宿主 combat 作 attacker | [[reference/prefabs/stalker_shield1|复活的骨架骨盾格挡]] |

## 典型接入

### 给自定义 Boss 复刻双角突袭

- 前提与选择：宿主必须是主模拟联网实体并装有 `combat`（伤害来源）；`target` 必须有效。角只在主模拟有 `SetUp`／`OnUpdate`，客户端只有工厂阶段的动画与物理壳。
- 生效与撤销：两角生成即自行飞行、结算、转碎影；命中后 1 秒给宿主推 `reappear`。宿主若要自己处理重现，监听该事件并 `Show()`；不处理就会保持 `Hide()` 的隐身态。
- 原版依据：[双发](../../../raw/current/scripts/stategraphs/SGruinsnightmare.lua#L114-L121)、[SetUp](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L127-L158)。

```lua
local function LaunchHornPair(owner, target)
    local left = SpawnPrefab("ruinsnightmare_horn_attack")
    left:SetUp(owner, target)

    local right = SpawnPrefab("ruinsnightmare_horn_attack")
    right:SetUp(owner, target, left) -- 以左角朝向落在另一侧，两角互删
end
```

- 调整与验证：改伤害要改宿主 `combat` 的默认伤害或 `planardamage`，角本身没有伤害字段；改距离／半径要改文件头常量并复制工厂。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：角在命中前没有 `FX` 标签。** 工厂只加物理与动画，`FX` 是 `TurnIntoCollisionFx` 才加的；按 `FX` 标签筛选实体（例如某些清场或统计逻辑）在飞行阶段看不到它。[标签](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L49-L49)
- **边界：两只角各自去重。** `targets` 不共享，同一目标可能被左右角各命中一次；不要把它当"每次突袭只结算一次"。[初始化](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L194-L194)、[去重](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L117-L117)
- **边界：落点固定，目标移动会"打空但仍收尾"。** 碰撞判定只比较角与 `SetUp` 时记录的目标旧坐标；角会在旧点转碎影并叫回宿主，不会追到新位置。[落点](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L136-L137)、[判定](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L58-L64)
- **边界：宿主重现任务只在命中分支里排。** 如果角在到达落点前被外部 `Remove`（调用方清场等），`reappear` 不会排，宿主会停在 `invisible` 状态，自定义清理要自己补 `PushEvent("reappear")`。`entitysleep` 自删监听是 `TurnIntoCollisionFx` 命中后才安装的，不是飞行阶段的删除路径。[碎影自删](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L51-L52)、[排任务](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L60-L62)
- **边界：`ignorehitrange` 是宿主 combat 的全局临时开关。** 角在每次 `OnUpdate` 里置真、循环末置假；如果宿主同时还有别的攻击逻辑，理论上会在这段窗口内也忽略距离（同步执行，窗口极短）。[开关](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L84-L84)
- **边界：裂隙态才有击退。** 击退分支包在 `owner.components.planarentity ~= nil` 内；普通世界只有伤害，没有 `knockback`。[分支](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L107-L115)
- **误区：它是"纯视觉投射物"。** 伤害真实存在，且以宿主为 attacker；删掉角或让它无法命中会直接改变 Boss 的反击伤害。[伤害](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L78-L122)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/ruinsnightmare_horn_attack.lua`（205 行）；按需读取 `stategraphs/SGruinsnightmare.lua`（事件区 L10-L30、传送 L97-L112、双发 L114-L121、`horn_attack` 状态 L302-L329、`appear` L345-L365）、`prefabs/nightmarecreature.lua`（依赖表 L6-L11、`MakeShadowCreature` 的 combat 挂载 L249-L251、裂隙态 planardamage 挂载 L141-L172、`ruinsnightmare` 数据项 L301-L315、`RuinsNightmare_CheckRift` L141-L172）、`components/combat.lua`（`CanAttack` 的 ignorehitrange L792-L793、`CanHitTarget` L1072-L1073、`DoAttack` 与 `GetAttacked_Internal` L1120-L1245）、`tuning.lua`（`RUINSNIGHTMARE_*` L8375-L8385）。全域反查（`grep -rn "ruinsnightmare_horn_attack" --include=*.lua`）：仅本文件注册 L206、`prefablist.lua:1074` 文件清单、`nightmarecreature.lua:8` 依赖声明、`SGruinsnightmare.lua:117-118` 双发；无其它生成点。台账分诊结论（1 注册、生成 SGruinsnightmare L117-118、伤害经 owner combat）复核属实；补充实核：角初始无 `FX` 标签、`targets` 每角独立、落点固定、`reappear` 只在命中分支排；`entitysleep` 自删只在碎影形态安装，不能当作飞行阶段的清理。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.RUINSNIGHTMARE`＝"潜伏梦魇"（msgid "Lurking Nightmare"，[L348819-L348822](../../../raw/current/scripts/languages/chinese_s.po#L348819-L348822)）；`ruinsnightmare_horn_attack` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——单注册单工厂，但需承载双角布置、宿主 SG 触发／传送／重现、每角去重、裂隙态击退与两种伤害归属；无多状态生命周期，未升 complex。
- 未覆盖：潜伏梦魇本体 AI／闪避与其它攻击状态、`SGruinsnightmare` 的 `appear` 动画细节、`knockback` 在玩家 SG 的最终表现、裂隙刷怪与掉落表；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/stalker_shield1|复活的骨架骨盾格挡]]（同为外围 FX 的真实伤害归属对照）
- [潜伏梦魇宿主工厂](../../../raw/current/scripts/prefabs/nightmarecreature.lua)（本体源码入口；Wiki 暂无专页）
- 返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]（潜伏梦魇）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处，tuning.lua 常量段 +1 行；CRLF 页按原字节格式宿主编辑），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-01）。
