---
title: "重型障碍持握族（glassspike 五注册＋oceantreenut）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
size_class: standard
summary: "六个 BODY heavy 注册的两条生命周期：玻璃尖刺/玻璃城堡由沙刺点燃玻璃化生成，靠 heavyobstaclephysics 在障碍与物品两态间切换，可锤毁但不掉材料；疙瘩树果从大树干撞落，沉入 underwater_salvageable 后按 1.2～2.7 天计时长成疙瘩树，船/树遮挡则每 60 秒重试。"
aliases: ["glassspike", "glassspike_short", "glassspike_med", "glassspike_tall", "glassblock", "oceantreenut", "玻璃尖刺", "玻璃城堡", "疙瘩树果", "Glass Spike", "Glass Castle", "Knobbly Tree Nut", "重型障碍", "重型持握物"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/glass_spike.lua", "prefabs/sand_spike.lua", "prefabs/oceantreenut.lua", "prefabs/watertree_pillar.lua", "prefabs/oceantree.lua", "prefabs/underwater_salvageable.lua", "components/equippable.lua", "components/inventoryitem.lua", "components/heavyobstaclephysics.lua", "components/submersible.lua", "components/symbolswapdata.lua", "standardcomponents.lua", "entityscript.lua", "simutil.lua", "constants.lua", "tuning.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 重型障碍持握族（glassspike 五注册＋oceantreenut）

## 快速判断

- 职责与适用：两个源文件、六个 BODY heavy 注册。`glass_spike.lua` 注册三种固定尺寸玻璃尖刺、一个随机尺寸入口与玻璃城堡；`oceantreenut.lua` 注册疙瘩树果。两者都不是 HANDS 武器/工具，而是"重型障碍物理＋可装备搬运＋沉水容器"的物件；适合查 heavy 的 item/obstacle 双态、玻璃化产物、沉水生长链与官方名。
- 优先使用：装备槽、`walkspeedmult` 与 `symbolswapdata` 查 [[reference/components/equippable]]、[[reference/components/symbolswapdata]]；重型障碍两态查 [[reference/components/heavyobstaclephysics]]；沉水与容器查 [[reference/components/submersible]]、[[reference/components/inventoryitem]]。要写同类"双手抱持、进 BODY、移速惩罚"的物品，先看 [[reference/prefabs/giant-crops]] 与 [[reference/prefabs/moon-altar-pieces]] 的 heavy 装配；完整造物见 [[cookbook/equipment/body-armor]]。
- 重要副作用：两族都占 BODY 槽、`walkspeedmult=TUNING.HEAVY_SPEED_MULT`（0.15）、`cangoincontainer=false`、`SetSinks(true)`；落海先走 `submersible`：可能换到附近可达的水点、也可能被挪上岸（`move_to_land`），只有真的沉下去才进 `underwater_salvageable`。玻璃尖刺/城堡锤 1 下只播破碎与侵蚀删除，**无掉落**；疙瘩树果锤 1 下掉树枝类战利品，**不返果**。
- 修改边界：`glassspike` 是随机尺寸入口，固定尺寸应直接 Spawn `glassspike_short/med/tall`；`glassblock` 官方名是 Glass Castle，不要按文件名写成 Glass Block。`oceantreenut` 不是砍疙瘩树掉落，本版可核的入口是大树干（`watertree_pillar`）被船撞落；`oceantree.lua` 的 `drop_oceantreenut_chance` 只赋值、未见消费。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `glassspike_short`／`_med`／`_tall` | `MakeSpikeFn("spike", size)`：公共端 bank/build `sand_spike`、动画 `<size>_glass_idle`、`MakeHeavyObstaclePhysics(radius)` → 主模拟 inspectable、inventoryitem（不可入容器、沉水）、equippable(BODY)、workable(HAMMER 1)、submersible、symbolswapdata、hauntable | 固定半径 0.12／0.25／0.3；onequip 将 `swap_body` 换成 `swap_glass_spike` 的 `swap_body_<size>`；无战利品。[工厂](../../../raw/current/scripts/prefabs/glass_spike.lua#L80-L165)、[注册](../../../raw/current/scripts/prefabs/glass_spike.lua#L168-L180) |
| `glassspike` | 同一工厂但 `size=nil`：随机抽 `SPIKE_SIZES`，`SetPrefabName("glassspike_"..size)`，`SetPrefabNameOverride("glassspike")`；依赖三固定变体＋`underwater_salvageable`＋`splash_green` | 随机尺寸入口；生成后 prefab 名变为 `glassspike_<size>`，仅显示名/描述仍指 `glassspike`。[随机段](../../../raw/current/scripts/prefabs/glass_spike.lua#L90-L96)、[注册](../../../raw/current/scripts/prefabs/glass_spike.lua#L177) |
| `glassblock` | `MakeSpikeFn("block")`：bank/build `sand_block`、动画 `block_glass_idle`、半径 0.45；依赖 `underwater_salvageable` | 官方名 `Glass Castle`（不是 Glass Block）；onequip 将 `swap_body` 换成 `swap_glass_block` 的 `swap_body`。[工厂](../../../raw/current/scripts/prefabs/glass_spike.lua#L97-L99)、[注册](../../../raw/current/scripts/prefabs/glass_spike.lua#L180) |
| `oceantreenut` | fn：公共端 heavy＋`MakeHeavyObstaclePhysics(0.75)`＋`SetPhysicsRadiusOverride` → 主模拟 inspectable、lootdropper、heavyobstaclephysics、inventoryitem、equippable(BODY)、workable(HAMMER 1)、timer、submersible、symbolswapdata、hauntable | 官方名 `Knobbly Tree Nut`；`OnSubmerge` 起生长计时、`OnTimerDone` 受阻重试；锤毁掉树枝并自删。[工厂](../../../raw/current/scripts/prefabs/oceantreenut.lua#L162-L231)、[装备](../../../raw/current/scripts/prefabs/oceantreenut.lua#L199-L206) |

## 机制与生命周期

### 重型障碍的 item/obstacle 双态

`MakeHeavyObstaclePhysics` 给实体加 `blocker` 标签与质量 0 的障碍碰撞；`heavyobstaclephysics:SetRadius` 记录最大半径并监听 `onputininventory`／`ondropped`。装备时（物品进入 owner 的 inventory）`ChangeToItem` 切到质量 1、`ITEMS` 组、与 `WORLD`／`OBSTACLES`／`SMALLOBSTACLES` 碰撞；**丢地**（`ondropped`，单独 `Unequip` 不算落地）后 `ChangeToObstacle` 延迟 0.5 秒切回质量 0——大件进 `OBSTACLES` 组，`MakeSmallObstacle` 分支（月坛部件）进 `SMALLOBSTACLES`——并先立刻按最近角色算一次半径、之后每 0.5 秒复查，半径到 `maxradius` 就取消任务。有 falling 接线的物件还会在 `stopfalling` 直接回障碍态、不等这 0.5 秒。

[标准件](../../../raw/current/scripts/standardcomponents.lua#L698-L715)、[切物品态](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L35-L58)、[切障碍态](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L113-L125)、[半径收缩](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L62-L89)

两族都显式覆写 `EQUIPSLOTS.BODY`，并设 `walkspeedmult=TUNING.HEAVY_SPEED_MULT`。速度是**两层**修正：装备层 `GetWalkSpeedMult` 在持有 `vigorbuff` 时把小于 1 的倍率加 0.25（上限 1），0.15 在这一层变 0.40；locomotor 层再判强壮态（服务端 `mightiness:GetState()=="mighty"`、客户端标签 `mightiness_mighty`），把任何小于 1 的装备倍率当成 1——真正“豁免重物减速”的是 locomotor 的强壮分支，`vigorbuff` 只是把基础倍率往上抬。猪皮包页按同样两层写，可对照。装备时手动 `OverrideSymbol`，同时挂 `symbolswapdata` 供换装/皮肤链读取（[SetData](../../../raw/current/scripts/components/symbolswapdata.lua#L5-L9)）。

[装备槽](../../../raw/current/scripts/prefabs/glass_spike.lua#L128-L138)、[常量](../../../raw/current/scripts/tuning.lua#L417)、[减速修正](../../../raw/current/scripts/components/equippable.lua#L126-L144)、[BODY 枚举](../../../raw/current/scripts/constants.lua#L654-L660)

两族都 `inventoryitem.cangoincontainer=false`、`SetSinks(true)`（[SetSinks](../../../raw/current/scripts/components/inventoryitem.lua#L310-L311)），并带 `submersible`。在海上丢下且无 owner 时，`Submersible:OnLanded` 先接管并 `Submerge`；`Submerge` 会看可达水域与近岸地形——可能把水点挪到附近放得下平台的位置，也可能走 `move_to_land` 直接上岸，**落海不保证沉水**。真的沉下去时 `MakeSunken` 生成 `underwater_salvageable`（单格容器、`winchtarget`）并 `GiveItem` 收进去（返回值没有检查），随后才推 `on_submerge`。玻璃尖刺/城堡没有监听该事件，只是沉在水下待绞盘取回；疙瘩树果则用它启动生长计时。

[沉水入口](../../../raw/current/scripts/components/submersible.lua#L73-L82)、[沉水容器](../../../raw/current/scripts/components/submersible.lua#L181-L194)、[容器结构](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L49-L75)、[不可入包/沉水](../../../raw/current/scripts/prefabs/glass_spike.lua#L124-L126)

### 玻璃尖刺／玻璃城堡：点燃玻璃化与锤毁

沙刺/沙块被点燃时，burnable 的 `OnIgnite` 接管：加 `NOCLICK`、无敌、清战斗与燃烧回调，播放 `<animname>_transform`，监听 `animover` 后 `ChangeToGlass`。`ChangeToGlass` 删掉沙刺，在同点 Spawn `glassspike_<size>` 或 `glassblock`，并调用 `Sparkle()`。玻璃化 FX 由同一文件的 `PlayGlassFX` 在 `GLASS_TIME-2` 帧单独落点，和换生动画不是同一条时间线，不能从帧数推出固定先后。

[点燃接管](../../../raw/current/scripts/prefabs/sand_spike.lua#L175-L188)、[换生](../../../raw/current/scripts/prefabs/sand_spike.lua#L162-L168)、[GLASS_TIME](../../../raw/current/scripts/prefabs/sand_spike.lua#L29-L29)

`Sparkle` 每 4～9 秒随机推一段 `<size>_glass_sparkle1..3` 动画；实体睡眠时取消，唤醒时若任务为空则重排，`workleft<=0` 后不再排。锤击回调只播 `<size>_glass_hit`；锤满 1 下后 `onworkfinished` 加 `NOCLICK`、关物理、清睡眠/唤醒钩子，播放 `<size>_glass_break`，再在 `animover` 交给 `ErodeAway`。`ErodeAway` 默认 1 秒侵蚀后 `Remove`，全程没有 `lootdropper`，所以**锤掉玻璃尖刺/城堡不会有任何材料返还**。

[闪烁](../../../raw/current/scripts/prefabs/glass_spike.lua#L55-L78)、[锤击与锤毁](../../../raw/current/scripts/prefabs/glass_spike.lua#L41-L53)、[侵蚀](../../../raw/current/scripts/simutil.lua#L575-L596)

### 疙瘩树果：撞落、沉水计时与长成

获取入口是大树干（`watertree_pillar`，官方名 Great Tree Trunk）被船撞击：`OnCollide` 要求 `hit_velocity>0.8`，且距上次掉落满 1.5 天；`generate_items_to_drop` 在 `num_oceantreenuts>0` 时加入一个果，再配 10～14 个小物品。`DropItems` 在距树干 8～20、高度 10 生成；heavy 果至多检查 5 个候选点（每次重掷距离与角度，`ATTEMPT_DROP_OCEANTREENUT_RADIUS=6` 内查 `tree`），**找不到无树点也接受最后一次随机位置**，再由 `make_heavy_object_falling` 抛落。初始 1 个、上限 2 个；掉落后按 480＋rand×240 秒补回，满 2 个即停。

[撞落](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L238-L264)、[抛落](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L101-L123)、[掉落](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L126-L172)、[掉落清单](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L174-L185)、[补回](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L266-L277)、[常量](../../../raw/current/scripts/tuning.lua#L6339-L6341)

果只有在**真的沉水**、收到 `on_submerge` 之后才起表：`Submerge` 被挤时只有走 `move_to_land`（挪上岸）才跳过沉水、不发事件；换到附近可用水点后仍会 `MakeSunken` 发 `on_submerge`。`OnSubmerge` 只在**没有** `grow` 计时时新建（抽 576＋rand×720 秒＝9.6～21.6 分钟），已有计时则 `ResumeTimer` 沿用剩余时间，若 `should_attempt_grow` 已置位则同时开 60 秒周期重试；该重试任务在实体睡眠时取消、唤醒时按标记恢复。`OnTimerDone` 到点先试 `CanGrow`：先从 `submersible:GetUnderwaterObject()` 取沉水容器，再要求点位是海、2 格内无 `tree` 标签，且以容器为圆心 0.85 格 8 个方向都没有 `GetPlatformAtPoint`（船/平台）；不满足就置 `should_attempt_grow=true` 并等下一次 60 秒重试。

[沉水回调](../../../raw/current/scripts/prefabs/oceantreenut.lua#L90-L100)、[生长判定](../../../raw/current/scripts/prefabs/oceantreenut.lua#L61-L72)、[长成](../../../raw/current/scripts/prefabs/oceantreenut.lua#L74-L82)、[重试](../../../raw/current/scripts/prefabs/oceantreenut.lua#L113-L123)、[计时常量](../../../raw/current/scripts/tuning.lua#L6361-L6362)

`Grow` 在沉水容器位置 Spawn `oceantree_short`，调 `sproutfn()`，再删果；容器监听 `itemlose`，下一帧自删。读档时 `OnSave` 只写 `should_attempt_grow`，`OnLoadPostPass` 在无沉水容器时把 y 归零，有容器且待重试时恢复周期任务；`on_salvaged` 事件到达时停表并取消任务（[OnSalvaged](../../../raw/current/scripts/prefabs/oceantreenut.lua#L102-L111)）。锤毁走 `OnWorkedFinished`：先 `DropLoot`（twigs×1、twigs 50%、spoiled_fish_small 1%；走 `SetChanceLootTable('oceantreenut')`，两条非必掉由 `GetChance` 按有效击杀者/最后工作者的幸运修正），再在同点 Spawn `collapse_small`（material stone）并 `Remove`，不回果。

[长成树](../../../raw/current/scripts/prefabs/oceantreenut.lua#L74-L82)、[容器回收](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L64-L73)、[读档补挂](../../../raw/current/scripts/prefabs/oceantreenut.lua#L138-L160)、[锤毁与掉落](../../../raw/current/scripts/prefabs/oceantreenut.lua#L29-L37)、[掉落表](../../../raw/current/scripts/prefabs/oceantreenut.lua#L13-L17)

## 典型接入

### 做同类"BODY heavy＋可沉水"物件

- 物理与两态：`MakeHeavyObstaclePhysics(inst, radius)`＋`heavyobstaclephysics:SetRadius(radius)`；后者会在 `onputininventory`／`ondropped` 自动切 item/obstacle，不要再手写质量与碰撞组。
- 装备与容器：`inventoryitem.cangoincontainer=false`、`SetSinks(true)`；`equippable.equipslot=EQUIPSLOTS.BODY`、`walkspeedmult=TUNING.HEAVY_SPEED_MULT`；需要换装外观时同时配 `symbolswapdata` 与 `onequip`／`onunequip`。
- 沉水：`submersible` 负责落海分流（换水点／上岸／沉水），只有沉水那支才进 `underwater_salvageable`；若物体会在沉水后变化，监听 `on_submerge`（**事件到达才起表**，不是“落海即起表”），并在 `on_salvaged` 停表/清任务。
- 工作与清理：`workable` 的 `OnFinishCallback` 自己决定掉落与删除；重型障碍的 `ErodeAway` 只做侵蚀删除，不会自动补 loot。

参照页面：[[reference/prefabs/giant-crops]]、[[reference/prefabs/moon-altar-pieces]]、[[cookbook/equipment/body-armor]]。

## 常见陷阱与限制

- **官方名不是文件名**：`glassblock` 是 **Glass Castle**（玻璃城堡），`glassspike` 是 **Glass Spike**（玻璃尖刺），`oceantreenut` 是 **Knobbly Tree Nut**（疙瘩树果）。[英文名](../../../raw/current/scripts/strings.lua#L2228-L2229)、[果名](../../../raw/current/scripts/strings.lua#L3870-L3871)、[玻璃中文名](../../../raw/current/scripts/languages/chinese_s.po#L343290-L343307)、[果中文名](../../../raw/current/scripts/languages/chinese_s.po#L346570-L346572)
- **`glassspike` 是随机尺寸入口**：固定尺寸应直接 Spawn `glassspike_short/med/tall`；生成随机入口后 prefab 名会被改成 `glassspike_<size>`，`SetPrefabNameOverride` 只改显示名，不改实际 prefab。[随机段](../../../raw/current/scripts/prefabs/glass_spike.lua#L90-L96)、[override 语义](../../../raw/current/scripts/entityscript.lua#L840-L850)
- **锤玻璃尖刺没有返还**：`glassspike_*` 与 `glassblock` 的 `onworkfinished` 只播破碎、关物理、1 秒侵蚀删除；不要按"玻璃制品"预期掉玻璃/石头。[锤毁](../../../raw/current/scripts/prefabs/glass_spike.lua#L45-L53)
- **锤疙瘩树果不返果**：`OnWorkedFinished` 掉的是树枝（twigs×1、twigs 50%、spoiled_fish_small 1%）＋collapse_small，然后删果；这两条百分比是**基础概率**，`SetChanceLootTable` 走 `GetChance` 会按幸运值修正。想留种就不要锤。[掉落](../../../raw/current/scripts/prefabs/oceantreenut.lua#L29-L37)
- **两件都占 BODY，不是 HANDS**：`equipslot=BODY`、减速 0.15、不可入容器；`vigorbuff` 可 +0.25 抵消一部分，但仍是重物。[装备](../../../raw/current/scripts/prefabs/glass_spike.lua#L128-L138)、[减速修正](../../../raw/current/scripts/components/equippable.lua#L126-L144)
- **落海不等于沉水，也不是删除**：`Submerge` 可能把物件换到附近水点或直接挪上岸，只有真沉水才进 `underwater_salvageable` 并推 `on_submerge`；玻璃尖刺沉下去只是待取回，疙瘩树果则从收到事件起才计生长。`SetSinks(true)` 只影响物品落水表现，别读成“落水销毁”。[沉水](../../../raw/current/scripts/components/submersible.lua#L181-L194)
- **果不是砍树掉的**：本版 `oceantree.lua` 的 `drop_oceantreenut_chance` 只在短/中/高三档赋值，未见消费；可核入口是 `watertree_pillar` 撞船。`oceantree_short` 只是果长成的目标 prefab。[树侧赋值](../../../raw/current/scripts/prefabs/oceantree.lua#L51-L53)、[撞落](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L238-L264)
- **沉水生长会被船/树挡住**：`CanGrow` 检查 2 格内 `tree` 与 8 方向 0.85 格的平台；不满足就置 `should_attempt_grow` 等 60 秒周期重试（实体睡眠时任务取消、唤醒恢复），计时本身是 9.6～21.6 分钟，不是到点必成。[判定](../../../raw/current/scripts/prefabs/oceantreenut.lua#L61-L72)
- **边界：`CanGrow` 没有 nil 容器分支**：`GetUnderwaterObject()` 取不到容器时，源码下一行仍访问 `container.inst`；取回后 `on_salvaged` 未到达时，计时到点是否走到这里未运行验证。[取容器](../../../raw/current/scripts/prefabs/oceantreenut.lua#L61-L64)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读 `glass_spike.lua`、`sand_spike.lua`、`oceantreenut.lua`、`watertree_pillar.lua`、`oceantree.lua`、`underwater_salvageable.lua` 及 heavy/equippable/inventoryitem/submersible/standardcomponents 相关段；逐项核对 6 个注册名、固定/随机尺寸分派、item/obstacle 双态、玻璃化与锤毁、撞船掉落、沉水容器、生长计时与受阻重试、掉落表、读档补挂；官方名与中文名按 `strings.lua`／`chinese_s.po` 的 `NAMES` 键核对；数值 `HEAVY_SPEED_MULT=0.15`、`OCEANTREENUT_GROW_TIME=1.2×480`、`VARIANCE=1.5×480`、`REGENERATE=480+rand×240`、`ATTEMPT_GROW_FREQUENCY=60` 均为源码推导。
- 游戏运行：未运行；`glassspike` 玻璃化动画时长、`collapse_small` 表现、沉水容器绞盘交互未做运行时验证；文中"9.6～21.6 分钟"按 480 秒总日长换算。
- 结构检查：`validate_page.py`（含 `--check-indexes`）通过；standard 档上限 240 行／32768 字节；`validate_source_links.py --strict` 0 error 0 warning。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
