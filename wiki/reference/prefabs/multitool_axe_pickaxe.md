---
title: "多用斧镐（multitool_axe_pickaxe）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版双功能手持工具单注册：一个 tool 登记 CHOP+MINE 双动作并按动作分档扣次数、默认 tough work 门禁豁免反弹、影子等级接入，可作一工具多动作的组装参照。"
aliases: ["multitool_axe_pickaxe", "多用斧镐", "Pick/Axe", "多功能工具斧镐", "斧镐"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["actions.lua", "bufferedaction.lua", "components/builder.lua", "components/equippable.lua", "components/finiteuses.lua", "components/playercontroller.lua", "components/shadowlevel.lua", "components/tool.lua", "components/weapon.lua", "components/workable.lua", "constants.lua", "languages/chinese_s.po", "map/static_layouts/oceanmonument.lua", "messagebottletreasures.lua", "prefabs/altar_prototyper.lua", "prefabs/axe_pickaxe.lua", "prefabs/chest_mimic.lua", "prefabs/charliearena_spike.lua", "prefabs/daywalker_pillar.lua", "prefabs/lunarrift_crystals.lua", "prefabs/nightmarefissure.lua", "prefabs/skinprefabs.lua", "prefabs/support_pillar.lua", "prefabs/waxwell.lua", "prefabskin.lua", "prefabskins.lua", "recipes.lua", "recipes_filter.lua", "scenarios/chest_labyrinth.lua", "scenarios/chest_labyrinth_mimic.lua", "scenarios/sunkenchest_oceanmonument.lua", "standardcomponents.lua", "stategraphs/SGshadowwaxwell.lua", "stringutil.lua", "tuning.lua", "widgets/itemtile.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 多用斧镐（multitool_axe_pickaxe）

## 快速判断

- 职责与适用：原版单注册手持双工具（可拾取物品＋手部装备，无 SG／Brain）：同一个 tool 组件登记 CHOP 与 MINE 两个工作动作，weapon 提供伤害，finiteuses 按动作分档扣总次数，tough work 目标可通过默认门禁豁免反弹，shadowlevel 提供静态影子等级 1。[工厂](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L28-L92)、[注册](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L94)
- 优先使用：借鉴"一工具多动作＋分动作消耗"组装读下方注册与构造、关联机制导航；改数值改 `TUNING.MULTITOOL_*` 键（见导航）；改运行行为用 `AddPrefabPostInit`（典型接入）；工作目标侧（workable）机制见 [[reference/components/workable#关键接口]]。
- 重要副作用：挖掘每次扣 3 次、砍伐每次扣 1 次、攻击命中扣 1 次；次数耗尽即从世界移除且原版无修理途径；在默认 tough work 门禁下不反弹（普通斧镐会被弹开且扣 0 次工作量；自定义反弹回调仍优先）；影子等级被麦斯威尔的通告、影随从伤害、描述文本与客户端库存特效消费。
- 修改边界：组件配置全部在 `SetPristine` 之后的主模拟段（服务端）；客户端只有动画、标签与 floater。本页未覆盖：引擎侧动画资源实际内容、皮肤资源包、网络同步细节、皮肤应用还原图标链路。[端分界](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L56-L60)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `multitool_axe_pickaxe` | `fn()`：CreateEntity＋Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank／build `multitool_axe_pickaxe`、`AddTag("sharp")` → 预置 `tool`／`weapon`／`shadowlevel` 标签 → `MakeInventoryFloatable` → `SetPristine` → 非主模拟返回 → 主模拟挂 weapon、tool、finiteuses、inspectable、inventoryitem、equippable、shadowlevel，最后 `MakeHauntableLaunch` | 物品＋手部装备形态，无运行时形态切换；无 common／master postinit 分层，全部初始化在同一 `fn`。[实体段](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L29-L54)、[主模拟段](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L62-L89) |

- 名称核验：`STRINGS.NAMES.MULTITOOL_AXE_PICKAXE`＝"多用斧镐"（英文 msgid "Pick/Axe"，[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L345979-L345982)）；配方描述"加倍实用。"（[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L357769-L357772)）。
- 获取（配方）：黄金斧×1＋黄金鹤嘴锄×1＋铥矿×2，`TECH.ANCIENT_FOUR`（`{ANCIENT=4}`，[键定义](../../../raw/current/scripts/constants.lua#L1254)），`nounlock=true`——制造不永久解锁配方（`Builder:UnlockRecipe` 对 nounlock 直接跳过，[实现](../../../raw/current/scripts/components/builder.lua#L465-L468)），每次都要在满足 ANCIENT=4 的合成站旁制作；完整远古祭坛 `ancient_altar` 给 `ANCIENTALTAR_HIGH`（ANCIENT=4），破损的 `ancient_altar_broken` 只给 ANCIENT=2。[配方](../../../raw/current/scripts/recipes.lua#L805)、[完整祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L348)、[破损祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L456)、[两注册名](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L503-L504)、[档位值](../../../raw/current/scripts/tuning.lua#L1284-L1286)；合成栏归 CRAFTING_STATION 过滤器的 ancients 组（[清单](../../../raw/current/scripts/recipes_filter.lua#L410)）。
- 生成点（实测引用链）：漂流瓶沉箱 `sunkenchest` 的 `splunker` 预设保证含 1 件（[预设](../../../raw/current/scripts/messagebottletreasures.lua#L126-L136)）；大型华丽宝箱场景 `chest_labyrinth`（5% 三选一，[条目](../../../raw/current/scripts/scenarios/chest_labyrinth.lua#L52-L56)；经 `chest_mimic` 的 scenariorunner 挂载，[调用](../../../raw/current/scripts/prefabs/chest_mimic.lua#L163)）及其 mimic 变体（[条目](../../../raw/current/scripts/scenarios/chest_labyrinth_mimic.lua#L75-L77)）；海洋纪念碑沉箱场景（85% 三选一，[条目](../../../raw/current/scripts/scenarios/sunkenchest_oceanmonument.lua#L50-L52)；由静态布局挂载，[调用](../../../raw/current/scripts/map/static_layouts/oceanmonument.lua#L66)）。`scenarios/chest_minotaur.lua` 也列有本 prefab（50% 四选一），但在当前快照全库检索不到该场景脚本被引用，不列为实际生成点。
- 皮肤：唯一皮肤 `multitool_axe_pickaxe_pickaxeaxe`（中文名"镐斧"，[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L401169-L401172)），`CreatePrefabSkin` 挂在本 prefab 上（[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8506-L8515)、[init／clear](../../../raw/current/scripts/prefabskin.lua#L874-L875)、[皮肤清单](../../../raw/current/scripts/prefabskins.lua#L1317-L1319)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | `SetOnEquip`／`SetOnUnequip` 挂持握回调；不改 `equipslot`，即默认 `EQUIPSLOTS.HANDS`（"hands"） | 主模拟挂载；装备时切持有者 `ARM_carry`／卸下还原 | [[reference/components/equippable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L82-L84)、[默认槽位](../../../raw/current/scripts/components/equippable.lua#L29)、[槽位值](../../../raw/current/scripts/constants.lua#L654-L660)、[Equip 回调与事件](../../../raw/current/scripts/components/equippable.lua#L92-L107) |
| 组件：tool | `SetAction(ACTIONS.CHOP, 4/3)`＋`SetAction(ACTIONS.MINE, 4/3)`，产生 `CHOP_tool`／`MINE_tool` 标签；`EnableToughWork(true)` 置 tough 标志 | 两动作共用同一效率值；`tool` 标签公共端预置 | [[reference/components/tool#关键接口]]；[登记](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L65-L68)、[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45)、[tough 标志](../../../raw/current/scripts/components/tool.lua#L29-L35)、[动作集](../../../raw/current/scripts/constants.lua#L2187-L2196) |
| 组件：weapon | `SetDamage(TUNING.MULTITOOL_DAMAGE)`；`attackwear` 保持默认，每次命中扣 1 次耐久 | 主模拟；攻击与工作互不影响各自的扣次路径 | [[reference/components/weapon#关键接口]]；[伤害](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L62-L63)、[攻击扣次](../../../raw/current/scripts/components/weapon.lua#L111-L120) |
| 组件：finiteuses | `SetMaxUses/SetUses(800)`、`SetOnFinished(inst.Remove)`、`SetConsumption(CHOP,1)`、`SetConsumption(MINE,3)`（MINE 同时把 REMOVELUNARBUILDUP 设为 3） | 耗尽时加 `usesdepleted` 标签、调 onfinished 移除实体；持有者收到 `toolbroke` 事件 | [[reference/components/finiteuses#关键接口]]；[配置](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L70-L75)、[分档映射](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)、[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)、[扣次入口](../../../raw/current/scripts/bufferedaction.lua#L26-L31)、[toolbroke](../../../raw/current/scripts/components/tool.lua#L1-L9) |
| 组件：shadowlevel | `SetDefaultLevel(TUNING.MULTITOOL_AXE_PICKAXE_SHADOW_LEVEL)`（1），无 `SetLevelFn`，等级静态不随耐久变化 | 主模拟挂组件；`shadowlevel` 标签公共端预置，客户端消费只看标签 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L86-L87)、[取值链](../../../raw/current/scripts/components/shadowlevel.lua#L14-L24) |
| 消费链：工作动作 | CHOP／MINE 经 `DoToolWork`：读 tool 效率 4/3 乘人物倍率 → 目标 `ShouldRecoil`→ 按反弹结果与角色特殊工作量倍率结算；动作成功后 `BufferedAction:Do` 调 `OnUsedAsItem` 按消耗表扣次（乘 efficientuser 倍率） | 服务端；动作必须匹配目标 workable 的 workaction | [DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1738)、[效率取值](../../../raw/current/scripts/actions.lua#L1703-L1717)、[反弹与 tooltooweak](../../../raw/current/scripts/actions.lua#L1719-L1731)、[MINE 动作](../../../raw/current/scripts/actions.lua#L1767-L1772)、[动作扣次](../../../raw/current/scripts/components/finiteuses.lua#L101-L114) |
| 消费链：tough work | 惧惧竞技场尖刺和梦魇裂缝的惧石矿使用默认 tough 门禁（MINE，[尖刺](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L277)、[惧石矿](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L547)）。月裂水晶的自定义回调也认可 `CanDoToughWork()`（MINE，[判定](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L78-L101)）。被囚者石柱最低档仍反弹，tough 能力可触发振动反馈（[反弹](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L407-L408)、[安装](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L421)、[振动](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L492-L507)）。惧石支撑柱要求 HAMMER，超出本工具动作集（[动作](../../../raw/current/scripts/prefabs/support_pillar.lua#L598-L603)） | `toughworker` 标签也可通过上述认可 tough 能力的判定 | [默认与自定义反弹判定](../../../raw/current/scripts/components/workable.lua#L189-L205) |
| 消费链：影子等级 | 持有者查看描述追加影子条目（要求查看者 `shadowmagic`）；麦斯威尔装备／卸下时按等级通告；影随从（protector）把领袖全部装备的 `GetCurrentLevel` 求和换算伤害加成；客户端库存格对已装备且 `shadowlevel` 标签物品显示影子特效（要求本地玩家 `shadowmagic`） | 组件与求和在服务端；库存特效为客户端 | [[reference/components/shadowlevel#典型接入]]；[描述](../../../raw/current/scripts/stringutil.lua#L343)、[通告](../../../raw/current/scripts/prefabs/waxwell.lua#L193)、[监听](../../../raw/current/scripts/prefabs/waxwell.lua#L317-L318)、[随从求和与伤害](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L144-L160)、[库存特效](../../../raw/current/scripts/widgets/itemtile.lua#L830-L836) |
| 组件：inspectable／inventoryitem | 基础检视与拾取，无专属配置 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L78-L80) |
| 组件：floater／hauntable | `MakeInventoryFloatable("med", 0.05, {0.7,0.4,0.7}, true, -13, swap 数据)` 水面漂浮并换 swap 装；`MakeHauntableLaunch` 被作祟时弹开（默认必弹、小冷却、HAUNT_TINY） | 两 helper 都在实体公共段调用 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L53-L54)、[漂浮 helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：TUNING | `MULTITOOL_DAMAGE＝wilson_attack×1.25＝34×1.25＝42.5`；`MULTITOOL_AXE_PICKAXE_EFFICIENCY＝4/3`；`MULTITOOL_AXE_PICKAXE_USES＝800`；`MULTITOOL_AXE_PICKAXE_SHADOW_LEVEL＝1` | 服务端组件读取；改键即改全族 | [伤害键](../../../raw/current/scripts/tuning.lua#L374)、[基数](../../../raw/current/scripts/tuning.lua#L31-L37)、[效率](../../../raw/current/scripts/tuning.lua#L191)、[次数](../../../raw/current/scripts/tuning.lua#L181)、[影子等级](../../../raw/current/scripts/tuning.lua#L2859) |
| API／数据：皮肤接线 | OnEquip 读 `GetSkinBuild`：有皮肤走 `equipskinneditem`＋`OverrideItemSkinSymbol`，否则普通 `OverrideSymbol`；OnUnequip 发 `unequipskinneditem` | 同一默认回调覆盖皮肤与无皮肤两态 | [持握分支](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L7-L17)、[卸下](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L19-L26) |
| 资源：动画 | `anim/multitool_axe_pickaxe.zip`＋`anim/swap_multitool_axe_pickaxe.zip` | 声明并实际用于 bank／build／swap | [assets](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L1-L5) |

## 典型接入

### 把多用斧镐的挖掘消耗改成与砍伐一致

- 前提与选择：放目标模组 `modmain.lua`，服务端组件配置沿用原版 prefab，不新增资源；原版挖掘（3 次／次）与砍伐（1 次／次）消耗不对称，统一后 800 次可同时支持约 800 次挖掘或砍伐。
- 生效与撤销：影响此模组下新实例；`SetConsumption(ACTIONS.MINE, ...)` 会同时把 REMOVELUNARBUILDUP 设成同值（组件耦合），清理月尘 buildup 也按新值扣次。停用后新实例恢复原版分档。
- 原版依据：分档消耗由 `SetConsumption` 登记，REMOVELUNARBUILDUP 的耦合与 `MINE_tool` 标签动作指派见关联导航；本例不改效率、伤害与 tough 标志。[分档映射](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)、[动作指派](../../../raw/current/scripts/components/playercontroller.lua#L2009-L2010)

```lua
AddPrefabPostInit("multitool_axe_pickaxe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local finiteuses = inst.components.finiteuses
    if finiteuses ~= nil then
        -- MINE 与 CHOP 都改为每次扣 1（REMOVELUNARBUILDUP 被组件同步为 1）
        finiteuses:SetConsumption(GLOBAL.ACTIONS.MINE, 1)
        finiteuses:SetConsumption(GLOBAL.ACTIONS.CHOP, 1)
    end
end)
```

- 调整与验证：数值单位是"次数"；已有实例的当前次数不受影响，存档载入按 `finiteuses` 的 OnLoad 恢复绝对次数。本例经源码审阅，未运行游戏；与其他同时改消耗的模组需自行核对最终值。

## 常见陷阱与限制

- **"800 次"不是 800 次挖矿：** MINE 每次动作扣 3 次、CHOP 扣 1 次、攻击命中扣 1 次（`attackwear` 默认 1），800 次约支撑 266 次挖掘动作；效率 4/3 是对目标的工作量加成，不改变扣次。[配置](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L70-L75)、[攻击扣次](../../../raw/current/scripts/components/weapon.lua#L111-L120)
- **tough work 不等于更高工作量：** 多用斧镐凭 `EnableToughWork(true)` 可以通过默认的 `SetRequiresToughWork(true)` 门禁（`Workable:ShouldRecoil` 的 `self.tough` 分支）。自定义 `SetShouldRecoilFn` 优先执行，返回 nil 才回落到默认门禁；回调返回 true 时以其给出的工作量进入后续结算，省略该值时按 0 处理。只有反弹、结算工作量为 0，且持有者 SG 有 `recoilstate` 时才发 `tooltooweak`；对普通树木矿石没有额外加成。[默认门禁](../../../raw/current/scripts/components/workable.lua#L199-L204)、[自定义回调优先](../../../raw/current/scripts/components/workable.lua#L189-L198)、[反弹与 tooltooweak](../../../raw/current/scripts/actions.lua#L1726-L1731)
- **耗尽即消失且不可修复：** 本 prefab 未挂 repairable，`SetOnFinished(inst.Remove)` 在次数归零时直接移除实体；改造成可修物品需自行加 repairable 并理解 finiteuses 对它的联动。[耗尽](../../../raw/current/scripts/components/finiteuses.lua#L68-L75)、[修理联动](../../../raw/current/scripts/components/finiteuses.lua#L1-L8)
- **改 MINE 消耗会连带 REMOVELUNARBUILDUP：** 组件在 MINE 映射时强制同步该键；手持带 `MINE_tool` 标签的工具（本 prefab 满足）会被客户端指派清理月尘动作。[耦合](../../../raw/current/scripts/components/finiteuses.lua#L29-L31)、[指派](../../../raw/current/scripts/components/playercontroller.lua#L2009-L2010)
- **影子等级是静态值：** 无 `SetLevelFn`，等级恒为 1；等级消费（随从伤害求和、通告、描述条目）要求持有者或查看者带 `shadowmagic`（麦斯威尔），普通角色查看无该条目。[取值](../../../raw/current/scripts/components/shadowlevel.lua#L22-L24)、[判定](../../../raw/current/scripts/stringutil.lua#L343)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394。完整读取 `prefabs/axe_pickaxe.lua` 与 equippable／tool／weapon／finiteuses／shadowlevel／workable 组件；核验 DoToolWork 效率与反弹链、BufferedAction 扣次入口、tough work 全部消费端（charliearena_spike／nightmarefissure／support_pillar／lunarrift_crystals／daywalker_pillar）、shadowlevel 四类消费者（stringutil／waxwell／SGshadowwaxwell／itemtile）、配方与 `nounlock` 语义、四个实际生成点引用链、皮肤注册与名称键。TUNING 实际值由键定义推导（`wilson_attack=34`、乘数 1），未实测运行数值。未覆盖：引擎侧资源内容、皮肤资源包、网络同步、`chest_minotaur` 场景为何留存未引用。
- 游戏运行：未运行；所有结论来自当前快照源码审阅。
- 结构检查：validate_page.py 与 strict 均通过。

相关：[[reference/prefabs/axe|斧头族（配方原料黄金斧）]]、[[reference/prefabs/pickaxe|鹤嘴锄（配方原料黄金鹤嘴锄）]]、[[reference/components/tool|工作动作与效率]]、[[reference/components/workable#关键接口|目标侧工作顺序]]、[[reference/components/finiteuses|按动作分档扣次]]。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
