---
title: 露西斧（lucy）
created: 2026-11-06
updated: 2026-11-08
type: prefab
summary: 伍迪专属会说话的有灵斧 lucy：无限耐久、砍树效率 2、伤害 13.6；由伍迪出生自带、砍树时替换普通斧产生，非 woodcutter 持有即被丢出，主人实体移除 90 秒后还原为替换前的斧。
aliases: ["lucy", "露西斧", "Lucy the Axe", "露西", "露西的斧头", "会说话的斧子", "伍迪的斧头"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lucy.lua", "prefabs/lucy_classified.lua", "prefabs/lavaarena_lucy.lua", "prefabs/woodie.lua", "prefabs/player_common.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "recipes.lua", "components/possessedaxe.lua", "components/sentientaxe.lua", "components/equippable.lua", "components/tool.lua", "components/weapon.lua", "components/talker.lua", "components/inventoryitem.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhermitcrab.lua", "fx.lua", "strings.lua", "tuning.lua", "play_commonfn.lua", "components/workable.lua", "components/inventory.lua", "screens/redux/scrapbookdata.lua", "prefabs/evergreens.lua", "prefabs/deciduoustrees.lua", "components/spooked.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 露西斧（lucy）

## 快速判断

- 职责与适用：`prefabs/lucy.lua` 单注册的可拾取手持斧（物品＋装备＋CHOP 工具），伍迪专属"有灵斧"：不挂 finiteuses（无限耐久），经 sentientaxe＋talker＋`lucy_classified` 自主说话；伍迪出生自带，并经"替换普通斧"机制流转。[工厂](../../../raw/current/scripts/prefabs/lucy.lua#L129-L232)、[注册](../../../raw/current/scripts/prefabs/lucy.lua#L234)
- 优先使用：调数值／组件查下方[数值](#数值与配置)与 [[reference/components/possessedaxe]]、[[reference/components/sentientaxe]]；让自定义角色持有露西用 `woodcutter` 标签（[典型接入](#典型接入)）；从零造同类手持工具查 [[cookbook/equipment/hand]]；普通斧侧见 [[reference/prefabs/axe]]。
- 重要副作用：伍迪装备 `possessable_axe` 砍 CHOP 目标会被替换成本页 lucy（记录原斧名与耐久）；非 woodcutter 拿起即被丢出并说话；主人移除 90 秒后还原替换前的斧；仅地面／装备栏分支播特效，普通容器直接交还原斧。[替换链](../../../raw/current/scripts/prefabs/woodie.lua#L681-L705)、[还原](../../../raw/current/scripts/components/possessedaxe.lua#L167-L202)
- 修改边界：标签与 talker 挂公共端、其余组件挂主模拟；客户端台词经 `lucy_classified` 代理（[同页说明](#lucyclassified-同步)）；Forge 用独立注册 `lavaarena_lucy`（投掷 AoE 武器），本页不覆盖。[Forge 注册](../../../raw/current/scripts/prefabs/lavaarena_lucy.lua#L224-L225)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lucy` | `fn()` 直接构造，无包装工厂与 postinit | 物品＋装备＋工具＋有灵组件；4 款皮肤为 CreatePrefabSkin 换装（[皮肤](#皮肤)）。[fn](../../../raw/current/scripts/prefabs/lucy.lua#L129-L232)、[注册](../../../raw/current/scripts/prefabs/lucy.lua#L234) |
| `lucy_classified` | `fn()` 构造的隐藏网络代理实体：`entity:Hide()`＋`CLASSIFIED` 标签＋`persists = false` | 只承载台词与音效索引同步，无独立玩法；lucy 主模拟生成后 `SetParent` 挂本体、随本体清理。[构造](../../../raw/current/scripts/prefabs/lucy_classified.lua#L97-L138)、[挂载与清理](../../../raw/current/scripts/prefabs/lucy.lua#L30-L41)、[生成](../../../raw/current/scripts/prefabs/lucy.lua#L189-L192) |

`lucy` 共同初始化（[lucy.lua#L129-L232](../../../raw/current/scripts/prefabs/lucy.lua#L129-L232)）：实体／动画／标签 → talker（SetPristine 前）→ 本地音效实体 `localsounds` → `MakeInventoryFloatable(...)` → `SetPristine` → 主模拟 sentientaxe → `lucy_classified` → weapon／tool／inspectable／inventoryitem／equippable → possessedaxe → `MakeHauntableLaunch`＋`AddHauntableCustomReaction`。

### 数值与配置

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键／来源 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.AXE_DAMAGE`×0.5 | 27.2×0.5＝13.6 生命值（源码推导，图鉴同值 [scrapbookdata.lua#L796](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L800)） |
| 砍树效率 | `tool:SetAction(ACTIONS.CHOP, 2)` | 2 工作量/动作，对一切 CHOP 目标无差别，工厂无对松树／月树的特殊效率分支（[tool 挂载](../../../raw/current/scripts/prefabs/lucy.lua#L212-L213)） |
| 耐久 | 工厂不挂 finiteuses | 无限：无 CHOP 消耗；weapon 磨损结算有 finiteuses 判空，本实体不扣耐久（[weapon OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121)） |
| 装备槽 | equippable 默认 | HANDS（[equippable.lua#L29](../../../raw/current/scripts/components/equippable.lua#L29)） |
| 还原等待时长 | `TUNING.LUCY_REVERT_TIME` | 90 秒（[tuning.lua#L2040](../../../raw/current/scripts/tuning.lua#L2040)） |
| 雕刻台词限频 | `TUNING.LUCY_CARVING_TALK_COOLDOWN` | 15 秒（[tuning.lua#L7827](../../../raw/current/scripts/tuning.lua#L7828)） |
| 非伍迪装备伤害 | `TUNING.LUCY_BITE_DAMAGE` | 5（[tuning.lua#L2041](../../../raw/current/scripts/tuning.lua#L2041)）；**本 build 全库无脚本消费者**，见[陷阱](#常见陷阱与限制) |

名称与台词：官方名 `STRINGS.NAMES.LUCY`＝Lucy the Axe／简中"露西斧"，经 .po `msgctxt` 核验（[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L344849-L344852)）；检视语 `DESCRIBE.LUCY`（[.po](../../../raw/current/scripts/languages/chinese_s.po#L14849-L14852)）。台词表 `STRINGS.LUCY` 定义于 strings.lua（equipped／on_ground／on_haunt／in_container／other_owner／on_pickedup／on_chopped／beaver_down_*／transform_*／carve_* 等，[strings.lua#L16522](../../../raw/current/scripts/strings.lua#L16522)）。

### 获取途径

- 无配方：全库无 `Recipe2("lucy",...)`；雕刻配方只以 `Ingredient("lucy", 0)` 做"持有即可、不消耗"的原料。
- 伍迪出生自带：`GAMEMODE_STARTING_ITEMS.DEFAULT.WOODIE = {"lucy"}`（[tuning.lua#L3460](../../../raw/current/scripts/tuning.lua#L3460)）→ woodie.lua 换算 `start_inv` 并声明依赖（[L90-L95](../../../raw/current/scripts/prefabs/woodie.lua#L90-L95)）、`inst.starting_inventory`（[L1798](../../../raw/current/scripts/prefabs/woodie.lua#L1798)）→ player_common 的 `GivePlayerStartingItems` 发放（[player_common.lua#L1481](../../../raw/current/scripts/prefabs/player_common.lua#L1481)）。
- 伍迪砍树替换（主流转）：见[替换与还原链](#伍迪替换与还原链)。

### 皮肤

装备接线与斧头族一致：`GetSkinBuild()` 非空时装备／卸装推 `equipskinneditem`／`unequipskinneditem`，持握贴图用 `OverrideItemSkinSymbol("swap_object", skin_build, "swap_lucy_axe", ...)`，否则普通 `OverrideSymbol`（[onequip](../../../raw/current/scripts/prefabs/lucy.lua#L86-L96)、[onunequip](../../../raw/current/scripts/prefabs/lucy.lua#L98-L105)）。已注册 4 款，只换贴图＋主模拟 `AddSkinSounds`（[lucy_init_fn](../../../raw/current/scripts/prefabskin.lua#L1069-L1079)），名称经 .po `SKIN_NAMES` 核验（[.po](../../../raw/current/scripts/languages/chinese_s.po#L400755-L400772)），均 Distinguished／Woven：

`lucy_broadaxe`（阔斧，[L7594-L7603](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7594-L7603)）、`lucy_chained`（链斧，[L7605-L7614](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7605-L7614)）、`lucy_picnic`（花格子露西尔，[L7616-L7625](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7616-L7625)）、`lucy_punk`（朋克斧，[L7627-L7636](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7627-L7636)）。

## lucy_classified 同步

角色与目的：lucy 是网络实体但台词正文不进 netvar；服务端只同步"哪张台词表＋第几句＋哪个音效"三个索引，客户端按本机 `STRINGS.LUCY` 还原。

| 环节 | 服务端（主模拟） | 客户端 | 证据 |
|---|---|---|---|
| 变量 | `string_list`（net_smallbyte：`STRINGS.LUCY` 子表索引）、`string_id`（net_smallbyte：句号）、`sound_override`（net_tinybyte：`TALK_SOUNDS` 索引，dirty 事件名 `saydirty`） | 同名 netvar 镜像，索引表由同一 strings 数据构建 | [netvars](../../../raw/current/scripts/prefabs/lucy_classified.lua#L108-L110)、[索引表](../../../raw/current/scripts/prefabs/lucy_classified.lua#L1-L13) |
| 发言 | `lucy_classified:Say(list, id, sound)`：写三个索引，1 秒超时清零 | `saydirty` → `OnSayDirty`：`talker:Say(string, nil, nil, nil, true)`（nobroadcast） | [Say](../../../raw/current/scripts/prefabs/lucy_classified.lua#L51-L60)、[OnSayDirty](../../../raw/current/scripts/prefabs/lucy_classified.lua#L24-L32)、[Talker:Say](../../../raw/current/scripts/components/talker.lua#L267) |
| 接收者与音效 | `SetTarget(target)` 定向；进背包→`topocket` 设持有链最终主人，掉落→`toground` 设 nil（广播） | 仅 `target == ThePlayer`（或 nil 全员）监听 `saydirty`；`GetTalkSound` 取 `TALK_SOUNDS`（lucy_warn_1／2／3／lucy_transform），无 override 循环播 `lucytalk_LP` | [SetTarget](../../../raw/current/scripts/prefabs/lucy_classified.lua#L62-L73)、[topocket／toground](../../../raw/current/scripts/prefabs/lucy.lua#L65-L82)、[TALK_SOUNDS](../../../raw/current/scripts/prefabs/lucy_classified.lua#L7-L13)、[localsounds](../../../raw/current/scripts/prefabs/lucy.lua#L165-L176) |

台词的发起与场景触发在 [[reference/components/sentientaxe]] 维护，本页不重复；分片迁移时 talker 静音 0.2 秒防串音（[topocket](../../../raw/current/scripts/prefabs/lucy.lua#L72-L75)）。

## 伍迪替换与还原链

1. **产生**：伍迪监听自身的 `working` 事件（onworked），`data.target` 为目标（`worked` 是目标侧事件），目标 `workable.action == ACTIONS.CHOP`、装备位是带 `possessable_axe` 标签的斧、`Inventory:FindItem` 在物品栏／鼠标持有物／溢出容器中找不到 lucy（不扫装备栏）时：`SpawnPrefab("lucy")`、`revert_prefab`＝原斧注册名、`revert_uses`＝原斧 finiteuses 剩余（无则 nil）、Remove 原斧、装备 lucy、在 `swap_object` 符号上播 `lucy_transform_fx`。旧斧先存耐久并被移除，本次 `BufferedAction` 因 `invobject` 失效跳过 `OnUsedAsItem`，触发替换的一斧不扣旧耐久。[onworked](../../../raw/current/scripts/prefabs/woodie.lua#L681-L705)、[事件派发](../../../raw/current/scripts/components/workable.lua#L145-L152)、[FindItem](../../../raw/current/scripts/components/inventory.lua#L623-L636)
2. **认主**：lucy 进背包后 possessedaxe 归属检查通过（主人有 `woodcutter` 标签，[woodie.lua#L1746](../../../raw/current/scripts/prefabs/woodie.lua#L1746)）→ `LinkToPlayer` 按 userid 绑定并推 `axepossessedbyplayer` → sentientaxe `SetOwner` 开始说话；不通过则 `Drop`＋推 `axerejectedowner`（说 other_owner）或 `axerejectedotheraxe`（说 on_woodie_pickedup_other 并取消旧露西未发台词）。[OnCheckOwner](../../../raw/current/scripts/components/possessedaxe.lua#L16-L33)、[LinkToPlayer](../../../raw/current/scripts/components/possessedaxe.lua#L136-L158)
3. **丢弃不还原**：掉落只清 classified 目标（广播）＋说 on_dropped（2 帧延迟）；`ondropped` 事件不带 owner，`currentowner` 清空但 `player`／`userid` 链接保持——不飞回、不消失、不开始计时，仍对 15 距离内的主人闲聊。[toground](../../../raw/current/scripts/prefabs/lucy.lua#L78-L82)、[inventoryitem ondropped](../../../raw/current/scripts/components/inventoryitem.lua#L392)、[地面闲聊](../../../raw/current/scripts/components/sentientaxe.lua#L187-L210)
4. **还原**：仅三条路径——玩家实体 `onremove`（掉线／被移除）进入 `WaitForPlayer(userid)`，90 秒（`LUCY_REVERT_TIME`）超时；同玩家被另一把 lucy `LinkToPlayer`（玩家侧 `possessedaxe` 事件）；等待期内重连玩家不合格。`Revert()` 按 `revert_prefab`（默认 "axe"）重生斧、还原耐久 `math.max(1, revert_uses)`、按地面／装备栏／容器三位置替换本体；地面／装备栏分支播 `revert_fx`，普通容器分支直接 `GiveItem` 交还旧斧。读档时 `OnLoad` 可按 `waittimeremaining` 恢复等待计时。[WaitForPlayer](../../../raw/current/scripts/components/possessedaxe.lua#L107-L121)、[Revert](../../../raw/current/scripts/components/possessedaxe.lua#L167-L202)、[OnLoad](../../../raw/current/scripts/components/possessedaxe.lua#L204-L226)
5. **月圆／技能变身不受影响**：wereness 变形路径没有针对 lucy 的移除或还原逻辑；sentientaxe 只监听 `startwereplayer`／`stopwereplayer` 说 transform_beaver／transform_woodie，变身中（transform／wereplayer 标签）不闲聊。[事件推送](../../../raw/current/scripts/prefabs/woodie.lua#L1072-L1076)、[闲聊门禁](../../../raw/current/scripts/components/sentientaxe.lua#L171-L178)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：talker | SetPristine 前挂载：fontsize 28／TALKINGFONT／颜色 (0.9,0.4,0.4)／symbol `swap_object` | 公共端；正文由 lucy_classified 驱动（nobroadcast） | [[reference/components/talker#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lucy.lua#L157-L162) |
| 组件：sentientaxe | 人格与台词触发链（拾起／掉落／砍完树／雕刻／变身／闲聊） | 主模拟；依赖 lucy_classified 与 equippable | [[reference/components/sentientaxe#关键接口]]；[安装](../../../raw/current/scripts/prefabs/lucy.lua#L187) |
| 组件：possessedaxe | woodcutter 归属校验、userid 链接、90 秒等待、Revert 还原与特效 | 主模拟；`revert_fx`＝lucy_ground_transform_fx、`transform_fx`＝lucy_transform_fx | [[reference/components/possessedaxe#关键接口]]；[接线](../../../raw/current/scripts/prefabs/lucy.lua#L224-L226) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：swap_object 覆盖、ARM_carry／ARM_normal 显隐、皮肤事件 | 主模拟；槽默认 HANDS；`equipped` 事件驱动拾起台词 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/lucy.lua#L220-L222)、[事件源](../../../raw/current/scripts/components/equippable.lua#L102) |
| 组件：tool | `SetAction(ACTIONS.CHOP, 2)` 挂砍树工具与 `CHOP_tool` 标签 | 主模拟；效率 2 | [[reference/components/tool#关键接口]]；[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45) |
| 组件：weapon | `SetDamage(AXE_DAMAGE×0.5)`＝13.6；无 onattack 回调 | 主模拟；无 finiteuses 故攻击无磨损 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lucy.lua#L208-L209) |
| 组件：inspectable／inventoryitem | 检视与拾取；掉落地面显示小地图图标 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lucy.lua#L217-L218)、[图标](../../../raw/current/scripts/prefabs/lucy.lua#L139) |
| 组件：hauntable | `MakeHauntableLaunch`（弹飞）＋`AddHauntableCustomReaction(CustomOnHaunt, true, false, true)`：弹飞后补说 `STRINGS.LUCY.on_haunt`，以弹飞结果为最终返回 | 主模拟；作祟冷却沿用组件默认 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/lucy.lua#L228-L229)、[secondrxn 语义](../../../raw/current/scripts/standardcomponents.lua#L1528-L1557)、[CustomOnHaunt](../../../raw/current/scripts/prefabs/lucy.lua#L121-L127) |
| 组件：floater | `MakeInventoryFloatable("small", 0.05, {1.2, 0.75, 1.2}, true, -11, swap_data)`：浮水换 swap 贴图 | 公共端 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/lucy.lua#L178-L179) |
| API／数据：`woodcutter` 标签 | possessedaxe 认主的唯一判定；另有多个消费端（见陷阱） | 玩家实体标签 | [判定](../../../raw/current/scripts/components/possessedaxe.lua#L1-L3)、[加标](../../../raw/current/scripts/prefabs/woodie.lua#L1746) |
| API／数据：`cancarveboards` 标签、雕刻与专属待机 | 伍迪技能 `woodie_human_lucy_1` 后按 `inventory:Has("lucy", 1, true)` 增删 `cancarveboards`；三个雕刻配方用 `Ingredient("lucy", 0)`（持有即可、不消耗）＋`sg_state` 进 SGwilson 雕刻状态（按 lucy 皮肤换贴图）；手持 lucy 时 `customidleanimfn` 返回 `idle_woodie` 待机动画 | 主模拟标签／配方 | [cancarveboards](../../../raw/current/scripts/prefabs/woodie.lua#L1715-L1717)、[配方](../../../raw/current/scripts/recipes.lua#L173-L175)、[SG 皮肤](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8553-L8560)、[待机](../../../raw/current/scripts/prefabs/woodie.lua#L1660-L1663) |
| 关联 Prefab：axe（还原目标） | `revert_prefab` 默认 "axe"；被替换斧的注册名与耐久在替换时记入，还原时重生 | 主模拟；goldenaxe／moonglassaxe 及自定义同标签斧同理 | [[reference/prefabs/axe]]；[Revert](../../../raw/current/scripts/components/possessedaxe.lua#L167-L202) |
| 关联 Prefab：lucy_transform_fx／lucy_ground_transform_fx | 替换与还原的变身特效（bank `lucy_axe_fx`／build `axe_transform_fx`，data-driven FX） | 主模拟生成、跟随 `swap_object`；也被 SGwilson／SGhermitcrab 作通用接物／换装特效复用（非 lucy 专属） | [FX 定义](../../../raw/current/scripts/fx.lua#L1062-L1071)、[复用例](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2121)、[SGhermitcrab](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L400) |
| 关联 Prefab：lavaarena_lucy | Forge 玩法独立注册（投掷 AoE 武器），与 `lucy` 无工厂共享 | 仅 Forge；master postinit 来自 event_server_data | [注册](../../../raw/current/scripts/prefabs/lavaarena_lucy.lua#L224-L225) |
| 关联 Prefab：舞台剧系统 | `play_commonfn` 的 `findlucy`／`lucytalk` 可点名在场 lucy 说台词 | 舞台剧演出 | [findlucy／lucytalk](../../../raw/current/scripts/play_commonfn.lua#L133-L152) |
| 资源：anim／图标 | lucy_axe.zip（bank）、swap_lucy_axe.zip（持握）、minimap `lucy_axe.png` | 静态声明 | [声明](../../../raw/current/scripts/prefabs/lucy.lua#L1-L6) |

## 典型接入

### 调整露西的砍树效率与伤害

- 前提与选择：服务端 `modmain.lua`。露西组件只在主模拟，`TheWorld`／`ACTIONS` 是运行时全局，modmain 环境只直挂 `TUNING` 等白名单（mods.lua CreateEnvironment 无 `__index=_G`），须经 `GLOBAL.` 取。
- 生效与撤销：postinit 在工厂后执行，重载存档时组件按新工厂值装配，新旧实例均生效。
- 原版依据：tool 效率与 weapon 伤害是工厂里两次直接设置，互不影响；耐久无关（无 finiteuses）。[tool](../../../raw/current/scripts/prefabs/lucy.lua#L212-L213)、[weapon](../../../raw/current/scripts/prefabs/lucy.lua#L208-L209)

```lua
-- modmain.lua：露西砍树效率 2→3、武器伤害 13.6→20
AddPrefabPostInit("lucy", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        inst.components.tool:SetAction(GLOBAL.ACTIONS.CHOP, 3)
        inst.components.weapon:SetDamage(20)
    end
end)
```

- 调整与验证：效率单位为工作量/动作，伤害为生命值。改 `TUNING.AXE_DAMAGE` 会连带普通斧（露西读 `AXE_DAMAGE×0.5`），只想改露西用上面的直调。

### 让自定义角色持有露西

- 前提与选择：服务端 `modmain.lua`。possessedaxe 认主只看玩家 `woodcutter` 标签（[判定](../../../raw/current/scripts/components/possessedaxe.lua#L1-L3)）。
- 生效与撤销：标签常驻即持续合法；移除后，露西下一次归属重检（进出背包／放置容器时触发）会把露西丢出。
- 原版依据：归属重检由 `onputininventory`／`ondropped` 触发、延迟 0 帧执行。[OnChangeOwner](../../../raw/current/scripts/components/possessedaxe.lua#L35-L63)

```lua
-- modmain.lua：给自定义角色加 woodcutter 标签，让露西的 possessedaxe 认主
AddPlayerPostInit(function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.prefab == "mymodchar" then
        inst:AddTag("woodcutter")
    end
end)
```

- 调整与验证：`woodcutter` 不只服务 Lucy 认主——见[陷阱](#常见陷阱与限制)副作用清单；给其他原版角色加标签前先确认这些表现可接受。

## 常见陷阱与限制

- **误区：非伍迪装备露西会掉 5 点血**：`TUNING.LUCY_BITE_DAMAGE = 5` 的注释写着"非伍迪装备 lucy 受到的伤害"，但本 build 全库 grep 该键仅 tuning.lua 一处定义、无脚本消费者；实际拒绝逻辑是 possessedaxe 归属检查直接 `Drop`，不扣血。[键定义](../../../raw/current/scripts/tuning.lua#L2041)、[拒绝路径](../../../raw/current/scripts/components/possessedaxe.lua#L21-L23)
- **边界：`woodcutter` 标签的副作用**：除露西认主外，还改变砍树／攻击动画（woodie_chop 系列，[SGwilson L5016](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5016)、[L5044-L5046](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5044-L5046)）、树精守卫（Leif）与桦木树精怪出现几率 ×1.5（[evergreens.lua#L536-L538](../../../raw/current/scripts/prefabs/evergreens.lua#L536-L538)、[tuning.lua#L4641](../../../raw/current/scripts/tuning.lua#L4641)、[deciduoustrees.lua#L551-L553](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L551-L553)）、部分惊吓判定提前到 8 帧（[spooked.lua#L97](../../../raw/current/scripts/components/spooked.lua#L97)）。
- **边界：露西无限耐久但被替换斧的耐久会保留**：露西本体不存耐久；还原时原斧耐久取 `math.max(1, revert_uses)`（不足 1 按 1 复活）。[还原耐久](../../../raw/current/scripts/components/possessedaxe.lua#L171-L172)
- **边界：死亡不启动 90 秒倒计时**：possessedaxe 的 `onplayerdied` 回调已定义但全库无监听点；伍迪死亡只让露西掉落（保持链接继续说话），掉线／被移除（onremove）才开始计时。[onplayerdied 无监听](../../../raw/current/scripts/components/possessedaxe.lua#L82)、[掉落路径](../../../raw/current/scripts/components/possessedaxe.lua#L160-L165)
- **边界：箱子中静置**：放入无 inventoryitem 的容器（如箱子）时 `GetGrandOwner` 返回容器本身、无 inventory 组件，归属检查跳过——不拒绝、不还原、不绑定。[GetGrandOwner](../../../raw/current/scripts/components/inventoryitem.lua#L501-L506)、[检查入口](../../../raw/current/scripts/components/possessedaxe.lua#L18-L20)
- **边界：变身 FX 有外部消费者**：`lucy_transform_fx`／`lucy_ground_transform_fx` 名字挂在 lucy 名下，但 SGwilson／SGhermitcrab 把前者当通用"接物／换装"特效复用（回旋镖接取、装备表现）；修改其外观会波及这些场景。[复用例](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2121)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 lucy.lua／lucy_classified.lua／possessedaxe.lua／sentientaxe.lua 全文，woodie.lua 的 working 监听与替换链，player_common、equippable／tool／weapon／talker／inventoryitem、tuning、strings.lua、fx.lua、皮肤注册、recipes.lua、SGwilson、scrapbookdata；LUCY_BITE_DAMAGE 全库无消费者，无 Recipe2("lucy")。未展开 talker 渲染与 hauntable 内部。
- 游戏运行：未运行。未覆盖：13.6 伤害与效率 2 为源码推导（图鉴数据印证），非运行测量；90 秒还原时序、classified 定向广播、皮肤实装外观、Forge 变体行为未实测。
- 结构检查：validate_page.py 与 strict 均通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
