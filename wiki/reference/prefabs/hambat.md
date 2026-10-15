---
title: "火腿棒（hambat）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版手持武器 hambat：单注册 HANDS 槽近战武器，无耐久上限，伤害随新鲜度在 59.5→29.75 生命值线性衰减，0 新鲜度时整体替换为腐坏食物；可借鉴『新鲜度驱动武器伤害』组装。"
aliases: ["hambat", "火腿棒", "Ham Bat", "火腿武器", "新鲜度武器", "火腿棒伤害"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/hambat.lua", "components/weapon.lua", "components/perishable.lua", "components/equippable.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/forcecompostable.lua", "components/combat.lua", "components/inventory.lua", "standardcomponents.lua", "mathutil.lua", "tuning.lua", "constants.lua", "recipes.lua", "prefabs/compostingbin.lua", "prefabs/spoiledfood.lua", "containers.lua", "widgets/itemtile.lua", "map/boons.lua", "worldgen_main.lua", "screens/redux/scrapbookdata.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "skin_gifts.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 火腿棒（hambat）

## 快速判断

- 职责与适用：`prefabs/hambat.lua` 顶层 `fn` 直接注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋perishable 新鲜度，**不挂 finiteuses**（DST 无耐久上限），伤害随新鲜度 59.5→29.75 线性衰减，腐坏后整个实体替换为 `spoiled_food`；edible 组件整段被注释，不可食用；同文件无 quagmire 分支。[源文件](../../../raw/current/scripts/prefabs/hambat.lua#L43-L105)
- 优先使用：借鉴"新鲜度驱动武器伤害"按[注册与构造](#注册与构造)的公式与[典型接入](#典型接入)复刻；改伤害／腐坏查下方数值表；从零造手持武器查 [[cookbook/equipment/hand]]；耐久型武器对照 [[reference/prefabs/spear]]（finiteuses 磨损模型）。
- 重要副作用：新鲜度一直计时（装备／背包／容器／地面都在腐坏，速率按位置修正），0 新鲜度时实体被整体替换为 spoiled_food（优先按原格交回，装备中查不到原格则 GiveItem 自动找可用槽）；`icebox_valid` 可放入冰箱，带 `fridge` 标签的容器（含保鲜背包）走 ×0.5 档；腐坏速率还会叠乘季节温度、湿度、局部与全局倍率；`show_spoilage` 让物品栏显示腐坏度条。[Perish 替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/hambat.lua#L65-L69)）；皮肤事件在装备回调内；不覆盖皮肤贴图内容与 consolecommands／debugcommands 调试入口。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hambat` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `hambat_init_fn`。 [注册](../../../raw/current/scripts/prefabs/hambat.lua#L107)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L190-L191) |

共同初始化（[hambat.lua#L43-L105](../../../raw/current/scripts/prefabs/hambat.lua#L43-L105)）：实体组件＋`MakeInventoryPhysics` → `SetBank("ham_bat")`＋`SetBuild("ham_bat")`＋`PlayAnimation("idle")` → 公共端标签 `show_spoilage`、`icebox_valid`、`weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/hambat.lua#L59-L60)）→ `MakeInventoryFloatable("med", nil, {1.0, 0.5, 1.0}, true, -13, {sym_build="swap_ham_bat", bank="ham_bat"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 perishable → weapon → forcecompostable → `inst.OnLoad = OnLoad` → inspectable → inventoryitem → `MakeHauntableLaunchAndPerish` → equippable。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[inspectable 构造挂标签](../../../raw/current/scripts/components/inspectable.lua#L2-L5)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.HAMBAT`=火腿棒（Ham Bat）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L343699-L343702)

伤害公式（`UpdateDamage`，[hambat.lua#L7-L13](../../../raw/current/scripts/prefabs/hambat.lua#L7-L13) 原文）：

```lua
local dmg = TUNING.HAMBAT_DAMAGE * inst.components.perishable:GetPercent()
dmg = Remap(dmg, 0, TUNING.HAMBAT_DAMAGE, TUNING.HAMBAT_MIN_DAMAGE_MODIFIER*TUNING.HAMBAT_DAMAGE, TUNING.HAMBAT_DAMAGE)
inst.components.weapon:SetDamage(dmg)
```

- `GetPercent()` = `min(1, 剩余腐坏时间 / 总腐坏时间)`，恒在 [0,1]；`Remap` 是**无钳制**线性映射 [mathutil.lua#L19-L21](../../../raw/current/scripts/mathutil.lua#L19-L21)、[GetPercent](../../../raw/current/scripts/components/perishable.lua#L215-L221)。
- 等效化简（源码推导）：`伤害 = 59.5 × (0.5 + 0.5 × 新鲜度)`——满鲜 59.5、50% 新鲜 44.625、0 新鲜 29.75 生命值。

数值（tuning 键＋实际值；`wilson_attack`=34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 满新鲜度伤害 | `TUNING.HAMBAT_DAMAGE`=`wilson_attack`×1.75 | 59.5 生命值（源码推导，[tuning.lua#L353](../../../raw/current/scripts/tuning.lua#L353)） |
| 伤害下限系数 | `TUNING.HAMBAT_MIN_DAMAGE_MODIFIER` | 0.5 → 最低 29.75 生命值（[tuning.lua#L354](../../../raw/current/scripts/tuning.lua#L354)） |
| 腐坏总时长 | `TUNING.PERISH_MED`=10×`total_day_time`×`perish_warp` | 4800 秒＝10 游戏日（total_day_time=480、perish_warp=1，[L19-L20](../../../raw/current/scripts/tuning.lua#L19-L20)、[L47](../../../raw/current/scripts/tuning.lua#L47)、[L2048](../../../raw/current/scripts/tuning.lua#L2048)） |
| 耐久 | 不挂 finiteuses；`TUNING.HAMBAT_USES`=100 | 全 scripts 无消费者（[tuning.lua#L179](../../../raw/current/scripts/tuning.lua#L179)，单机版遗留）；每次攻击磨损分支因 finiteuses 判空跳过（[weapon.lua#L111-L120](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |

佐证：图鉴数据条 `hambat` 记 `weapondamage=59.5`、`perishable=4800`，与源码推导一致。[scrapbookdata.lua#L673](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L673)

更新时机：`UpdateDamage` 只在装备（[onequip](../../../raw/current/scripts/prefabs/hambat.lua#L19-L20)）、卸装（[onunequip](../../../raw/current/scripts/prefabs/hambat.lua#L33-L34)）、每次攻击结算（`SetOnAttack`，[hambat.lua#L76-L78](../../../raw/current/scripts/prefabs/hambat.lua#L76-L78)）与读档（[OnLoad](../../../raw/current/scripts/prefabs/hambat.lua#L15-L17)）触发；`Combat:DoAttack` 先 `CalcDamage` 取 `weapon.damage` 结算伤害、后调 `weapon:OnAttack`（[combat.lua#L1212](../../../raw/current/scripts/components/combat.lua#L1212)、[L1224](../../../raw/current/scripts/components/combat.lua#L1224)），即每次攻击用的是上一次更新点的快照，连续攻击近似实时。

获取途径（均在快照内核验）：

| 途径 | 证据 |
|---|---|
| 配方：猪皮 1＋树枝 2＋肉 2，`TECH.SCIENCE_TWO`（={SCIENCE=2}） | [配方](../../../raw/current/scripts/recipes.lua#L480)、[TECH](../../../raw/current/scripts/constants.lua#L1246) |
| 世界生成：Rare 组 `Level4Boon` 的 `item_area` 从 8 件武器随机抽 1（含 hambat）；抽到 Rare 组约 2% 概率保留 | [boons](../../../raw/current/scripts/map/boons.lua#L163-L166)、[Boons 组装](../../../raw/current/scripts/map/boons.lua#L171-L174)、[Rare 98% 跳过](../../../raw/current/scripts/worldgen_main.lua#L271)、[AddSingleSetPeice](../../../raw/current/scripts/worldgen_main.lua#L300-L307)、[数量循环](../../../raw/current/scripts/worldgen_main.lua#L368-L372) |

原料中文名经 .po 核验：Pig Skin=猪皮（[L346941](../../../raw/current/scripts/languages/chinese_s.po#L346941)）、Twigs=树枝（[L351126](../../../raw/current/scripts/languages/chinese_s.po#L351126)）、Meat=肉（[L345251](../../../raw/current/scripts/languages/chinese_s.po#L345251)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：perishable | `SetPerishTime(PERISH_MED)`＋`StartPerishing`；`onperishreplacement="spoiled_food"` | 主模拟；计时不受装备状态影响，速率按持有位置修正（冰箱 ×0.5、地面 ×1.5、湿/冬/夏另有乘区） | [[reference/components/perishable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hambat.lua#L71-L74)、[Update 修正链](../../../raw/current/scripts/components/perishable.lua#L60-L154) |
| 组件：weapon | `SetDamage(HAMBAT_DAMAGE)`；`SetOnAttack(UpdateDamage)` | 主模拟；无射程／投射／electric 定制 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hambat.lua#L76-L78)、[SetDamage](../../../raw/current/scripts/components/weapon.lua#L44-L46) |
| 组件：forcecompostable | `green = true`：放入堆肥箱按 1 绿计算 | 主模拟 | [[reference/components/forcecompostable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hambat.lua#L80-L81)、[消费](../../../raw/current/scripts/prefabs/compostingbin.lua#L197-L199) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：先 `UpdateDamage` 再维护 swap_object、ARM 显隐与皮肤事件 | 主模拟；槽默认 HANDS；皮肤推 `equipskinneditem`＋`OverrideItemSkinSymbol` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/hambat.lua#L100-L102)、[onequip](../../../raw/current/scripts/prefabs/hambat.lua#L19-L31)、[onunequip](../../../raw/current/scripts/prefabs/hambat.lua#L33-L41) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hambat.lua#L94-L96)、[replica 同步](../../../raw/current/scripts/components/inventoryitem.lua#L1-L9) |
| 组件：hauntable | `MakeHauntableLaunchAndPerish`：被作祟按 `HAUNT_CHANCE_ALWAYS` 弹飞；helper 内"作祟加速腐坏"分支整体注释（#HAUNTFIX），实际只弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/hambat.lua#L98)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1340) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", nil, {1.0, 0.5, 1.0}, true, -13, swap_data)`，浮水换 swap_ham_bat | 公共端；`SetBankSwapOnFloat(true, -13, swap_data)` | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/hambat.lua#L62-L63)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| API／数据：`show_spoilage` 标签 | 物品栏 itemtile 按标签显示腐坏度小部件 | 客户端 UI，物品在栏内时 | [消费](../../../raw/current/scripts/widgets/itemtile.lua#L737) |
| API／数据：`icebox_valid` 标签 | 冰箱 `itemtestfn` 直接放行；入带 `fridge` 标签的容器（冰箱、保鲜背包）后走 `fridge` 分支 ×`PERISH_FRIDGE_MULT`=0.5 | 放入判定与腐坏速率 | [itemtestfn](../../../raw/current/scripts/containers.lua#L1019-L1022)、[fridge 分支](../../../raw/current/scripts/components/perishable.lua#L81-L86)、[值](../../../raw/current/scripts/tuning.lua#L2079)、[icebox 挂 fridge 标签](../../../raw/current/scripts/prefabs/icebox.lua#L59) |
| 关联 Prefab：spoiled_food | `onperishreplacement`：腐坏瞬间 SpawnPrefab 替换；优先按原格转移（`GiveItem(goop, slot)`）；装备中 `GetItemSlot` 只查 itemslots 查不到，slot 传 nil 时由 `GiveItem` 自动找可用槽（可并入同类堆叠），找不到合适位置按 `GiveItem` 回退 | 腐坏时，主模拟 | [替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)、[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263)、[GetItemSlot](../../../raw/current/scripts/components/inventory.lua#L592-L598)、[空槽回退](../../../raw/current/scripts/components/inventory.lua#L1094-L1096) |
| 皮肤 | `hambat_nautical`（TWITCH_DROP）／`hambat_potroast`／`hambat_spiralcut` 共 3 种，统一 `basic_init_fn(inst, build, "ham_bat")` | 装备/卸装时推事件并换持握符号 | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6295-L6325)、[init](../../../raw/current/scripts/prefabskin.lua#L190-L191)、[gifts](../../../raw/current/scripts/skin_gifts.lua#L40) |
| 资源：anim／swap 动画包 | ham_bat.zip（bank＋build）／swap_ham_bat.zip（持握符号） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/hambat.lua#L1-L5) |

## 典型接入

### 调整火腿棒的新鲜度衰减曲线

- 前提与选择：服务端 `modmain.lua`；沿用原 perishable＋weapon 组装，只替换 `SetOnAttack` 的更新逻辑（原 `UpdateDamage` 是文件内局部函数，无法直接改）。改腐坏速率另见 [[reference/components/perishable]]。
- 生效与撤销：postinit 影响新实例；读档（`inst.OnLoad`）与装备/卸装仍会按原公式重算一次快照——只换 `SetOnAttack` 时这三处仍走原 50% 下限；示例只改攻击后的快照，首次装备或读档后的第一击仍按原下限，要完全一致须同时包装 `OnEquip`／`OnUnequip` 与 `inst.OnLoad`。
- 原版依据：公式与 Remap 无钳制见[注册与构造](#注册与构造)；结算顺序（先 CalcDamage 后 OnAttack）见上文更新时机。

```lua
-- modmain.lua：把火腿棒伤害下限从 50% 提到 80%（0 新鲜度仍有 47.6 伤害）
AddPrefabPostInit("hambat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local weapon = inst.components.weapon
    if weapon ~= nil then
        weapon:SetOnAttack(function(inst)
            local perishable = inst.components.perishable
            if perishable ~= nil then
                local dmg = TUNING.HAMBAT_DAMAGE * perishable:GetPercent()
                dmg = GLOBAL.Remap(dmg, 0, TUNING.HAMBAT_DAMAGE,
                    0.8 * TUNING.HAMBAT_DAMAGE, TUNING.HAMBAT_DAMAGE)
                inst.components.weapon:SetDamage(dmg)
            end
        end)
    end
end)
```

- 调整与验证：单位为生命值；`TUNING` 已注入 mod 环境，`TheWorld`／`Remap` 须经 `GLOBAL` 取；下限系数沿同一 Remap 结构改，勿另设分段。未运行验证。

## 常见陷阱与限制

- **误区：DST 火腿棒有 100 次攻击耐久**：源码不挂 finiteuses，磨损分支判空跳过；`TUNING.HAMBAT_USES=100` 在全 scripts 中只有 tuning.lua 定义一处、无消费者（单机版遗留数值），不能当 DST 行为依据。[tuning.lua#L179](../../../raw/current/scripts/tuning.lua#L179)、[磨损判空](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **边界：有新鲜度不等于可食用**：edible 组件整段被注释（[hambat.lua#L85-L92](../../../raw/current/scripts/prefabs/hambat.lua#L85-L92)），运行时无进食路径；`show_spoilage`／`icebox_valid` 只驱动腐坏显示与冰箱准入，别据此当食物喂人。
- **边界：腐坏是"整体替换"不是"永久低伤害形态"**：0 新鲜度时实体被 Remove 并换成 spoiled_food（原格转移）；长存手段有冰箱／保鲜背包的 ×0.5 档、季节与湿度乘区，或改腐坏时长，装备中同样计时。[替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)
- **边界：伤害快照滞后一击**：`DoAttack` 先 `CalcDamage` 后 `OnAttack`，首击伤害取装备时快照；连续攻击第二击起才反映当前新鲜度。[结算顺序](../../../raw/current/scripts/components/combat.lua#L1212-L1224)
- **边界：装备中腐坏时替换物不在手上**：perishable 用 `Inventory:GetItemSlot`（只查 itemslots）定位原格；装备中查不到，slot 传 nil 由 `GiveItem` 自动找可用槽（可并入同类堆叠），手部装备槽清空。[GetItemSlot](../../../raw/current/scripts/components/inventory.lua#L592-L598)、[空槽回退](../../../raw/current/scripts/components/inventory.lua#L1094-L1096)
- **边界：`weapon` 标签是公共端手打的 pristine 镜像**（客户端优化用）；同文件无 `GetServerGameMode` 调用，quagmire 下组件照常挂载。[注释](../../../raw/current/scripts/prefabs/hambat.lua#L59-L60)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 OnLoad 漏项、fridge 倍率叠加与非冰箱容器、替换落点）。早前 2026-11-06，Steam buildid 26335394。核验 prefabs/hambat.lua 全文（fn 组装、UpdateDamage、onequip/onunequip、edible 注释块）；weapon/perishable 关键方法（GetPercent、Update 修正链、Perish 替换）；equippable 默认 HANDS；combat DoAttack 结算顺序；Remap 定义；forcecompostable→compostingbin 消费；icebox_valid／show_spoilage 消费端；配方行与原料 .po 名；boons→worldgen 放置链；`HAMBAT_USES` 全 scripts grep 仅 tuning.lua 定义一处；3 种皮肤注册与 gifts 来源；名称键以 chinese_s.po `msgctxt` 定向核验；scrapbookdata 的 59.5／4800 与源码推导互证。
- 游戏运行：未运行。未覆盖：实机腐坏节奏与冰箱／季节修正、装备中腐坏的替换落点、boon 实际落点与频率、皮肤实装外观；59.5／29.75／44.625／4800 均为源码推导，非运行测量。
- 结构检查：validate_page.py 0 error（129 行／17337 字节，standard 档）与 validate_source_links.py --strict（74 条源码引用、25 个源文件，0 error 0 warning）实际执行通过；两个 lua 围栏语法完整。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
