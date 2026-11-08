---
title: "阴郁回旋镖（voidcloth_boomerang）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版虚布武器 voidcloth_boomerang：充能时 weapon:SetProjectile 发射代理投射物（本体不离手、不可接回），武器/位面伤害随飞行尺寸 5→27.2 缩放、对月相 ×1.25；在场弹体 3 枚（套装 4）即放电退化近战，耐久掷出与放电近战都磨、耗尽残破可修复。"
aliases: ["voidcloth_boomerang", "阴郁回旋镖", "Gloomerang", "虚布回旋镖", "投掷武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/voidcloth_boomerang.lua", "prefabs/forge_repair_kits.lua", "prefabs/voidcloth.lua", "prefabs/shadow_forge.lua", "prefabs/walter.lua", "prefabs/wolfgang.lua", "components/weapon.lua", "components/projectile.lua", "components/equippable.lua", "components/finiteuses.lua", "components/planardamage.lua", "components/damagetypebonus.lua", "components/rechargeable.lua", "components/shadowlevel.lua", "components/forgerepairable.lua", "components/forgerepair.lua", "components/spdamageutil.lua", "components/combat.lua", "components/sanity.lua", "components/locomotor.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "recipes.lua", "recipes_filter.lua", "fx.lua", "widgets/itemtile.lua", "brains/wx78_possessedbodybrain.lua", "entityscript.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 阴郁回旋镖（voidcloth_boomerang）

## 快速判断

- 职责与适用：`prefabs/voidcloth_boomerang.lua` 同文件注册三实体——本体 `voidcloth_boomerang`（物品＋装备：weapon＋finiteuses＋equippable HANDS＋rechargeable＋shadowlevel）、持握光层 `voidcloth_boomerang_fx`、代理投射物 `voidcloth_boomerang_proj`。特色是**充能时本体不离手**：`weapon:SetProjectile` 经 weapon 组件发射独立弹体，伤害/位面都挂在弹体上且随飞行尺寸缩放；放电退化为默认 10 伤近战。无 SG/Brain、无皮肤注册、无 quagmire 分支。[工厂与注册](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L674-L677)
- 优先使用：复刻"代理投射武器"按[投掷机制对照](#与普通回旋镖的投掷机制对照)＋[[reference/components/weapon#关键接口]] 的 `SetProjectile`/`LaunchProjectile` 接线；只查数值用[数值表](#注册与构造)；弹体飞行细节对照 [[reference/prefabs/boomerang#关联机制导航]]。
- 重要副作用：每次掷出即磨 1 耐久（未命中也磨），放电近战每击同样扣 1；在场弹体达缓存上限（3，套装 4）即放电退化近战，计数低于上限才回充；装备期约 −0.056 理智/秒（magic 型）＋1.1 倍移速；耐久耗尽变残破（移除 equippable），需虚空修补套件修复。[掷出磨损](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L452-L458)、[放电](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L202-L205)
- 修改边界：本页覆盖三注册的工厂、组件与飞行/修复机制；fx 细节归 [[reference/prefabs/voidcloth_boomerang_fx]]、launch/impact 特效归 [[reference/prefabs/voidcloth_boomerang_launch_fx]]；玩家 SG 动作只给入口（见对照节）。

## 注册与构造

| 注册名/变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `voidcloth_boomerang` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；残破移除 equippable、修复重组件。[注册](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L674-L675) |
| `voidcloth_boomerang_fx` | 同文件 `FollowSymbolFxFn`：联网跟随光层骨架 | 持握/浮水视觉；`persists = false`。机制归 [[reference/prefabs/voidcloth_boomerang_fx]]；[工厂](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L638-L670) |
| `voidcloth_boomerang_proj` | 同文件 `ProjectileFn`：联网代理投射物 | 自带 weapon/planardamage/damagetypebonus，`persists = false`；见下两节。[工厂](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L477-L550) |

`voidcloth_boomerang_launch_fx`/`voidcloth_boomerang_impact_fx` **不在本文件注册**：二者是 [fx.lua 表代理](../../../raw/current/scripts/fx.lua#L3657-L3676)，由弹体 OnThrown/OnHit 生成，见 [[reference/prefabs/voidcloth_boomerang_launch_fx]]。

本体共同初始化（[fn](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L233-L327)）：`MakeInventoryPhysics`→`RemovePhysicsColliders`→bank＝build `boomerang_voidcloth`＋`idle`→公共端标签 `shadow_item`/`magicweapon`/`rangedweapon`/`show_broken_ui`/`weapon`/`shadowlevel`/`rechargeable`→floater＋`isbroken` net_bool＋`SetIsBroken(false)`→`SetPristine`（非主模拟只听 `isbrokendirty`）→主模拟：随机帧同步 fx（[L290-L297](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L290-L297)）→SetupComponents（equippable）→inspectable→inventoryitem→finiteuses(85)→weapon(射程＋投射物)→rechargeable→shadowlevel→`OnPreLoad`→`MakeForgeRepairable`→`MakeHauntableLaunch`。[equippable 默认 HANDS](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.VOIDCLOTH_BOOMERANG`＝阴郁回旋镖（Gloomerang）；配方描述＝"它会一直回来！"；残破态检视名 `BROKEN_FORGEDITEM`＝"它破损了，但是我可以把它修好。"。[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351349-L351352)、[RECIPE_DESC](../../../raw/current/scripts/languages/chinese_s.po#L359519-L359521)、[BROKEN](../../../raw/current/scripts/languages/chinese_s.po#L9959-L9961)

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键/来源 | 实际值 |
|---|---|---|
| 弹体武器伤害 | `TUNING.VOIDCLOTH_BOOMERANG_DAMAGE`＝{min 5, max `wilson_attack`×.8} | 5→27.2 随尺寸 Lerp（源码推导） |
| 弹体位面伤害 | `TUNING.VOIDCLOTH_BOOMERANG_PLANAR_DAMAGE`＝{min 5, max `wilson_attack`×.8} | 5→27.2 同步缩放 |
| 对月相加成 | `TUNING.VOIDCLOTH_BOOMERANG_VS_LUNAR_BONUS` | 目标带 `lunar_aligned` ×1.25 |
| 耐久上限 | `TUNING.VOIDCLOTH_BOOMERANG_USES` | 85；掷出与放电近战命中各磨 1（默认值，受磨损/效率修正） |
| 武器射程 | `..._ATTACK_DIST`＝10/`..._ATTACK_DIST_MAX`＝14 | weapon `attackrange`/`hitrange`（[设置](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L311)） |
| 弹体速度 | `..._PROJECTILE.LAUNCH_SPEED`/`RETURN_SPEED` | 15（去程）/7.5（回程起步，随时间 ×max(1, t/3) 加速） |
| 在场弹体上限 | `..._PROJECTILE.MAX_ACTIVE`/`SETBONUS_MAX_ACTIVE` | 3/4（套装） |
| 套装伤害 | `WEAPONS_VOIDCLOTH_SETBONUS_DAMAGE_MULT`/`..._PLANAR_DAMAGE` | 伤害 ×1.1、位面 +5（[调值](../../../raw/current/scripts/tuning.lua#L7819-L7820)） |
| 装备理智 | `dapperness`＝−`DAPPERNESS_MED`（[tuning.lua#L2215](../../../raw/current/scripts/tuning.lua#L2215)） | ≈−0.056/秒（约 −26.7/游戏天）；`is_magic_dapperness`＝true（Walter 也受影响） |
| 移速倍率 | `TUNING.VOIDCLOTH_BOOMERANG_SPEEDMULT` | 1.1（equippable.walkspeedmult） |
| 影子等级 | `TUNING.VOIDCLOTH_BOOMERANG_SHADOW_LEVEL` | 3（[调值](../../../raw/current/scripts/tuning.lua#L2884)） |

获取途径（`SpawnPrefab("voidcloth_boomerang")` 字面检索无命中；map／worldgen 布局也未检索到引用，WeaponBoon 候选不含它——落点仍按未穷尽核验）：

| 途径 | 证据 |
|---|---|
| 配方（唯一）：纯粹恐惧（horrorfuel）×4＋暗影碎布（voidcloth）×2，`TECH.SHADOWFORGING_TWO`（＝{SHADOWFORGING=2}，[constants.lua#L1335](../../../raw/current/scripts/constants.lua#L1335)），`station_tag="shadow_forge"`＋`nounlock=true` → 需暗影术基座（树 `PROTOTYPER_TREES.SHADOW_FORGE`，[tuning.lua#L1452-L1454](../../../raw/current/scripts/tuning.lua#L1452-L1454)、[接线](../../../raw/current/scripts/prefabs/shadow_forge.lua#L152)） | [配方](../../../raw/current/scripts/recipes.lua#L870)、[过滤分组](../../../raw/current/scripts/recipes_filter.lua#L455-L463)、[原料名锚点](../../../raw/current/scripts/languages/chinese_s.po#L351340-L351342) |

## 关联机制导航

### 与普通回旋镖的投掷机制对照

同文件 `projectile` 组件机制见 [[reference/prefabs/boomerang#关联机制导航]] 与 [[reference/components/projectile#关键接口]]，不在此复制；两族差异：

| 环节 | 普通回旋镖 `boomerang` | 阴郁回旋镖 `voidcloth_boomerang` |
|---|---|---|
| 发射 | **本体即投射物**：combat 走 `projectile` 标签分支 DropItem 后 `projectile:Throw`（SG 另按 `thrown` 标签进 `throw`），武器脱手 | **代理投射**：`weapon:SetProjectile("voidcloth_boomerang_proj")` → combat 走 `CanRangedAttack` 分支 [LaunchProjectile](../../../raw/current/scripts/components/combat.lua#L1185-L1189)；本体无 `thrown` 标签，SG 落普通 `attack` 状态，武器不离手（[接线](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L312)、[SG 选动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1410-L1416)） |
| 伤害来源 | 武器本体的 weapon 组件（固定 27.2） | 弹体自身：`has_damage_set=true` 时 [Hit 以弹体为 weapon](../../../raw/current/scripts/components/projectile.lua#L243-L246) 结算，伤害/位面/对月全读弹体组件且随尺寸每帧重设 |
| 磨损时机 | 命中结算（含回程自伤再磨 1） | 掷出瞬间（弹体 OnThrown 调 [本体 weapon:OnAttack](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L452-L458)）；放电近战每击也经本体 `weapon:OnAttack` 扣 1，默认值受磨损/效率修正 |
| 回程 | 可接回（`SetCanCatch`＋CATCH 动作）；未接住回程弹会命中掷出者自伤 | **不可接回**（无 `SetCanCatch`，cancatch 默认 false）；回程由弹体自身 updatelooper 驱动，接近/越过/失效即 Remove，[不与任何实体结算](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L382-L416) |
| 目标丢失 | owner＝玩家（有 combat），飞满武器射程才 Miss | owner＝武器本体（无 combat），目标失效立即 [Miss 返回](../../../raw/current/scripts/components/projectile.lua#L428-L438) |

弹体飞行链：`Throw` 后 15 速追踪（homing、hitdist 1、range 20）→尺寸 0.4→1 按 `ATTACK_DIST/LAUNCH_SPEED` 放大，[OnUpdateFn](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L418-L439) 每帧按 `Remap(scale,.4,1,0,1)` 重设伤害/位面→Hit 生成 impact_fx（Miss 不生成）→[ReturnToThrower](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L347-L359) 解除边界、7.5 起步加速返航（缩 0.4）→1 格内（[收集半径](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L332)）Remove。掷出前 [DelayVisibility(FRAMES)](../../../raw/current/scripts/components/weapon.lua#L143-L145) 隐藏 1 帧。

### 组件与导航

| 类型/对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki/源码入口及证据 |
|---|---|---|---|
| 组件：weapon（本体） | 仅 `SetRange(10,14)`＋`SetProjectile`；**无 SetDamage**：放电近战为默认 10 伤，主实体无 planardamage/damagetypebonus | 主模拟 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L310-L312)、[默认伤害](../../../raw/current/scripts/components/weapon.lua#L13) |
| 组件：rechargeable | 不当冷却用：在场弹体达缓存上限（3/套装 4）→ `Discharge(999999)` 清射程/投射物；注销后计数低于 `max_projectiles` 才 `SetPercent(1)` 恢复 | 主模拟；`OnPreLoad` 清 rechargeable 存档，读档恒满充（[顺序](../../../raw/current/scripts/entityscript.lua#L1988-L1996)） | [[reference/components/rechargeable#关键接口]]；[计数](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220)、[充放电回调](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L202-L210) |
| 组件：equippable | 默认 HANDS；`dapperness`（magic 型，[汇总](../../../raw/current/scripts/components/sanity.lua#L511-L523)、[Walter 限定](../../../raw/current/scripts/prefabs/walter.lua#L754-L760)）＋`walkspeedmult=1.1`（[结算](../../../raw/current/scripts/components/locomotor.lua#L191-L200)）；OnEquip/OnUnequip 只做 ARM 显隐＋`ClearOverrideSymbol`＋SetFxOwner/SetBuffOwner（**不设 swap_object 贴图**） | 主模拟；持握外观归 [[reference/prefabs/voidcloth_boomerang_fx]] | [[reference/components/equippable#关键接口]]；[SetupComponents](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L133-L140)、[OnEquip](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L16-L24) |
| 组件：finiteuses | 上限＝当前＝85；磨损由 `weapon:OnAttack` 统一扣（掷出/近战各 1 次）；归零加 `usesdepleted` 走 MakeForgeRepairable | 主模拟 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L306-L308)、[归零链](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：planardamage/damagetypebonus（弹体） | 弹体挂 `planardamage`（随尺寸 SetBaseDamage）＋`damagetypebonus:AddBonus("lunar_aligned", 1.25)`；位面经 SpDamageUtil 汇入（[Wiki 待建](../../../raw/current/scripts/components/spdamageutil.lua#L22-L29)） | 主模拟；命中时以弹体身份被读 | [[reference/components/planardamage#关键接口]]、[[reference/components/damagetypebonus#关键接口]]；[弹体组件](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L526-L533) |
| 组件：shadowlevel | `SetDefaultLevel(3)`；消费端（麦斯威尔影像按领导者 shadowlevel 加伤）同铥矿棒页 | 主模拟；pristine 同名标签 | [[reference/components/shadowlevel#关键接口]]、[[reference/prefabs/ruins-bat#关联机制导航]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L318-L319) |
| 组件：floater | 直接 `AddComponent("floater")`（无 MakeInventoryFloatable）：`SetBankSwapOnFloat(true, -6, {sym_build="boomerang_voidcloth"})`，残破/正常两档尺寸 | 公共端；`isbroken` net_bool 双端消费 | [[reference/components/floater#关键接口]]；[SetIsBroken](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L163-L177)、[两档](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L149-L159) |
| 组件：forgerepairable | `MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH, OnBroken, OnRepaired)`：耐久归零先卸下再进残破态；REPAIR 回满后 OnRepaired 重组件 | 主模拟；`SetRepairable(current < total)` 未满即可修 | [[reference/components/forgerepairable#关键接口]]、[[reference/components/forgerepair#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)、[修复链](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L179-L198)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1290-L1296) |
| 组件：inspectable/inventoryitem/hauntable | 检视（残破换 BROKEN_FORGEDITEM 名）；拾取/掉落；`MakeHauntableLaunch` | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L303-L304) |
| 同步：netvar | `isbroken`（isbrokendirty→双端 floater 两档）；fx 的 `equiptoggle` | 公共端声明、双端消费 | [isbroken](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L264)、[[reference/prefabs/voidcloth_boomerang_fx]] |
| API/数据：标签 | `magicweapon`→被 Wolfgang 位面技能加成门排除（该门还要求 item 自带 planardamage，本武器位面在弹体上）；`rangedweapon`→远程攻击豁免虚空断档（同 boomerang）；`show_broken_ui`→物品栏耐久/残破 UI；`weapon`/`projectile`/`rechargeable`/`shadowlevel` 为 pristine 镜像 | 运行时读取 | [wolfgang.lua#L568-L585](../../../raw/current/scripts/prefabs/wolfgang.lua#L568-L585)、[combat.lua#L815](../../../raw/current/scripts/components/combat.lua#L815)、[itemtile.lua#L601](../../../raw/current/scripts/widgets/itemtile.lua#L601) |
| API/数据：`_bonusenabled` 套装门禁 | `SetBuffOwner` 监听持有者 HEAD 槽 equip/unequip：装备 `voidclothhat`（按注册名判定）→上限 3→4；弹体掷出时复制标记加位面 +5、伤害 ×1.1 | 主模拟；换戴其他头饰/摘帽即取消 | [[reference/prefabs/shadow-battleaxe#装备副作用与虚空风帽套装]]（同套装模式）；[SetBuffOwner](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L49-L89)、[帽名](../../../raw/current/scripts/languages/chinese_s.po#L351344-L351347) |
| 消费端：WX-78 附身机体脑 | `#GLOOMERANG_HACK`：附身机体持此武器按 `ATTACK_DIST` 判攻击距离，放电时不主动攻击 | 宿主为被附身机体；读 rechargeable:IsCharged | [脑](../../../raw/current/scripts/brains/wx78_possessedbodybrain.lua#L589-L594)、[L707-L709](../../../raw/current/scripts/brains/wx78_possessedbodybrain.lua#L707-L709) |
| 关联 Prefab：`voidcloth_boomerang_proj` | 唯一生成点＝weapon `LaunchProjectile`；`OnRemoveEntity` 从 `_projectiles` 注销并重算充能；`OnEntitySleep` 离屏 Stop＋Remove | 主模拟；`persists = false` 不存档 | [生成](../../../raw/current/scripts/components/weapon.lua#L129-L145)、[注销](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L339-L345)、[休眠](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L470-L473) |
| 关联 Prefab：launch_fx/impact_fx | OnThrown 在攻击者朝目标一侧生成 launch；OnHit 在目标身后偏移生成 impact（Miss 不生成） | 主模拟；均为 fx.lua 表代理 | [[reference/prefabs/voidcloth_boomerang_launch_fx]]；[OnThrown](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L460-L467)、[OnHit](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L364-L375) |
| 关联 Prefab：`voidcloth_kit` | 修复材料（材质 voidcloth，耗 1 个）；暗影碎布原料**无** forgerepair 组件、不能直接修 | 主模拟；REPAIR 动作 | [MakeKit](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56-L58)、[原料](../../../raw/current/scripts/prefabs/voidcloth.lua#L27-L33)、[Kit 名](../../../raw/current/scripts/languages/chinese_s.po#L351354-L351357) |
| 资源：anim | `boomerang_voidcloth.zip`（bank＝build，含 idle/projectile/broken/swap_loop_*/launch_fx/impact_fx） | 静态声明；皮肤贴图未注册（prefabskin/skinprefabs 无条目，OnEquip 无皮肤接线） | [assets](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L1-L4)、[prefabs 声明](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L6-L12) |

## 典型接入

### 调整阴郁回旋镖的弹体上限、返程速度与耐久

- 前提与选择：服务端 `modmain.lua`。返程速度每次现读 `TUNING`；弹体上限在构造/套装切换时缓存到 `inst.max_projectiles`，改 `TUNING` 影响之后新建实例，已有实例在下次套装状态切换（戴/摘虚空风帽）时也会重读上限（modmain 直挂白名单，[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）；耐久是实例构造值，用 `AddPrefabPostInit` 改。`TheWorld` 非 modmain 白名单字段，须经 `GLOBAL.` 读取。
- 生效与撤销：TUNING 改动停用模组后需自行还原（原值见[数值表](#注册与构造)）；postinit 只影响新实例，已有实例当前耐久按存档恢复。
- 原版依据：上限缓存于 `inst.max_projectiles`（构造读一次、套装切换重读），`OnProjectileCountChanged` 只比缓存（[计数](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220)）；回程速度每次 `OnUpdateFn` 现读 `RETURN_SPEED`（[L402-L414](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L402-L414)）。

```lua
-- modmain.lua：在场弹体上限 5（套装 6）、回程速度 10、耐久上限 120
TUNING.VOIDCLOTH_BOOMERANG_PROJECTILE.MAX_ACTIVE = 5
TUNING.VOIDCLOTH_BOOMERANG_PROJECTILE.SETBONUS_MAX_ACTIVE = 6
TUNING.VOIDCLOTH_BOOMERANG_PROJECTILE.RETURN_SPEED = 10

AddPrefabPostInit("voidcloth_boomerang", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.finiteuses:SetMaxUses(120)
end)
```

- 调整与验证：改 `TUNING` 的新值要到下次套装状态切换才对已有实例生效，否则仍是旧值；也可直接改字段并调 `OnProjectileCountChanged`。回程尺寸插值＝`BOOMERANG_DISTANCE/RETURN_SPEED`（默认约 1.6 秒，提前抵达/越过/失效即移除）；去程＝`ATTACK_DIST/LAUNCH_SPEED`。耐久单位为耐久点/次磨损（掷出或近战命中）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：像普通回旋镖那样"脱手飞行、可以接回"**：本体经 `SetProjectile` 代理投射、不离手；弹体未设 `SetCanCatch`，玩家收不到 CATCH 动作；回程弹接近投掷者即 Remove、不与路径实体结算（见[对照节](#与普通回旋镖的投掷机制对照)）。
- **边界：耐久在掷出瞬间磨 1，未命中也磨**：磨损由弹体 `OnThrown` 调本体 `weapon:OnAttack` 完成，命中/未命中不重复扣；放电近战每击同样经 `weapon:OnAttack` 扣 1，所以 85 耐久不等于 85 次掷出。[OnThrown](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L452-L458)
- **边界：放电不是冷却计时**：只反映在场弹体数是否达到缓存上限；增删与换帽时比较缓存，在场数量降到当前上限以下才回充恢复投射；套装退回普通上限后，须降到新上限以下。[计数](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220)、[缓存](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L38-L46)
- **边界：伤害随距离缩放，贴身掷出只保底 5**：去程 0.4→1 尺寸对应伤害/位面 5→27.2 线性；近距命中接近下限，满程命中才到 max；回程伤害不参与结算。[OnUpdateFn](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L418-L439)
- **边界：残破可修复而非消失**：耐久归零移除 equippable 进残破态（`broken` 标签、换检视名、隐藏 fx）；修复物是虚空修补套件，暗影碎布原料无 forgerepair 组件、直接修无效。[OnBroken/OnRepaired](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L179-L198)
- **边界：读档恒为满充**：`OnPreLoad` 删除存档里的 rechargeable 数据（弹体亦不持久化），放电状态不跨存档；要"掷空后冷却"需另接逻辑。[OnPreLoad](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L224-L229)
- **边界：投掷动作的玩家 SG 不在本页**：本体无 `thrown` 标签，SG 选动作落到普通 `attack`（不是普通回旋镖的 `throw`），再由 combat 的 `CanRangedAttack` 分支发射代理弹体。[combat 分支](../../../raw/current/scripts/components/combat.lua#L1185-L1189)、[SG 选动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1410-L1416)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394。核验 prefabs/voidcloth_boomerang.lua 全文；weapon/projectile/finiteuses/equippable/rechargeable/shadowlevel/planardamage/damagetypebonus/forgerepairable 关键方法；combat 的 CanRangedAttack 与 OnAttack 调用点；sanity/locomotor；REPAIR 门禁；OnPreLoad 顺序；wx78 脑/wolfgang/itemtile；tuning 全键；配方与 shadow_forge 树；.po 六键；皮肤与弹体生成点反查。2026-11-07 第四轮复检：`max_projectiles` 缓存、掷出/近战双磨损、返航 `BOOMERANG_DISTANCE` 常量、SG 标签分流。未覆盖：fx 页表现、throw SG、动画资源。
- 游戏运行：未运行。27.2 上限、返程加速曲线、−0.056 理智/秒等均为源码推导，非运行测量。
- 结构检查：validate_page.py 通过（139 行／24643 字节，standard 档内）；validate_source_links.py --strict 通过（0 error 0 warning）。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
