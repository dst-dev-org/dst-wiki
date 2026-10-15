---
title: "木头短剑（cutless）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版粉猴手持武器 cutless（官方简中名木头短剑）：伤害 27.2、耐久 150 仅攻击磨损；每次命中从目标物品栏或容器敲落一件无 nosteal 物品并随机弹飞（起点分别为目标位置与容器坐标）；可作 BURNABLE 燃料（45）；无配方、仅粉猴生态获取。"
aliases: ["cutless", "Cutless", "木头短剑", "弯刀"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/cutless.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/thief.lua", "components/container.lua", "components/fuel.lua", "components/fueled.lua", "components/inventory.lua", "components/combat.lua", "components/piratespawner.lua", "prefabs/monkeyhut.lua", "prefabs/powdermonkey.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "mods.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 木头短剑（cutless）

## 快速判断

- 职责与适用：`prefabs/cutless.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备），粉猴海盗的木剑：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽＋fuel 燃料，特色是命中从目标物品栏敲落物品；无 tool、无 SG／Brain、无配方，同文件无 quagmire 分支。[源文件](../../../raw/current/scripts/prefabs/cutless.lua#L31-L87)
- 优先使用：改伤害／耐久／燃料查[数值与获取](#数值与获取)；复刻"命中敲落物品"读[命中敲落机制](#命中敲落机制)与[典型接入](#典型接入)；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：每次攻击命中 `thief:StealItem(target)`——两分支都会随机弹飞：目标有物品栏时敲落第一件无 `nosteal` 物品，从目标位置起弹；目标是容器时从容器取出，以容器坐标为起点弹飞；目标收 `onitemstolen` 事件（容器无可偷物时仍推 `item=nil` 但返回 false）。耐久耗尽 `SetOnFinished(inst.Remove)` 整个实体移除。[onattack](../../../raw/current/scripts/prefabs/cutless.lua#L27-L29)、[StealItem](../../../raw/current/scripts/components/thief.lua#L15-L58)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/cutless.lua#L45-L54)）；装备回调有皮肤分支但本 build 无 cutless 皮肤注册（见[陷阱](#常见陷阱与限制)）；不覆盖粉猴持有端行为（见[数值与获取](#数值与获取)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cutless` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；无皮肤注册。[注册](../../../raw/current/scripts/prefabs/cutless.lua#L87) |

共同初始化（[cutless.lua#L31-L85](../../../raw/current/scripts/prefabs/cutless.lua#L31-L85)）：实体组件＋`MakeInventoryPhysics` → `SetBank("cutless")`＋`SetBuild("cutless")`＋`PlayAnimation("idle")` → 公共端标签 `weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/cutless.lua#L44-L45)）→ `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -18, swap_data)` → `SetPristine`（非主模拟返回）→ 主模拟依次 thief → weapon → fuel → finiteuses → inspectable → inventoryitem → equippable → `MakeHauntableLaunch`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称经简中 .po `msgctxt` 定向核验：`STRINGS.NAMES.CUTLESS`＝木头短剑（Cutless）；通用检视语＝"虽然它切不了东西，但是它木刺多呀。"；图鉴特性行＝"把物品从目标的物品栏中打出来。"。英文名 Cutless 谐音 Cutlass（弯刀），"弯刀"是社区俗称，官方简中名不是弯刀。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L341949-L341952)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L11654-L11657)、[.po 图鉴](../../../raw/current/scripts/languages/chinese_s.po#L361449-L361452)

### 命中敲落机制

服务端执行序：combat 结算调用 `weapon:OnAttack`（[combat.lua#L1223-L1225](../../../raw/current/scripts/components/combat.lua#L1223-L1225)）→ 先执行 onattack 回调（[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）→ 再按 1 耐久点扣磨损。onattack 只有一行 `inst.components.thief:StealItem(target)`（[cutless.lua#L27-L29](../../../raw/current/scripts/prefabs/cutless.lua#L27-L29)），`StealItem` 流程（[thief.lua#L15-L58](../../../raw/current/scripts/components/thief.lua#L15-L58)）：

1. 目标须有 inventory 且非 `IsThiefProof`（[inventory.lua#L262-L264](../../../raw/current/scripts/components/inventory.lua#L262-L264)），或有 container 且非 thiefproof；都不满足直接返回 false，本击伤害与磨损照常。
2. **物品栏分支**：`FindItem` 取第一件无 `nosteal` 的物品（[thief.lua#L11-L13](../../../raw/current/scripts/components/thief.lua#L11-L13)、[L21](../../../raw/current/scripts/components/thief.lua#L21)），命中后 `DropItem(item, false, direction)`——第三形参名义为 `randomdir`，方向向量只作真值判断，`DoDropPhysics` 对 truthy 值改用随机角度弹飞（[L28-L29](../../../raw/current/scripts/components/thief.lua#L28-L29)、[inventoryitem.lua#L414-L428](../../../raw/current/scripts/components/inventoryitem.lua#L414-L428)）；推 `onitemstolen`（[L33](../../../raw/current/scripts/components/thief.lua#L33)）。找不到物品返回 false、不推事件。
3. **容器分支**：同样按 `nosteal` 选物，但走 `container:DropItem(item)`；`DropItemAt` 在容器坐标调用 `OnDropped(true)`，同样按随机角度弹飞，差别只是起点（[thief.lua#L38-L50](../../../raw/current/scripts/components/thief.lua#L38-L50)、[container.lua#L256-L262](../../../raw/current/scripts/components/container.lua#L256-L262)）；容器无可偷物时 `DropItemAt(nil)` 直接返回，`StealItem` 仍推 `onitemstolen(item=nil)` 再返回 false（[container.lua#L236-L252](../../../raw/current/scripts/components/container.lua#L236-L252)）。
4. 调用不传 `attack`，不触发额外 `DoAttack`；cutless 未设 `SetOnStolenFn`，无附加回调。

### 数值与获取

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.CUTLESS_DAMAGE`＝`wilson_attack`×0.8 | 27.2 生命值（源码推导，[tuning.lua#L6705](../../../raw/current/scripts/tuning.lua#L6705)） |
| 耐久上限 | `TUNING.CUTLESS_USES` | 150（[tuning.lua#L6706](../../../raw/current/scripts/tuning.lua#L6706)） |
| 燃料值 | `TUNING.MED_FUEL`＝`seg_time`×1.5 | 45（fuel 组件 `fuelvalue`，`seg_time`＝30 秒；经 `Fueled:TakeFuelItem` 按该值 DoDelta 进宿主 fueled 燃料量，[tuning.lua#L1675](../../../raw/current/scripts/tuning.lua#L1675)、[消费](../../../raw/current/scripts/components/fueled.lua#L210-L212)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L114)） |

获取途径（配方：全 scripts grep recipes.lua "cutless／cutlass" 零命中，无配方）：

| 途径 | 证据 |
|---|---|
| 海盗船粉猴船员 | piratespawner 生成 `powder_monkey` 后 `SpawnPrefab("cutless")`、加 `personal_possession` 标签并直接装备 | [装备链](../../../raw/current/scripts/components/piratespawner.lua#L344-L347) |
| 猴子小屋子猴 | 每只生成子猴 `give_child_gear(child, "cutless")`（30% 概率另给帽） | [giver](../../../raw/current/scripts/prefabs/monkeyhut.lua#L68-L74)、[调用](../../../raw/current/scripts/prefabs/monkeyhut.lua#L87) |
| 击杀粉猴掉落 | 粉猴 inventory 默认 `dropondeath=true`，死亡事件 `DropEverything(true)` 掉落全部物品，含装备中的 cutless | [OnDeath](../../../raw/current/scripts/components/inventory.lua#L1-L6)、[dropondeath](../../../raw/current/scripts/components/inventory.lua#L59-L60)、[DropEverything](../../../raw/current/scripts/components/inventory.lua#L1873-L1912)、[粉猴挂 inventory](../../../raw/current/scripts/prefabs/powdermonkey.lua#L274) |
| 数据依赖（非生成点） | powdermonkey `prefabs` 声明、scrapbook 图鉴（`weapondamage=27.2`、`fuelvalue=45` 旁证）、prefablist／debugcommands 清单 | [声明](../../../raw/current/scripts/prefabs/powdermonkey.lua#L18)、[图鉴数据](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L379) |

`personal_possession` 是获取链加在实例上的私产标记（非 cutless 构造行为）：粉猴脑清点战利品与回巢藏物时排除（[count_loot](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L373)、[is_stashable_item](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L495)），boatcrew 清点海盗战利品同样排除（[boatcrew.lua#L90](../../../raw/current/scripts/components/boatcrew.lua#L90)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(CUTLESS_DAMAGE)`＋`SetOnAttack(onattack)`；无射程／投射 | 主模拟；`OnAttack` 先回调后扣耐久，`attackwear` 缺省按 1 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cutless.lua#L58-L60) |
| 组件：thief | onattack 调 `StealItem`；仅作调用载体，未设 `SetOnStolenFn` | 主模拟；目标覆盖 inventory／container 两分支 | [[reference/components/thief#默认选物与两种掉落分支]]；[挂载](../../../raw/current/scripts/prefabs/cutless.lua#L56) |
| 组件：fuel | `fuelvalue = TUNING.MED_FUEL`＝45；fueltype 用默认 BURNABLE（挂 `burnable_fuel` 标签） | 主模拟；火堆等 fueled 宿主 `TakeFuelItem` 消费 | [[reference/components/fuel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cutless.lua#L64-L65)、[默认](../../../raw/current/scripts/components/fuel.lua#L10-L15) |
| 组件：finiteuses | 上限＝当前＝150；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cutless.lua#L69-L72)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖持握 `swap_object`、ARM 显隐 | 主模拟；槽默认 HANDS；皮肤事件在回调内，本 build 无皮肤注册 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/cutless.lua#L78-L80)、[onequip](../../../raw/current/scripts/prefabs/cutless.lua#L6-L16) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cutless.lua#L74-L76) |
| 组件：floater | `MakeInventoryFloatable("med", 0.05, {1.1, 0.5, 1.1}, true, -18, swap_data)` → `SetBankSwapOnFloat(true, -18, swap_data)` | 公共端；`swap_data` 指定持握符号 `swap_cutless` | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/cutless.lua#L47-L48)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟按 `HAUNT_CHANCE_ALWAYS` 弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`nosteal` 标签 | `StealItem` 选物排除条件 | StealItem 选物时读取 | [定义](../../../raw/current/scripts/components/thief.lua#L11-L13) |
| 关联 Prefab：powder_monkey（Wiki 待建） | 持有者：船员装备链与死亡掉落来源；非 cutless 工厂生成 | 生成／装备见获取途径；`personal_possession` 由外部加在实例上 | [组件挂载](../../../raw/current/scripts/prefabs/powdermonkey.lua#L274)、[掉落监听](../../../raw/current/scripts/components/inventory.lua#L59-L60) |
| 资源：anim／swap 动画包 | cutless.zip（bank `cutless`、build `cutless`），持握符号 `swap_cutless` | 静态声明 | [声明](../../../raw/current/scripts/prefabs/cutless.lua#L1-L4) |

## 典型接入

### 改写命中敲落行为

- 前提与选择：服务端 `modmain.lua`。`Weapon:SetOnAttack`（[weapon.lua#L53-L55](../../../raw/current/scripts/components/weapon.lua#L53-L55)）整体替换回调；modmain 环境只直挂 `TUNING` 等白名单、无 `__index=_G` 回退，`TheWorld` 须经 `GLOBAL.` 前缀（[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)）。
- 生效与撤销：postinit 只影响新实例；回调不存档，已有实例下次加载即用新回调；停用模组恢复原回调。
- 原版依据：原回调只调一次 `StealItem`（[cutless.lua#L27-L29](../../../raw/current/scripts/prefabs/cutless.lua#L27-L29)）；磨损结算在回调之后，与本改动独立（[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）。

```lua
-- modmain.lua：木头短剑命中改为最多敲落 3 件（原版 1 件）；伤害、耐久不变
AddPrefabPostInit("cutless", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetOnAttack(function(inst, attacker, target)
        local thief = inst.components.thief
        if thief ~= nil and target ~= nil and target:IsValid() then
            for _ = 1, 3 do
                if not thief:StealItem(target) then
                    break
                end
            end
        end
    end)
end)
```

- 调整与验证：`StealItem` 每次调用独立选物，无可偷物品返回 false 跳出；要整体禁用敲落把回调设为空函数或 nil 即可（`Weapon:OnAttack` 判空跳过回调，磨损照常，[weapon.lua#L106-L108](../../../raw/current/scripts/components/weapon.lua#L106-L108)）。

## 常见陷阱与限制

- **边界：两分支都随机弹飞，差别在起点**：物品栏分支的 `direction` 传给 `DropItem` 的 `randomdir` 形参只作真值判断，从目标位置随机弹飞；容器分支由 `DropItemAt` 在容器坐标 `OnDropped(true)`，从容器位置随机弹飞。捡拾位置要按来源区分。[thief.lua#L28-L29](../../../raw/current/scripts/components/thief.lua#L28-L29)、[DoDropPhysics](../../../raw/current/scripts/components/inventoryitem.lua#L414-L428)
- **边界：敲落失败不影响攻击**：目标 thiefproof、无 inventory／container 或无可偷物品时 `StealItem` 返回 false，本击伤害与磨损照常结算。[thief.lua#L15-L20](../../../raw/current/scripts/components/thief.lua#L15-L20)
- **边界：无配方**：recipes.lua 无 cutless／cutlass 命中，只能从粉猴生态获取（击杀掉落、或在目标被命中时捡拾弹飞物），见[数值与获取](#数值与获取)。
- **边界：皮肤分支为通用模板**：onequip／onunequip 有 `GetSkinBuild` 分支，但本 build 无 cutless 皮肤注册（prefabskin／skinprefabs 零命中），分支不触发。[onequip](../../../raw/current/scripts/prefabs/cutless.lua#L6-L16)
- **误区：官方中文名是"弯刀"**：.po 键 `STRINGS.NAMES.CUTLESS` 官方简中名为"木头短剑"；弯刀是社区俗称（英文名谐音 Cutlass）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L341949-L341952)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 thief 物品栏／容器两分支、空容器事件与两分支随机弹飞起点）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/cutless.lua 全文（单注册、onattack 敲落、装备皮肤接线、floater 参数、无 quagmire 分支）；weapon／finiteuses／fuel／fueled（TakeFuelItem）／thief（StealItem 两分支与选物）／inventory（OnDeath、DropEverything、DropItem、IsThiefProof）／inventoryitem（OnDropped、DoDropPhysics）／equippable（默认槽）／inspectable 关键方法与挂载顺序；`MakeInventoryFloatable`／`MakeHauntableLaunch` helper；tuning 四键与 `wilson_attack` 链；获取链 piratespawner／monkeyhut／powdermonkey 与 `personal_possession` 消费端（powdermonkeybrain、boatcrew）；recipes.lua 零命中确认无配方；全 scripts grep "cutless" 复核其余命中（debugcommands、prefablist、scrapbook 数据、skins_defs_data 图鉴键）均非新机制。名称键以 chinese_s.po `msgctxt` 定向核验。
- 游戏运行：未运行。未覆盖：27.2 伤害与 45 燃料值为源码推导（scrapbookdata 旁证），非运行测量；敲落弹飞实际表现、粉猴刷新与掉落频率未实测。
- 结构检查：validate_page.py（0 error，125 行／16795 字节，standard 档）与 validate_source_links.py --strict（66 条源码引用、21 个源文件，0 error 0 warning）实际执行通过；首跑 3 条 `source-blank` warning（两处引用起点落在空行）已改为实义行后复跑归零。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
