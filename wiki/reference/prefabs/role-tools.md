---
title: "角色与活动专用工具族（火焰笔／劣质工具／清洁扫把／鞍角／鱼食／出逃腿靴／气球／枕头／剧场服装）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "十个 prefab 文件注册的 26 个角色或活动专用装备：火笔（book_fire 产出的远程点火武器）、鱼人劣质工具（restrictedtag 门禁 NPC 工具）、清洁扫把（对目标循环换肤）、鞍角（UNSADDLE 卸牛鞍）、鱼食（海面抛掷诱鱼）、出逃腿靴（海面双向旋涡传送）、Wes 气球双注册与 YotR 枕头四材质、Charlie 剧场八套 body 服装，覆盖注册、槽位、门禁、配方与获取。"
aliases: ["firepen", "merm_tool", "merm_tool_upgraded", "reskin_tool", "saddlehorn", "chum", "bootleg", "balloon", "balloonspeed", "balloon_mapicon", "bodypillow_petals", "bodypillow_kelp", "bodypillow_beefalowool", "bodypillow_steelwool", "handpillow_petals", "handpillow_kelp", "handpillow_beefalowool", "handpillow_steelwool", "costume_doll_body", "costume_queen_body", "costume_king_body", "costume_blacksmith_body", "costume_mirror_body", "costume_tree_body", "costume_fool_body", "costume_princess_body", "火焰笔", "劣质工具", "不那么劣质的工具", "清洁扫把", "鞍角", "鱼食", "出逃腿靴", "气球", "迅捷气球", "枕头", "枕头盔甲", "剧场服装", "花朵枕头盔甲", "海带枕头盔甲", "皮弗娄牛枕头盔甲", "钢丝绵枕头盔甲", "花朵枕头", "海带枕头", "皮弗娄牛毛枕头", "钢丝绵枕头", "玩偶服", "女王服", "国王服", "铁匠服", "镜子服", "树木服", "小丑服", "公主服", "角色工具", "活动工具", "换肤工具", "卸鞍"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pillow_equipment.lua", "prefabs/pillow_defs.lua", "prefabs/pillow_common.lua", "prefabs/costumes.lua", "prefabs/firepen.lua", "prefabs/merm_tool.lua", "prefabs/reskin_tool.lua", "prefabs/saddlehorn.lua", "prefabs/chum.lua", "prefabs/chum_aoe.lua", "prefabs/bootleg.lua", "prefabs/balloonspeed.lua", "prefabs/balloon.lua", "prefabs/balloons_common.lua", "components/equippable.lua", "components/inventory.lua", "components/inventoryitem_replica.lua", "components/unsaddler.lua", "components/stackable.lua", "components/fueled.lua", "prefabs/merm.lua", "brains/mermbrain.lua", "prefabs/merm_supply_structures.lua", "prefabs/books.lua", "prefabs/sharkboi.lua", "prefabs/charlie_stage.lua", "prefabs/statueharp_hedgespawner.lua", "play_the_doll.lua", "prefabs/yotr_rabbitshrine.lua", "brains/oceanfishbrain.lua", "recipes.lua", "tuning.lua", "constants.lua", "actions.lua", "componentactions.lua", "skins_defs_data.lua", "prefabs/hats.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 角色与活动专用工具族（火焰笔／劣质工具／清洁扫把／鞍角／鱼食／出逃腿靴／气球／枕头／剧场服装）

## 快速判断

- 职责与适用：十个文件注册的 26 个"绑定角色或活动场景"的装备：25 个可拾取装备（BODY 12、HANDS 13）＋1 个不装备的地图图标代理 `balloon_mapicon`（槽位见[注册与构造](#注册与构造)）。共性是绕开通用工具链，用角色 tag、事件活动或特定生成者限定获取与使用：`firepen` 只能由 book_fire 按火数"充能"获得；`merm_tool` 走 [restrictedtag 门禁](#鱼人工具restrictedtag-门禁mermtoollua)；枕头与服装绑定 YotR 枕头大战／Charlie 剧场；气球绑定 Wes 的 balloonomancer tag。
- 优先使用：给自定义角色或 NPC 做专属装备查 `merm_tool.lua` 工厂与 [[reference/components/equippable]] 的 restrictedtag 判定；做"无伤害但有命中效果"的武器查 `firepen.lua`／`handpillow_*`；做抛掷物查 `chum.lua`（[[reference/components/oceanthrowable]]）与 `bootleg.lua`（[[reference/components/complexprojectile]]）；做易碎跟随物查 [[reference/prefabs/balloon_held_child]]；完整造物工程见 [[cookbook/equipment/hand]]。
- 重要副作用：火笔命中会点燃／投料／解冻／唤醒目标（[OnAttack](../../../raw/current/scripts/prefabs/firepen.lua#L17-L63)）；清洁扫把直接改写目标实体皮肤（含虫洞与洞穴入口跨 shard，[spellCB](../../../raw/current/scripts/prefabs/reskin_tool.lua#L252-L281)）；出逃腿靴在海面生成双向传送旋涡；迅捷气球手持提供 1.0–1.3 移速，**未耗尽**时丢在地上会飞走、耗尽后掉落为静止物品（[SetHeliumLevel](../../../raw/current/scripts/prefabs/balloonspeed.lua#L88-L104)、[flyoff](../../../raw/current/scripts/prefabs/balloonspeed.lua#L56-L71)）。
- 修改边界：本页只覆盖注册、装备链、门禁与获取途径；`balloon_held_child` 表现层归 [[reference/prefabs/balloon_held_child]]，`attackfx_handpillow_*` 四注册归 [[reference/prefabs/attackfx_handpillow_petals]]，chum_aoe 的鱼群 AI、剧场演出版本流程未展开。所有结论基于源码审阅，未运行游戏验证。

## 注册与构造

十文件均为 `Prefab(...)` 直接注册（`pillow_equipment.lua` 按 `pillow_defs` 四材质循环拼名后 `unpack` 返回）。装备槽默认 `EQUIPSLOTS.HANDS`（[equippable.lua:29](../../../raw/current/scripts/components/equippable.lua#L29)），下表只写非默认值。官方中文名核验自 `languages/chinese_s.po` 的 `STRINGS.NAMES.*` 键（见[验证记录](#验证记录)）。

### 枕头与剧场服装（pillow_equipment.lua＋costumes.lua）

| 注册名 | 工厂与槽位 | 差异与证据 |
|---|---|---|
| `bodypillow_petals/kelp/beefalowool/steelwool`（花朵枕头盔甲／海带枕头盔甲／皮弗娄牛枕头盔甲／钢丝绵枕头盔甲） | `MakeBodyPillow(material, pillowdata)` → [Prefab 拼名](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L115)，BODY 槽（[:101](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L101)） | 四材质只差 `pillow_defs` 数据（strengthmult／laglength／defense_amount／prize 值，[pillow_defs.lua](../../../raw/current/scripts/prefabs/pillow_defs.lua#L1-L43)）；manrabbit 装备者改写 `belt` 符号（[:31-35](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L31-L35)）；`named` 组件＋`_named` pristine 优化（[:93](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L93)） |
| `handpillow_petals/kelp/beefalowool/steelwool`（花朵枕头／海带枕头／皮弗娄牛毛枕头／钢丝绵枕头） | `MakeHandPillow` → [Prefab 拼名](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L239)，HANDS 槽（[:219](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L219)） | weapon 伤害 0（`TUNING.PILLOW_DAMAGE`，[tuning.lua:6909](../../../raw/current/scripts/tuning.lua#L6910)）、加长攻击距离 2.5（`PILLOW_HIT_RANGE`，[:6910](../../../raw/current/scripts/tuning.lua#L6911)）；命中时若目标是 `bunnyman_pillowfighting` 小游戏参与者则按 `pillow_common.DoKnockback` 击退，否则只推 0 伤害 attacked 事件（[onweaponattack](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L153-L176)） |
| `costume_doll/queen/king/blacksmith/mirror/tree/fool_body`（玩偶服／女王服／国王服／铁匠服／镜子服／树木服／小丑服） | `makecostume(name, ...)` 共用工厂 → [return 列表](../../../raw/current/scripts/prefabs/costumes.lua#L111-L118)，BODY 槽（[:65](../../../raw/current/scripts/prefabs/costumes.lua#L65)） | 纯外观：OverrideSymbol `swap_body`，有皮肤时走 `OverrideItemSkinSymbol`＋equipskinneditem 事件（[onequip](../../../raw/current/scripts/prefabs/costumes.lua#L19-L27)）；默认可燃，`data.noburn` 可关（[:69-72](../../../raw/current/scripts/prefabs/costumes.lua#L69-L72)）；`scrapbook_specialinfo="COSTUME"`（[:48](../../../raw/current/scripts/prefabs/costumes.lua#L48)） |
| `costume_princess_body`（公主服，YotC） | 同工厂＋princess 三参（[:88](../../../raw/current/scripts/prefabs/costumes.lua#L88)、[:111-118](../../../raw/current/scripts/prefabs/costumes.lua#L111-L118)），BODY 槽 | 唯一带防护与套装的服装：`armor` 耐久＝wilson_health×1×耐久修正、减伤 60%（[tuning.lua:9288-9289](../../../raw/current/scripts/tuning.lua#L9289-L9290)，[挂载](../../../raw/current/scripts/prefabs/costumes.lua#L101-L109)）；`setbonus` 名 `YOTH_PRINCESS`，启用时加 `unluckysource` tag（[princess_onsetbonus_enabled](../../../raw/current/scripts/prefabs/costumes.lua#L94-L99)）；`metal`＋`hardarmor` tag、不可燃、金属 foley（[:88-92](../../../raw/current/scripts/prefabs/costumes.lua#L88-L92)） |

### 火笔与鞍角（firepen.lua＋saddlehorn.lua）

| 注册名 | 工厂与槽位 | 差异与证据 |
|---|---|---|
| `firepen`（火焰笔） | 单注册，[return](../../../raw/current/scripts/prefabs/firepen.lua#L126)，HANDS 槽 | tags `weapon/rangedweapon/rangedlighter/firepen`（[:83-86](../../../raw/current/scripts/prefabs/firepen.lua#L83-L86)）；weapon 伤害 0、射程 (8,10)、投影 `fire_projectile`、命中结算见[快速判断](#快速判断)；`finiteuses` 上限 10（`FIREPEN_MAXUSES`，[tuning.lua:6818](../../../raw/current/scripts/tuning.lua#L6819)）且 `SetDoesNotStartFull(true)`、初始 1 用（[:106-110](../../../raw/current/scripts/prefabs/firepen.lua#L106-L110)）；`skin_sound` 字段预留在命中音（[:18](../../../raw/current/scripts/prefabs/firepen.lua#L18)）；`controlled_burner` 持有者发射时给弹体打同 tag（[:65-69](../../../raw/current/scripts/prefabs/firepen.lua#L65-L69)） |
| `saddlehorn`（鞍角） | 单注册，[return](../../../raw/current/scripts/prefabs/saddlehorn.lua#L69)，HANDS 槽 | 武器伤害 wilson_attack×0.5（`SADDLEHORN_DAMAGE`，[tuning.lua:378](../../../raw/current/scripts/tuning.lua#L378)）、`attackwear=3`（[:44-45](../../../raw/current/scripts/prefabs/saddlehorn.lua#L44-L45)）；挂 `unsaddler` 标记组件（[:48](../../../raw/current/scripts/prefabs/saddlehorn.lua#L48)，[component](../../../raw/current/scripts/components/unsaddler.lua#L1-L5)）；`finiteuses` 10 次（`SADDLEHORN_USES`，[tuning.lua:187](../../../raw/current/scripts/tuning.lua#L187)）、UNSADDLE 动作每次扣 1（[:50-54](../../../raw/current/scripts/prefabs/saddlehorn.lua#L50-L54)） |

### 鱼人工具（restrictedtag 门禁）（merm_tool.lua）

| 注册名 | 工厂与槽位 | 差异与证据 |
|---|---|---|
| `merm_tool`（劣质工具） | `CreateMermTool(data)` → [return](../../../raw/current/scripts/prefabs/merm_tool.lua#L130-L137)，HANDS 槽（默认，未显式赋值） | `restrictedtag = "merm_npc"`（[:109](../../../raw/current/scripts/prefabs/merm_tool.lua#L109)）；`tool` 组件 CHOP/MINE/DIG 效率 0.25、消耗 1/3/3；`AddInherentAction(TILL)`＋`farmtiller`（[:99-L104](../../../raw/current/scripts/prefabs/merm_tool.lua#L99-L104)）；`finiteuses` 50 次（`MERM_TOOL`，[tuning.lua:8307-8317](../../../raw/current/scripts/tuning.lua#L8308-L8318)） |
| `merm_tool_upgraded`（不那么劣质的工具） | 同工厂，[return](../../../raw/current/scripts/prefabs/merm_tool.lua#L139-L145)，HANDS 槽 | 效率 0.5、100 次（`MERM_TOOL_UPGRADED`，[tuning.lua:8319-8329](../../../raw/current/scripts/tuning.lua#L8320-L8330)），其余同上 |

门禁链：`Equippable:IsRestricted` 只对带 `player`/`possessedbody` 的目标生效，玩家缺 `merm_npc` tag 即永远不能装备（[equippable.lua:147-162](../../../raw/current/scripts/components/equippable.lua#L147-L162)）；装备入口在 `Inventory:Equip` 拦截（[inventory.lua:1268-1275](../../../raw/current/scripts/components/inventory.lua#L1268-L1275)），玩家间转移／换装同样判定（[:139-185](../../../raw/current/scripts/components/inventory.lua#L139-L185)），tag 走 `classified.equiprestrictedtag` 同步到客户端（[inventoryitem_replica.lua:431-442](../../../raw/current/scripts/components/inventoryitem_replica.lua#L431-L442)）。`merm_npc` tag 由 `MakeMerm` 共用初始化授予（[merm.lua:1064](../../../raw/current/scripts/prefabs/merm.lua#L1070)），宿主为 merm/mermguard 及 shadow/lunar 变体共六注册（[merm.lua:1773-1778](../../../raw/current/scripts/prefabs/merm.lua#L1782-L1787)）。武器伤害回调对 merm 持有者取其 `combat.defaultdamage` 与工具基础值（wilson_attack×0.5）的较大者（[getDamage](../../../raw/current/scripts/prefabs/merm_tool.lua#L40-L46)）。

### 鱼食、出逃腿靴与清洁扫把（chum.lua＋bootleg.lua＋reskin_tool.lua）

| 注册名 | 工厂与槽位 | 差异与证据 |
|---|---|---|
| `chum`（鱼食） | 单注册，[return](../../../raw/current/scripts/prefabs/chum.lua#L147)，HANDS 槽、`equipstack=true` 整组手持（[:131-134](../../../raw/current/scripts/prefabs/chum.lua#L131-L134)） | `oceanthrowable` 抛掷：水平速度 15、重力 -35（[OnAddProjectile](../../../raw/current/scripts/prefabs/chum.lua#L76-L82)）；落点陆地则变回物品，海面则生成 `chum_aoe` 诱鱼圈并记录投掷者（[OnHit](../../../raw/current/scripts/prefabs/chum.lua#L19-L33)）；`stackable` 默认上限 20（`STACK_SIZE_MEDITEM`，[stackable.lua:28](../../../raw/current/scripts/components/stackable.lua#L28)、[tuning.lua:82](../../../raw/current/scripts/tuning.lua#L82)）；`forcecompostable` 可作绿肥（[:139-140](../../../raw/current/scripts/prefabs/chum.lua#L139-L140)）；reticule 仅在海洋或虚拟海洋实体上有效（[ReticuleValidFn](../../../raw/current/scripts/prefabs/chum.lua#L72-L74)） |
| `bootleg`（出逃腿靴） | 单注册，[return](../../../raw/current/scripts/prefabs/bootleg.lua#L164)，HANDS 槽 | 注释明言 `oceanthrowable` 未完成，改用 `complexprojectile`＋`TOSS` 动作（[:151-157](../../../raw/current/scripts/prefabs/bootleg.lua#L151-L157)，动作 distance 8，[actions.lua:501](../../../raw/current/scripts/actions.lua#L501)）；tag `complexprojectile_showoceanaction`/`action_pulls_up_map`/`allow_action_on_impassable`（[:121-123](../../../raw/current/scripts/prefabs/bootleg.lua#L121-L123)）；地图投掷函数组 `CanTossOnMap`（须站在船上）／`InitMapDecorations`／`CalculateMapDecorations`／`valid_map_actions`（[:70-103](../../../raw/current/scripts/prefabs/bootleg.lua#L70-L103)）；落海生成两个 `oceanwhirlportal` 互设出口成双向传送，落在陆地则恢复为物品（[onhit](../../../raw/current/scripts/prefabs/bootleg.lua#L50-L68)、[CreateOceanWhirlportal](../../../raw/current/scripts/prefabs/bootleg.lua#L11-L21)）；`stackable` 上限 10（[:148-149](../../../raw/current/scripts/prefabs/bootleg.lua#L148-L149)） |
| `reskin_tool`（清洁扫把） | 单注册，[return](../../../raw/current/scripts/prefabs/reskin_tool.lua#L417)，HANDS 槽 | `spellcaster`：可对目标／尸体／背包内施法、`veryquickcast`，`spelltype="RESKIN"`（[:392-398](../../../raw/current/scripts/prefabs/reskin_tool.lua#L392-L398)、[:367](../../../raw/current/scripts/prefabs/reskin_tool.lua#L367)）；每次施法把目标皮肤循环到下一款已拥有皮肤：beard 皮肤走 `prefab.."_beard"` 键（[:120-123](../../../raw/current/scripts/prefabs/reskin_tool.lua#L120-L123)）、其余经 `TheSim:ReskinEntity`，虫洞同步配对端、洞穴入口走 `SHARD_RPC.ReskinWorldMigrator`（[:261-278](../../../raw/current/scripts/prefabs/reskin_tool.lua#L261-L278)）；目标可通过 `reskin_tool_cannot_target_this`／`reskin_tool_target_redirect`／`_playerlink` 改判可否与转接（[:217-223](../../../raw/current/scripts/prefabs/reskin_tool.lua#L217-L223)）；命中特效缩放表 `reskin_fx_info` 按 prefab 覆盖（[:13-101](../../../raw/current/scripts/prefabs/reskin_tool.lua#L13-L101)）；万圣夜活动期挂 `sweeper` 组件（[:403-406](../../../raw/current/scripts/prefabs/reskin_tool.lua#L403-L406)，[[reference/components/sweeper]]）；`fuel` 组件 `MED_FUEL`（[:400-401](../../../raw/current/scripts/prefabs/reskin_tool.lua#L400-L401)） |

### 气球（balloon.lua＋balloonspeed.lua，Wes 专属）

| 注册名 | 工厂与槽位 | 差异与证据 |
|---|---|---|
| `balloon`（气球） | 单注册，[return](../../../raw/current/scripts/prefabs/balloon.lua#L102)，HANDS 槽 | 9 种形状随机（[SetBalloonShape](../../../raw/current/scripts/prefabs/balloon.lua#L23-L28)）、6 色随机染色＋4 种绳结（[SetColour](../../../raw/current/scripts/prefabs/balloons_common.lua#L13-L17)、[SetRopeShape](../../../raw/current/scripts/prefabs/balloons_common.lua#L193-L195)）；存档记 `num`/`colour_idx`（[:30-44](../../../raw/current/scripts/prefabs/balloon.lua#L30-L44)）；共享初始化 `MakeBalloonMasterInit`：`poppable`（`DoPop_Floating`）、`combat` 爆伤 5（`BALLOON_DAMAGE`）、health 1 不可谋杀（[balloons_common.lua:134-156](../../../raw/current/scripts/prefabs/balloons_common.lua#L134-L156)、[tuning.lua:5922-5923](../../../raw/current/scripts/tuning.lua#L5922-L5923)）；漂浮物理胶囊 (0.25,2)、弹性 1（[MakeFloatingBallonPhysics](../../../raw/current/scripts/prefabs/balloons_common.lua#L166-L187)） |
| `balloonspeed`（迅捷气球） | 单注册，[return](../../../raw/current/scripts/prefabs/balloonspeed.lua#L224-L225)，HANDS 槽 | `walkspeedmult` 初值 1.3（[:161-164](../../../raw/current/scripts/prefabs/balloonspeed.lua#L161-L164)），`fueled` MAGIC 燃料、时长 `total_day_time×0.25`（[tuning.lua:5926](../../../raw/current/scripts/tuning.lua#L5926)，即 120 秒）、3 段 section 回调降档到 {1, 1.1, 1.2, 1.3}（[SetHeliumLevel](../../../raw/current/scripts/prefabs/balloonspeed.lua#L88-L104)，[TUNING.BALLOON_SPEEDS](../../../raw/current/scripts/tuning.lua#L5925)）；耗尽加 `fueldepleted` tag（[fueled.lua:76](../../../raw/current/scripts/components/fueled.lua#L76)），显示名变"气球"（[DisplayNameFn](../../../raw/current/scripts/prefabs/balloonspeed.lua#L110-L112)）；丢在地上未耗尽会飞离，15 秒或高度 ≥35 移除，并留 10 秒 `balloon_mapicon` 全局地图图标（[flyoff](../../../raw/current/scripts/prefabs/balloonspeed.lua#L56-L71)、[map_fn](../../../raw/current/scripts/prefabs/balloonspeed.lua#L195-L222)） |
| `balloon_mapicon`（迅捷气球地图图标） | `map_fn` → [return](../../../raw/current/scripts/prefabs/balloonspeed.lua#L224-L225) | 非实体图标代理：MiniMapEntity `balloonspeed.png`＋`globalmapicon` 跟随（[:188-193](../../../raw/current/scripts/prefabs/balloonspeed.lua#L188-L193)），`persists=false` 10 秒自毁 |

两注册的手持表现共用 `BALLOONS.OnEquip_Hand`：清空 swap_object、生成 `balloon_held_child` 联网代理、监听持有者 attacked，经 `ShouldProcOnAttackedOrBlocked`（非重定向、有攻击者、非自身）才掉落（[balloons_common.lua:93-113](../../../raw/current/scripts/prefabs/balloons_common.lua#L93-L113)）；表现层详见 [[reference/prefabs/balloon_held_child]]，Wes 消耗理智做气球的工厂组件见 [[reference/components/balloonmaker]]。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：equippable` | 25 个装备注册的核心：槽位、穿脱回调、`restrictedtag`、`walkspeedmult`；`balloon_mapicon` 不挂 | 玩家／可装备实体；restrictedtag 只拦 player/possessedbody | [[reference/components/equippable]]；[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L162) |
| `组件：inventory` | restrictedtag 的实际装备拦截点；`Equip`/转移/换装三处判定 | 服务端装备行为 | [[reference/components/inventory]]；[Equip 门禁](../../../raw/current/scripts/components/inventory.lua#L1268-L1275) |
| `组件：weapon／tool／finiteuses` | handpillow/firepen/saddlehorn/chum 挂 weapon；merm_tool 挂 tool+weapon；耐久计数 | 服务端；`finiteuses` 耗尽即 `inst.Remove` | [[reference/components/weapon]]、[[reference/components/tool]]、[[reference/components/finiteuses]]；[merm_tool 挂载](../../../raw/current/scripts/prefabs/merm_tool.lua#L95-L118) |
| `组件：armor／setbonus` | `costume_princess_body` 唯一挂 armor 与 setbonus，套装名 `YOTH_PRINCESS` | 头件为 `mask_princess`（[fns.princess](../../../raw/current/scripts/prefabs/hats.lua#L6406)，同名 setbonus 挂载 [:6432-6434](../../../raw/current/scripts/prefabs/hats.lua#L6432-L6434)，启用时同样加 `unluckysource` 并设 `luckuser` 运源 [:6345-6354](../../../raw/current/scripts/prefabs/hats.lua#L6345-L6354)）；树篱掉落表用的 `mask_princesshat` 是另一注册，本页未核验 | [[reference/components/armor]]、[[reference/components/setbonus]] |
| `组件：spellcaster` | `reskin_tool` 的施法入口（canuseontargets/canuseondead/veryquickcast） | 服务端裁决＋客户端 can_cast 预检（[:284-327](../../../raw/current/scripts/prefabs/reskin_tool.lua#L284-L327)） | [[reference/components/spellcaster]] |
| `组件：oceanthrowable／complexprojectile` | `chum` 用 oceanthrowable（含 reticule）；`bootleg` 注释说明弃用 oceanthrowable 改 complexprojectile | 抛掷动作服务端发起 | [[reference/components/oceanthrowable]]、[[reference/components/complexprojectile]]、[[reference/components/reticule]]；[bootleg 注释](../../../raw/current/scripts/prefabs/bootleg.lua#L151) |
| `组件：poppable／combat／health` | 气球共享初始化：被打／点燃即爆，`DoPop_Floating` 面积伤害 5、半径 2 | 服务端；漂浮物理共基 | [[reference/components/poppable]]；[MakeBalloonMasterInit](../../../raw/current/scripts/prefabs/balloons_common.lua#L134-L156) |
| `组件：fueled` | `balloonspeed` 移速档位驱动：MAGIC 燃料 120 秒 3 段 section 回调 | 耗尽加 `fueldepleted` | [[reference/components/fueled]]；[section 回调](../../../raw/current/scripts/prefabs/balloonspeed.lua#L106-L108) |
| `组件：minigame_participator` | `handpillow_*` 命中裁决：`CurrentMinigameType()=="bunnyman_pillowfighting"` 才走击退分支 | 服务端；小游戏由 yotr_fightring 主持 | [[reference/components/minigame_participator]]、[[reference/prefabs/attackfx_handpillow_petals]]；[onweaponattack](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L165-L175) |
| `组件：sweeper` | `reskin_tool` 万圣夜附加：手持扫过地面拾取物 | 仅 `SPECIAL_EVENTS.HALLOWED_NIGHTS` 活动期 | [[reference/components/sweeper]]；[reskin_tool.lua:403-406](../../../raw/current/scripts/prefabs/reskin_tool.lua#L403-L406) |
| `API／数据：TUNING／recipes` | 全部数值与配方：`MERM_TOOL`、`BALLOON_SPEEDS`、`PILLOW_*`、TECH.LOST（不可原型化，[constants.lua:1324](../../../raw/current/scripts/constants.lua#L1324)） | merms 用 `TUNING[data.tuning]` 间接取值（[merm_tool.lua:54](../../../raw/current/scripts/prefabs/merm_tool.lua#L54)），mod 可改表 | [tuning.lua](../../../raw/current/scripts/tuning.lua#L8308-L8330)、[recipes.lua](../../../raw/current/scripts/recipes.lua#L373-L377) |
| `关联 Prefab：chum_aoe／chumpiece` | `chum` 落海产物：20 秒诱鱼圈，每 2.5–5 秒请 `schoolspawner` 生成鱼群（至多 5 次），并撒最多 7 个可食 chumpiece | 服务端 FX，tag `chum` 供鱼群寻路 | 源码 [chum_aoe.lua](../../../raw/current/scripts/prefabs/chum_aoe.lua#L49-L74)；消费者 [oceanfishbrain.lua:219](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219)；鱼群细节 Wiki 待建 |
| `关联 Prefab：oceanwhirlportal` | `bootleg` 落海产物：双向传送门，enter/exit 互设 | `TOSS` 落点为入口、出手点为出口 | 源码 [bootleg.lua:11-21](../../../raw/current/scripts/prefabs/bootleg.lua#L11-L21)；门体行为 Wiki 待建 |
| `关联 Prefab：fire_projectile` | `firepen` 的 `SetProjectile` 弹体 | 远程命中即进 OnAttack | [firepen.lua:116](../../../raw/current/scripts/prefabs/firepen.lua#L116)；弹体页 Wiki 待建 |
| `关联 Prefab：attackfx_handpillow_*` | `handpillow_*` 命中碎屑（玩家偏移 1.0、NPC 2.5） | 与本体同循环注册，本页不持主归属 | [[reference/prefabs/attackfx_handpillow_petals]]；[生成](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L161-L163) |
| `关联 Prefab：balloon_held_child(_client)` | 气球手持表现代理，`OnEquip_Hand` 生成 | 联网代理＋本地动画两层 | [[reference/prefabs/balloon_held_child]]；[生成点](../../../raw/current/scripts/prefabs/balloons_common.lua#L101-L102) |
| `关联 Prefab：book_fire` | `firepen` 的唯一产出者：按周围 16 距离内火数 extinguish 并充能／新发火笔 | Wickerbottom 书本；背包已有火笔先充能 | 源码 [books.lua:614-690](../../../raw/current/scripts/prefabs/books.lua#L614-L690)；`BOOK_FIRE_RADIUS` [tuning.lua:6817](../../../raw/current/scripts/tuning.lua#L6818) |
| `关联 Prefab：merm 工具链` | merm 从 `merm_toolshed(_upgraded)` 取工具，brain 优先升级版 | `merm_toolshed` 供 `merm_tool`、升级版供 upgraded（[merm_supply_structures.lua:497/509](../../../raw/current/scripts/prefabs/merm_supply_structures.lua#L491-L511)）；[PickupTool 优先级](../../../raw/current/scripts/brains/mermbrain.lua#L269-L287) | [[reference/prefabs/merm_splash]] 为 merm 现有页；工具链页 Wiki 待建 |
| `关联 Prefab：sharkboi` | `bootleg` 唯一非配方来源：交易奖励 LaunchAt 发放（最多 5 次） | 服务端 NPC 交易 | [sharkboi.lua:365-367](../../../raw/current/scripts/prefabs/sharkboi.lua#L365-L367) |
| `关联 Prefab：charlie_stage／hedgehound` | 服装来源：剧场 mannequin 配装（[charlie_stage.lua:267](../../../raw/current/scripts/prefabs/charlie_stage.lua#L267)）；树篱刺犬按 `COSTUME_ITEMS` 表随机掉 2 身体＋2 头具，骑士祭坛激活追加公主服（[statueharp_hedgespawner.lua:44-67](../../../raw/current/scripts/prefabs/statueharp_hedgespawner.lua#L44-L67)） | 剧场流程归 playbill 演出，Wiki 待建 | [[reference/components/storytellingprop]] 为相关组件页 |
| `皮肤／条件 → reskin_tool 四皮肤 → 生成与清理 → 入口` | `reskin_tool_bouquet/brush/toilet/wand` 为受限 build（[skins_defs_data.lua:3226-3229](../../../raw/current/scripts/skins_defs_data.lua#L3226-L3229)），施法爆炸 FX 按皮肤查 `SKIN_FX_PREFAB`（[reskin_tool.lua:233-237](../../../raw/current/scripts/prefabs/reskin_tool.lua#L233-L237)） | 皮肤注册走 restricted build，非玩家自选商店皮肤 | 其余物品本快照未见物品皮肤（firepen/merm_tool/saddlehorn 等仅 scrapbook key，[skins_defs_data.lua:5833/6185-6186/6541/6586](../../../raw/current/scripts/skins_defs_data.lua#L5833)） |

## 典型接入

### 给自定义 NPC 或 mod 角色放行 merm_tool

- 前提与选择：`merm_tool` 的 `restrictedtag="merm_npc"` 只对 player/possessedbody 判定：NPC 本来就不受该门禁约束（`IsRestricted` 对非玩家直接返回 false），加 tag 也不会让 NPC 新装备被拒；标签同时是 `getDamage` 的鱼人伤害分支开关，并要求宿主有 `combat`。要放行玩家，得给具体玩家角色加 `merm_npc`（或改 `restrictedtag`）。
- 生效与撤销：`AddTag` 后该实体即可装备 `merm_tool(_upgraded)` 并触发 `getDamage` 的 merm 伤害分支；移除 tag 后已装备的不自动卸下（`Equip` 只在装备时判定），新装备会被拦。
- 原版依据：门禁判定 [equippable.lua:147-162](../../../raw/current/scripts/components/equippable.lua#L147-L162)、tag 授予同源机制 [merm.lua:1064](../../../raw/current/scripts/prefabs/merm.lua#L1070)。

```lua
-- modmain.lua：给自建玩家角色放行 merm_tool；该标签也会启用鱼人伤害分支
AddPlayerPostInit(function(inst)
    if inst.prefab == "mycharacter" and GLOBAL.TheWorld.ismastersim then
        inst:AddTag("merm_npc") -- 装备门禁 + merm_tool 的 getDamage 分支
    end
end)
```

- 调整与验证：替换点为角色 prefab 名；`merm_tool` 消耗 CHOP/MINE/DIG/TILL 耐久，宿主需有 `inventory` 组件才能持握；该标签不改变 `IsRestricted` 对非玩家的早退，也不会把 NPC 变成可拒绝对象。未经游戏运行验证，仅源码依据。

### 扩展 Wes 气球移速档与燃料

- 前提与选择：`balloonspeed` 在模块加载时把 `TUNING.BALLOON_SPEEDS` 存成文件级 `balloon_speeds` 引用，之后都读这张表。**原地改表内元素**会生效；把 `TUNING.BALLOON_SPEEDS` 换成新表不会更新已加载脚本的引用。
- 生效与撤销：section 回调把 0 基 section 映射为 `balloon_speeds[section+1]`（[onfuelsectionchange](../../../raw/current/scripts/prefabs/balloonspeed.lua#L106-L108)）；`SetSections` 在构造时按 `#balloon_speeds - 1` 算出（默认 3）。改数值后，已有实例的 `walkspeedmult` 要等下一次 section 回调才更新，新实例才用新初值。
- 原版依据：[TUNING.BALLOON_SPEEDS](../../../raw/current/scripts/tuning.lua#L5925-L5926)、[SetHeliumLevel](../../../raw/current/scripts/prefabs/balloonspeed.lua#L88-L104)。

```lua
-- modmain.lua：只改既有四档（原地改元素）；替换整张表不会更新已加载脚本的引用
GLOBAL.TUNING.BALLOON_SPEEDS[1] = 1.1
GLOBAL.TUNING.BALLOON_SPEEDS[2] = 1.2
GLOBAL.TUNING.BALLOON_SPEEDS[3] = 1.3
GLOBAL.TUNING.BALLOON_SPEEDS[4] = 1.6
```

- 调整与验证：单位为移速倍率与秒（`total_day_time×0.25`）。真正加第 5 档还要处理：`SetSections` 重设、`balloonspeed*` 图标名（1–3 档专用、第 4 档用裸名）、`balloon_shapes_speed/balloon_N` 动画符号，以及已加载脚本持有的原表引用（只能原地插入或改脚本）。未经游戏运行验证。

## 常见陷阱与限制

- **鞍角是卸鞍工具，不是上鞍工具**：`saddlehorn` 挂的是 `unsaddler` 标记组件，供 `ACTIONS.UNSADDLE`（对 `rideable` 目标 `SetSaddle(doer, nil)`，目标战斗中不可卸）识别手持物（[actions.lua:4190-4200](../../../raw/current/scripts/actions.lua#L4190-L4200)、[componentactions.lua:1883](../../../raw/current/scripts/componentactions.lua#L1883)）；上鞍走 `rideable` 的 saddle 装备链，与本工具无关。
- **玩家永远无法装备 merm_tool**：`restrictedtag="merm_npc"` 且无任何玩家 prefab 授予该 tag；`IsRestricted` 又只对 player/possessedbody 生效，所以它在 NPC 手里是工具、在玩家背包里是摆设。误区是把 `restrictedtag` 理解成"仅限某角色"，实际语义是"缺该 tag 的玩家不可装备"，且对非玩家不生效。
- **bodypillow 的受击监听当前是预留扩展点**：装备时监听 owner 的 `blocked`/`attacked`（[pillow_equipment.lua:37-38](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L37-L38)），但回调要求 `inst._defense_callback` 非空（[:23-28](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L23-L28)），而 `pillow_defs` 四材质均未定义 `defense_callback`，故现版无命中触发效果；bodypillow 的实际作用是 `pillow_common.DoKnockback` 里用 `_defense_amount` 削减击退力度并 `forcelanded`（[pillow_common.lua:11-22](../../../raw/current/scripts/prefabs/pillow_common.lua#L11-L22)），且只在枕头大战小游戏内结算。
- **firepen 无配方**：`recipes.lua` 无 `firepen` 配方，唯一获取是读 `book_fire` 按附近火数充能或新发（[books.lua:642-673](../../../raw/current/scripts/prefabs/books.lua#L642-L673)）；`finiteuses:SetDoesNotStartFull(true)`＋初始 1 用只是模板兜底，实际初始用量由 fire_count 决定。
- **chum 落在陆地变回物品**：`OnHit` 判 `IsOceanAtPoint`，非海面时直接生成 `chum` 物品落地（[chum.lua:19-33](../../../raw/current/scripts/prefabs/chum.lua#L19-L33)）；诱鱼圈只在海面生效，鱼群被 16 距离内的 `chum` tag 吸引（`OCEANFISH_SEE_CHUM_DIST`，[tuning.lua:335](../../../raw/current/scripts/tuning.lua#L335)）。
- **chum 配方不能在原型站解锁**：`TECH.LOST = {MAGIC=10, SCIENCE=10, ANCIENT=10}` 是不可达科技（[constants.lua:1324](../../../raw/current/scripts/constants.lua#L1324)），实际解锁靠隐修者商店 7 级买 `chum_blueprint`（[recipes.lua:946-947](../../../raw/current/scripts/recipes.lua#L946-L947)）；鱼食本体也可直接购买（1 漂流瓶 3 份）。
- **bootleg 必须站在船上对地图使用**：`CanTossOnMap` 要求 `doer:GetCurrentPlatform()` 为 boat（[bootleg.lua:70-73](../../../raw/current/scripts/prefabs/bootleg.lua#L70-L73)）；抛射参数 15/-35 与 chum 相同，但入口是 `TOSS` 动作而非 oceanthrowable（作者注释明言后者未完成，[:151](../../../raw/current/scripts/prefabs/bootleg.lua#L151)）。
- **reskin_tool 不创造皮肤，只循环已拥有皮肤**：`GetNextSkin` 经 `TheInventory:CheckClientOwnership` 过滤（[reskin_tool.lua:141-196](../../../raw/current/scripts/prefabs/reskin_tool.lua#L141-L196)）；找不到下一款时，只要目标当前有皮肤且**不属于** `PREFAB_SKINS_SHOULD_NOT_SELECT`，仍可通过并去掉现有皮肤回到基础外观（[:311-326](../../../raw/current/scripts/prefabs/reskin_tool.lua#L311-L326)）；禁止基础外观的目标才在无候选时失败。皮肤列表随活动开关（`SKINS_EVENTLOCK`，[:162](../../../raw/current/scripts/prefabs/reskin_tool.lua#L162)）。
- **边界：`balloon` 配方的 `Ingredient("balloons_empty", 0)` 数量为 0**：Wes 造气球的实际消耗是 5 点理智，balloons_empty 是触发物（[recipes.lua:373-374](../../../raw/current/scripts/recipes.lua#L373-L374)），不要按"0 个气球"字面理解。
- **边界：迅捷气球燃料耗尽不消失**：只降回 1.0 移速并改名"气球"；`OnDropped` 只在**未耗尽**时启动 `flyoff`（[balloonspeed.lua:73-79](../../../raw/current/scripts/prefabs/balloonspeed.lua#L73-L79)），耗尽后掉落即静止物品，不会飞走。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394。逐行实读十个主文件全文，核验 26 个本页主归属注册名（含 `unpack` 动态注册）、equipslot（BODY×12、HANDS×13，另 1 个 balloon_mapicon 非装备）、restrictedtag 门禁链（组件→replica→inventory 三消费端）、皮肤（reskin_tool 四 restricted build，其余物品未见物品皮肤）、配方与获取（recipes.lua＋books.lua＋sharkboi.lua＋charlie_stage.lua＋statueharp_hedgespawner.lua＋yotr_rabbitshrine.lua＋merm_supply_structures.lua）。官方中文名经 `D:/Code/dst-wiki-develop/dst-wiki/.venv/Scripts/python.exe` 以 UTF-8 解析 `languages/chinese_s.po` 的 `STRINGS.NAMES.*` 键逐个取回（firepen=火焰笔、merm_tool=劣质工具、merm_tool_upgraded=不那么劣质的工具、reskin_tool=清洁扫把、saddlehorn=鞍角、chum=鱼食、bootleg=出逃腿靴、balloonspeed=迅捷气球、balloon=气球、8 件枕头与 8 件服装名），.po 已计入 sources，其文件变化表示名称依据待复核。
- 未覆盖：`chum_aoe` 鱼群 AI 与 schoolspawner 细节、`oceanwhirlportal` 门体行为、`fire_projectile` 弹体参数、charlie_stage 演出版本流程、`balloon_held_child` 表现层（引用已有页）均只登记入口未展开；全页未运行游戏验证（validation: source-reviewed）。
- 结构检查：validate_page.py 通过，实测 153 行／35790 字节，size_class: complex（上限 480 行／48 KiB）；validate_source_links.py --strict 0 error 0 warning（152 条源码引用、35 个来源文件）；validate_source_hashes.py check 通过（基线已由主代理 record）。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 11 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
