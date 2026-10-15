---
title: "长矛（spear）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版基础手持武器 spear：单注册 HANDS 槽近战武器，伤害 34、耐久 150 仅攻击磨损，耗尽整个实体移除；可借鉴最小武器组装。"
aliases: ["spear", "长矛", "Spear", "基础武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/spear.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/combat.lua", "components/retrofitforestmap_anr.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "recipes.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabs/hutch.lua", "prefabs/charlie_stage.lua", "prefabs/hound.lua", "map/layouts.lua", "map/pointsofinterest.lua", "map/levels/forest.lua", "worldgen_main.lua", "map/static_layouts/cave_base_1.lua", "map/static_layouts/chessy_1.lua", "map/static_layouts/teleportato_box_layout.lua", "map/static_layouts/skeleton_warrior.lua", "map/static_layouts/skeleton_hunter.lua", "map/static_layouts/skeleton_hunter_swamp.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 长矛（spear）

## 快速判断

- 职责与适用：`prefabs/spear.lua` 顶层 `fn` 直接注册的单注册可拾取手持武器（物品＋装备），原版基础近战武器模板：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽；无 tool、投射、onattack 定制，同文件无 quagmire 分支。[源文件](../../../raw/current/scripts/prefabs/spear.lua#L28-L79)
- 优先使用：从零造简单近战武器按[注册与构造](#注册与构造)复刻组装并查 [[cookbook/equipment/hand]]；改伤害/耐久查下方数值表。
- 重要副作用：耐久只有攻击磨损一条路径；耗尽 `SetOnFinished(inst.Remove)` 整个实体移除、无掉落。[OnAttack 磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/spear.lua#L49-L53)）；皮肤事件在装备回调内；不覆盖矛族其他 prefab 与皮肤贴图内容。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spear` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `spear_init_fn`。 [注册](../../../raw/current/scripts/prefabs/spear.lua#L79)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L756-L757) |

共同初始化（[spear.lua#L28-L77](../../../raw/current/scripts/prefabs/spear.lua#L28-L77)）：实体组件＋`MakeInventoryPhysics` → `SetBank("spear")`＋`SetBuild("swap_spear")`＋`PlayAnimation("idle")` → 公共端标签 `sharp`／`pointy`／`weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/spear.lua#L44-L45)）→ `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -9)` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → equippable → `MakeHauntableLaunch`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.SPEAR`=长矛（Spear）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L349745-L349747)

数值（tuning 键＋实际值；`wilson_attack`=34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 耐久上限 | `TUNING.SPEAR_USES` | 150（[tuning.lua#L166](../../../raw/current/scripts/tuning.lua#L166)） |
| 武器伤害 | `TUNING.SPEAR_DAMAGE`=`wilson_attack` | 34 生命值（源码推导，[tuning.lua#L355](../../../raw/current/scripts/tuning.lua#L355)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool）（[OnAttack](../../../raw/current/scripts/components/weapon.lua#L114)） |

获取途径（全 scripts `SpawnPrefab("spear")`／`SpearSpawn` grep 零命中）：

| 途径 | 证据 |
|---|---|
| 配方：twigs 2＋rope 1＋flint 1，`TECH.SCIENCE_ONE`（={SCIENCE=1}） | [配方](../../../raw/current/scripts/recipes.lua#L478)、[TECH](../../../raw/current/scripts/constants.lua#L1245) |
| 世界生成：6 个静态布局含 `type="spear"` 放置点 | [cave_base_1](../../../raw/current/scripts/map/static_layouts/cave_base_1.lua#L196)、[chessy_1](../../../raw/current/scripts/map/static_layouts/chessy_1.lua#L120)、[teleportato_box](../../../raw/current/scripts/map/static_layouts/teleportato_box_layout.lua#L315)、[skeleton_warrior](../../../raw/current/scripts/map/static_layouts/skeleton_warrior.lua#L142)、[skeleton_hunter](../../../raw/current/scripts/map/static_layouts/skeleton_hunter.lua#L193)、[skeleton_hunter_swamp](../../../raw/current/scripts/map/static_layouts/skeleton_hunter_swamp.lua#L205) |
| 消费端：`Chessy_1`→森林 level；skeleton 三布局→POI 群系表→世界生成；CaveBase／TeleportatoBox 仅注册、无按名消费端（全 scripts grep） | [注册](../../../raw/current/scripts/map/layouts.lua#L852)、[forest](../../../raw/current/scripts/map/levels/forest.lua#L20)、[POI 表](../../../raw/current/scripts/map/pointsofinterest.lua#L33-L44)、[Any 组](../../../raw/current/scripts/map/pointsofinterest.lua#L89-L95)、[挂载](../../../raw/current/scripts/worldgen_main.lua#L352)、[两注册行](../../../raw/current/scripts/map/layouts.lua#L667-L724) |
| 特殊事件：查理舞台演出后从 `REWARDPOOL` 抽组刷 hedgehound_bush，击杀掉落（hunter 组含 spear）；旧世界 retrofit 接入 | [REWARDPOOL](../../../raw/current/scripts/prefabs/charlie_stage.lua#L154-L162)、[刷 bush](../../../raw/current/scripts/prefabs/charlie_stage.lua#L174-L184)、[掉落](../../../raw/current/scripts/prefabs/hound.lua#L864-L871)、[retrofit](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L989-L990) |

配方原料：firestaff、icestaff、mermwatchtower（merm_builder）、shadowduelist_builder、fence_rotator（[recipes.lua#L109](../../../raw/current/scripts/recipes.lua#L109)、[L300](../../../raw/current/scripts/recipes.lua#L300)、[L492-L493](../../../raw/current/scripts/recipes.lua#L492-L493)、[L606](../../../raw/current/scripts/recipes.lua#L606)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | 仅 `SetDamage(SPEAR_DAMAGE)`；无 onattack／射程／投射 | 主模拟；推 `weapononattack` 后按 `(attackwear or 1)×multipliers` 扣耐久 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/spear.lua#L55-L56)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | 上限＝当前＝150；耗尽 `inst.Remove`；未设 SetConsumption | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/spear.lua#L60-L64)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖持握 swap_object、ARM 显隐 | 主模拟；槽默认 HANDS；皮肤推 `equipskinneditem`＋`OverrideItemSkinSymbol` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/spear.lua#L70-L72)、[onequip](../../../raw/current/scripts/prefabs/spear.lua#L7-L17) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/spear.lua#L66-L68) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -9)` | 公共端；`SetBankSwapOnFloat(true, -9)` | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/spear.lua#L47)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟按 `HAUNT_CHANCE_ALWAYS` 弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/spear.lua#L74)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`sharp` 标签 | `Combat:GetImpactSound` 按武器 `sharp` 切命中音效档 | 命中结算时读取 | [消费](../../../raw/current/scripts/components/combat.lua#L745) |
| API／数据：`pointy` 标签 | Hutch 的伤害反弹电池（`DamageReflectBattery`），FUGU 形态条件含 pointy | 放进 Hutch 时判定 | [电池](../../../raw/current/scripts/prefabs/hutch.lua#L138-L140)、[FUGU](../../../raw/current/scripts/prefabs/hutch.lua#L320-L323) |
| 皮肤 | bee／chess／forge_gungnir／forge_lance／hockey／northern／ragged／rose／rw_swamp／simple 共 10 种 | 统一 `basic_init_fn(inst, build, "swap_spear")` | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L11415-L11533)、[init](../../../raw/current/scripts/prefabskin.lua#L756-L757) |
| 资源：anim／swap 动画包 | spear.zip（bank `spear`）／swap_spear.zip（build＋持握符号） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/spear.lua#L1-L5) |

## 典型接入

### 修改长矛的伤害与耐久

- 前提与选择：服务端 `modmain.lua`；spear 组装极简无专属机制，`AddPrefabPostInit` 直接改组件数值。
- 生效与撤销：postinit 在构造与读档重建时执行，运行中已存在的实例不会重跑；`SetMaxUses` 只改上限，读档后存档 `uses` 会覆盖 `SetUses` 设置的当前值（`SetUses` 不按新上限钳制）；停用模组后新实例恢复原上限，已写入存档的当前耐久仍可能高于原上限。
- 原版依据：伤害在 weapon、耐久在 finiteuses，两条独立；无动作消耗端，改 `SetConsumption` 无效。

```lua
-- modmain.lua：长矛伤害 45、耐久 300
AddPrefabPostInit("spear", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.weapon:SetDamage(45)
    inst.components.finiteuses:SetMaxUses(300)
    inst.components.finiteuses:SetUses(300)
end)
```

- 调整与验证：单位为生命值/击、耐久点/次；磨损仍按 `attackwear or 1`，与上限独立。

## 常见陷阱与限制

- **误区：耐久耗尽会掉残骸**：`SetOnFinished(inst.Remove)` 直接 Remove 整个实体，无掉落。[回调](../../../raw/current/scripts/prefabs/spear.lua#L64)
- **边界：build 与 bank 不同名**：build 是 `swap_spear` 而 bank 是 `spear`，自定义变体推 `SetBuild("spear")` 会拿错 build。[初始化](../../../raw/current/scripts/prefabs/spear.lua#L37-L39)
- **边界：`weapon` 标签是公共端手打的 pristine 镜像**（客户端优化用）；同文件无 `GetServerGameMode` 调用，quagmire 下组件照常挂载。[注释](../../../raw/current/scripts/prefabs/spear.lua#L44-L45)
- **边界：耐久只有攻击磨损一条消耗路径**：不挂 tool，`OnUsedAsItem` 动作消耗链不触达；补采集功能需另挂 tool 并设消耗。[有限耐久组件](../components/finiteuses.md)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 PostInit 仅在构造／读档重建执行、存档 uses 覆盖当前值）。早前 2026-11-06，Steam buildid 26335394。核验 prefabs/spear.lua 全文、六组件关键方法、tuning 两键与 wilson_attack、配方行与 5 处原料消费、6 个布局放置点与消费链（Chessy_1→levels；skeleton→POI；CaveBase／TeleportatoBox 仅注册）、charlie_stage REWARDPOOL→hedgehound 掉落链与 retrofit 接入、sharp／pointy 消费端、10 种皮肤注册与 init。名称键以 chinese_s.po `msgctxt` 定向核验；hound.lua 的 spear 清单另核为 scrapbook 依赖数据（非生成点）。
- 游戏运行：未运行。未覆盖：布局实际落点与频率、charlie_stage 新世界生成路径与事件期实机奖励、皮肤实装外观；34 伤害为源码推导，非运行测量。
- 结构检查：validate_page.py 0 error（103 行，simple）与 validate_source_links.py --strict 0 error 0 warning 通过。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
