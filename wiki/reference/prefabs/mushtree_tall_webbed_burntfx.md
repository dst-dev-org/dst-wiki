---
title: "生物残骸与植物／菌类次级表现（mushtree_tall_webbed_burntfx 等 6 注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "牧者残骸、致命亮茄藤蔓节、蛛网蓝蘑菇树焦壳、尖刺灌木侵蚀壳、梦魇城墙地裂与远古守护者大血滴六个外围注册的工厂、生成点、父挂与清理边界。"
aliases: ["mushtree_tall_webbed_burntfx", "lunar_grazer_debris", "lunarthrall_plant_vine", "burnt_marsh_bush_erode", "nightmaregrowth_crack", "minotaur_blood_big", "牧者残骸", "藤蔓节", "菌树焦壳", "焦灌木侵蚀", "梦魇城墙地裂", "远古守护者大血滴", "生物残骸与植物次级表现"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lunar_grazer.lua", "prefabs/lunarthrall_plant.lua", "prefabs/mushtree_webbed.lua", "prefabs/marsh_bush.lua", "prefabs/nightmaregrowth.lua", "prefabs/minotaur.lua", "prefabs/bigshadowtentacle.lua", "stategraphs/SGlunar_grazer.lua", "stategraphs/SGlunarthrall_plant_vine.lua", "stategraphs/SGbigshadowtentacle.lua", "components/burnable.lua", "components/colouradder.lua", "components/freezable.lua", "simutil.lua", "standardcomponents.lua", "entityscript.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 生物残骸与植物／菌类次级表现（mushtree_tall_webbed_burntfx 等 6 注册）

## 快速判断

- 职责与适用：六个注册都是生物／植物／菌类的**外围表现或次级实体**，不是本体：牧者残骸（`lunar_grazer_debris`）、致命亮茄藤蔓节（`lunarthrall_plant_vine`）、蛛网蓝蘑菇树焦壳（`mushtree_tall_webbed_burntfx`）、尖刺灌木焦壳侵蚀（`burnt_marsh_bush_erode`）、梦魇城墙地裂（`nightmaregrowth_crack`）、远古守护者大血滴（`minotaur_blood_big`，兼生成触手）。六个工厂都不是 `fx.lua` 通用表条目，按工厂、端、生成点与清理分组核验。
- 优先使用：查上述六种残骸／焦壳／地裂／血滴的生成点、父挂与收尾读本页；宿主本体、战斗链与掉落读各自页面（多数 Wiki 待建）；通用表 FX 的代理／本地实体协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
- 重要副作用：清理分五类——宿主移除（残骸显示态、地裂）、头部 `removetail` 的 `ErodeAway`（藤蔓节）、`animover`（菌树焦壳、大血滴）、动画时长兜底（菌树焦壳）、0.5 秒壳回收＋本地 `animover`（焦灌木侵蚀）；大血滴在 21 帧时生成 `bigshadowtentacle`，伤害归触手。六个工厂都设 `persists=false`，但它只表示不入存档，不等于自动消失。
- 修改边界：`lunarthrall_plant_vine` 有普通 FX 形态与 `makeweak` 战斗形态两种；`burnt_marsh_bush_erode` 是联网空壳，视觉在非联网本地代理；`nightmaregrowth_crack` 注册不带 assets；`minotaur_blood_big` 的 21 帧回调与动画长度无关。`size_class: complex` 因本页覆盖 6 个文件、6 个注册、6 类宿主与多种清理／端模型，需要整体说明。

## 注册与构造

六个独立工厂，均无参数；`assets` 多数随注册传入（`nightmaregrowth_crack` 不传）。工厂多数在 `SetPristine` 后按 `TheWorld.ismastersim` 分界（`burnt_marsh_bush_erode` 先在非专服端排本地代理），客户端只保留表现实体（含本地代理），主模拟追加字段、计时器与清理。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunar_grazer_debris` | `debrisfn`：Transform＋AnimState＋Network → NOCLICK／DECOR → `MakeInventoryPhysics`（只加物理，不加 inventoryitem）→ bank/build `lunar_grazer`、初始 `rock_01`、`rock_blob` 符号 α0.4／LightOverride 0.1／Bloom → SetPristine → 客户端早退 → 主模拟 `variation=math.random(5)`（1 保留 `rock_01`，2-5 播 `rock_0N`）、`persists=false`；注册带 `assets`（`anim/lunar_grazer.zip`）。 | 5 变体；首次 `ShowDebris` 一次生成 6 枚，之后复用；无自删计时器。[工厂](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L553-L586)、[变体](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L578-L581)、[注册](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L590-L592) |
| `lunarthrall_plant_vine` | `vinefn`：Transform＋AnimState＋SoundEmitter＋Network → bank/build `lunarthrall_plant_vine`、`idle` 循环、FinalOffset 1、SixFaced、缩放 1.2 → fx／NOCLICK／soulless／plantcreature → `SetPrefabNameOverride("lunarthrall_plant_vine_end")` → SetPristine → 客户端早退 → 主模拟 `override_combat_fx_height="low"`、colouradder、`MakeMediumFreezableCharacter`＋抗性 6、`MakeMediumBurnableCharacter`＋燃烧 8×`PLANTMOB_BURNTIME_MULT`、`persists=false`、`makeweak`、SG `SGlunarthrall_plant_vine`、`sg.mem.burn_on_electrocute=true`、`OnRemoveEntity=vine_onremoveentity`；注册带 `vineassets`（普通＋`_big` 两包）。 | 普通形态是藤蔓节表现；头端带 `weakvine` 时 `makeweak` 追加 health／combat／planarentity、切 `_big` bank、摘 fx／NOCLICK、加 hostile／segment／lunar_aligned，伤害重定向到头端。不是 `_END` 头端本体。[工厂](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L583-L631)、[makeweak](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L535-L575)、[注册](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L867-L870) |
| `mushtree_tall_webbed_burntfx` | `burntfxfn`：Transform＋AnimState＋SoundEmitter＋Network → bank/build `mushroom_tree_webbed`、`chop_burnt`、`treeCrumble` → FX → SetPristine → 客户端早退 → 主模拟 `persists=false`、`animover` Remove、`GetCurrentAnimationLength()+FRAMES` 兜底 Remove；注册带宿主同套 `assets`（`anim/mushroom_tree_webbed.zip`＋minimap 声明）。 | 无父挂、无掉落；由宿主烧尽回调在宿主坐标生成。[工厂](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L222-L250)、[兜底](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L244-L247)、[注册](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L339-L340) |
| `burnt_marsh_bush_erode` | `burnt_erode_fn`：Transform＋Network（无 AnimState／SoundEmitter）→ TwoFaced、FX → 非专服 `DoTaskInTime(0, PlayErodeAnim)` → SetPristine → 客户端早退 → 主模拟随机旋转、`persists=false`、0.5 秒 Remove；注册带 `erode_assets`（`anim/ash.zip`）。本地 `PlayErodeAnim`：非联网、`SetCanSleep(false)`、`persists=false`、`SetFromProxy(proxy.GUID)`、bank `ashes`／build `ash`、`disappear`、`SetMultColour(.4,.4,.4,1)`（RGB 压暗、alpha 1）、SetFrame 13、`animover` Remove。 | 联网空壳＋本地代理；专服不建本地动画。[壳](../../../raw/current/scripts/prefabs/marsh_bush.lua#L194-L223)、[本地](../../../raw/current/scripts/prefabs/marsh_bush.lua#L169-L192)、[注册](../../../raw/current/scripts/prefabs/marsh_bush.lua#L225-L227) |
| `nightmaregrowth_crack` | `crackfn`：Transform＋AnimState＋Network → bank/build `nightmaregrowth`、`crack_idle` 非循环、SortOrder 3、OnGround、`LAYER_BACKGROUND` → FX／NOCLICK／DECOR → SetPristine → 客户端早退 → 主模拟 `persists=false`、随机旋转 0-360；注册不带 assets。 | 地面裂隙装饰；无自删；宿主持有 `_crack` 并在移除时清理。[工厂](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L162-L192)、[注册](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L262-L263) |
| `minotaur_blood_big` | `bloodfn`：Transform＋AnimState＋SoundEmitter＋Network → EightFaced、bank/build `rook_rhino_blood_big_fx`、`blood_drop`、α0.5 → FX → SetPristine → 客户端早退 → 主模拟 `SpawnTentacle` 字段、`persists=false`、0 帧 `blood_splurt_large`、21×FRAMES 生成触手、`animover` Remove；注册带 `bloodassets`（`anim/rook_rhino_blood_big_fx.zip`）。 | 混合项：血滴本身无 combat／health；伤害与存在感在生成的 `bigshadowtentacle`。[工厂](../../../raw/current/scripts/prefabs/minotaur.lua#L693-L727)、[注册](../../../raw/current/scripts/prefabs/minotaur.lua#L731-L733) |

六个注册名均未命中 `languages/chinese_s.po` 的 `STRINGS.NAMES` 键（按 `LUNAR_GRAZER_DEBRIS` 等六个精确键检索）；标题与别名中的“残骸／藤蔓节／焦壳／侵蚀／地裂／大血滴”均为已核验效果描述，不是官方译名。`lunarthrall_plant_vine` 的工厂把显示名覆盖到 `_END`，该键命中“钻地藤蔓”，但覆盖只改 name／description，不改 `inst.prefab`，且本实体没有 inspectable，不能据此把本注册当官方命名。[po `_END`](../../../raw/current/scripts/languages/chinese_s.po#L344909-L344912)、[覆盖实现](../../../raw/current/scripts/entityscript.lua#L846-L850)

宿主官方名只用于正文指认，不是本页注册名：牧者（`lunar_grazer`）、蛛网蓝蘑菇树（`mushtree_tall_webbed`）、尖刺灌木（`burnt_marsh_bush`）、梦魇城墙（`nightmaregrowth`）、远古守护者（`minotaur`）、守护者暗影触手（`bigshadowtentacle`）。[牧者](../../../raw/current/scripts/languages/chinese_s.po#L344924-L344927)、[菌树](../../../raw/current/scripts/languages/chinese_s.po#L346054-L346057)、[焦灌木](../../../raw/current/scripts/languages/chinese_s.po#L340009-L340012)、[城墙](../../../raw/current/scripts/languages/chinese_s.po#L346154-L346157)、[守护者](../../../raw/current/scripts/languages/chinese_s.po#L345474-L345477)、[触手](../../../raw/current/scripts/languages/chinese_s.po#L339444-L339447)

## 生成点、挂接与清理

以下生成点均按完整注册名在 `raw/current/scripts` 全域反查；`prefabs`／依赖数组、bank/build 字符串与注释不计生成点。

| 注册名 | 生成点（唯一 SpawnPrefab） | 父挂／跟随 | 清理与端 |
|---|---|---|---|
| `lunar_grazer_debris` | `ShowDebris` 在宿主首次显示残骸时一次生成 6 枚（`NUM_DEBRIS=6`）；全 scripts 仅 `SpawnPrefab` 一处，另有依赖声明与注册。[生成](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L87-L102)、[数量](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L73) | 隐藏时逐枚 `RemoveFromScene`＋`SetParent(inst.entity)`＋本地 `(0,0,0)`；显示时解父＋`ReturnToScene`；位置由 `ScatterDebris`／`TossDebris`／`DropDebris` 用物理传送／速度设置，不是 Follower。隐藏态父挂。[隐藏](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L75-L85)、[散开](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L104-L117)、[抛掷](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L119-L138)、[掉落](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L140-L159) | 宿主 `OnRemoveEntity` 在显示态逐枚 Remove；隐藏态仅 SetParent，删除级联未核验；force despawn 路径先 `ErodeCB(v,1)` 再 1 秒后移除宿主；`persists=false`；存档只记 `data.debris` 布尔，读档回 `dissipated` 重建。[移除](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L161-L168)、[强制消散](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L150-L170)、[存档](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L362-L372) |
| `lunarthrall_plant_vine` | 头端 SG 的 `teleportahead` 在前进时生成；全 scripts 唯一 `SpawnPrefab`；`lunarplanttentacle.lua` 与 `lunarthrall_plant.lua` 其余命中只是 bank/build 字符串。[生成](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L57-L70)、[前进事件](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L158-L166) | 无 entity parent；`nub.headplant` 指回头端、`headplant.tails` 列表持有；`parent.components.colouradder:AttachChild(nub)` 接加色；`freezable:SetRedirectFn(nub_addcoldness)` 把冷量转给头端。[接线](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L62-L77)、[冷量](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L3-L9) | `persists=false`；头端 death／onremove 监听调 `removetail`，按 0.1 秒递增 `ErodeAway`（默认 1 秒）；`teleportback` 直接 Remove 最后一节；`OnRemoveEntity` 从 tails 摘除。无自删计时器。[尾部清理](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L739-L757)、[头端监听](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L841-L853)、[回缩](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L42-L51)、[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597) |
| `mushtree_tall_webbed_burntfx` | 宿主烧尽回调 `tree_burnt` 在宿主坐标生成；读档 `data.burnt` 为真时也走同一回调；全 scripts 唯一 `SpawnPrefab`。[烧尽](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L152-L159)、[回调接线](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L308-L311)、[读档](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L205-L209) | 无父挂；只复制世界坐标。[定位](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L157) | 服务端 `animover` Remove；`GetCurrentAnimationLength()+FRAMES` 兜底 Remove；`persists=false`；客户端在 master 分界处早退。[清理](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L244-L247) |
| `burnt_marsh_bush_erode` | 焦灌木 `OnActivateBurnt` 在宿主位置生成；`activatable.OnActivate` 接线；全 scripts 唯一 `SpawnPrefab`。[激活](../../../raw/current/scripts/prefabs/marsh_bush.lua#L61-L68)、[接线](../../../raw/current/scripts/prefabs/marsh_bush.lua#L162-L164) | 壳不父挂；本地代理 `SetFromProxy(proxy.GUID)` 跟随壳的 Transform。[代理](../../../raw/current/scripts/prefabs/marsh_bush.lua#L181) | 壳 0.5 秒 Remove；本地代理 `animover` Remove；宿主 `ErodeAway(inst)` 默认 1 秒；`persists=false`；专服不建本地代理。[壳清理](../../../raw/current/scripts/prefabs/marsh_bush.lua#L217-L220)、[本地清理](../../../raw/current/scripts/prefabs/marsh_bush.lua#L191)、[专服](../../../raw/current/scripts/prefabs/marsh_bush.lua#L204-L209) |
| `nightmaregrowth_crack` | 宿主 `SpawnCrack` 在初始 0 帧任务与 `grow` 时生成；全 scripts 唯一 `SpawnPrefab`。[生成](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L47-L67)、[初始任务](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L152)、[生长](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L73-L80) | 无 entity parent；宿主 `inst._crack` 字段持有；读档 `_crack_rotation` 恢复旋转。[持有](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L58-L65)、[旋转存档](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L92-L105) | 宿主 `onremove` → `OnRemove` Remove；`persists=false`；无自删计时器；读档后 0 帧重建。[清理](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L86-L90)、[监听](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L154) |
| `minotaur_blood_big` | 远古守护者受击进入二阶段后，0.2 秒周期检查按 1%-5% 概率调用 `SpawnBigBloodDrop`；5 范围内 `bigshadowtentacle` 少于 5 时，在半径 4 的随机角生成；全 scripts 唯一 `SpawnPrefab`。[周期检查](../../../raw/current/scripts/prefabs/minotaur.lua#L166-L178)、[启动](../../../raw/current/scripts/prefabs/minotaur.lua#L180-L197)、[生成](../../../raw/current/scripts/prefabs/minotaur.lua#L306-L327) | 无父挂；调用方设置位置与朝向（`theta/DEGREES-180`）。[定位](../../../raw/current/scripts/prefabs/minotaur.lua#L322-L325) | 自身 `animover` Remove；`persists=false`；21 帧时在自身位置生成 `bigshadowtentacle`；触手独立 30 秒 `testremove`→`leave`→`animover` Remove。[自身清理](../../../raw/current/scripts/prefabs/minotaur.lua#L716-L724)、[触手生成](../../../raw/current/scripts/prefabs/minotaur.lua#L298-L303)、[触手清理](../../../raw/current/scripts/prefabs/bigshadowtentacle.lua#L39-L45)、[leave](../../../raw/current/scripts/stategraphs/SGbigshadowtentacle.lua#L32-L45) |

### 混合项：大血滴的伤害与生成归属

- **血滴本身无伤害**：`bloodfn` 只加 `FX` 标签、动画、声效与计时器，没有 combat／health；唯一玩法产物是 21 帧回调里的 `SpawnTentacle`。[工厂](../../../raw/current/scripts/prefabs/minotaur.lua#L693-L727)
- **伤害在触手**：`bigshadowtentacle` 的 combat 默认伤害为 `TUNING.BIG_TENTACLE_DAMAGE=60`，攻击距离 4、攻击周期 2 秒；触手在 30 秒后尝试 `leave`，播完 `breach_pst` 再 Remove。[伤害](../../../raw/current/scripts/prefabs/bigshadowtentacle.lua#L80-L84)、[数值](../../../raw/current/scripts/tuning.lua#L1181-L1182)、[伤害值](../../../raw/current/scripts/tuning.lua#L1196)
- **生成上限只数已存在触手**：`SpawnBigBloodDrop` 用 `TheSim:FindEntities` 在半径 5 内数 `bigshadowtentacle`，少于 5 才落血滴；血滴要 21 帧（0.7 秒，`FRAMES=1/30`）后才变触手，因此快速连续检查可以在触手出现前排队多枚血滴。[计数](../../../raw/current/scripts/prefabs/minotaur.lua#L309-L317)、[帧长](../../../raw/current/scripts/prefabs/minotaur.lua#L723)、[FRAMES](../../../raw/current/scripts/constants.lua#L16)

## 关联机制导航

| 类型／对象 | 本族如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：colouradder | 藤蔓节作为头端／宿主植物的加色子项；子项移除时自动摘除 | 主模拟；`AttachChild` 监听子项 `onremove` | [[reference/components/colouradder]]；[接线](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L68-L69)、[实现](../../../raw/current/scripts/components/colouradder.lua#L24-L48) |
| 组件：freezable | 藤蔓节挂 freezable，但冷量经 `SetRedirectFn` 转给头端；头端冻结时只把带 health 的 weak 节切 `sync_frozen` | 主模拟；普通节不进入冻结态 | [[reference/components/freezable]]；[重定向](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L3-L9)、[SetRedirectFn](../../../raw/current/scripts/components/freezable.lua#L133-L140)、[同步](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L499-L560) |
| 组件：burnable | 藤蔓节挂 burnable，燃烧 8×`PLANTMOB_BURNTIME_MULT`；工厂没有 `SetOnBurntFn`，烧完不自删 | 主模拟；清理仍由头端 `removetail` 驱动 | [[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L618-L619)、[完成回调](../../../raw/current/scripts/components/burnable.lua#L291-L312) |
| 组件：propagator | `MakeMediumBurnableCharacter` 除 burnable 外还调用 `MakeSmallPropagator`，并把 `acceptsheat=false`；藤蔓节因此带传播组件 | 主模拟；随角色燃烧 helper 安装，不单独由工厂挂载 | [[reference/components/propagator]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L258-L268) |
| 组件：health／combat／planarentity | 仅 weak 形态由 `makeweak` 追加；health 200，伤害经 `health.redirect` 转给头端 | 主模拟；头端带 `weakvine` 时 | [[reference/components/health]]、[[reference/components/combat]]、[[reference/components/planarentity]]；[makeweak](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L535-L575)、[血量](../../../raw/current/scripts/tuning.lua#L7653) |
| SG：SGlunar_grazer | 残骸的显示／散开／抛掷／掉落全部由宿主 SG 的 debris 状态驱动；离开 debris 状态时隐藏 | 主模拟；宿主 SG，客户端不执行这些方法 | Wiki 待建；[消散](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L150-L207)、[生成](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L209-L275)、[抛掷](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L377-L386) |
| SG：SGlunarthrall_plant_vine | 节进入 `nub_idle`；头端移动时由 `onmovetailtask` 切 `nub_forward`／`nub_reverse`；`nub_spawn`／`nub_retract` 是头端状态 | 主模拟；头端与节共用同一 SG | Wiki 待建；[状态驱动](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L12-L15)、[nub_idle](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_vine.lua#L409-L430) |
| SG：SGbigshadowtentacle | 血滴生成的触手用 `arrive`／`idle`／`leave`；30 秒 `testremove` 后离场 | 主模拟；独立实体 | Wiki 待建；[leave](../../../raw/current/scripts/stategraphs/SGbigshadowtentacle.lua#L32-L45) |
| API：ErodeAway／ErodeCB | 藤蔓节与焦灌木宿主的渐变移除；牧者强制消散时对残骸用 `ErodeCB(v,1)` | 全局 helper；依赖 AnimState | [ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597)、[ErodeCB](../../../raw/current/scripts/simutil.lua#L599-L622) |
| API：FRAMES | 血滴 21 帧生成触手；菌树焦壳兜底为动画长度＋1 帧 | 常量 1/30 秒 | [FRAMES](../../../raw/current/scripts/constants.lua#L16) |
| 关联 Prefab：bigshadowtentacle | 血滴 21 帧后生成；伤害与 30 秒寿命都在触手侧 | 主模拟；无父挂 | Wiki 待建；[生成](../../../raw/current/scripts/prefabs/minotaur.lua#L298-L303)、[触手工厂](../../../raw/current/scripts/prefabs/bigshadowtentacle.lua#L47-L100) |
| 关联 Prefab：collapse_small | 梦魇城墙 `SpawnCrack` 在生成地裂前破坏 4 范围内可工作的 structure／tree／boulder；属宿主行为 | 主模拟；与地裂同帧 | Wiki 待建；[破坏](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L50-L56) |
| 资源 | bank/build：`lunar_grazer`、`lunarthrall_plant_vine`（＋`_big`）、`mushroom_tree_webbed`、`ashes`／`ash`、`nightmaregrowth`、`rook_rhino_blood_big_fx` | 声明与使用已核验；包内帧数与视觉未解包 | 各注册行与 assets 行 |

## 典型接入

### 复用焦灌木侵蚀壳

- 前提与选择：主模拟；自定义宿主已有 AnimState 且可移除。`burnt_marsh_bush_erode` 是联网空壳，客户端会自动建非联网 ashes 动画，专服不建；若还要掉灰，照原版先排 0.25-0.3 秒的 `DropAsh` 任务。
- 生效与撤销：壳 0.5 秒自删；本地代理按 `animover` 自删；宿主 `ErodeAway` 默认 1 秒后 Remove。壳不父挂、不随宿主；本地代理只在本端存在。
- 原版依据：焦灌木激活回调在宿主位置生成壳并 `ErodeAway(inst)`（[激活](../../../raw/current/scripts/prefabs/marsh_bush.lua#L61-L68)）；壳的本地代理与 0.5 秒回收见[工厂](../../../raw/current/scripts/prefabs/marsh_bush.lua#L169-L223)。

```lua
-- modmain.lua：给自定义灌木复用焦灌木的侵蚀壳
local function erode_custom_bush(inst)
    local pos = inst:GetPosition()
    GLOBAL.SpawnPrefab("burnt_marsh_bush_erode").Transform:SetPosition(pos:Get())
    inst.persists = false
    GLOBAL.ErodeAway(inst) -- 默认 1 秒；本地壳 0.5 秒后回收
end
```

- 调整与验证：壳没有 AnimState，不能给它加动画；它仍有 Transform／Network，可以 `SetParent`，但不要把它当带动画的普通 FX；`ErodeAway` 依赖宿主 AnimState；示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`persists=false` 会自己消失**：藤蔓节与梦魇城墙地裂都没有自删计时器，分别依赖头端 `removetail` 与宿主 `OnRemove`；单独生成后不管会留到宿主移除或场景卸载。[藤蔓清理](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L739-L757)、[地裂清理](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L86-L90)
- **边界：藤蔓节不是 `_END` 头端**：`SetPrefabNameOverride("lunarthrall_plant_vine_end")` 只改显示名／描述，`inst.prefab` 仍是 `lunarthrall_plant_vine`；普通节没有 health／combat，只有头端带 `weakvine` 时 `makeweak` 才追加，并把伤害转给头端。[覆盖](../../../raw/current/scripts/entityscript.lua#L846-L850)、[makeweak](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L535-L575)
- **边界：焦灌木侵蚀壳没有 AnimState**：视觉全在非联网本地代理，专服不建；不要把壳当可加动画的普通 FX。它仍有 Transform／Network，可以 `SetParent`，但原版不父挂，额外父挂后的表现未运行验证。[壳工厂](../../../raw/current/scripts/prefabs/marsh_bush.lua#L194-L223)
- **误区：大血滴自己造成伤害**：血滴无 combat／health，伤害在 21 帧后生成的 `bigshadowtentacle`（60）；生成上限只数已存在触手，不数排队中的血滴。[血滴](../../../raw/current/scripts/prefabs/minotaur.lua#L693-L727)、[上限](../../../raw/current/scripts/prefabs/minotaur.lua#L309-L317)、[触手伤害](../../../raw/current/scripts/prefabs/bigshadowtentacle.lua#L80-L84)
- **边界：菌树焦壳的双清理**：`animover` 与“动画长度＋1 帧”兜底同时存在，后者在构造时取当前动画长度；复用到更长动画时不能只依赖 `animover`，否则离屏睡眠可能留下实体。[兜底](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L244-L247)
- **边界：地裂注册不带 assets**：bank/build 依赖宿主 `nightmaregrowth` 的 `anim/nightmaregrowth.zip`；自定义加载路径要显式声明该资源。`OnGround`＋`LAYER_BACKGROUND`＋SortOrder 3 也假定它贴地。[注册](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L262-L263)、[宿主资源](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L1-L4)、[层次](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L173-L175)
- **边界：牧者残骸的变体号不是 netvar**：`variation` 只在主模拟赋值，客户端工厂在 `SetPristine` 后早退，不持有该字段；客户端看到的是联网实体同步后的动画，不要在客户端读 `inst.variation` 做逻辑。[赋值](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L578-L581)
- **边界：`MakeInventoryPhysics` 不是 inventoryitem**：残骸只获得物理体（COLLISION.ITEMS），没有 inventoryitem 组件，不能当可拾取物。[物理](../../../raw/current/scripts/standardcomponents.lua#L371-L390)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。逐注册通读六个工厂（`debrisfn`、`vinefn`、`burntfxfn`、`burnt_erode_fn`、`crackfn`、`bloodfn`）及注册行，核验标签、bank/build/anim、SetPristine 与主模拟分界、`persists=false`、生成点、父挂与清理；另核 `SGlunar_grazer` 的 debris 状态、`SGlunarthrall_plant_vine` 的 nub／sync 状态、`SGbigshadowtentacle` 的 leave、`bigshadowtentacle` 的伤害与 30 秒清理、`ErodeAway`／`ErodeCB`、`MakeInventoryPhysics`／`MakeMediumBurnableCharacter`／`MakeMediumFreezableCharacter`、`SetPrefabNameOverride`、`FRAMES` 与相关 TUNING。按六个完整注册名在 `raw/current/scripts` 全域反查；`lunarthrall_plant_vine` 另区分 bank/build 字符串与唯一 SpawnPrefab。`languages/chinese_s.po` 按六个 `STRINGS.NAMES` 精确键检索无命中，宿主名按 `LUNAR_GRAZER`／`MUSHTREE_TALL_WEBBED`／`BURNT_MARSH_BUSH`／`NIGHTMAREGROWTH`／`MINOTAUR`／`BIGSHADOWTENTACLE` 条目核对。
- 分档说明：覆盖 6 个文件、6 个注册、6 类宿主、联网空壳＋本地代理、藤蔓节双形态与血滴生成触手等混合机制，需要整体对照保留，故 `size_class: complex`。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；123 行、27942 字节，complex 档 400 行／32768 字节内）；validate_source_links --strict 通过（111 处行引用、0 error／0 warning）；Lua 示例另存临时文件经 `parser_probe.exe --syntax-only` 通过（只查语法，未执行，临时文件已删）。
- 未覆盖：各动画包的实际帧数与视觉、客户端动画同步细节、专服实际表现、宿主本体战斗链与掉落、`minotaur_blood_big` 21 帧回调与 `animover` 的实际先后、多客户端晚加入表现。

## 相关链接

- [[reference/prefabs/minotaur_blood1|远古守护者通用血滴（minotaur_blood1 等）]]：同文件三条通用表血滴；本页只收独立工厂 `minotaur_blood_big`。
- [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]：本页六个注册均非 `fx.lua` 通用表条目，用于对照工厂协议。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]。
