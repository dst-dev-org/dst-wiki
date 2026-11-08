---
title: "鹤嘴锄（pickaxe）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "原版基础挖矿工具两注册的 MakePickaxe 工厂组装：MINE 挖掘、耐久消耗、武器伤害与皮肤接线，可作手持工具族同文件多注册参照。"
aliases: ["pickaxe", "goldenpickaxe", "鹤嘴锄", "黄金鹤嘴锄", "Opulent Pickaxe", "镐", "挖矿工具"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pickaxe.lua", "components/equippable.lua", "components/tool.lua", "components/weapon.lua", "components/finiteuses.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "actions.lua", "bufferedaction.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 鹤嘴锄（pickaxe）

## 快速判断

- 职责与适用：原版基础挖矿工具两注册 `pickaxe`／`goldenpickaxe`，同用 `MakePickaxe` 工厂组装"可拾取 + 手部装备 + MINE 工具 + 武器 + 有限次数"的手持镐；金色变体同构，仅动画数据、每次消耗折算与装备音效不同。[工厂](../../../raw/current/scripts/prefabs/pickaxe.lua#L32-L122)、[两注册](../../../raw/current/scripts/prefabs/pickaxe.lua#L244-L246)
- 优先使用：改伤害／效率／耐久分别走 [[reference/components/weapon#关键接口]]、[[reference/components/tool#关键接口]]、[[reference/components/finiteuses#关键接口]]；借鉴"单文件多注册工具族"写法读下方注册与构造；改金工具折算见典型接入。
- 重要副作用：挖掘与武器攻击都扣耐久（金色攻击也按 1/4 扣）；次数耗尽即 Remove 实体；金色版装备时持有者播金工具音。[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)、[音效](../../../raw/current/scripts/prefabs/pickaxe.lua#L134-L145)
- 修改边界：服务端配置组件，公共端只设外观、标签与 floater；本页不覆盖同文件第三注册 fumarolepickaxe 的机制、皮肤清单与 workable 消费端细节。[端分界](../../../raw/current/scripts/prefabs/pickaxe.lua#L85-L89)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pickaxe` | `MakePickaxe("pickaxe", nil, nil, nil, assets)` → 公共实体／动画／标签 → SetPristine → 主模拟 inspectable、inventoryitem、equippable、weapon、tool、finiteuses、MakeHauntableLaunch；无 common／master postinit | 可拾取手持装备；bank／build `pickaxe`、swap `swap_pickaxe`。[注册](../../../raw/current/scripts/prefabs/pickaxe.lua#L244)、[主模拟段](../../../raw/current/scripts/prefabs/pickaxe.lua#L91-L112) |
| `goldenpickaxe` | 同一 `MakePickaxe`，common 仍 nil，master postinit 为 `gold_master_postinit`；数据 `GOLD_DATA` 换 bank／build `goldenpickaxe`、swap `swap_goldenpickaxe` | 折算：MINE 消耗与 weapon.attackwear 同设 `1/TUNING.GOLDENTOOLFACTOR`；监听 `equipped` 播金工具音。[数据](../../../raw/current/scripts/prefabs/pickaxe.lua#L126-L132)、[专属初始化](../../../raw/current/scripts/prefabs/pickaxe.lua#L140-L145) |

- 名称核验：`STRINGS.NAMES.PICKAXE`＝"鹤嘴锄"（[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L346849-L346852)）；`STRINGS.NAMES.GOLDENPICKAXE`＝"黄金鹤嘴锄"（英文 msgid 为 Opulent Pickaxe，[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L343359-L343362)）。
- 获取：`pickaxe` 配方树枝×2＋燧石×2（TECH.NONE），`goldenpickaxe` 树枝×4＋金块×2（TECH.SCIENCE_TWO）。[配方](../../../raw/current/scripts/recipes.lua#L395-L396)
- 同文件还有第三注册 `fumarolepickaxe`（气孔镐，[返回处](../../../raw/current/scripts/prefabs/pickaxe.lua#L244-L246)、专属数据与回调 [L126-L240](../../../raw/current/scripts/prefabs/pickaxe.lua#L173-L240)）：走同一工厂但有 fumaroletool 接线与破损／修复流程，机制归气孔工具族页 [[reference/prefabs/fumarole-tools]]（组件细节见 [[reference/components/fumaroletool]]），本页不展开。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | `SetOnEquip`／`SetOnUnequip` 挂默认持握回调；不改 `equipslot`，即默认 `EQUIPSLOTS.HANDS`（"hands"） | 两注册同；SetPristine 后仅主模拟挂载 | [[reference/components/equippable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/pickaxe.lua#L94-L96)、[默认槽位](../../../raw/current/scripts/components/equippable.lua#L29)、[槽位值](../../../raw/current/scripts/constants.lua#L654-L660) |
| 组件：tool | `SetAction(ACTIONS.MINE)`，效率缺省 1，并加 `MINE_tool` 标签 | 两注册同；`tool` 标签在公共端预置 | [[reference/components/tool#关键接口]]；[挂载与登记](../../../raw/current/scripts/prefabs/pickaxe.lua#L101-L102)、[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45) |
| 组件：weapon | `SetDamage(TUNING.PICK_DAMAGE)`；`attackwear` 保持默认（每次命中扣 1 次） | 金色在 master postinit 改 `attackwear＝1/4` | [[reference/components/weapon#关键接口]]；[伤害](../../../raw/current/scripts/prefabs/pickaxe.lua#L98-L99)、[攻击扣次](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | `SetMaxUses/SetUses(TUNING.PICKAXE_USES)`、`SetOnFinished(inst.Remove)`、`SetConsumption(ACTIONS.MINE, 1)` | 金色消耗改 `1/TUNING.GOLDENTOOLFACTOR`；`nofiniteuses` 数据可关（两注册均未用） | [[reference/components/finiteuses#关键接口]]；[配置](../../../raw/current/scripts/prefabs/pickaxe.lua#L104-L110)、[金折算](../../../raw/current/scripts/prefabs/pickaxe.lua#L141) |
| 消费链：动作与攻击 | 挖掘经 `ACTIONS.MINE.fn`→`DoToolWork` 读 tool 效率，动作成功后 `BufferedAction:Do` 调 `OnUsedAsItem` 按消耗表扣次；攻击在 `Weapon:OnAttack` 按 `attackwear` 扣次 | 服务端；efficientuser 倍率两条链分别生效 | [MINE 动作](../../../raw/current/scripts/actions.lua#L1767-L1778)、[DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1738)、[动作扣次入口](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[消耗表](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)、[攻击扣次](../../../raw/current/scripts/components/weapon.lua#L111-L120) |
| 组件：inspectable／inventoryitem | 基础检视与拾取，无专属配置 | 两注册同；主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/pickaxe.lua#L91-L92) |
| 组件：floater／hauntable | `MakeInventoryFloatable("med", 0.05, {0.75,0.4,0.75}, true, -11, swap 数据)` 漂浮换装；`MakeHauntableLaunch` 作祟弹开 | 两注册同（swap 数据按各自动画） | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/pickaxe.lua#L79)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：皮肤接线 | 默认 OnEquip 读 `GetSkinBuild`：有皮肤时在持有者上发 `equipskinneditem` 并 `OverrideItemSkinSymbol`，否则普通 `OverrideSymbol`；OnUnequip 发 `unequipskinneditem`；皮肤应用／还原图标由 prefabskin 的 `pickaxe_init_fn`／`goldenpickaxe_init_fn` 及对应 clear 走 `basic_init_fn`／`basic_clear_fn`（`_invisible` 皮肤关 floater 换装） | 两注册同一默认回调；皮肤注册（base_prefab／init_fn／skin_tags）在 `prefabs/skinprefabs.lua`；具体 build／图标名清单未逐项核验 | [持握与皮肤分支](../../../raw/current/scripts/prefabs/pickaxe.lua#L43-L55)、[卸装](../../../raw/current/scripts/prefabs/pickaxe.lua#L21-L28)、[init／clear 注册](../../../raw/current/scripts/prefabskin.lua#L925-L934)、[golden 版](../../../raw/current/scripts/prefabskin.lua#L958-L967)、[basic helper](../../../raw/current/scripts/prefabskin.lua#L106-L142) |
| API／数据：TUNING | `PICKAXE_USES＝33`；`PICK_DAMAGE＝wilson_attack×0.8`（`wilson_attack＝34×1`，当前值 27.2）；`GOLDENTOOLFACTOR＝4×1` | 服务端读取；修正常量即改全族 | [耐久](../../../raw/current/scripts/tuning.lua#L162)、[伤害](../../../raw/current/scripts/tuning.lua#L359)、[基数](../../../raw/current/scripts/tuning.lua#L31-L37)、[金折算](../../../raw/current/scripts/tuning.lua#L32)、[键](../../../raw/current/scripts/tuning.lua#L95) |
| 资源：动画 | `anim/pickaxe.zip`＋`swap_pickaxe.zip`；金版 `anim/goldenpickaxe.zip`＋`swap_goldenpickaxe.zip` | 注册时声明并实际用于 bank／build／swap | [assets](../../../raw/current/scripts/prefabs/pickaxe.lua#L1-L11) |

## 典型接入

### 同步修改金鹤嘴锄的挖掘与攻击消耗折算

- 前提与选择：放目标模组 `modmain.lua`；原版金工具把"MINE 消耗"与"攻击磨损"两处分别设为同一折算值，只改一处会导致挖矿与打怪扣耐久速度不一致。沿用原版组件，不新增资源。
- 生效与撤销：影响此模组下新的 `goldenpickaxe` 实例；折算分母为 2 时每次挖掘／攻击扣 0.5 次，33 次总耐久约可挖 66 次。停用后新实例恢复原版 `1/4` 折算。
- 原版依据：`gold_master_postinit` 同时设置消耗与 attackwear，并在同处挂装备音效监听；本例不覆盖音效与皮肤接线。[原版折算](../../../raw/current/scripts/prefabs/pickaxe.lua#L140-L145)

```lua
AddPrefabPostInit("goldenpickaxe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local factor = 2 -- 每次挖掘/攻击消耗 1/factor；原版为 TUNING.GOLDENTOOLFACTOR(4)
    if inst.components.finiteuses ~= nil then
        inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.MINE, 1 / factor)
    end
    if inst.components.weapon ~= nil then
        inst.components.weapon.attackwear = 1 / factor
    end
end)
```

- 调整与验证：`SetConsumption(MINE, x)` 会连带把 `REMOVELUNARBUILDUP` 消耗设为同一 x（组件联动，非本页行为）；`attackwear` 只作用于攻击扣次链。当前世界已生成实例不追溯；读档重建会先走本 postinit，再按 `OnLoad` 恢复存档当前使用数；满耐久通常不写 `uses`，读档保留本次构造/PostInit 的初始值，只有存档写有 `uses` 时才覆盖当前耐久。消耗/磨损设置会作用于读档对象。[联动](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)、[attackwear 消费](../../../raw/current/scripts/components/weapon.lua#L111-L120)、[保存条件](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)、[读档恢复](../../../raw/current/scripts/components/finiteuses.lua#L54-L58)

### 为镐族接入皮肤时的接线检查点

- 原版镐的皮肤路径分三段：皮肤实例的动画与图标由 prefabskin 的 init／clear helper 换 build／改图标名；装备到手上时走本文件默认 OnEquip 的 `GetSkinBuild` 分支（发 `equipskinneditem`＋`OverrideItemSkinSymbol`）；卸下发 `unequipskinneditem`。自建皮肤只需提供皮肤定义与对应 init／clear 接线，不必重写持握回调；若替换 OnEquip／OnUnequip，须保留皮肤分支，否则皮肤持握会退化为普通外观或丢事件。[分支](../../../raw/current/scripts/prefabs/pickaxe.lua#L43-L55)、[helper](../../../raw/current/scripts/prefabskin.lua#L106-L142)
- 皮肤注册可在 `prefabs/skinprefabs.lua` 核验：pickaxe 7 种（含共用动画 build、但独立注册的 `pickaxe_scythe` 与 `pickaxe_scythe_alt`）、goldenpickaxe 2 种；build／图标名到资源的对应仍未逐项核验。[pickaxe 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L9140-L9214)、[goldenpickaxe 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6102-L6122)

## 常见陷阱与限制

- **误区：金鹤嘴锄"耐久上限是 33×4＝132"**：`finiteuses` 总次数仍是 `PICKAXE_USES＝33`，折算发生在每次消耗（MINE 扣 0.25、攻击扣 0.25），约可执行 132 次挖掘；百分比按 33 总量显示。[总次数](../../../raw/current/scripts/prefabs/pickaxe.lua#L104-L110)、[折算](../../../raw/current/scripts/prefabs/pickaxe.lua#L141-L142)
- **误区：改了挖掘消耗就等于改了攻击磨损**：两条扣次链独立（动作消耗表 vs `weapon.attackwear`），原版金色变体是两处同步设置；只改其一会出现挖矿与打怪耐久速度不一致。[两条链](../../../raw/current/scripts/prefabs/pickaxe.lua#L141-L142)、[攻击侧](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **边界：weapon 组件没有 quagmire 分支**：`pickaxe.lua` 全文件无 `quagmire` 判断（与 axe.lua 的游戏模式分支不同）；移植镐族写法时不要把 axe 的模式分支照搬过来。[工厂全文](../../../raw/current/scripts/prefabs/pickaxe.lua#L32-L122)
- **边界：耗尽即消失并通知持有者**：`SetOnFinished(inst.Remove)` 使次数归零时直接删除实体（仅正转非正触发一次）；`tool` 组件另监听 `percentusedchange`，给持有者发 `toolbroke`。修复类组件未挂载，本族不可修理。[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)、[toolbroke](../../../raw/current/scripts/components/tool.lua#L1-L9)
- **边界：动作消耗的组件级联动**：`SetConsumption(ACTIONS.MINE, x)` 自动同设 `REMOVELUNARBUILDUP` 为 x，核对月岩坍落物清除消耗时按同一折算理解。[联动](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)

## 验证记录

- 源码审阅：2026-11-05，Steam buildid 26335394。覆盖 `MakePickaxe` 工厂全文与两注册（`pickaxe`／`goldenpickaxe`）、金色专属初始化（折算＋装备音效）、equippable 默认 HANDS、tool→MINE、weapon 伤害与 attackwear 消费、finiteuses 配置／耗尽／消耗联动、MINE 动作与动作扣次入口、floater／hauntable helper、皮肤接线（本文件回调＋prefabskin init／clear）、TUNING 三键取值、两注册配方与 `.po` 名称键。未覆盖：fumarolepickaxe 机制、皮肤 build/图标资源逐项清单、workable/workmultiplier 消费端细则、皮肤网络时序。
- 游戏运行：未运行；未验证实际持握外观、耐久消耗的逐帧表现与多人端表现。
- 结构检查：2026-11-05，`validate_page.py` 与 `validate_source_links.py`（--strict）通过。

相关：[[reference/prefabs/index|Prefab 索引]]；[[reference/components/fumaroletool|同文件气孔镐的温档组件]]。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
