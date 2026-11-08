---
title: "锤子（hammer）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "原版手持拆除工具 hammer：MakeHammer 工厂单注册的 HAMMER 工具兼 HANDS 槽武器，耐久 75、耗尽整个实体移除；可借鉴手持工具的组件组装与 HAMMER 动作接入。"
aliases: ["hammer", "锤子", "Hammer", "拆除建筑的工具", "拆建筑"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hammer.lua", "components/equippable.lua", "components/tool.lua", "components/weapon.lua", "components/finiteuses.lua", "actions.lua", "bufferedaction.lua", "entityscript.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "recipes.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "stategraphs/SGmole.lua", "componentactions.lua", "prefabs/pickaxe_lunarplant.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 锤子（hammer）

## 快速判断

- 职责与适用：`prefabs/hammer.lua` 用 `MakeHammer` 工厂注册的可拾取手持锤，对象类别为物品＋装备。原版职责是拆解可工作的建筑／设施（HAMMER 工具动作，目标侧消费端为 workable）并兼作 HANDS 槽近战武器；本页只覆盖 `hammer` 单注册。
- 优先使用：做"可拆解"手持工具或改耐久／伤害查[注册与构造](#注册与构造)数值表与 [有限耐久组件](../components/finiteuses.md)；从零造同类手持工具查 [[cookbook/equipment/hand]]；改拆解效率走 tool 组件 `SetAction` 的第二参（见[典型接入](#修改锤子的耐久与拆解效率)）。
- 重要副作用：装备回调维护 swap_object、手臂显隐与皮肤事件；`SetOnFinished(inst.Remove)` 耐久耗尽整个实体移除、无掉落；拆解与攻击两条路径都扣同一份耐久。[耗尽回调](../../../raw/current/scripts/prefabs/hammer.lua#L98-L104)
- 修改边界：标签与 floater 挂在公共端、组件挂在主模拟（[端分界](../../../raw/current/scripts/prefabs/hammer.lua#L79-L83)）；HAMMER 动作对目标何时生效在 workable 一侧，本页不复述；不覆盖同文件注册的 `fumarolehammer`（见[注册与构造](#注册与构造)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hammer` | `MakeHammer("hammer", nil, nil, nil, assets)`，data 为 nil 走全默认，common／master postinit 均无 | 静态资源 anim/hammer.zip＋swap_hammer.zip；皮肤经 `hammer_init_fn`。 [注册](../../../raw/current/scripts/prefabs/hammer.lua#L215)、[工厂](../../../raw/current/scripts/prefabs/hammer.lua#L26-L116) |

共同初始化（`fn`，[hammer.lua#L51-L113](../../../raw/current/scripts/prefabs/hammer.lua#L51-L113)）：实体组件与 `MakeInventoryPhysics` → `SetBank("hammer")`＋`SetBuild("swap_hammer")`＋`PlayAnimation("idle")`（build 默认是 `swap_hammer`，与 bank `"hammer"` 不同名，[默认值](../../../raw/current/scripts/prefabs/hammer.lua#L27-L34)）→ 公共端标签 `hammer`、`tool`、`weapon`（后两个是 tool／weapon 组件标签的 pristine 镜像，[源注释](../../../raw/current/scripts/prefabs/hammer.lua#L69-L73)）→ `MakeInventoryFloatable("med", 0.05, {0.7, 0.4, 0.7}, true, -13, swap_data)` → `common_postinit`（本注册为 nil）→ `SetPristine`（非主模拟返回）→ 主模拟依次 `inspectable`、`inventoryitem`、`equippable`（接线 `_OnEquip`／`OnUnequip`）、`weapon`（伤害 `TUNING.HAMMER_DAMAGE`）、`tool`（`SetAction(ACTIONS.HAMMER)`，效率取默认 1）→ `finiteuses`（上限＝当前＝`TUNING.HAMMER_USES`，耗尽 `inst.Remove`，HAMMER 消耗 1）→ `MakeHauntableLaunch` → `master_postinit`（nil）。data 还支持 `nofiniteuses`／`maxfiniteuses`／`onequipfn`／`onunequipfn`／bank／build／swap_build／sym_name／anim／floater_scale 覆盖项，本注册全部未用。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.HAMMER`=锤子（Hammer）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L343704-L343707)

数值（tuning 键＋实际值；`wilson_attack`=34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 耐久上限 | `TUNING.HAMMER_USES` | 75（[tuning.lua#L158](../../../raw/current/scripts/tuning.lua#L158)） |
| 每 HAMMER 动作消耗 | `SetConsumption(ACTIONS.HAMMER, 1)` | 1 耐久点（[挂载](../../../raw/current/scripts/prefabs/hammer.lua#L103)） |
| 武器伤害 | `TUNING.HAMMER_DAMAGE`=`wilson_attack`×0.5 | 17 生命值（源码推导，[tuning.lua#L362](../../../raw/current/scripts/tuning.lua#L362)） |
| 每次攻击磨损 | weapon `attackwear` 缺省（从未赋值） | 1 耐久点（[OnAttack](../../../raw/current/scripts/components/weapon.lua#L114-L119)） |

获取途径：配方 twigs 3＋rocks 3＋cutgrass 6，`TECH.NONE`（[recipes.lua#L401](../../../raw/current/scripts/recipes.lua#L401)）；按注册名全量 grep，scripts 内未检索到 `SpawnPrefab("hammer")` 的世界生成或掉落点。

同文件第二注册 `fumarolehammer`（气孔锤，走 FUMAROLE_DATA＋`MakeFumaroleTool` 温度机制）本页不覆盖，机制见 [[reference/prefabs/fumarole-tools]]（[注册行](../../../raw/current/scripts/prefabs/hammer.lua#L215-L216)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | `SetOnEquip(_OnEquip)`／`SetOnUnequip(OnUnequip)`：覆盖持握 swap_object、ARM_carry／ARM_normal 显隐 | 主模拟；皮肤 build 非空时推 `equipskinneditem`／`unequipskinneditem` 并用 `OverrideItemSkinSymbol` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/hammer.lua#L88-L90)、[_OnEquip](../../../raw/current/scripts/prefabs/hammer.lua#L37-L47)、[OnUnequip](../../../raw/current/scripts/prefabs/hammer.lua#L15-L22) |
| 组件：tool | `SetAction(ACTIONS.HAMMER)`：效率 1，加 `HAMMER_tool` 标签 | 主模拟；与公共端手打的 `hammer` 标签是两个标签 | [[reference/components/tool#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hammer.lua#L95-L96)、[SetAction 加标签](../../../raw/current/scripts/components/tool.lua#L41-L45) |
| 组件：weapon | 仅 `SetDamage(TUNING.HAMMER_DAMAGE)`，无 onattack／射程设置 | 主模拟；`OnAttack` 每击按 `attackwear or 1` 扣 finiteuses | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hammer.lua#L92-L93)、[OnAttack 磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | 上限＝当前＝`HAMMER_USES`；`SetOnFinished(inst.Remove)`；HAMMER 消耗 1 | 主模拟；经 `BufferedAction:Do` → `OnUsedAsItem` 结算 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hammer.lua#L98-L104)、[消耗链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[分发](../../../raw/current/scripts/entityscript.lua#L1811-L1816) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/hammer.lua#L85-L86) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", 0.05, {0.7, 0.4, 0.7}, true, -13, swap_data)`，浮水换 swap 贴图 | 公共端；invisible 皮肤会关 `do_bank_swap` | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/hammer.lua#L67)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟按概率弹飞（默认 HAUNT_CHANCE_ALWAYS） | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/hammer.lua#L106)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：HAMMER 动作与消耗链 | `ACTIONS.HAMMER.fn` → `DoToolWork`（tool 效率×worker／workmultiplier）→ `workable:WorkedBy_Internal`；动作成功后 `OnUsedAsItem` 扣耐久 | 服务端动作结算；目标侧 `SetWorkAction` 须为 HAMMER 才可拆 | [[reference/components/workable#关键接口]]；[动作定义](../../../raw/current/scripts/actions.lua#L429)、[DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1735)、[HAMMER.fn](../../../raw/current/scripts/actions.lua#L1780-L1791) |
| API／数据：`hammer` 标签消费端 | 对鼹鼠：允许选中攻击，且命中武器带该标签时掉落全部物品并进入眩晕；目标带 `whackable` 时 ATTACK 描述串切 "WHACK" 措辞 | 原版打此标签的有 `hammer`、pickaxe_lunarplant 与 hammer_mjolnir | [鼹鼠可攻击](../../../raw/current/scripts/componentactions.lua#L2553)、[命中眩晕](../../../raw/current/scripts/stategraphs/SGmole.lua#L32-L41)、[WHACK 措辞](../../../raw/current/scripts/actions.lua#L1995-L1996)、[另一打标](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L172) |
| 关联 Prefab：fumarolehammer | 同文件同工厂第二注册，走温度机制 | 本页不覆盖，见 [[reference/prefabs/fumarole-tools]] | [注册行](../../../raw/current/scripts/prefabs/hammer.lua#L215-L216) |
| 资源：anim／swap 动画包 | hammer.zip（bank `hammer`）／swap_hammer.zip（build 与持握符号 `swap_hammer`） | 按 data.bank／build／swap_build 对应 | [资源声明](../../../raw/current/scripts/prefabs/hammer.lua#L1-L5) |

皮肤（[skinprefabs.lua#L6327-L6384](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6327-L6384)）：hammer_crowbar、hammer_forge、hammer_hammush（hit 音效 terraria1/skins/hammush）、hammer_invisible、hammer_rw_startingforest 共 5 种；init 统一 `basic_init_fn(inst, build_name, "swap_hammer")`，invisible 另关 floater 换 bank（[hammer_init_fn](../../../raw/current/scripts/prefabskin.lua#L736-L748)）。

## 典型接入

### 修改锤子的耐久与拆解效率

- 前提与选择：服务端 `modmain.lua`；hammer 主模拟组件齐备且无专属机制，直接 `AddPrefabPostInit` 改组件数值，无需新资源。
- 生效与撤销：postinit 影响新构造实例；读档重建也会先走 postinit，再按 `OnSave`／`OnLoad` 恢复当前耐久。满耐久通常不写 `uses`，读档保留本次构造/PostInit 的初始值，只有存档写有 `uses` 时才覆盖当前耐久；上限/效率设置仍会作用于读档对象。停用模组后新建/重建回默认，已存使用数可能高于默认上限。[保存条件](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)、[读档恢复](../../../raw/current/scripts/components/finiteuses.lua#L54-L58)
- 原版依据：上限与消耗端独立（见[注册与构造](#注册与构造)数值表）；`SetAction` 重复调用覆盖效率、不打重复标签。[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45)

```lua
-- modmain.lua：锤子耐久改为 150，拆解效率翻倍（每次 HAMMER 结算 2 点工作量）
AddPrefabPostInit("hammer", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.finiteuses:SetMaxUses(150)
    inst.components.finiteuses:SetUses(150)
    inst.components.tool:SetAction(GLOBAL.ACTIONS.HAMMER, 2)
end)
```

- 调整与验证：耐久单位为点/次、效率单位为工作量/动作；更多上限改法见 [[reference/components/finiteuses#典型接入]]。

## 常见陷阱与限制

- **误区：耐久耗尽会掉残骸或变废品**：`SetOnFinished(inst.Remove)` 直接 Remove 整个实体，无掉落无替换物；"broken 形态"是 fumarolehammer 的机制，与 `hammer` 无关。[回调](../../../raw/current/scripts/prefabs/hammer.lua#L102)
- **边界：当武器攻击也扣耐久**：`Weapon:OnAttack` 每击 `Use(attackwear or 1)`，与 HAMMER 动作消耗独立；攻击与拆解共享同一份 75 点耐久。[OnAttack](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **边界：`hammer` 与 `HAMMER_tool` 是两个标签**：prefab 公共端手打 `hammer`（[L65](../../../raw/current/scripts/prefabs/hammer.lua#L65)），tool 组件 `SetAction` 打 `HAMMER_tool`（[tool.lua#L44](../../../raw/current/scripts/components/tool.lua#L44)）；按注册名推标签名会找不到消费端（`hammer` 标签消费端见[关联机制导航](#关联机制导航)）。
- **边界：build 默认 `swap_hammer`**：地面／背包贴图 build 与 bank（`hammer`）不同名，自定义变体从注册名推 `SetBuild("hammer")` 会拿错 build。[默认值](../../../raw/current/scripts/prefabs/hammer.lua#L27-L34)
- **边界：耗尽顺序**：`SetUses` 先推 `percentusedchange`（tool 组件监听，持有者存在时向其推 `toolbroke` 事件），再加 `usesdepleted` 标签并调用 Remove 回调。[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)、[toolbroke](../../../raw/current/scripts/components/tool.lua#L1-L9)

## 验证记录

- 源码审阅：2026-11-05，Steam buildid 26335394。核验 prefabs/hammer.lua 全文（MakeHammer 工厂默认值、`hammer` 单注册与 fumarolehammer 注册行、装备／皮肤事件接线、floater 参数）、equippable 默认 HANDS、tool／weapon／finiteuses 关键方法、tuning 两键、`MakeInventoryFloatable`／`MakeHauntableLaunch` helper、配方行、5 种皮肤注册与 init、`hammer` 标签三处消费端、HAMMER 动作 fn→`DoToolWork`→`WorkedBy_Internal` 及 `OnUsedAsItem` 耐久链。名称键以 chinese_s.po `msgctxt` 定向核验（`STRINGS.NAMES.HAMMER`）。
- 游戏运行：未运行。未覆盖：拆解工作量结算的运行时序、皮肤实装外观、鼹鼠交互实机表现；17 伤害为源码推导（wilson_attack=34×0.5），非运行测量。
- 结构检查：validate_page.py 与 strict 均通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
