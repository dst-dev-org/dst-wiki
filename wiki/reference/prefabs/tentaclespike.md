---
title: "触手尖刺（tentaclespike）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版掉落型手持武器 tentaclespike：无配方，击杀触手（0.5）与大触手（0.50）掉落；伤害 51、耐久 100 仅攻击磨损，耗尽整个实体移除；无 onattack、无皮肤，可借鉴最小武器组装与掉落获取。"
aliases: ["tentaclespike", "触手尖刺", "Tentacle Spike", "触手掉落武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
size_class: simple
sources: ["prefabs/tentaclespike.lua", "prefabs/tentacle.lua", "prefabs/tentacle_pillar.lua", "scenarios/chest_random_good.lua", "scenarios/chestfunctions.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/floater.lua", "components/combat.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 触手尖刺（tentaclespike）

## 快速判断

- 职责与适用：`prefabs/tentaclespike.lua` 顶层 `fn` 直接注册的单注册可拾取手持武器（物品＋装备），原版"掉落型中阶武器"参照：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽；无 tool、无 onattack、无投射，同文件无 quagmire 分支、无皮肤注册。[源文件](../../../raw/current/scripts/prefabs/tentaclespike.lua#L18-L67)
- 优先使用：改伤害／耐久查下方数值表与[典型接入](#修改触手尖刺的伤害与耐久)；查获取途径查[数值与获取](#数值与获取)；从零造简单近战武器按[注册与构造](#注册与构造)复刻并查 [[cookbook/equipment/hand]]。
- 重要副作用：耐久只有攻击磨损一条路径；耗尽 `SetOnFinished(inst.Remove)` 整个实体移除、无掉落。[OnAttack 磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/tentaclespike.lua#L39-L43)）；掉落来源 tentacle／tentacle_pillar 的生物机制不在本页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tentaclespike` | 无包装工厂：`fn()` 直接构造，无 postinit，同文件仅此一注册 | 物品＋装备；无皮肤变体。[注册](../../../raw/current/scripts/prefabs/tentaclespike.lua#L67) |

共同初始化（[tentaclespike.lua#L18-L65](../../../raw/current/scripts/prefabs/tentaclespike.lua#L18-L65)）：实体组件＋`MakeInventoryPhysics` → `SetBank("spike")`＋`SetBuild("tentacle_spike")`＋`PlayAnimation("idle")` → 公共端标签 `sharp`、`weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/tentaclespike.lua#L31-L32)）→ `MakeInventoryFloatable("med", 0.05, {0.9, 0.5, 0.9}, true, -17, {sym_build = "swap_spike", bank = "spike"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → equippable → `MakeHauntableLaunch`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.TENTACLESPIKE`＝触手尖刺（Tentacle Spike）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L350329-L350332)

### 数值与获取

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 耐久上限 | `TUNING.SPIKE_USES` | 100（[tuning.lua#L169](../../../raw/current/scripts/tuning.lua#L169)） |
| 武器伤害 | `TUNING.SPIKE_DAMAGE`＝`wilson_attack`×1.5 | 51 生命值（源码推导，[tuning.lua#L352](../../../raw/current/scripts/tuning.lua#L352)；图鉴数据同值佐证，[scrapbookdata.lua#L1503](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1503)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |

获取途径（全 scripts `grep tentaclespike` 复核；`SpawnPrefab("tentaclespike")` 无直接命中，仅下表 scenario 经 chestfunctions 间接生成）：

| 途径 | 内容与证据 |
|---|---|
| 触手（tentacle）掉落 | 共享掉落表 `monstermeat` 1.0×2＋`tentaclespike` 0.5＋`tentaclespots` 0.2；lootdropper 挂 `SetChanceLootTable('tentacle')`。[掉落表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21)、[接线](../../../raw/current/scripts/prefabs/tentacle.lua#L124-L125) |
| 大触手（tentacle_pillar）掉落 | 掉落表 `tentaclespike` 0.50（另 turf_marsh 0.25、tentaclespots 0.40、rocks 1.00）；死亡时（未 overtaken）从 y=20 处 DropLoot。[掉落表](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L20-L26)、[OnDeath](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L224-L229)、[接线](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L369-L370) |
| scenario 宝箱 | `chest_random_good` 随机池 6–9 抽候选之一，生成时耐久随机设 25%–100%（initfn 在 SpawnPrefab 后调用，会覆盖同帧 postinit 的耐久设置）；快照 scripts 内未检索到该 scenario 的按名消费端。[池条目](../../../raw/current/scripts/scenarios/chest_random_good.lua#L76-L78)、[initfn 时序](../../../raw/current/scripts/scenarios/chestfunctions.lua#L34-L42) |

无配方、无世界生成放置：recipes.lua／recipes_filter.lua／map/ 全目录 grep 零命中。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | 仅 `SetDamage(SPIKE_DAMAGE)`；无 onattack／射程／投射 | 主模拟；`OnAttack` 按 `(attackwear or 1)×multipliers` 扣耐久 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespike.lua#L45-L46) |
| 组件：finiteuses | 上限＝当前＝100；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespike.lua#L50-L53)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖持握 `swap_object`、ARM_carry／ARM_normal 显隐；回调内不处理皮肤事件 | 主模拟；槽默认 HANDS | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/tentaclespike.lua#L58-L60)、[onequip](../../../raw/current/scripts/prefabs/tentaclespike.lua#L7-L11) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespike.lua#L55-L56) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", 0.05, {0.9, 0.5, 0.9}, true, -17, swap_data)` | 公共端；浮水播 bank `spike` 并用 `swap_spike` 覆写 `swap_spear` 符号（floater 固定符号名） | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/tentaclespike.lua#L36-L37)、[符号覆写](../../../raw/current/scripts/components/floater.lua#L133-L139)、[bank 切换](../../../raw/current/scripts/components/floater.lua#L198-L205) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟按 `HAUNT_CHANCE_ALWAYS` 弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/tentaclespike.lua#L62)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`sharp` 标签 | `Combat:GetImpactSound` 按武器有无 `sharp` 切命中音效档 | 命中结算时读取 | [[reference/components/combat#关键接口]]；[消费](../../../raw/current/scripts/components/combat.lua#L745) |
| 关联 Prefab：触手（tentacle）／大触手（tentacle_pillar，Wiki 待建） | 掉落来源：击杀各自按共享掉落表 0.5／0.50 概率掉本武器 | 击杀结算时 | [tentacle 表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21)、[pillar 表](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L20-L26) |
| 资源：anim／swap 动画包 | tentacle_spike.zip（bank `spike`）／swap_spike.zip（build＋持握符号） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/tentaclespike.lua#L1-L5) |

## 典型接入

### 修改触手尖刺的伤害与耐久

- 前提与选择：服务端 `modmain.lua`；组装极简无专属机制，`AddPrefabPostInit` 直接改组件数值。
- 生效与撤销：postinit 在构造与读档重建时执行，运行中已存在的实例不会重跑；`SetMaxUses` 只改上限，读档后存档 `uses` 会覆盖 `SetUses` 设置的当前值（`SetUses` 不按新上限钳制，[finiteuses.lua#L48-L77](../../../raw/current/scripts/components/finiteuses.lua#L48-L77)）；停用模组后新实例恢复原上限，已写入存档的当前耐久仍可能高于原上限。
- 原版依据：伤害在 weapon、耐久在 finiteuses，两条独立；无动作消耗端，改 `SetConsumption` 无效。

```lua
-- modmain.lua：触手尖刺伤害 60、耐久 150
AddPrefabPostInit("tentaclespike", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.weapon:SetDamage(60)
    inst.components.finiteuses:SetMaxUses(150)
    inst.components.finiteuses:SetUses(150)
end)
```

- 调整与验证：单位为生命值/击、耐久点/次；磨损仍按 `attackwear or 1`，与上限独立。伤害与耐久在实例构造时读 `TUNING`（运行时改 `TUNING.SPIKE_DAMAGE`／`SPIKE_USES` 只对新实例生效）。

## 常见陷阱与限制

- **误区：触手尖刺可以合成**：无配方，获取只靠 tentacle／tentacle_pillar 掉落与 chest_random_good scenario 池。[途径](#数值与获取)
- **边界：耐久耗尽即整个实体 Remove**：`SetOnFinished(inst.Remove)` 直接移除，无残骸掉落。[回调](../../../raw/current/scripts/prefabs/tentaclespike.lua#L53)
- **边界：只有 `sharp` 没有 `pointy`**：Hutch 的 pointy 伤害反弹电池不含本武器（对照长矛 [[reference/prefabs/spear#关联机制导航]]）；`sharp` 在快照内的消费端只有命中音效档。[标签](../../../raw/current/scripts/prefabs/tentaclespike.lua#L29-L32)
- **边界：build 与 bank 不同名**：build 是 `tentacle_spike` 而 bank 是 `spike`，自定义变体推 `SetBuild("spike")` 会拿错 build。[初始化](../../../raw/current/scripts/prefabs/tentaclespike.lua#L25-L27)

## 验证记录

- 源码审阅：2026-11-07（第三轮修订：复核 PostInit 作用域、存档 uses 覆盖与“大触手”名称）。早前 2026-11-06，steam buildid 26335394（raw/current/BUILD.txt）。核验 tentaclespike.lua 全文（单注册、无 onattack、无 quagmire、无皮肤事件）、weapon／finiteuses／equippable／inspectable／inventoryitem／floater、`sharp` 消费端、tuning 两键、两处掉落表与 scenario 池 initfn 时序、po 名称；全 scripts grep 其余命中均为数据／调试，非生成点；图鉴数值与 tuning 推导一致。
- 游戏运行：未运行。未覆盖：51 伤害与 0.5 掉率为源码推导，非运行测量；触手／大触手实机掉落频率、chest_random_good 实机生成路径未验证。
- 结构检查：validate_page.py 与 validate_source_links.py --strict 实际执行通过（实测 104 行／12075 字节）。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
