---
title: "实用与投掷装备（瓶中信／路标导航仪／种壳）"
created: "2026-11-07"
updated: "2026-11-09"
type: prefab
size_class: standard
summary: "三组 HANDS 实用/投掷装备：瓶中信系（显影/空瓶装恶液/海獭 AI 投掷瓶/恶液罐）、路标导航仪（三分支指针、marker/visual 清理、宝库生成与拆解）、种壳（equipstack 整叠投掷、59.5 伤害、AOE；PvP 下未命中落船板才漏水；waterplant 运行时武器 excluded）。"
aliases: ["messagebottle", "messagebottleempty", "messagebottle_throwable", "gelblob_bottle", "vault_compass", "vault_compass_marker", "waterplant_bomb", "瓶中信", "空瓶子", "恶液罐", "路标导航仪", "种壳", "Message in a Bottle", "Vault Compass", "Sea Wort Herb Seed"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/messagebottle.lua", "prefabs/vault_compass.lua", "prefabs/vaultroom_defs.lua", "prefabs/waterplant_seed.lua", "prefabs/waterplant.lua", "brains/hermitcrabbrain.lua", "components/equippable.lua", "components/mapspotrevealer.lua", "components/messagebottlemanager.lua", "components/bottler.lua", "components/waterproofer.lua", "components/complexprojectile.lua", "components/weapon.lua", "components/stackable.lua", "components/furnituredecor.lua", "components/inventoryitem.lua", "recipes.lua", "tuning.lua", "constants.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 实用与投掷装备（瓶中信／路标导航仪／种壳）

## 快速判断

- 职责与适用：三组 HANDS 实用/投掷链——瓶中信系的 READ 显影与空瓶装恶液、路标导航仪的三分支指针与虚拟房间、种壳的整叠投掷 AOE。适合查“非武器道具的 equippable 命中”“装备态生成 marker/visual”“投掷后落海与船板分流”。
- 优先使用：显影与读信查 [[reference/components/mapspotrevealer]]、[[reference/components/messagebottlemanager]]；导航仪的 visual/marker 见 [[reference/prefabs/vault_compass_visual]]；投掷与 AOE 查 [[reference/components/complexprojectile]]、[[reference/components/weapon]]。
- 重要副作用：瓶中信主体**无 equippable**；空瓶对 `gelblob_small_fx` 使用变恶液罐；恶液罐落海只漂浮消失、不释放恶液；罗盘装备生成 marker/visual，卸装/装模型要 ClearMarker；种壳是整叠装备（`equipstack=true`），PvP 开启且未命中有效目标、落点又满足船板距离阈值时才开小漏水。
- 修改边界：宝库 virtualroom 玩法与 `vaultroom_defs` 布局、海草 boss 的 AI 武器、messagebottlemanager 的完整世界任务不展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `messagebottle` | commonmakebottle：公共端 waterproofer/mapspotrevealer 标签 → 主模拟 waterproofer(0)、mapspotrevealer | 主体**无 equippable**——地图显影物品（READ）；读后都会变空瓶，0.4 概率只决定走便签台词还是真正显影。[工厂](../../../raw/current/scripts/prefabs/messagebottle.lua#L139-L195)、[显影](../../../raw/current/scripts/prefabs/messagebottle.lua#L113-L137) |
| `messagebottleempty` | emptybottlefn：公共端 waterproofer 标签 → 主模拟 bottler、stackable(20)、waterproofer(0) | 空瓶：商店配方材料；对 `gelblob_small_fx` 使用变恶液罐。[工厂](../../../raw/current/scripts/prefabs/messagebottle.lua#L232-L272)、[装瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228) |
| `messagebottle_throwable` | commonmakebottle(throwing_postinit)：追加 equippable(HANDS)＋complexprojectile(15 速／-35 重力) | hermitcrab 专用投掷瓶：Brain 内生成投掷，玩家常规流程不持有。[装配](../../../raw/current/scripts/prefabs/messagebottle.lua#L323-L338)、[投掷者](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L452-L454) |
| `gelblob_bottle` | gelblobbottlefn：公共端 projectile 标签 → 主模拟 stackable(10)、equippable(HANDS, `equipstack=true`)、complexprojectile | 玩家可持有：掷地释放恶液，落海漂浮后消失。[工厂](../../../raw/current/scripts/prefabs/messagebottle.lua#L429-L481)、[命中](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L401) |
| `vault_compass` | fn：公共端 weapon/furnituredecor 标签＋漂浮 → 主模拟 inspectable(KEYROOM/NOTVAULT)、inventoryitem、furnituredecor、equippable(HANDS)、weapon(UNARMED) | 装备生成 `vault_compass_marker`（挂 owner）＋`vault_compass_visual`（专页 [[reference/prefabs/vault_compass_visual]]）。[工厂](../../../raw/current/scripts/prefabs/vault_compass.lua#L140-L186)、[装备](../../../raw/current/scripts/prefabs/vault_compass.lua#L82-L100) |
| `vault_compass_marker`／`vault_compass_visual` | markerfn：空动画挂 owner；visualpointerfn：OnGround＋背景层＋泛光指针 | 均带 `staysthroughvirtualrooms`；visual `persists=false`，由 ClearMarker 清理。[marker](../../../raw/current/scripts/prefabs/vault_compass.lua#L190-L210)、[visual](../../../raw/current/scripts/prefabs/vault_compass.lua#L214-L243)；指针三态与清理归 [[reference/prefabs/vault_compass_visual#路标指针罗盘装备期的三层结构与方位判定]] |
| `waterplant_bomb` | fn：公共端 reticule＋projectile/weapon 标签 → 主模拟 complexprojectile、weapon、combat、inventoryitem、stackable、equippable(HANDS, `equipstack=true`) | 种壳：海草掉落，玩家可拾取整叠投掷，命中 AOE；boss 侧 `waterplant_projectile` 无 equippable。[工厂](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L148-L221)、[投掷命中](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L111-L127) |

## 机制

### 瓶中信系：显影、空瓶与恶液罐

`messagebottle` 走 `mapspotrevealer` 的 READ 动作，`getrevealtargetpos` 委托 TheWorld 的 `messagebottlemanager`；`prereveal` 在“已用过瓶/已见隐士”成立时 `TryLuckRoll(0.4)`（MESSAGEBOTTLE_NOTE_CHANCE）——命中就走便签分支：立即 `turn_empty` 且不显影；未命中才真正显影，而显影完成的 `on_reveal_map_spot_pst` 同样调 `turn_empty`——两条路最终都给空瓶，概率只决定给地图还是便签。另有 `ShouldForceMapReveal`：珍珠相关状态可判定且场上找不到珍珠/裂珍珠时，把 `hermit.gotcrackedpearl` 置 true 并强制显影；标记已经为真时也直接放行，属于防卡死补丁而不是“复位”。

空瓶 `messagebottleempty` 是 bottler：对 `gelblob_small_fx` 使用，消耗一瓶生成 `gelblob_bottle` 给使用者。恶液罐 `gelblob_bottle` 以 `equipstack=true` 整叠装备，`complexprojectile` 掷地释放恶液（SetLifespan=1 天）；落海只漂浮后随动画消失，恶液损失。`messagebottle_throwable` 是 hermitcrab Brain 内生成的 AI 投掷瓶，玩家常规流程不持有。

[显影/便签](../../../raw/current/scripts/prefabs/messagebottle.lua#L113-L137)、[防卡死](../../../raw/current/scripts/prefabs/messagebottle.lua#L86-L111)、[装瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228)、[恶液罐装配](../../../raw/current/scripts/prefabs/messagebottle.lua#L459-L478)、[落海分流](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L401)

### 路标导航仪：三分支指针与清理

`OnEquip` 生成 marker 挂 owner、visual 贴地指针，并开 `DoPeriodicTask(0, OnUpdateDirection)` 逐帧更新；卸装或装模型时 `ClearMarker`。`OnUpdateDirection` 三分支：VAULT 虚室→指向 key1 房（抵达切 success 动画）；LOBBYVAULT→指向 `vaultmarker_lobby_to_vault`；他处→指向 `archive_portal`（2.5 格内 success）。`inspectable:GetStatus` 在 key1 房内给 `KEYROOM`、虚室外给 `NOTVAULT`。

导航仪由宝库 key1 房与 lobby 固定点生成（`vaultroom_defs`），拆解返铥矿碎片 1＋月岩 1；weapon 伤害 `UNARMED_DAMAGE=10`，同时是 `furnituredecor` 可作桌面装饰。visual 是 `persists=false` 的纯表现，不能把它当存档实体。

[指针生成](../../../raw/current/scripts/prefabs/vault_compass.lua#L82-L100)、[三分支](../../../raw/current/scripts/prefabs/vault_compass.lua#L21-L80)、[清理](../../../raw/current/scripts/prefabs/vault_compass.lua#L102-L117)、[生成](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L465-L465)、[拆解](../../../raw/current/scripts/recipes.lua#L1684-L1684)

### 种壳：整叠投掷与 AOE

`waterplant_bomb` 以 `equipstack=true` 整叠装备为手持物，换持握图标 `swap_barnacle_burr`；weapon 伤害 `wilson_attack×1.75=59.5`、射程 8-10。投掷 OnHit `do_bomb`：对 `_combat` 实体结算 AOE，`stunnedbybomb` 触发 stunbomb；漏水只走 PvP 分支——`TheNet:GetPVPEnabled()` 为真时以 `break_boats=true` 结算，且必须未命中有效目标、落点到船板距离小于 `GOOD_LEAKSPAWN_PLATFORM_RADIUS`；PvE 分支照常结算 AOE，但不开漏水。获取为海草共享掉落表（普通 2–3、黄 3–5），stackable 默认上限（scrapbook 记 20）。

分诊：`waterplant.lua` 的 equippable 在 `equip_ranged_weapon` 里是运行时 `CreateEntity` 临时武器（weapon＋inventoryitem＋equippable＋`nosteal`），**不注册 prefab**、只 Equip 给海草 boss 自身，玩家不可获取，记 excluded；玩家可拾取的 equippable 是 `waterplant_seed.lua` 的 `waterplant_bomb`。`waterplant_projectile` 是 boss 弹，无 equippable、不可拾取。

[种壳装配](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L210-L213)、[do_bomb](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L30-L74)、[weapon](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L195-L197)、[掉落表](../../../raw/current/scripts/prefabs/waterplant.lua#L29-L43)、[分诊](../../../raw/current/scripts/prefabs/waterplant.lua#L198-L218)

## 常见陷阱与限制

- **瓶中信主体不是投掷武器**：`messagebottle` 无 equippable（READ 显影物品）；文件内两处 equippable 是 hermitcrab AI 专用投掷瓶与玩家恶液罐。[主体](../../../raw/current/scripts/prefabs/messagebottle.lua#L139-L195)
- **恶液罐落海不释放恶液**：OnHit 按地/海分流，掷地才 Spawn 恶液并 ReleaseFromBottle，落海漂浮后消失（内容损失）。[分流](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L401)
- **罗盘依赖 virtualroom**：三分支都读 virtualroommanager；不在宝库/大厅时只指向 archive_portal，不能按“永远指向宝库”写。[三分支](../../../raw/current/scripts/prefabs/vault_compass.lua#L21-L80)
- **种壳是整叠装备**：`equipstack=true`，不是单件；投掷命中与船板漏水是两条分支——漏水仅 PvP 且未命中有效目标时成立，别只写 AOE，也别写成 PvE 通用机制。[装配](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L210-L213)、[漏水门禁](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L64-L71)、[PvP 分派](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L109-L126)
- **waterplant 命中是 boss 临时武器**：`waterplant.lua` 的 equippable 不注册 prefab，玩家不可获取；可拾取的是 `waterplant_bomb`。[分诊](../../../raw/current/scripts/prefabs/waterplant.lua#L198-L218)
- **官方名**：MESSAGEBOTTLE＝瓶中信、MESSAGEBOTTLEEMPTY＝空瓶子、GELBLOB_BOTTLE＝恶液罐、VAULT_COMPASS＝路标导航仪、WATERPLANT_BOMB＝种壳。[瓶中信名](../../../raw/current/scripts/languages/chinese_s.po#L345409-L345412)、[空瓶名](../../../raw/current/scripts/languages/chinese_s.po#L345414-L345417)、[恶液罐名](../../../raw/current/scripts/languages/chinese_s.po#L343149-L343152)、[罗盘名](../../../raw/current/scripts/languages/chinese_s.po#L351189-L351192)、[种壳名](../../../raw/current/scripts/languages/chinese_s.po#L351844-L351847)

## 验证记录

- 源码审阅：2026-11-07（第二轮检视修订），Steam buildid 26335394。复核瓶中信两条变空瓶路径与 gotcrackedpearl 的修补语义、种壳 PvP 漏水门禁。实读 messagebottle、vault_compass、waterplant_seed、waterplant 与 hermitcrabbrain 的投掷段；核验主体/空瓶/投掷瓶/恶液罐四条 equippable 命中、prereveal 0.4、罗盘三分支与 marker/visual 清理、种壳 equipstack/59.5/AOE/船板漏水、waterplant 运行时武器分诊；官方名五键抽查。
- 游戏运行：未运行；宝库 virtualroom 布局、messagebottlemanager 世界任务、海草 boss 战斗表现未做运行时验证。
- 结构检查：validate_page 实测 79 行（standard 档 480 行／24 KiB 内）通过；strict 39 引用 0 错 0 警、30 唯一目标、8 源文件；record 后 21 组来源 unchanged。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
