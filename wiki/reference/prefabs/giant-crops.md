---
title: "巨型作物 heavy 持握（oversized／waxed／rotten 三变体工厂）"
created: "2026-11-07"
updated: "2026-11-09"
type: prefab
size_class: standard
summary: "veggies.lua 的 14 种有种子蔬菜×3 变体共 42 个 BODY heavy 注册：oversized／oversized_waxed／oversized_rotten 的工厂、0.15 减速与不可入容器、常温×4 腐烂三分支、蜂蜡无缝换手、gymweight 与 HAMMER 1 掉落；无种子 6 种不注册。"
aliases: ["oversized", "waxed", "rotten", "veggies 巨型作物", "巨型作物", "超大作物", "打蜡巨型作物", "腐烂巨型作物", "carrot_oversized", "carrot_oversized_waxed", "carrot_oversized_rotten", "巨型胡萝卜", "巨型腐烂胡萝卜", "Trophy Scale 巨型作物"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/veggies.lua", "components/equippable.lua", "components/inventoryitem.lua", "components/heavyobstaclephysics.lua", "components/perishable.lua", "components/waxable.lua", "components/wax.lua", "components/weighable.lua", "components/mightygym.lua", "components/symbolswapdata.lua", "components/workable.lua", "components/lootdropper.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 巨型作物 heavy 持握（oversized／waxed／rotten 三变体工厂）

## 快速判断

- 职责与适用：`veggies.lua` 在 `MakeVeggie` 工厂内为**有种子**蔬菜追加三件 BODY heavy 变体——`<base>_oversized`、`<base>_oversized_waxed`、`<base>_oversized_rotten`。20 种蔬菜中 14 种有种子，故 42 个注册；`SEEDLESS` 的 6 种（浆果、香蕉、仙人掌肉、多汁浆果、无花果、海带叶）不注册。
- 优先使用：BODY heavy 的通用物理与装备链查 [[reference/prefabs/heavy-items]] 与 [[reference/components/heavyobstaclephysics]]；腐烂／打蜡／称重分别查 [[reference/components/perishable]]、[[reference/components/waxable]]、[[reference/components/weighable]]。
- 重要副作用：占 BODY 槽、`walkspeedmult=0.15`、不可入容器、沉水；普通 oversized 常温保鲜期×4；腐烂时按持有者/地面/健身架三分支处理；打蜡时若正被 heavy 持握会**卸下并直接装备蜡制版**，不是原地替换。蜡制版**没有 perishable**，不会腐坏；腐烂版带 `pickable`，锤或采摘均可取出。
- 修改边界：本页只覆盖三件 oversized 变体；普通蔬菜、种子、农田与 `wurt_swamp_terraformer` 地形主体不展开（植株侧见 [[reference/prefabs/farm-plants]]；种子与敲碎掉种见 [[reference/prefabs/farm-crop-seeds]]；普通物品形态见 [[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]）。蜡制版只掉 spoiled_food，腐烂版掉 `loot_oversized_rot`，都不是原作物。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `<base>_oversized` | `fn_oversized`：公共端 heavy＋`MakeHeavyObstaclePhysics` → 主模拟 perishable、inspectable、inventoryitem（不可入容器/沉水）、equippable(BODY)、workable(HAMMER 1)、waxable、submersible、symbolswapdata、weighable、lootdropper | 基础巨型作物；外观按 `plant_def.build` 覆盖 `swap_body`；`gymweight=4`。[工厂](../../../raw/current/scripts/prefabs/veggies.lua#L835-L926)、[装配](../../../raw/current/scripts/prefabs/veggies.lua#L886-L895) |
| `<base>_oversized_waxed` | `fn_oversized_waxed`：同骨架，但 `SetPrefabNameOverride(base.."_oversized")`、打蜡外观、无 `perishable`／`waxable`；创建、`ondropped`、`OnEntityWake` 各安排 **20～40 秒后播一次**打蜡动画，入包/睡眠 `CancelWaxTask`；掉落只有 spoiled_food | 蜡制版：普通蜂蜡 WAX 的 `dowaxfn` 生成；若玩家正 heavy 持握，先卸下再 `Equip(waxedveggie)`，否则原地播打蜡动画。[蜡制工厂](../../../raw/current/scripts/prefabs/veggies.lua#L928-L1006)、[换手](../../../raw/current/scripts/prefabs/veggies.lua#L290-L309) |
| `<base>_oversized_rotten` | `fn_oversized_rotten`：同骨架，腐烂外观／`swap_body_rotten`、`gymweight=3`、掉落 `loot_oversized_rot` | 腐烂版由普通 oversized 的 `oversized_onperish` 生成：非健身架持有者掉 spoiled_food；无持有者/健身架则原地 Spawn 腐烂版，健身架内转 rotten 并 `LoadWeight` 保持配重。[腐烂工厂](../../../raw/current/scripts/prefabs/veggies.lua#L1008-L1082)、[onperish](../../../raw/current/scripts/prefabs/veggies.lua#L237-L264) |

工厂覆盖由 `MakeVeggie` 的 `SEEDLESS` 判定与导出循环决定：只有 `not SEEDLESS[name]` 才把三件 Prefab 插入 `exported_prefabs`，并附带种子资源。[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)、[遍历](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)、[导出](../../../raw/current/scripts/prefabs/veggies.lua#L1086-L1091)

## 机制

### 装备与 heavy 物理

三件变体都显式 `equipslot=EQUIPSLOTS.BODY`、`walkspeedmult=TUNING.HEAVY_SPEED_MULT`（0.15）、`inventoryitem.cangoincontainer=false`、`SetSinks(true)`，并带 `submersible`。装备时 `oversized_onequip` 用 `symbolswapdata` 的 build 覆盖 owner 的 `swap_body`；卸装清符号。`heavyobstaclephysics` 负责 item/obstacle 双态：装备进 inventory 切 item 物理，丢地 0.5 秒后切回障碍态。

[装备槽](../../../raw/current/scripts/prefabs/veggies.lua#L886-L890)、[onequip](../../../raw/current/scripts/prefabs/veggies.lua#L212-L219)、[不可入包/沉水](../../../raw/current/scripts/prefabs/veggies.lua#L882-L884)、[常量](../../../raw/current/scripts/tuning.lua#L417)

### 腐烂三分支

`oversized_onperish` 先取 `inventoryitem:GetGrandOwner()`，并以 owner 是否带 `gym` 标签分流：

1. **有持有者、非健身架**：把 lootdropper 的每项替换成 `spoiled_food`，`DropLoot` 后移除。
2. **无持有者或健身架**：在原地 Spawn `inst.prefab.."_rotten"`；如果是健身架，记录原槽位。
3. **健身架**：Spawn 腐烂版后调用 `gym.components.mightygym:LoadWeight(rot, slot)`，保持配重槽位。

腐烂版自身 `gymweight=3`（普通 4），`loot_oversized_rot` 由工厂配置。[腐烂分流](../../../raw/current/scripts/prefabs/veggies.lua#L237-L264)、[腐烂装配](../../../raw/current/scripts/prefabs/veggies.lua#L1025-L1025)

### 打蜡换手与称重

普通蜂蜡的 `waxable:SetWaxfn(dowaxfn)` 生成 `inst.prefab.."_waxed"`。若 `doer` 正在 heavy lifting 且 BODY 装备就是该作物，`dowaxfn` 先 `Unequip(BODY)` 再 `Equip(waxedveggie)`，实现无缝换手；否则把蜡制版放原地并播 `wax_oversized` 动画。蜡制版**没有 perishable**（不会腐坏），也没有 `waxable`；它的动画是**一次性延迟**而非周期：创建、`ondropped`、`OnEntityWake` 各排一个 20～40 秒的 `DoTaskInTime`，到点只播 `wax_oversized`；`PlayWaxAnimation` 既不清 `_waxtask` 也不续排，要等 `CancelWaxTask`（入包/睡眠）把句柄清空，下一次 `ondropped`/唤醒才会重排一次。锤毁只掉 spoiled_food。

普通 oversized 另有 `weighable`，用 `TROPHYSCALE_TYPES.OVERSIZEDVEGGIES` 初始化重量区间，再按 `oversized_calcweightcoefficient` 取 Lerp 值；三件的 `gymweight` 为普通/蜡制 4、腐烂 3，供 `mightygym` 计重。

[打蜡挂接](../../../raw/current/scripts/prefabs/veggies.lua#L897-L898)、[蜡制装配](../../../raw/current/scripts/prefabs/veggies.lua#L973-L977)、[蜡制状态](../../../raw/current/scripts/prefabs/veggies.lua#L997-L1003)、[称重](../../../raw/current/scripts/prefabs/veggies.lua#L904-L909)、[gymweight](../../../raw/current/scripts/prefabs/veggies.lua#L851-L851)

### 工作与战利品

`workable` 为 HAMMER、`OVERSIZED_MAXWORK=1`；`oversized_onfinishwork` 调 `lootdropper` 掉 `product×2＋seeds×2＋(75% product|seeds)`，腐烂版走 `loot_oversized_rot`。可燃体为 `MakeMediumBurnable`，烧毁走 `oversized_onburnt`（不属于装备链）。[战利品生成](../../../raw/current/scripts/prefabs/veggies.lua#L231-L235)、[workable](../../../raw/current/scripts/prefabs/veggies.lua#L892-L895)

## 常见陷阱与限制

- **14×3 不是 20×3**：`SEEDLESS` 的 6 种蔬菜没有 oversized 系；把“所有蔬菜都有巨型作物”当作事实会多算 18 个注册。[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)
- **蜡制版是“固定形态”，不是“耐腐版”**：它没有 `perishable`（不会腐烂），也没有 `waxable`；锤毁只掉 spoiled_food，别按“可继续种植/收获”写。[蜡制工厂](../../../raw/current/scripts/prefabs/veggies.lua#L928-L1006)
- **腐烂版不是原作物**：`_oversized_rotten` 的 `gymweight=3`、掉落 `loot_oversized_rot`，带 `pickable`（`remove_when_picked`）且用 `inspectable.nameoverride="VEGGIE_OVERSIZED_ROTTEN"`，获取路径与普通/蜡制版不同。[腐烂工厂](../../../raw/current/scripts/prefabs/veggies.lua#L1008-L1082)
- **打蜡不是原地换模型**：正被 heavy 持握时会先卸装再直接装备蜡制版；写 mod 替换逻辑时要按 `dowaxfn` 的分支处理，不能假定只换 AnimState。[换手](../../../raw/current/scripts/prefabs/veggies.lua#L290-L309)
- **锤 1 下是拆巨型作物，不是拆普通作物**：掉落含种子与产品，腐烂版另走 loot_oversized_rot；不要与普通蔬菜收获混用。[workable](../../../raw/current/scripts/prefabs/veggies.lua#L892-L895)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读 `veggies.lua` 的工厂、`SEEDLESS`、`oversized`/`waxed`/`rotten` 三段、导出循环与 `oversized_onperish`/`dowaxfn`/`oversized_makeloots`；核验 14×3=42 注册、0.15 减速、不可入容器/沉水、常温×4、腐烂三分支、蜡制换手、gymweight 4/3、HAMMER 1 掉落；官方名按 `chinese_s.po` 的 `CARROT_OVERSIZED` 等键抽查。
- 游戏运行：未运行；`LoadWeight` 的健身架表现、蜡制动画时长未做运行时验证。
- 结构检查：`validate_page.py`（含 `--check-indexes`）通过；standard 档上限 240 行／32768 字节；`validate_source_links.py --strict` 0 error 0 warning。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
