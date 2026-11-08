---
title: "基础法杖族（staffs）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "原版宝石法杖的共用工厂与冰、火、传送、闪现、拆解五类施法效果的实现、消耗与修改入口。"
aliases: ["staffs", "icestaff", "icestaff2", "icestaff3", "firestaff", "telestaff", "orangestaff", "greenstaff", "冰魔杖", "闪冻魔杖", "深冻魔杖", "火魔杖", "传送魔杖", "懒人魔杖", "拆解魔杖", "冰杖", "火杖", "传送杖", "懒人杖", "拆解杖", "Ice Staff", "Flash Freeze Staff", "Deep Freeze Staff", "Fire Staff", "Telelocator Staff", "The Lazy Explorer", "Deconstruction Staff"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/staff_projectile.lua", "prefabs/telebase.lua", "prefabs/king_cane.lua", "prefabs/shellweaver.lua", "prefabskin.lua", "prefabskins.lua", "components/spellcaster.lua", "components/blinkstaff.lua", "components/weapon.lua", "components/projectile.lua", "components/combat.lua", "components/freezable.lua", "components/staffsanity.lua", "components/shadowlevel.lua", "components/quaker.lua", "components/weather.lua", "actions.lua", "stategraphs/SGwilson.lua", "componentutil.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 基础法杖族（staffs）

## 快速判断

- 职责与适用：[staff.lua](../../../raw/current/scripts/prefabs/staff.lua#L301-L309) 一次注册 9 支宝石法杖，全部共用 `commonfn`（物品、装备持握、皮肤、有限次数、漂浮）与 [staff_common.lua](../../../raw/current/scripts/prefabs/staff_common.lua#L49) 的分色效果工厂。本页覆盖 icestaff／icestaff2／icestaff3、firestaff、telestaff、orangestaff、greenstaff 七个注册与共用结构；yellowstaff、opalstaff 同文件注册但已有独立页，本页只互链。选 `complex` 是因为七支法杖与国王魔杖共用 staff_common，分档与各效果分支必须整体对照，不能按注册名拆页。
- 优先使用：查次数／精神／耐久与共用初始化看[注册与构造](#注册与构造)；改冰火攻击看[冰火攻击链](#冰火攻击链)，改传送、闪现、拆解看对应小节；新物品想复用原版法杖效果时参照 [kingcane 的复用模式](#复用与换装kingcane)。施法动作门禁与消耗流程复用 [[reference/components/spellcaster#原版法杖的施法模式]]。
- 重要副作用：施法与攻击经持有者的 staffsanity 或 sanity 路径扣精神；冰杖冻结与灭火、火杖引燃或投喂燃料；传送杖在洞穴只触发地震且仍扣次数；拆解杖拆除配方实体且宝石类材料不返还。证据入口在各小节。
- 修改边界：效果与消耗逻辑全部在主模拟（`TheWorld.ismastersim` 分支之后），客户端仅 reticule 与皮肤；本页不展开玩家 SG 施法状态机全文、动作过滤器（componentactions）与 yellowstaff／opalstaff 的产物、月台转换机制。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icestaff` | blue → blue_common("blue",1) → commonfn → 主模拟 stafffns.blue、20 次 | 冷度 1；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L174-L176)、[注册](../../../raw/current/scripts/prefabs/staff.lua#L301) |
| `icestaff2` | 同上，coldness=2，动画后缀 "2"（bluestaff2） | 冷度 2；无独立皮肤键；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L177-L179) |
| `icestaff3` | 同上，coldness=3，动画后缀 "3" | 冷度 3；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L180-L182) |
| `firestaff` | red → commonfn("red",…,{"firestaff","weapon","rangedweapon","rangedlighter"},true,true) → 主模拟 stafffns.red、20 次 | 引燃；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L117-L147) |
| `telestaff` | purple → commonfn("purple",…,{"nopunch"},true,true) → 主模拟 stafffns.purple、5 次 | spellcaster 施放传送；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L184-L202) |
| `orangestaff` | orange → commonfn("orange",…,{"weapon"},true,true) → 主模拟 stafffns.orange + weapon(CANE_DAMAGE) + walkspeedmult、20 次 | 闪现杖兼手杖武器；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L247-L270) |
| `greenstaff` | green → commonfn("green",…,{"nopunch"},true,true) → 主模拟 stafffns.green、5 次 | 拆解；[工厂](../../../raw/current/scripts/prefabs/staff.lua#L231-L245) |
| `yellowstaff`／`opalstaff` | 同文件的 yellow／opal 工厂与 20／50 次 | 效果本体见 [[reference/prefabs/yellowstaff]]、[[reference/prefabs/opalstaff]]；[注册](../../../raw/current/scripts/prefabs/staff.lua#L308-L309) |

### 共用工厂 commonfn

- 客户端段（SetPristine 之前）：Transform／AnimState／SoundEmitter／Network、`MakeInventoryPhysics`、bank 与 build 均为 "staffs"、动画 `colour.."staff"..suffix`、按 tags 加标签、漂浮换装数据；冰杖的 `hasshadowlevel` 还会先加 `shadowlevel` pristine 标签做优化。[commonfn](../../../raw/current/scripts/prefabs/staff.lua#L31-L67)
- 主模拟段：`finiteuses`（耗尽回调播 gem_shatter 后 Remove）、`inspectable`、`inventoryitem`、`tradable`、`equippable`（hasskin 全部为 true，装备时按皮肤 build 换持握符号）；hasshadowlevel 时挂 `shadowlevel` 组件并设 `TUNING.STAFF_SHADOW_LEVEL`（本版 1，仅 opal 传 false 不挂）。[挂载](../../../raw/current/scripts/prefabs/staff.lua#L69-L113)、[耗尽](../../../raw/current/scripts/prefabs/staff.lua#L13-L16)、[常量](../../../raw/current/scripts/tuning.lua#L2858)
- 各分色工厂在主模拟再设满 finiteuses：`ICESTAFF_USES`／`FIRESTAFF_USES` 20、`TELESTAFF_USES`／`GREENSTAFF_USES` 5、`ORANGESTAFF_USES` 20。[常量](../../../raw/current/scripts/tuning.lua#L175-L178)、[L221](../../../raw/current/scripts/tuning.lua#L221)
- staff_common 的 `fns.red/blue/…` 系列函数带第二参数 notinit，供复用宿主补打标签；普通法杖传默认值，kingcane 传 true。[签名示例](../../../raw/current/scripts/prefabs/staff_common.lua#L228-L232)、[复用](../../../raw/current/scripts/prefabs/king_cane.lua#L158)

### 名称核验与皮肤

- 名称键（简中 `languages/chinese_s.po`）：`STRINGS.NAMES.ICESTAFF`“冰魔杖”[L344095-L344097](../../../raw/current/scripts/languages/chinese_s.po#L344095-L344097)、`ICESTAFF2`“闪冻魔杖”[L344100-L344102](../../../raw/current/scripts/languages/chinese_s.po#L344100-L344102)、`ICESTAFF3`“深冻魔杖”[L344105-L344107](../../../raw/current/scripts/languages/chinese_s.po#L344105-L344107)、`FIRESTAFF`“火魔杖”[L342815-L342817](../../../raw/current/scripts/languages/chinese_s.po#L342815-L342817)、`TELESTAFF`“传送魔杖”[L350310-L350312](../../../raw/current/scripts/languages/chinese_s.po#L350310-L350312)、`ORANGESTAFF`“懒人魔杖”／The Lazy Explorer[L346690-L346692](../../../raw/current/scripts/languages/chinese_s.po#L346690-L346692)、`GREENSTAFF`“拆解魔杖”[L343445-L343447](../../../raw/current/scripts/languages/chinese_s.po#L343445-L343447)。“冰杖／火杖／传送杖／懒人杖／拆解杖”为派发任务提供的俗称，一并收入 aliases。
- 皮肤清单按基础注册名记入 `PREFAB_SKINS`：firestaff 10 款、greenstaff 4 款、icestaff 9 款、orangestaff 8 款、telestaff 4 款；icestaff2／icestaff3 无键即无皮肤。[清单](../../../raw/current/scripts/prefabskins.lua#L842-L854)、[L963-L968](../../../raw/current/scripts/prefabskins.lua#L963-L968)、[L1093-L1103](../../../raw/current/scripts/prefabskins.lua#L1093-L1103)、[L1397-L1406](../../../raw/current/scripts/prefabskins.lua#L1397-L1406)、[L1891-L1896](../../../raw/current/scripts/prefabskins.lua#L1891-L1896)
- 皮肤初始化共用 `staff_init_fn`／`caststaff_init_fn`（换 build、覆写 grass 符号、按皮肤数据写 `skin_sound`／`skin_castsound` 等音效字段）；orangestaff 皮肤可另换 blink FX 与拖尾，yellowstaff 皮肤写 morph_skin 供月台转换。[共用入口](../../../raw/current/scripts/prefabskin.lua#L3280-L3306)、[音效字段](../../../raw/current/scripts/prefabskin.lua#L20-L51)、[orangestaff](../../../raw/current/scripts/prefabskin.lua#L3308-L3342)。攻击音消费：[火](../../../raw/current/scripts/prefabs/staff_common.lua#L72)、[冰](../../../raw/current/scripts/prefabs/staff_common.lua#L170-L172)；本次未逐款核验皮肤视觉 build。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：finiteuses` | commonfn 挂载并设 OnFinished；分色工厂设满次数，各效果回调内 `Use(1)` | 全体主模拟；耗尽碎裂移除 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/staff.lua#L74-L75)、[满次数](../../../raw/current/scripts/prefabs/staff.lua#L133-L134) |
| `组件：weapon` | firestaff/icestaff：damage 0、射程 8/10、弹射物、OnAttack；orangestaff：CANE_DAMAGE 与 walkspeedmult=CANE_SPEED_MULT | 仅红蓝橙挂 weapon；紫绿黄无 weapon（nopunch 只影响攻击表现） | [[reference/components/weapon#关键接口]]；[红](../../../raw/current/scripts/prefabs/staff_common.lua#L141-L157)、[橙武器](../../../raw/current/scripts/prefabs/staff.lua#L260-L263)、[常量](../../../raw/current/scripts/tuning.lua#L373)、[L414](../../../raw/current/scripts/tuning.lua#L414) |
| `组件：spellcaster` | telestaff：SetSpellFn(teleport_func)＋canuseontargets／frominventory／canonlyuseonlocomotorspvp；greenstaff：SetSpellFn(destroystructure)＋canonlyuseonrecipes | 施法门禁详见组件页“原版法杖的施法模式” | [[reference/components/spellcaster#原版法杖的施法模式]]；[紫](../../../raw/current/scripts/prefabs/staff_common.lua#L469-L479)、[绿](../../../raw/current/scripts/prefabs/staff_common.lua#L770-L784) |
| `组件：blinkstaff` | orangestaff 挂载，SetFX 沙尘前后景、onblinkfn=onblink（扣精神与次数） | 主模拟；皮肤可换 FX 与音效 | [[reference/components/blinkstaff#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/staff_common.lua#L538-L547)、[onblink](../../../raw/current/scripts/prefabs/staff_common.lua#L483-L497) |
| `组件：shadowlevel` | commonfn 按 hasshadowlevel 挂载，默认等级 STAFF_SHADOW_LEVEL=1 | 除 opal 外全体；供麦斯威尔影子系统与描述文本 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/staff.lua#L107-L110) |
| `组件：inspectable／inventoryitem／tradable／equippable／floater／hauntable` | 共用物品检视、拾取、交易、穿脱持握（含皮肤）、漂浮与作祟 | 全体；红黄设 floater 水面换装偏移，主模拟 MakeHauntableLaunch 后追加分色稀有作祟反应 | [[reference/components/equippable#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[作祟示例](../../../raw/current/scripts/prefabs/staff_common.lua#L156) |
| `组件：reticule／itemmimicconfig` | orange_client 配手柄选点（twinstickrange 15）；greenstaff 加 itemmimicconfig 强制 CASTSPELL 鉴伪 | 客户端／主模拟 | [[reference/components/reticule#关键接口]]；[橙](../../../raw/current/scripts/prefabs/staff_common.lua#L529-L536)、[绿](../../../raw/current/scripts/prefabs/staff_common.lua#L780-L783) |
| `API／数据：ACTIONS.CASTSPELL` | 动作 fn 取手上或目标物品的 spellcaster，CanCast 通过后 CastSpell(target,pos,doer) | 玩家 SG castspell 状态帧 13 播施法音、帧 53 PerformBufferedAction | [[reference/components/spellcaster#关键接口]]；[动作](../../../raw/current/scripts/actions.lua#L3445-L3473)、[定义](../../../raw/current/scripts/actions.lua#L455)、[SG 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1315)、[执行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16240-L16251) |
| `API／数据：ACTIONS.BLINK` | 动作 fn 直接调 `blinkstaff:Blink(act_pos, doer)`，SG 转 quicktele 状态 | 仅持闪现杖右键；动作 distance 36 | [定义](../../../raw/current/scripts/actions.lua#L457)、[fn](../../../raw/current/scripts/actions.lua#L3583-L3585)、[SG 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1345-L1352) |
| `API／数据：精神与次数` | 施法/攻击统一走 `staffsanity:DoCastingDelta` 优先、否则 `sanity:DoDelta`；共用函数另支持 rechargeable+dodischarge 分支 | 七支普通法杖无 rechargeable，永远走 finiteuses；dodischarge 仅国王魔杖进入腐化态后为 true（`OnCorruptedFn` 置位、`OnUncorruptedFn` 复位） | [[reference/components/staffsanity#关键接口]]；[模式示例](../../../raw/current/scripts/prefabs/staff_common.lua#L483-L497)、[标志](../../../raw/current/scripts/prefabs/king_cane.lua#L195) |
| `关联 Prefab：ice_projectile／fire_projectile` | weapon 弹射物，速度 50、无伤害；命中生成 shatter（非 freezable 目标）或 fire_fail_fx（未燃烧目标）后自删 | 主模拟生成、本端可见；[定义](../../../raw/current/scripts/prefabs/staff_projectile.lua#L47-L84) | [[reference/prefabs/shatter]]；fire_fail_fx 无页，源码 [staff_projectile.lua](../../../raw/current/scripts/prefabs/staff_projectile.lua#L25-L45) |
| `关联 Prefab：telebase` | teleport_func 用 FindNearestActiveTelebase(x,y,z,nil,1,"purplegem") 找激活传送台作目的地；到达消耗其一颗宝石 | 未选中时随机传送到地图节点中心；telebase Wiki 待建 | [查找](../../../raw/current/scripts/prefabs/telebase.lua#L84-L112)、[扣宝石 onteleto](../../../raw/current/scripts/prefabs/telebase.lua#L263) |
| `关联 Prefab：cutgrass／sand_puff_large_*／collapse_small` | 火杖投喂燃料、闪现沙尘、绿杖作祟灰尘 | staff_common prefabs 表声明且实际生成 | [声明](../../../raw/current/scripts/prefabs/staff_common.lua#L1-L48)、[作祟灰尘](../../../raw/current/scripts/prefabs/staff_common.lua#L759) |
| `关联 Prefab：kingcane` | 换宝石时 cleargemstaff 清效果，再按宝石名从 stafffns 查表装载本族行为（notinit=true）；其传送不走闪电（notlightningteleport），落地改 atrium_portal_fx | 机制见 [[reference/prefabs/staffs-more|法杖补充族]] 的国王魔杖小节；[[reference/prefabs/king_cane_fx]] 为其特效 | [装载](../../../raw/current/scripts/prefabs/king_cane.lua#L148-L171)、[清除](../../../raw/current/scripts/prefabs/staff_common.lua#L887-L906)、[传送标志](../../../raw/current/scripts/prefabs/king_cane.lua#L317-L318) |
| `资源：staffs／swap_staffs／staff_projectile 动画` | 本体、持握与弹射物；皮肤 build 经 skin 数据替换 | 资源声明与解包内容分开核验 | [声明](../../../raw/current/scripts/prefabs/staff.lua#L1-L6)、[弹射物](../../../raw/current/scripts/prefabs/staff_projectile.lua#L1-L4) |

### 冰火攻击链

攻击命中路径：combat:DoAttack 见 `weapon.components.projectile` 时走 LaunchProjectile 并 return（[combat.lua](../../../raw/current/scripts/components/combat.lua#L1185-L1188)）；弹射物飞抵后 `Projectile:Hit` 换回持物者再进一次 DoAttack（projectile 非空），在 [combat.lua L1223-L1225](../../../raw/current/scripts/components/combat.lua#L1223-L1225) 调 `weapon:OnAttack`——即 onattack 回调与战斗耐久扣减（[weapon.lua L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)，每次命中 1 次，efficientuser 可减）都发生在弹射物命中时，而非出手瞬间。冰火杖 attackwear 未设，基础每次 1 次。

- 火杖 onattack_red：扣 SANITY_SUPERTINY（1）精神；对未燃烧的 burnable 目标按状态引燃（`Ignite(true, attacker)`，仅 canlight 或有 combat 的目标）、对接受 BURNABLE 燃料的目标生成 cutgrass 投喂、对冻结目标先解冻；再 `AddColdness(-1)`、唤醒沉睡者、`SuggestTarget` 并推 `attacked`（damage 0）。[实现](../../../raw/current/scripts/prefabs/staff_common.lua#L63-L117)
- 冰杖 onattack_blue：扣 1 精神；燃烧目标 `Extinguish`、阴燃目标 `SmotherSmolder`；对 freezable 目标 `AddColdness(icestaff_coldness)` 并 SpawnShatterFX。freezable 默认 resistance 1，累计冷度达阈值即冻结，因此 2／3 档可一击冻结高冷抗目标。[实现](../../../raw/current/scripts/prefabs/staff_common.lua#L161-L204)、[AddColdness](../../../raw/current/scripts/components/freezable.lua#L137-L161)
- 动作资格：`extinguisher` 标签让燃烧／阴燃目标可被攻击（[CanExtinguishTarget](../../../raw/current/scripts/components/combat.lua#L1037-L1042)）；`rangedlighter` 让未燃烧的可点燃目标走点火攻击（[CanLightTarget](../../../raw/current/scripts/components/combat.lua#L1044-L1056)）。Willow 控火技能会让火杖弹射物带 `controlled_burner`（[发射回调](../../../raw/current/scripts/prefabs/staff_common.lua#L57-L61)、[技能](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L62)）。

### 传送链（telestaff）

施放入口为 CASTSPELL；teleport_func 里 `target = target or caster`，目标优先级：小游戏参与者（`minigame_participator:GetMinigame()`）→ `teleportedoverride:GetDestTarget()` → 被套索实体（`hitchable.hitched`）→ 目标非水栖时才找最近激活 telebase（紫宝石优先）。落点另按 `teleportedoverride:GetDestPosition()` → 无 loctarget 时随机位置（`getrandomposition`）→ `teletopos()` → 实体位置取值；水栖读 `locomotor:IsAquatic()`。[目标选择](../../../raw/current/scripts/prefabs/staff_common.lua#L431-L449)、[随机位置](../../../raw/current/scripts/prefabs/staff_common.lua#L252-L281)

流程（teleport_start）：先扣次数（1 次），再判定 `TheWorld:HasTag("cave")`——洞穴内改推 `ms_miniquake`（rad 3、num 5、duration 1.5，quaker 监听）并直接返回，不传送；地表则闪落雷（`ms_sendlightningstrike`，weather 监听；临时置 burning 位防自燃）、扣施放者 SANITY_HUGE（50）精神、`ms_deltamoisture` 加 500 湿度；玩家进 forcetele 状态，3 秒后 teleport_continue 位移，再 1 秒 teleport_end 落地播 castsound；非玩家目标跳过 3+1 秒等待，即时 continue/end。[流程](../../../raw/current/scripts/prefabs/staff_common.lua#L352-L449)、[quaker](../../../raw/current/scripts/components/quaker.lua#L743)、[weather](../../../raw/current/scripts/components/weather.lua#L817)、[L823](../../../raw/current/scripts/components/weather.lua#L823)

限制：`noteleport` 标签或 [IsTeleportLinkingPermittedFromPoint](../../../raw/current/scripts/componentutil.lua#L1450-L1466) 为假（ WagPunk 竞技场屏障、Vault、Charlie 竞技场）时 `no_teleport=true`——仍扣次数、仍有雷击与精神消耗，但不位移。

### 闪现链（orangestaff）

右键走 ACTIONS.BLINK：blinkstaff:Blink 校验 `IsTeleportingPermittedFromPointToPoint`（VRS／竞技场屏障，[componentutil.lua](../../../raw/current/scripts/componentutil.lua#L1390-L1408)）与落点可通行后，立即生成沙尘并回调 onblinkfn——即 [onblink](../../../raw/current/scripts/prefabs/staff_common.lua#L483-L497)：扣 SANITY_MED（15）精神与 1 次耐久——并推 `onblink`；位移安排在 0.25 秒后的 `OnBlinked`，届时再次校验落点与屏障，通过才 `Physics:Teleport`，失败则只留下已付消耗。[Blink](../../../raw/current/scripts/components/blinkstaff.lua#L62-L97) 期间 SG 有 onstartblinking/onstopblinking 挂钩；选点圈由 client helper 的 reticule 提供（[blinkstaff_reticuletargetfn](../../../raw/current/scripts/prefabs/staff_common.lua#L525-L527)）。本杖另有手杖武器面：CANE_DAMAGE（=wilson_attack*0.5，即 17）与 1.25 倍移速（[staff.lua L260-L263](../../../raw/current/scripts/prefabs/staff.lua#L260-L263)），并 `SetIgnoreCombatDurabilityLoss(true)` 让近战不耗次数。

### 拆解链（greenstaff)

spellcaster 限定 `canonlyuseonrecipes`：CanCast 只放行 `AllRecipes[target.prefab]` 存在且非 `no_deconstruction` 的目标（[spellcaster.lua L190-L191](../../../raw/current/scripts/components/spellcaster.lua#L190-L191)）。destroystructure：`ingredient_percent` 先取目标百分比再除以配方 `numtogive`（finiteuses 可用 `decon_ignores_finiteuses` 跳过；fueled／armor 只对带 `inventoryitem` 的目标取百分比）；正量材料（amount>0）按 `max(1, ceil(amount×percent))` 返还、0 量仍返 0；以 "gem" 结尾的材料不返还（"preciousgem" 例外）。拟态目标只走 `TurnEvil` 掉噩梦燃料，不再进入普通材料返还/清空/移除分支；非拟态目标才继续倾倒库存、释放子实体、推 `ondeconstructstructure`（telebase 监听它掉出宝石，[telebase.lua L285](../../../raw/current/scripts/prefabs/telebase.lua#L285)）并 Remove（堆叠只拆一个）；扣施放者 SANITY_MEDLARGE（20）精神与 1 次耐久。[实现](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L747)

### 复用与换装（kingcane）

国王魔杖声明 `Asset("SCRIPT", "scripts/prefabs/staff_common.lua")`，合并 staff_common.prefabs 作依赖；玩家嵌入新宝石时按 `GetGemId(item.prefab)` 从 `stafffns` 查表调用对应分色函数（notinit=true）装载行为（含标签），并先把旧宝石返还给玩家；cleargemstaff 恢复手杖本体。本体无 finiteuses，挂 rechargeable：进入腐化态时 `dodischarge=true`（`OnCorruptedFn` 置位、`OnUncorruptedFn` 复位）使共用函数改走 `Discharge(KING_CANE_*_COOLDOWN)`（紫 90 秒、橙 10 秒、绿 120 秒，seg_time=30 推导值），未充能时加 `nomagiccast` 直接禁施法。[装载](../../../raw/current/scripts/prefabs/king_cane.lua#L148-L171)、[清除](../../../raw/current/scripts/prefabs/staff_common.lua#L887-L906)、[充能回调](../../../raw/current/scripts/prefabs/king_cane.lua#L118-L132)、[冷却常量](../../../raw/current/scripts/tuning.lua#L9759-L9764)

## 典型接入

### 调整冰杖冷度或火杖引燃行为

- 前提与选择：改冷度只动 `blue_common` 的 coldness 实参或 onattack_blue 的 `AddColdness`；改引燃条件改 onattack_red 的 burnable 分支。两者都在 staff_common，会影响全部冰杖档位与 kingcane 蓝宝石形态。
- 生效与撤销：主模拟命中时生效；无需撤销逻辑，弹射物命中即回调。注意 onattack_blue 对已冻结目标不再推 `attacked`（避免二次仇恨），删除该判定会改变仇恨表现。
- 原版依据：[冰](../../../raw/current/scripts/prefabs/staff_common.lua#L161-L204)、[火](../../../raw/current/scripts/prefabs/staff_common.lua#L63-L117)、[冷度档位](../../../raw/current/scripts/prefabs/staff.lua#L149-L182)。

### 复用原版法杖效果制作新物品（原版参照 kingcane）

- 前提与选择：自建 prefab 需在 assets 声明 `Asset("SCRIPT", "scripts/prefabs/staff_common.lua")` 并把 `staff_common.prefabs` 对应分色并入依赖表，否则弹射物／特效不预载；运行时先 `local staff_common = require("prefabs/staff_common")` 取 `staff_common.fns`，再 `stafffns.<gem>(inst, true)`。**本节以国王魔杖为实现参照**：绿分支解引用 `itemmimicconfig`、蓝分支读冷度等宿主字段，新宿主须按所选分色先备好组件与字段。
- 生效与撤销：装载在主模拟；撤销可照 `cleargemstaff` 的**国王魔杖专用**清理顺序（移除 spellcaster／blinkstaff／weapon 后重建国王魔杖武器、重挂 hauntable、要求 `itemmimicconfig`），但自建宿主须按自己挂过的组件与标签逐项配对撤销；直接 AddComponent 不配对移除会残留标签。
- 原版依据与调整：完整流程见 [king_cane.lua](../../../raw/current/scripts/prefabs/king_cane.lua#L148-L171)；精神倍率接入见 [[reference/components/staffsanity#自制法杖采用同一人物倍率]]。

## 常见陷阱与限制

- **误区：普通宝石法杖“自动充能”**：共用函数里的 `rechargeable + dodischarge` 分支只对 kingcane 成立；七支普通法杖只有 finiteuses，施法/命中一律 `Use(1)`。[分支](../../../raw/current/scripts/prefabs/staff_common.lua#L368-L372)
- **边界：传送杖在洞穴仍扣次数**：次数扣除先于洞穴判定，洞穴内结果是一次小型地震而非传送；对 `noteleport`／禁传送区目标同理。[顺序](../../../raw/current/scripts/prefabs/staff_common.lua#L368-L378)
- **边界：拆解杖不返还宝石**：以 "gem" 结尾的配方材料被跳过（"preciousgem" 例外），且 `no_deconstruction` 配方（如 purplegem、弹射物弹药）不可拆。[过滤](../../../raw/current/scripts/prefabs/staff_common.lua#L667-L673)
- **边界：火杖对燃烧目标既不引燃也不灭火**：onattack_red 的引燃分支要求目标未燃烧；灭火是冰杖职责。对接受 BURNABLE 燃料的目标（营火等）是投喂 cutgrass。[分支](../../../raw/current/scripts/prefabs/staff_common.lua#L74-L99)
- **误区：icestaff2/3 是独立 build 或换皮肤**：同 "staffs" build，仅动画后缀与冷度不同；PREFAB_SKINS 无 icestaff2／icestaff3 键，即无皮肤。[分档](../../../raw/current/scripts/prefabs/staff.lua#L149-L182)、[清单键](../../../raw/current/scripts/prefabskins.lua#L1093-L1103)
- **边界：orangestaff 的伤害与移速不是 blinkstaff 附带**：weapon(CANE_DAMAGE) 与 walkspeedmult 在 orange 工厂单独挂载；紫绿杖连 weapon 都没有，nopunch 仅改变攻击表现。[挂载](../../../raw/current/scripts/prefabs/staff.lua#L260-L263)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394；核验 staff.lua 全部工厂与 9 项注册、staff_common 七支分色工厂与作祟反应（yellow／opal 效果本体只核验到 createlight 归属，不复制到本页）、staff_projectile、telebase 查找／扣宝石、kingcane 装载／清除、皮肤清单与 init、spellcaster／blinkstaff／weapon／projectile／combat／freezable／staffsanity／shadowlevel 消费端、CASTSPELL／BLINK 动作链、洞穴地震与天气事件消费、TUNING、配方与 .po 官方名。
- 游戏运行：未运行；未审校玩家 SG 施法状态机全文、PVP 传送、皮肤资源包内容、componentactions 动作过滤与模组兼容。
- 结构检查：validate_page.py 实测 129 行 / 26,983 字节左右（complex 档 480 行 / 64 KiB 内）；validate_source_links.py --strict 0 error 0 warning；validate_source_hashes.py check 通过（基线已由主代理 record）。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
