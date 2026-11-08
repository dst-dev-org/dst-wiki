---
title: "海象牙（walrus_tusk）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品海象牙：宿主掉落只有成年海象 0.50（无幸运基准；小海象与海象营地都不掉），拆解法杖可从步行手杖、刷子回收 1 牙，10 叠大件与三条配方。"
aliases: ["walrus_tusk", "海象牙", "步行手杖材料", "刷子材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/walrus_tusk.lua", "prefabs/walrus.lua", "prefabs/walrus_camp.lua", "components/stackable.lua", "components/lootdropper.lua", "components/spellcaster.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 海象牙（walrus_tusk）

## 快速判断

- 职责与适用：海象牙是**唯一一个注册**的大件材料（简中实显 **海象牙**，10 叠），本体只有浮空、拾取与作祟抛飞；没有标签、没有燃烧、燃料、食用、鲜度或交易组件。[构造](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L16-L35)
- 优先使用：查它从哪只怪掉看[掉落来源](#掉落来源)；查三条配方看[用途与消耗](#用途与消耗)；查堆叠与共享端差异看[注册与构造](#注册与构造)。
- 重要副作用：**宿主掉落**只有成年海象 `walrus` 的 0.50（无幸运修正的基准）；小海象 `little_walrus` 只掉肉，海象营地 `walrus_camp` 根本没有 `lootdropper`。牙还能**回收**：拆解法杖拆步行手杖或刷子各返 1 牙。[海象表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)、[小海象表](../../../raw/current/scripts/prefabs/walrus.lua#L31-L34)、[营地构造](../../../raw/current/scripts/prefabs/walrus_camp.lua#L448-L484)
- 修改边界：海象的 AI、冰猎犬护卫与营地刷新不展开（S5）；步行手杖、刷子、怀表本体归各自页面。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `walrus_tusk` | `fn()`：实体 → Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank／build `walrus_tusk`、播 `idle` → 浮空 `("small", 0.08)` → `SetPristine()` → 主模拟挂 `inspectable`／`stackable`（10）／`inventoryitem` → `MakeHauntableLaunch` | 唯一注册，无变体、无标签、无皮肤；`assets` 只有 `anim/walrus_tusk.zip`。[工厂](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L6-L38) |

- **堆叠 10 是显式的大件档**：`stackable.maxsize = TUNING.STACK_SIZE_LARGEITEM`＝10，不是小件默认的 40（S7）。[赋值](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **没有任何显式标签**：全文件只有 `AddComponent`，没有 `AddTag`；因此不参与"有标签才算"的过滤（如容器 `itemtestfn`、食谱标签判定）。[构造](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L22-L35)
- **不可燃、不可当燃料**：既没有 `MakeSmallBurnable` 也没有 `fuel`，与触手皮、针刺同批但不是同类材料。
- **作祟只抛飞**：`MakeHauntableLaunch` 没有碎裂或转化分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

### 名称核验

`NAMES.WALRUS_TUSK`＝**海象牙**，是简中实显名（S1）；没有改名调用。[po 名称键](../../../raw/current/scripts/languages/chinese_s.po#L351714-L351717)

## 掉落来源

| 宿主 | 条目 | 同表其它产出 | 证据 |
|---|---|---|---|
| 海象 `walrus`（MacTusk） | 1 条 0.50（无幸运修正的基准） | 肉 1.00、吹箭 1.00、海象帽 0.25 | [表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)、[接线](../../../raw/current/scripts/prefabs/walrus.lua#L181)、[注册](../../../raw/current/scripts/prefabs/walrus.lua#L240) |
| 小海象 `little_walrus` | **无** | 只有肉 1.0（`walrus_wee_loot`） | [表](../../../raw/current/scripts/prefabs/walrus.lua#L31-L34)、[接线](../../../raw/current/scripts/prefabs/walrus.lua#L224) |
| 海象营地 `walrus_camp` | **无** | 只是刷怪宿主，没有 `lootdropper` | [构造](../../../raw/current/scripts/prefabs/walrus_camp.lua#L448-L484) |

- **0.50 是单条独立判定，也是无幸运基准**：共享表每条各掷一次，1.00 走必中分支、小于 1 的先过 `GetChance`（击杀者幸运按 `LuckFormulas.LootDropperChance` 改算）；海象死亡时肉与吹箭必得、海象帽 25%、海象牙 50%，互不影响。[掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)、[共享表](../../../raw/current/scripts/components/lootdropper.lua#L21-L24)
- **营地的产出是"生成海象"，不是掉落**：`walrus_camp` 在冬季占位并生成 `walrus`／`little_walrus`／`icehound`，死亡与锤击都不经过 `lootdropper`；宿主侧要牙只能打成年海象（材料回收见下）。[冬季占位](../../../raw/current/scripts/prefabs/walrus_camp.lua#L101-L103)、[组队生成](../../../raw/current/scripts/prefabs/walrus_camp.lua#L208-L262)、[注册](../../../raw/current/scripts/prefabs/walrus_camp.lua#L486)
- **边界：海象营地不掉海象牙。** `walrus_camp` 的建造函数全段没有 `AddComponent("lootdropper")`，全库也只有 `walrus.lua` 一张表含 `walrus_tusk`。[建造段](../../../raw/current/scripts/prefabs/walrus_camp.lua#L448-L484)、[唯一表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)

## 用途与消耗

| 去向 | 消耗 | 条件与产物 | 证据 |
|---|---|---|---|
| 步行手杖 `cane` | 金块 2＋海象牙 1＋树枝 4 | `TECH.SCIENCE_TWO`；手杖本体与加速倍率不展开 | [配方](../../../raw/current/scripts/recipes.lua#L552) |
| 刷子 `brush` | 钢丝棉 1＋海象牙 1＋金块 2 | `TECH.SCIENCE_TWO`；刷毛对象是皮弗娄牛 | [配方](../../../raw/current/scripts/recipes.lua#L685) |
| 回溯怀表 `pocketwatch_recall` | 怀表零件 2＋金块 2＋海象牙 1 | `TECH.MAGIC_TWO`、时钟匠；`no_deconstruction` 由 `pocketwatch_nodecon` 判定 | [配方](../../../raw/current/scripts/recipes.lua#L367) |

- **三条配方都只要 1 个**：海象牙是"一次性解锁件"，不是可堆量消耗品；10 叠的设置只影响搬运与仓储。[消耗行](../../../raw/current/scripts/recipes.lua#L552)、[堆叠](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L30-L31)
- **拆解读回**：解构法杖 `greenstaff` 对「有配方且 `no_deconstruction` 求值不为真」的目标生效，按 `AllRecipes[prefab].ingredients` 逐项返还——步行手杖与刷子各返 **1 牙**（手杖无耐久件、比例 1；刷子按剩余次数折算但 1 牙仍取整为 1）。回溯怀表把这个字段设成 `pocketwatch_nodecon`：怀表**仍可施法**（无 `pocketwatch_inactive` 标签）时该函数返回真、此时不可拆；怀表已带 `pocketwatch_inactive` 标签时返回假，仍可拆解。[拆解比例与生成](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[目标门禁](../../../raw/current/scripts/components/spellcaster.lua#L190-L191)、[怀表门禁](../../../raw/current/scripts/recipes.lua#L360-L368)
- **没有燃料／烹饪／燃烧去向**：上述三条制造是它唯一的加工路径，其余只有堆叠、搬运与被作祟抛飞（它没有 `tradable`，也不能交易）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 显式 10（`STACK_SIZE_LARGEITEM`） | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L30-L31) |
| 组件：`inventoryitem` | 可拾取、可落地 | 主模拟 | [[reference/components/inventoryitem]]；[挂载](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L33) |
| 组件：`hauntable` | 作祟只抛飞 | 主模拟 | [[reference/components/hauntable]]；[挂载](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L35) |
| 标签：无 | 全文件无 `AddTag`，不参与标签过滤 | — | [构造](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L22-L35) |
| 宿主：`walrus` | 0.50 宿主来源（另有拆解回收：手杖／刷子直接可拆，怀表需 `no_deconstruction` 求值为假） | 服务端死亡 `DropLoot` | [表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)、[接线](../../../raw/current/scripts/prefabs/walrus.lua#L181) |
| 宿主：`walrus_camp` | 只生成海象，不掉海象牙 | 冬季占位 | [生成](../../../raw/current/scripts/prefabs/walrus_camp.lua#L208-L262)、[冬季](../../../raw/current/scripts/prefabs/walrus_camp.lua#L101-L103) |
| 关联 Prefab：`cane`／`brush`／`pocketwatch_recall` | 三条配方产物 | 科学二本／魔二 | [手杖](../../../raw/current/scripts/recipes.lua#L552)、[刷子](../../../raw/current/scripts/recipes.lua#L685)、[怀表](../../../raw/current/scripts/recipes.lua#L367) |
| 资源：`anim/walrus_tusk.zip` | bank／build `walrus_tusk` | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L1-L4) |

## 常见陷阱与限制

- **误区：打倒海象营地或小海象也能拿牙。** 小海象表只有肉；营地没有掉落组件，只负责冬季生成队伍。[小海象表](../../../raw/current/scripts/prefabs/walrus.lua#L31-L34)、[营地构造](../../../raw/current/scripts/prefabs/walrus_camp.lua#L448-L484)
- **边界：0.50 不等于"每两次出一颗"。** 它是每条独立掷点（无幸运基准），期望值 0.5 只在大样本下成立；同表的海象帽 0.25 与吹箭 1.00 各算各的。[表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)
- **边界：10 叠不是 40。** 它走大件档 `STACK_SIZE_LARGEITEM`，按小件估仓储会差 4 倍。[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **边界：它不可燃、不能当燃料。** 同批的触手皮可作 15 秒燃料、针刺可燃烧，海象牙两样都没有，不能按"动物材料同族"推断。[构造](../../../raw/current/scripts/prefabs/walrus_tusk.lua#L28-L35)
- **未核验**：未运行游戏。0.50 与期望为源码推导；海象刷新周期与海象营地冬季刷新量未核验。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读 `walrus_tusk.lua`（全 40 行）；`walrus.lua` 的两张共享表、接线与两个注册；`walrus_camp.lua` 的建造函数、冬季占位与组队生成；`stackable`／`lootdropper`／`standardcomponents` 的相关函数；`recipes.lua` 的三条配方；`tuning.lua` 堆叠常量；`chinese_s.po` 的海象牙与海象名称键。
  未覆盖：海象 AI、冰猎犬护卫、营地刷新计时；手杖／刷子／怀表的成品机制。
- 游戏运行：未运行；概率与叠数为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付回报。

## 相关链接

- 宿主与刷新：海象营地 `walrus_camp` 的生成逻辑（Wiki 待建；[构造](../../../raw/current/scripts/prefabs/walrus_camp.lua#L448-L484)）
- 组件：[[reference/components/stackable]]、[[reference/components/lootdropper]]、[[reference/components/hauntable]]
- 同批页面：[[reference/prefabs/honeycomb|蜜脾]]、[[reference/prefabs/houndstooth|犬牙]]、[[reference/prefabs/stinger|针刺]]、[[reference/prefabs/tentaclespots|触手皮]]

返回 [[reference/prefabs/index|Prefab 索引]]。
