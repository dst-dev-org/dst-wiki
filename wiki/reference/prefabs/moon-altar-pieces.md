---
title: "天体祭坛部件与吸引人的结构（moon_altar_pieces）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
size_class: standard
summary: "moon_altar_pieces.lua 的 9 个注册：6 件 BODY heavy 天体部件（idol/glass/seed/crown/ward/icon）与 3 种“吸引人的结构”（Inviting Formation；「岩石包裹」为旧检索词）。部件可装备、可在裂隙入槽或作为祭坛分阶段补件、沉水后由 archive_resonator 检索；岩石用 MINE 20 下挖出部件。入槽三件 socket_product 分别为 moon_altar／cosmic／astral，另三件按 stage 当补件材料。"
aliases: ["moon_altar_idol", "moon_altar_glass", "moon_altar_seed", "moon_altar_crown", "moon_altar_ward", "moon_altar_icon", "moon_altar_rock_idol", "moon_altar_rock_glass", "moon_altar_rock_seed", "天体祭坛雕像", "天体祭坛底座", "天体祭坛宝球", "未激活天体贡品", "天体圣殿卫戍", "天体圣殿象征", "祭坛部件", "月坛部件", "月坛岩石", "岩石包裹", "吸引人的结构", "Inviting Formation"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moon_altar_pieces.lua", "prefabs/moon_fissure.lua", "prefabs/moon_altar.lua", "prefabs/archive_resonator.lua", "components/equippable.lua", "components/inventoryitem.lua", "components/heavyobstaclephysics.lua", "components/repairer.lua", "components/submersible.lua", "components/symbolswapdata.lua", "components/workable.lua", "components/lootdropper.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 天体祭坛部件与吸引人的结构（moon_altar_pieces）

## 快速判断

- 职责与适用：`moon_altar_pieces.lua` 导出 6 件 BODY heavy 天体部件与 3 种“吸引人的结构”（官方名 **Inviting Formation**；「岩石包裹」为旧检索词，现有证据不足以确认是玩家俗称）。部件是玩家可搬运/装备的祭坛零件，岩石是世界中的包裹体，挖穿后掉对应部件；两者都监听 `calling_moon_relics`，供 `archive_resonator` 全球检索。
- 优先使用：BODY heavy 通用物理与 0.15 减速查 [[reference/prefabs/heavy-items]]、[[reference/components/heavyobstaclephysics]]；修复语义查 [[reference/components/repairer]]；沉水与检索查 [[reference/components/submersible]]。
- 重要副作用：部件 `equipslot=BODY`、`walkspeedmult=0.15`、`cangoincontainer=false`、`SetSinks(true)`，并带 `irreplaceable`＋`nonpotatable`；只有 glass／crown／icon 三件设置 `socket_product`，分别对应 moon_altar／moon_altar_cosmic／moon_altar_astral。
- 修改边界：`moon_altar_marker` 是未进模块 return 的死代码，本版不存在；祭坛本体、天体贡品激活流程、`archive_resonator` 的完整任务链不在本页展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moon_altar_idol`／`glass`／`seed`／`crown`／`ward`／`icon` | `makepiece(name, socket_product)`：公共端 `MakeSmallHeavyObstaclePhysics(.1)`＋`SetPhysicsRadiusOverride`、heavy／irreplaceable／nonpotatable 标签、bank `moon_altar_pieces` → 主模拟 heavyobstaclephysics(MakeSmallObstacle)、inspectable、inventoryitem（不可入容器/沉水）、equippable(BODY)、repairer、submersible、symbolswapdata、hauntable | 6 件 BODY heavy；onequip 用 `swap_altar_<name>piece` 覆盖 `swap_body`；`socket_product` 仅 glass/crown/icon 有。[工厂](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L7-L97)、[装备](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L69-L74) |
| `moon_altar_rock_idol`／`rock_glass`／`rock_seed` | `makerockpiece(name)`：`MakeObstaclePhysics(1)`＋boulder 标签、bank/build `altar_<name>piece` → 主模拟 lootdropper（对应 `moon_altar_<name>`）、workable(MINE 20)、inspectable、MakeHauntableWork | 3 种“吸引人的结构”（Inviting Formation）；挖穿后 Spawn 对应部件＋`collapse_small`，并按 1/3、2/3 进度播三段台词。[工厂](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L129-L198)、[OnWork](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L99-L127) |

注册名与 return 顺序：`rock_idol`、`idol`、`rock_glass`、`glass`、`rock_seed`、`seed`、`crown`、`ward`、`icon`；`moon_altar_marker` 的 `makemarker` 虽定义，但未列入 return。[导出](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L224-L235)、[死代码](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L202-L222)

## 机制

### 装备、修复与 heavy 物理

六件部件都是 BODY heavy：`MakeSmallHeavyObstaclePhysics` 给质量 0 的小障碍碰撞，`heavyobstaclephysics:SetRadius(.1)` 后再 `MakeSmallObstacle`；装备时切 item 物理，丢地 0.5 秒后切回障碍态。装备覆盖 owner 的 `swap_body`，卸装清除；`symbolswapdata` 记录 `swap_altar_<name>piece` 与 `swap_body`。

`repairer` 对所有部件设置 `repairmaterial=MATERIALS.MOON_ALTAR`、`workrepairvalue=MOON_ALTAR_COMPLETE_WORK/3=1`，但“能放进哪个祭坛”分两条链：

1. **裂隙入槽**只认带 `_socket_product` 的三件——`moon_fissure` 的 `check_piece` 要求该字段非空，成功后 `SpawnPrefab(slotted_item._socket_product)`：glass→`moon_altar`、crown→`moon_altar_cosmic`、icon→`moon_altar_astral`。
2. **后续补件**由祭坛自己的 `repairable.checkmaterialfn` 按 stage 认件：`moon_altar` stage1 收 `moon_altar_seed`、stage2 收 `moon_altar_idol`；`moon_altar_astral` stage1 收 `moon_altar_ward`；成功即 `on_piece_slotted` → `set_stage(+1)`。

所以 idol／seed／ward 不是“没有对应祭坛”，它们不进裂隙、只作补件材料。补件的量也要看清：每件按 `workrepairvalue = MOON_ALTAR_COMPLETE_WORK/3 = 1` 给祭坛 `workable.workleft` **加 1 点**；普通祭坛满值 `MOON_ALTAR_COMPLETE_WORK = 3`（一件占 1/3），星界祭坛满值 `MOON_ALTAR_ASTRAL_COMPLETE_WORK = 2`（一件占 1/2）。所以不是“每件都能修所有祭坛”，也不是“修一次补满”。

[装备槽](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L69-L74)、[heavy 小障碍](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L37-L38)、[修复配置](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L76-L80)、[裂隙入槽](../../../raw/current/scripts/prefabs/moon_fissure.lua#L69-L83)、[祭坛补件 checkmaterialfn](../../../raw/current/scripts/prefabs/moon_altar.lua#L234-L270)、[workleft 加值](../../../raw/current/scripts/components/repairable.lua#L117-L123)、[常量](../../../raw/current/scripts/tuning.lua#L4065-L4070)

### 吸引人的结构（旧检索词“岩石包裹”）与挖取

三种岩石用 `MakeObstaclePhysics(1)`＋`boulder` 标签，`workable` 为 MINE、`MOONALTAR_ROCKS_MINE=20` 下。三个键 `MOON_ALTAR_ROCK_GLASS`／`_IDOL`／`_SEED` 的简中都译「吸引人的结构」（[.po](../../../raw/current/scripts/languages/chinese_s.po#L345789-L345802)）；「岩石包裹」保留作旧检索词，现有证据不足以确认它是玩家俗称。`OnWork` 在 `workleft<=0` 时按 `inst._altar_piece` Spawn `moon_altar_<name>` 与 `collapse_small`，并让玩家说 `ANNOUNCE_MOONALTAR_MINE` 的 `_REVEAL` 台词；未挖穿时按剩余 1/3、2/3 阈值播 `_LOW`／`_MED` 并切 `low`／`med`／`full` 动画。岩石的 lootdropper 也列了对应部件，但挖穿实际由 `OnWork` 的 Spawn 生成；不要把两者当成两条独立掉落。

[岩石工厂](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L129-L198)、[挖取回调](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L99-L127)、[常量](../../../raw/current/scripts/tuning.lua#L4070-L4070)

### 沉水与 `calling_moon_relics`

部件带 `SetSinks(true)`＋`submersible`，但**落海不保证沉水**：`Submersible:Submerge` 会看可达水域与近岸地形，可能换到附近水点、也可能被挪上岸，只有真的沉下去才进 `underwater_salvageable`（并推 `on_submerge`）；同时带 `irreplaceable` 标签，若走到 `SinkEntity` 的上岸/重生分支会按该标签处理。

检索侧分两段：①`archive_resonator` 先 `FindEntity` 一个未打 `marker_found` 的 `moon_altar_astral_marker`；②没有标记时推 `calling_moon_relics`，部件与岩石在事件里 `data.caller:RegisterDevice(inst)`，随后**按登记顺序同一遍遍历**——第一个满足「`INLIMBO`＋有 `submersible` 沉水容器」或「带 `boulder`」的对象胜出，两类之间没有全表优先级；③仍无目标时再推一次事件查 `moon_altar_crown`／`moon_altar_cosmic`，但它们只用来决定“跳过螃蟹之王兜底”，本身不会成为目标。这是“沉水部件可被全球检索”的实际接线，不是部件自己定时上报。

[沉水配置](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L65-L84)、[部件监听](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L89-L91)、[岩石监听](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L190-L192)、[检索发起](../../../raw/current/scripts/prefabs/archive_resonator.lua#L123-L150)

## 常见陷阱与限制

- **`moon_altar_marker` 不存在**：`makemarker` 定义并 `return Prefab("moon_altar_marker")`，但该函数没有进入模块 return；按注册名 Spawn 会失败，不能把它写进可生成清单。[死代码](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L202-L222)
- **六件对应关系分两条链，别混成一条**：`_socket_product` 只决定**裂隙**接受哪件、生成哪个祭坛（glass→moon_altar、crown→cosmic、icon→astral）；seed／idol／ward 是祭坛 `checkmaterialfn` 按 stage 认的**补件材料**（moon_altar 收 seed 再收 idol，astral 收 ward），不是“没有对应祭坛”。[socket](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L226-L235)
- **岩石不是部件**：岩石是 boulder＋MINE 20，挖穿才 Spawn 部件；部件本身是 BODY heavy、可装备、可修复，两者组件与获取路径不同。[岩石工厂](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L129-L198)
- **沉水不等于丢失**：部件带 `submersible` 与 `calling_moon_relics` 监听；但 `archive_resonator` 的检索依赖事件与 `registered_devices`，不能只凭“沉水”就断言任意时刻都能找回。[检索](../../../raw/current/scripts/prefabs/archive_resonator.lua#L123-L150)
- **官方名与文件名不同**：idol／glass／seed 是“天体祭坛雕像／底座／宝球”，crown／ward／icon 是“未激活天体贡品／天体圣殿卫戍／天体圣殿象征”；按文件名硬译会错。[中文名](../../../raw/current/scripts/languages/chinese_s.po#L345764-L345812)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读 `moon_altar_pieces.lua` 的 makepiece／makerockpiece／makemarker／OnWork／return，及 `archive_resonator.lua` 的 `calling_moon_relics` 检索段；核验 9 个注册、6 件 BODY heavy、3 种岩石、socket_product 三件、repairer 1/3 总量、MINE 20 与三段台词、`SetSinks`＋`submersible`＋`irreplaceable`、死代码 marker；官方名按 `chinese_s.po` 的 MOON_ALTAR_* 六键核对。
- 游戏运行：未运行；`archive_resonator` 的完整任务链、沉水部件的实际回收时序、祭坛修复表现未做运行时验证。
- 结构检查：`validate_page.py`（含 `--check-indexes`）通过；standard 档上限 240 行／32768 字节；`validate_source_links.py --strict` 0 error 0 warning。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
