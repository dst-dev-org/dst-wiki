---
title: "铲子与黄金铲子（shovel／goldenshovel）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "MakeShovel 工厂两注册的基础挖掘工具族：DIG 工具与弱武器、耐久消耗（金铲折算）、HANDS 装备持握与皮肤接线、耗尽即移除的组装入口。"
aliases: ["shovel", "goldenshovel", "铲子", "黄金铲子", "Shovel", "Regal Shovel", "金铲", "挖掘工具", "铲子挖坑"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/shovel.lua", "components/equippable.lua", "components/tool.lua", "components/weapon.lua", "components/finiteuses.lua", "actions.lua", "componentactions.lua", "bufferedaction.lua", "entityscript.lua", "standardcomponents.lua", "modutil.lua", "mainfunctions.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 铲子与黄金铲子（shovel／goldenshovel）

## 快速判断

- 职责与适用：原版基础挖掘工具族，`prefabs/shovel.lua` 的单一工厂 `MakeShovel` 注册 `shovel` 与 `goldenshovel`；两者都是可拾取的 HANDS 装备，`tool` 组件提供 DIG，非 quagmire 模式另配耐久与武器伤害。适合借鉴"一工厂＋data＋postinit"的工具组装与折算耐久模式。[工厂](../../../raw/current/scripts/prefabs/shovel.lua#L32-L127)、[注册](../../../raw/current/scripts/prefabs/shovel.lua#L249-L251)
- 优先使用：改耐久／伤害／单次消耗按"关联机制导航"查组件页；新建同类手持工具查 [[cookbook/equipment/hand]]；查两注册差异与中文名读"注册与构造"。
- 重要副作用：DIG 成功后统一经 `BufferedAction:Do` 按 `SetConsumption` 扣耐久，耗尽执行 `inst.Remove` 并让持有者收到 `toolbroke`；攻击每次按 `attackwear` 磨损（普通 1、金铲 1/4）。装备／卸装回调维护持握外观并推送皮肤事件。[扣次链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)
- 修改边界：公共端只有实体基础、标签与漂浮，组件装配在主模拟；quagmire 服务端游戏模式跳过 finiteuses／weapon。本页不覆盖合成配方与获取途径、workable 目标端细节（链接组件页）、皮肤定义端（prefabskin／skinprefabs）与引擎侧 inherent actions 读取。[端分界](../../../raw/current/scripts/prefabs/shovel.lua#L85-L115)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shovel` | `MakeShovel("shovel", nil, nil, nil, assets)`：data 为 nil 走全默认——bank／build `shovel`、anim `idle`、swap_build／sym_name `swap_shovel`、`maxfiniteuses=TUNING.SHOVEL_USES` | 铲子本体；无 common／master postinit，行为全部来自共同初始化。[调用](../../../raw/current/scripts/prefabs/shovel.lua#L249)、[默认值](../../../raw/current/scripts/prefabs/shovel.lua#L33-L41) |
| `goldenshovel` | `MakeShovel("goldenshovel", nil, gold_master_postinit, GOLD_DATA, gold_assets)`：共同初始化后追加 master postinit | GOLD_DATA 改 bank／build `goldenshovel`、swap_build／sym_name `swap_goldenshovel`（anim 仍默认 `idle`）；postinit 把 DIG 消耗与攻击磨损折算为 1/`TUNING.GOLDENTOOLFACTOR`，并接装备金音。[GOLD_DATA](../../../raw/current/scripts/prefabs/shovel.lua#L131-L137)、[postinit](../../../raw/current/scripts/prefabs/shovel.lua#L145-L150) |

名称键核验：`STRINGS.NAMES.SHOVEL`＝"铲子"（Shovel）、`STRINGS.NAMES.GOLDENSHOVEL`＝"黄金铲子"（Regal Shovel），均出自本快照 `languages/chinese_s.po`。[SHOVEL](../../../raw/current/scripts/languages/chinese_s.po#L349334-L349337)、[GOLDENSHOVEL](../../../raw/current/scripts/languages/chinese_s.po#L343369-L343372)

工厂共同初始化顺序：实体基础＋`MakeInventoryPhysics` → 动画 → 公共端 `AddTag("tool")`、非 quagmire 加 `AddTag("weapon")`、`MakeInventoryFloatable` → `SetPristine` 端分界（客户端到此返回）→ 主模拟 inspectable／inventoryitem／tool →（非 quagmire）finiteuses／weapon → `AddInherentAction(ACTIONS.DIG)` → equippable → `MakeHauntableLaunch` → master postinit。[fn 全文](../../../raw/current/scripts/prefabs/shovel.lua#L57-L124)

> 同文件 `prefabs/shovel.lua` 还注册 `fumaroleshovel`（气孔铲：共用工厂＋fumaroletool 机制与分档外观，官方名"呲啦铲"）；本页不展开，机制见 [[reference/prefabs/fumarole-tools]]。[同文件注册](../../../raw/current/scripts/prefabs/shovel.lua#L251)、[名称键](../../../raw/current/scripts/languages/chinese_s.po#L343074-L343077)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`tool` | `SetAction(ACTIONS.DIG)`，效率取默认 1，并加 `DIG_tool` 标签 | 两注册同；quagmire 也添加 | [[reference/components/tool#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shovel.lua#L94-L95)、[标签写入](../../../raw/current/scripts/components/tool.lua#L41-L45) |
| 组件：`finiteuses` | `SetMaxUses`／`SetUses(maxfiniteuses)`、`SetOnFinished(inst.Remove)`、`SetConsumption(ACTIONS.DIG, 1)` | 仅非 quagmire；总量默认 `TUNING.SHOVEL_USES`=25，金铲 postinit 改 DIG 消耗 1/4 | [[reference/components/finiteuses#关键接口]]；[配置](../../../raw/current/scripts/prefabs/shovel.lua#L98-L104)、[折算](../../../raw/current/scripts/prefabs/shovel.lua#L145-L150) |
| 组件：`weapon` | `SetDamage(TUNING.SHOVEL_DAMAGE)`；攻击扣次走 `attackwear`（组件默认 1，金铲 1/4） | 仅非 quagmire | [[reference/components/weapon#关键接口]]；[配置](../../../raw/current/scripts/prefabs/shovel.lua#L107-L108)、[攻击磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：`equippable` | `SetOnEquip`／`SetOnUnequip` 接持握与皮肤回调；槽位为组件默认 `EQUIPSLOTS.HANDS` | 两注册同（默认回调，未被 data 覆盖） | [[reference/components/equippable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shovel.lua#L113-L115)、[默认槽](../../../raw/current/scripts/components/equippable.lua#L29) |
| 组件：`inspectable`／`inventoryitem` | 主模拟挂载，检视与拾取 | 两注册同 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shovel.lua#L91-L92) |
| helper：漂浮与作祟 | 公共端 `MakeInventoryFloatable("med", 0.05, {0.8,0.4,0.8}, true, 7, swap_data)`；主模拟 `MakeHauntableLaunch` | swap_data 用各注册的 swap_build／sym_name，漂浮换装与持握一致 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/shovel.lua#L79)、[作祟](../../../raw/current/scripts/prefabs/shovel.lua#L117)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 动作：`ACTIONS.DIG` | DIG 是 TOOLACTIONS 成员（`rmb=true, invalid_hold_action=true`）；`fn` 走 `DoToolWork`，要求目标 `workable` 的工作动作匹配并按 tool 效率结算；成功后 `BufferedAction:Do` 调 `invobject:OnUsedAsItem` 扣次 | 目标端资格归 workable；Lua 侧动作提供走 `DIG_tool` 标签处理器（USEITEM／EQUIPPED 两处） | [[reference/components/workable#关键接口]]；[DIG 定义](../../../raw/current/scripts/actions.lua#L369)、[fn 与 validfn](../../../raw/current/scripts/actions.lua#L1793-L1804)、[DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1738)、[TOOLACTIONS](../../../raw/current/scripts/constants.lua#L2187-L2196)、[EQUIPPED 处理器](../../../raw/current/scripts/componentactions.lua#L2507-L2520)、[USEITEM 处理器](../../../raw/current/scripts/componentactions.lua#L1845-L1857)、[扣次入口](../../../raw/current/scripts/bufferedaction.lua#L22-L37) |
| API／登记：`AddInherentAction(ACTIONS.DIG)` | 把 DIG 登记为物品内置动作（写入 `inst.inherentactions`）并经 net_bytearray 同步客户端还原 | 登记、同步与 `EntityScript:CanDoAction` 的 Lua 消费均已核验（先查 `inherentactions[action]`，再查 `<action>_tool` 标签与持有者物品）；引擎侧动作扫描仍另有一层 | [登记](../../../raw/current/scripts/entityscript.lua#L419-L433)、[同步与还原](../../../raw/current/scripts/entityscript.lua#L101-L137)、[net 字段](../../../raw/current/scripts/entityscript.lua#L182)、[CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1835) |
| API／数据：`TUNING` | `SHOVEL_USES`=25；`SHOVEL_DAMAGE`=`wilson_attack*.5`，wilson_attack=34×`multiplayer_attack_modifier`(1)→17（源码推导值）；`GOLDENTOOLFACTOR`=4×`multiplayer_goldentool_modifier`(1)→4 | 构造时读取；基数与被注释的调档值同段 | [SHOVEL_USES](../../../raw/current/scripts/tuning.lua#L159)、[SHOVEL_DAMAGE](../../../raw/current/scripts/tuning.lua#L363)、[基数](../../../raw/current/scripts/tuning.lua#L31-L37)、[GOLDENTOOLFACTOR](../../../raw/current/scripts/tuning.lua#L95) |
| 资源：动画 | shovel：`shovel.zip`／`swap_shovel.zip`；goldenshovel：`goldenshovel.zip`／`swap_goldenshovel.zip` | 声明即使用：SetBank／SetBuild／swap 持握 | [assets](../../../raw/current/scripts/prefabs/shovel.lua#L1-L11)、[使用](../../../raw/current/scripts/prefabs/shovel.lua#L67-L69) |

### 皮肤接线与持握

默认装备回调（`data.onequipfn` 可覆盖，两注册都用默认）按 `inst:GetSkinBuild()` 分支：有皮肤时向 owner 推送 `equipskinneditem` 并 `OverrideItemSkinSymbol`（皮肤 build，`swap_build` 兜底）；无皮肤时 `OverrideSymbol("swap_object", swap_build, sym_name)`；两分支都 Show `ARM_carry`／Hide `ARM_normal`。默认卸装回调恢复手臂并对皮肤推送 `unequipskinneditem`。[装备](../../../raw/current/scripts/prefabs/shovel.lua#L43-L55)、[卸装](../../../raw/current/scripts/prefabs/shovel.lua#L21-L28)。金铲另监听 equippable 在装备时推送的 `equipped` 事件，在 owner 播 `dontstarve/wilson/equip_item_gold`。[金音](../../../raw/current/scripts/prefabs/shovel.lua#L139-L150)、[事件源](../../../raw/current/scripts/components/equippable.lua#L102)。皮肤实体定义与事件消费端不在本页核验范围。

## 典型接入

### 调整铲子耐久总量或每次消耗

- 前提与选择：modmain 用 `AddPrefabPostInit`（登记 [modutil.lua](../../../raw/current/scripts/modutil.lua#L593-L599)），回调在实体构造完成后、生成时执行（[调用点](../../../raw/current/scripts/mainfunctions.lua#L394-L411)），因此晚于 master postinit。改总量用 `SetMaxUses`＋`SetUses`（`SetMaxUses` 不重置当前值）；改单次消耗用 `SetConsumption`——金铲的折算即后者。
- 生效与撤销：当前世界已生成实体不追溯；读档重建会先走本 postinit，再按 `OnLoad` 恢复存档当前使用数。满耐久通常不写 `uses`，读档保留本次构造/PostInit 的初始值，只有存档写有 `uses` 时才覆盖当前耐久；上限/消耗设置仍会作用于读档对象。停用后新实例回默认。耗尽仍走 `inst.Remove`。[保存条件](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)、[读档恢复](../../../raw/current/scripts/components/finiteuses.lua#L54-L58)
- 原版依据：`shovel` 用默认 `maxfiniteuses=TUNING.SHOVEL_USES`；goldenshovel 的消耗 1/4 与磨损 1/4 是同接口的折算用法。[折算](../../../raw/current/scripts/prefabs/shovel.lua#L145-L150)

```lua
-- modmain.lua（模组环境，AddPrefabPostInit 由 mod env 提供）
AddPrefabPostInit("shovel", function(inst)
    if inst.components.finiteuses ~= nil then
        -- 方案一：总量 25 -> 50（SetMaxUses 不改当前值，须一并 SetUses）
        inst.components.finiteuses:SetMaxUses(50)
        inst.components.finiteuses:SetUses(50)
        -- 方案二：折算单次消耗（如金铲 1/4 那样）：0.5 点/次，50 点约 100 次
        -- inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.DIG, 0.5)
    end
end)
```

- 调整与验证：消耗允许小数（`Use` 直接相减），实际次数＝总量÷单次消耗；mod postinit 晚于工厂与 master postinit（金铲折算已先执行），对 `goldenshovel` 直接 `SetConsumption` 即整体覆盖折算值而非叠加；同类总次数改法见 [[reference/components/finiteuses#典型接入]]。

## 常见陷阱与限制

- **误区：金铲只能挖 25 次**：总量是 25"点"，DIG 每次消耗 1/`TUNING.GOLDENTOOLFACTOR`=1/4，约 100 次；攻击磨损同为 1/4（普通铲每击 1，约 25 击耗尽）。[折算](../../../raw/current/scripts/prefabs/shovel.lua#L145-L150)、[磨损取值](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **quagmire 分支**：`TheNet:GetServerGameMode() == "quagmire"` 时两注册都不加 finiteuses／weapon，公共端也不加 weapon 标签；此时构造 `goldenshovel` 会因 gold postinit 直接索引 `inst.components.finiteuses` 报错（原版代码未判空），是否有 quagmire 生成点未核验，模组接入须判空。[分支一](../../../raw/current/scripts/prefabs/shovel.lua#L74-L77)、[分支二](../../../raw/current/scripts/prefabs/shovel.lua#L97-L109)
- **耗尽即移除**：`SetOnFinished(inst.Remove)`，当前值降到 ≤0 且由正转非正时执行，物品直接消失并带 `usesdepleted` 标签；持有者另收 `toolbroke`（tool 组件监听 `percentusedchange`，无 rechargeable 时）。移除与进行中动作的时序未实测。[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)、[toolbroke](../../../raw/current/scripts/components/tool.lua#L1-L9)
- **AddInherentAction 不是 DIG 资格的唯一天然来源**：客户端 Lua 的动作提供走 `DIG_tool` 标签处理器；服务端 `EntityScript:CanDoAction` 也会先查 `inherentactions`，两者是不同入口，不能因为看到标签处理器就断言数组无 Lua 消费。[处理器](../../../raw/current/scripts/componentactions.lua#L2507-L2520)、[登记](../../../raw/current/scripts/entityscript.lua#L419-L433)、[CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1835)
- **TUNING 读取时机**：三个键在实体构造时读取，改 tuning 只影响此后生成的物品；`GOLDENTOOLFACTOR` 与被注释的调档基数同段，改基数按 4×基数直接改变折算值。[基数](../../../raw/current/scripts/tuning.lua#L31-L32)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid 26335394（`raw/current/BUILD.txt`）。完整读取 `prefabs/shovel.lua`；核验两注册参数、GOLD_DATA、gold postinit、默认装备／卸装与皮肤分支、quagmire 门禁、floater／hauntable／physics helper 调用与实现；核验 equippable 默认 HANDS 与 `equipped` 事件、tool `SetAction`／标签／`toolbroke`、weapon `SetDamage`／`attackwear` 扣次、finiteuses 消耗与耗尽回调、DIG 定义与 `DoToolWork`、componentactions 两处 tool 处理器、bufferedaction 扣次入口、entityscript 登记与同步、modutil 登记＋mainfunctions 调用的 postinit 时序、tuning 三键及基数、`.po` 两个名称键。
- 游戏运行：未运行。未覆盖：皮肤定义端（prefabskin／skinprefabs）与皮肤事件消费端、合成配方与获取途径、workable 目标端细节（链接组件页）、引擎侧 inherent actions 读取、作祟与漂浮实际表现、quagmire 实际生成点。
- 结构检查：validate_page.py 与 validate_source_links.py 已实际运行，无 error（详见交付说明）。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
