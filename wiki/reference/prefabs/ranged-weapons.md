---
title: "远程手持武器族（blowdart 五变体、houndstooth_blowpipe、slingshot 五变体、mosquitobomb）"
created: "2026-11-07"
updated: "2026-11-09"
type: prefab
summary: "远程手持武器族的 13 个主注册与 2 个关联注册分三条机制：吹箭族 weapon+projectile 同体、发射即整支脱手且效果写在 weapon.onattack 由命中链回放；嚎弹炮与弹弓 weapon+container 弹舱、装弹后 SetProjectile 指向独立 _proj 弹体；蚊子炸弹 weapon+complexprojectile 抛物线并范围爆蚊群；全部 HANDS 槽、效果组件挂主模拟。"
aliases: ["blowdart_sleep", "blowdart_fire", "blowdart_pipe", "blowdart_yellow", "blowdart_walrus", "houndstooth_blowpipe", "slingshot", "slingshotex", "slingshot999ex", "slingshot2", "slingshot2ex", "slingshotmodkit", "mosquitobomb", "催眠吹箭", "火焰吹箭", "吹箭", "雷电吹箭", "嚎弹炮", "Howlitzer", "可靠的弹弓", "弹弓", "蚊子炸弹", "Skeeter Bomb", "弹弓野外作业包", "远程武器", "远程武器族", "远程手持武器"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/blowdart.lua", "prefabs/houndstooth_blowpipe.lua", "prefabs/slingshot.lua", "prefabs/mosquitobomb.lua", "prefabs/impact.lua", "prefabs/walrus.lua", "prefabs/houndstooth.lua", "prefabs/slingshotmodkit.lua", "prefabs/slingshotpart_defs.lua", "prefabs/slingshotammo.lua", "prefabskins.lua", "components/projectile.lua", "components/complexprojectile.lua", "components/combat.lua", "components/combat_replica.lua", "components/weapon.lua", "components/container.lua", "components/equippable.lua", "components/stackable.lua", "components/sleeper.lua", "components/grogginess.lua", "containers.lua", "componentactions.lua", "componentutil.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "recipes.lua", "tuning.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 远程手持武器族（blowdart 五变体、houndstooth_blowpipe、slingshot 五变体、mosquitobomb）

## 快速判断

- 职责与适用：三组机制分支的手持远程武器——①吹箭五变体是"武器与弹体同体"：weapon＋projectile 组件挂在同一实体，发射即整支脱手飞行，命中效果写在 `weapon:SetOnAttack` 回调；②嚎弹炮（houndstooth_blowpipe）与弹弓五变体是"弹舱武器"：weapon＋container 同体，装弹后 `weapon:SetProjectile` 指向独立的 `_proj` 弹体注册；③蚊子炸弹（mosquitobomb）走 complexprojectile 抛物线、落地范围伤害并爆出蚊群。[吹箭工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L48-L114)、[弹舱接线](../../../raw/current/scripts/prefabs/slingshot.lua#L397-L403)、[抛物线配置](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L290-L295)
- 优先使用：做"命中效果消耗品武器"抄吹箭的 onattack 接线；做"弹舱＋弹药 prefab"武器抄弹弓/嚎弹炮的 OnAmmoLoaded→SetProjectile 与 OnProjectileLaunched 扣弹；做怪物远程武器可参考海象的[内联 weapon＋生成弹体](../../../raw/current/scripts/prefabs/walrus.lua#L88-L104)；单纯"落地 AoE 爆炸"更适用 [[reference/prefabs/throwables]] 的炸弹族。
- 重要副作用：催眠吹箭 `AddSleepiness(1, 15)`（有 sleeper 组件目标）或 grogginess 兜底；火焰吹箭引燃＋解冻＋0 火焰伤害；吹箭（pipe）100 伤害；雷电吹箭 60 电属性伤害＋电火花 FX；嚎弹炮 34 物理＋34 位面、对影 ×1.1；弹弓本体 0 伤、伤害全由弹药决定；蚊子炸弹范围 3 内 59.5（wilson_attack×1.75 源码推导）并生成 4 只（Wurt 技能 +2）跟随蚊群。[睡眠](../../../raw/current/scripts/prefabs/blowdart.lua#L125-L147)、[火焰](../../../raw/current/scripts/prefabs/blowdart.lua#L174-L200)、[蚊群](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L30-L73)
- 修改边界：标签、动画、reticule 与 netvar 挂公共端，weapon/projectile/container 等玩法组件挂主模拟；弹弓五变体仅 `slingshot_sharpshooter`（Walter）可装备并经 linkeditem 绑定主人；`blowdart_walrus` 是海象 AI 弹体、不是玩家武器。[装备限制](../../../raw/current/scripts/prefabs/slingshot.lua#L542-L546)、[绑定主人](../../../raw/current/scripts/prefabs/slingshot.lua#L415-L426)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `blowdart_sleep`（催眠吹箭） | `common("idle_purple", {"tranquilizer"})` 公共端工厂 → 主模拟 weapon(0 伤、射程 8-10)＋projectile(speed 60)＋`SetOnAttack(sleepattack)` | 命中：sleeper `AddSleepiness(1,15)`（无 sleeper 组件走 grogginess）、`SuggestTarget`、推 0 伤 `attacked`；飞行 `dart_purple`＋NOCLICK＋persists=false。[工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L149-L163)、[效果](../../../raw/current/scripts/prefabs/blowdart.lua#L125-L147) |
| `blowdart_fire`（火焰吹箭） | 同上，tags `firedart`、`SetOnAttack(fireattack)` | 引燃 burnable、freezable `Unfreeze`、`DoFireDamage(0)`、`SuggestTarget`；飞行 `dart_red`。[效果](../../../raw/current/scripts/prefabs/blowdart.lua#L174-L200)、[工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L202-L218) |
| `blowdart_pipe`（吹箭） | 同上，无附加 tag，weapon 伤害 `PIPE_DART_DAMAGE`(100)、装备换 `swap_blowdart_pipe` 持握 | 纯伤害弹，onattack 未设。[工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L235-L250) |
| `blowdart_yellow`（雷电吹箭） | 同上，weapon `YELLOW_DART_DAMAGE`(60)＋`SetElectric()`，onattack `SpawnElectricHitSparks` | 电属性 stimuli 走 combat 的电伤加成链；prefabs 数组多 `electrichitsparks`。[工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L266-L282) |
| `blowdart_walrus` | `common("idle_pipe", {"NOCLICK"}, true)` 第三参移除物理碰撞 → 主模拟 persists=false、projectile range 25（`WALRUS_DART_RANGE`）、homing=false、LaunchOffset(3,2,0)、HitDist √5 | 海象 AI 专用弹体；海象手持武器是另一内联实体（见[导航](#关联机制导航)）。[工厂](../../../raw/current/scripts/prefabs/blowdart.lua#L288-L310) |
| `houndstooth_blowpipe`（嚎弹炮） | `WeaponFn` → 公共端 weapon 标签＋floatable → 主模拟 weapon(`UNARMED_DAMAGE` 10)＋container(弹舱)＋equippable（皮肤分支 OnEquip） | 空舱近战 10；装弹才 `SetProjectile("houndstooth_proj")`＋射程 12-16＋增删 `blowpipe`/`rangedweapon` 标签。[工厂](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L179-L233)、[装弹](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80-L96) |
| 关联：`houndstooth_proj` | 装弹后由嚎弹炮设为独立弹体。[装弹](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80-L96) | 弹体自带物理与位面伤害，结算归属见 [[reference/prefabs/impact#伤害归属]]。 |
| `slingshot`（可靠的弹弓） | `MakeSlingshot` 包装工厂 → 公共端 `slingshotmods`＋`linkeditem`＋netvar(bandid/handleid/buildname) 分层图标 → 主模拟 weapon(0 伤、射程 10-14、projectile offset 1)＋container 1 弹槽＋burnable | 装弹 `SetProjectile(<弹药>_proj)`；安骨架部件替换为 `slingshot2`。[工厂](../../../raw/current/scripts/prefabs/slingshot.lua#L482-L586)、[装弹](../../../raw/current/scripts/prefabs/slingshot.lua#L397-L403) |
| `slingshotex` | 同上＋公共端 aoecharging 蓄力准星（reticulecharging）；`SetPrefabNameOverride("slingshot")` | 蓄力 k（0–1）传 `SetChargedMultiplier(k*k)`，弹体按默认 ×2 伤害／×1.25 速度上限插值；frame_wagpunk_0 部件替换而来。[蓄力](../../../raw/current/scripts/prefabs/slingshot.lua#L657-L673) |
| `slingshot999ex` | slingshotex＋主模拟 container `EnableInfiniteStackSize(true)` | 弹舱堆叠无视 STACK_SIZE_PELLET 上限（默认每发扣 1，拟态皮筋可免）。[工厂](../../../raw/current/scripts/prefabs/slingshot.lua#L730-L738) |
| `slingshot2` | slingshot＋container 双弹槽；发射后槽 1 空时自动从槽 2 补弹 | frame_bone 部件替换而来。[补弹](../../../raw/current/scripts/prefabs/slingshot.lua#L748-L757) |
| `slingshot2ex` | slingshot2＋aoetargeting/aoespell：CASTAOE 用槽 2 特殊弹药朝准星位置额外射一发（`overrideammoslot=2`） | frame_gems 部件替换而来。[副手弹](../../../raw/current/scripts/prefabs/slingshot.lua#L786-L817) |
| `slingshotmodkit`（弹弓野外作业包） | 独立文件注册：主模拟 `slingshotmodder` 开改装界面 | 不在 slingshot.lua 注册；配方与部件链见获取节。[工厂](../../../raw/current/scripts/prefabs/slingshotmodkit.lua#L10-L22) |
| `slingshotparts_fx` | 持握跟随 FX：六个 `swap_1_to_5`…`swap_17` 段跟随 `swap_object` 符号，OnEquip 重建、OnUnequip 移除 | 客户端注册进 owner `highlightchildren`。[FX 工厂](../../../raw/current/scripts/prefabs/slingshot.lua#L876-L908)、[装备重建](../../../raw/current/scripts/prefabs/slingshot.lua#L290-L328)；工厂、高亮登记与清理归 [[reference/prefabs/staff_lunarplant_fx#父挂跟随与高亮]] |
| `mosquitobomb`（蚊子炸弹；携带物见 [[reference/prefabs/mosquitomusk|小痒痒]]） | `fn` → 公共端 reticule＋weapon/projectile/complexprojectile 三标签 → 主模拟 locomotor＋stackable＋complexprojectile(speed 15、gravity −35)＋weapon(0 伤、射程 8-10)＋equippable(equipstack) | 命中 `DoImpactDamage`（范围 3、排除蚊/飞行/隐形；投掷者是玩家时排除其他玩家）＋`SpawnMosquitos`；飞行 `spin_loop`＋改物理。[工厂](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L239-L314)、[命中](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L154-L166) |

五吹箭 `return` 共五注册＋公共 onhit 只做 impact FX 与自移除；impact/electrichitsparks 是声明依赖与命中特效。[return 核对](../../../raw/current/scripts/prefabs/blowdart.lua#L313-L317)、[onhit](../../../raw/current/scripts/prefabs/blowdart.lua#L31-L41)

共同发射链（吹箭）：SG `blowdart` 状态按帧 `PerformBufferedAction` → combat `DoAttack` 见武器带 projectile 组件 → `DropItem` 脱手 → `projectile:Throw(player, target)` → 更新逼近 → `Hit` 内对攻击者再走一次 `combat:DoAttack(目标, 弹体, 弹体)`（临时 ignorehitrange）→ 伤害按弹体 weapon 结算并回调 `weapon:OnAttack`（即各 onattack 效果）→ `SetOnHitFn` 出 impact 并移除弹体。[SG 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10436-L10530)、[投掷分支](../../../raw/current/scripts/components/combat.lua#L1168-L1175)、[命中回放](../../../raw/current/scripts/components/projectile.lua#L236-L266)、[OnAttack 回调](../../../raw/current/scripts/components/combat.lua#L1223-L1225)

弹舱武器链：装弹事件 → `weapon:SetProjectile(<弹药或 houndstooth_proj>)` → 攻击时 `CanRangedAttack()` 成立 → `weapon:LaunchProjectile` 生成弹体并 `Throw` → `OnProjectileLaunched` 从弹舱 `RemoveItem(..., false)` 拆 1 发消耗（拟态皮筋 30%＋幸运可免扣，2ex 副手弹扣槽 2）。[LaunchProjectile](../../../raw/current/scripts/components/weapon.lua#L123-L153)、[拆 1 发](../../../raw/current/scripts/components/container.lua#L1019-L1043)

名称经简中 .po `msgctxt` 核验：`BLOWDART_SLEEP`＝催眠吹箭、`BLOWDART_FIRE`＝火焰吹箭、`BLOWDART_PIPE`＝吹箭（英文 Blow Dart）、`BLOWDART_YELLOW`＝雷电吹箭、`HOUNDSTOOTH_BLOWPIPE`＝嚎弹炮（Howlitzer）、`SLINGSHOT`＝可靠的弹弓（Trusty Slingshot）、`SLINGSHOTMODKIT`＝弹弓野外作业包、`MOSQUITOBOMB`＝蚊子炸弹（Skeeter Bomb）；slingshotex/999ex/2/2ex 无独立名称键，经 `SetPrefabNameOverride("slingshot")` 显示"可靠的弹弓"，被绑定的弹弓再走 `SLINGSHOT_FMT` 显示"<主人名>的弹弓"。[四吹箭](../../../raw/current/scripts/languages/chinese_s.po#L339499-L339527)、[嚎弹炮](../../../raw/current/scripts/languages/chinese_s.po#L344014-L344017)、[弹弓](../../../raw/current/scripts/languages/chinese_s.po#L349454-L349457)、[作业包](../../../raw/current/scripts/languages/chinese_s.po#L349559-L349562)、[显示名覆盖](../../../raw/current/scripts/prefabs/slingshot.lua#L647-L648)、[SLINGSHOT_FMT](../../../raw/current/scripts/strings.lua#L3477)

获取途径：

| 注册名 | 途径 | 证据 |
|---|---|---|
| 四玩家吹箭 | 配方 cutreeds 2＋尾羽/獠牙等、`TECH.SCIENCE_TWO`：pipe＝houndstooth＋冬鸟羽、fire＝charcoal＋红鸟羽、yellow＝goldnugget＋金丝雀羽、sleep＝stinger＋黑鸟羽 | [配方](../../../raw/current/scripts/recipes.lua#L485-L488) |
| `houndstooth_blowpipe` | 配方 security_pulse_cage_full 1＋lunarplant_husk 3＋purebrilliance 3＋moonrocknugget 3、`TECH.LUNARFORGING_TWO`＋`station_tag="lunar_forge"`＋nounlock | [配方](../../../raw/current/scripts/recipes.lua#L859) |
| `slingshot` | 配方 twigs 1＋mosquitosack 2、`builder_tag="pebblemaker"`（Walter 制作组）；弹弓变体均无配方，按安装的骨架部件替换实体且拆解沿用 slingshot 配方 | [配方](../../../raw/current/scripts/recipes.lua#L211)、[替换](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L116-L147)、[拆解](../../../raw/current/scripts/recipes.lua#L1706-L1713) |
| `slingshotmodkit` 与部件 | 均需 Walter 技能（`walter_slingshot_modding` 等）；骨架替换链：frame_bone→slingshot2、frame_gems→slingshot2ex、frame_wagpunk_0→slingshotex、frame_wagpunk→slingshot999ex | [作业包配方](../../../raw/current/scripts/recipes.lua#L234)、[部件配方](../../../raw/current/scripts/recipes.lua#L235-L245)、[四个替换定义](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L151-L239) |
| `mosquitobomb` | 配方 mosquitosack 1＋mosquito 4、`builder_skill="wurt_mosquito_craft_2"` | [配方](../../../raw/current/scripts/recipes.lua#L114) |
| `blowdart_walrus` | 无配方、不生成持有者：海象攻击时由其内联 weapon 的 `SetProjectile("blowdart_walrus")` 发射 | [海象装备](../../../raw/current/scripts/prefabs/walrus.lua#L88-L104) |

## 关联机制导航

### 组件与同步

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | 三种接线：吹箭本体 0/100/60 伤＋onattack 效果；弹舱武器 `SetProjectile` 动态指弹体、`SetOnProjectileLaunched` 扣弹；弹体 weapon 34＋electric/位面附加 | 主模拟；嚎弹炮/弹弓空舱时无 projectile＝近战结算 | [[reference/components/weapon]]；[CanRangedAttack](../../../raw/current/scripts/components/weapon.lua#L84-L86)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L114) |
| 组件：projectile | 吹箭本体与两个 _proj：SetSpeed/SetRange/SetHitDist/SetHoming/OnHit/OnMiss；Throw 后 StartUpdatingComponent 逼近目标 | 主模拟；homing 默认 true（嚎弹炮与海象弹显式关掉）、离屏 2s 判 Miss | [[reference/components/projectile]]；[Throw](../../../raw/current/scripts/components/projectile.lua#L160-L192)、[Hit/命中回放](../../../raw/current/scripts/components/projectile.lua#L236-L266) |
| 组件：complexprojectile | mosquitobomb：`SetHorizontalSpeed(15)`＋`SetGravity(-35)`＋LaunchOffset(.25,1,0)＋OnLaunch/OnHit | 主模拟；combat 投掷分支要求非 `ismeleeweapon`（默认空） | [[reference/components/complexprojectile]]；[配置](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L290-L295)、[默认值](../../../raw/current/scripts/components/complexprojectile.lua#L24) |
| 组件：container | 弹舱 type `hand_inv`（装备即开）：slingshot1 槽/2 双槽、blowpipe 深拷贝 slingshot 并限 `blowpipeammo` 标签；`stay_open_on_hide`＋发射从槽位默认 1 扣 1 发（副手弹用槽 2） | 主模拟；999ex `EnableInfiniteStackSize` 取消堆叠上限 | [[reference/components/container]]；[弹舱参数](../../../raw/current/scripts/containers.lua#L1463-L1538)、[blowpipe 参数](../../../raw/current/scripts/containers.lua#L1941-L1948) |
| 组件：equippable | 四组文件均未设槽位→默认 `EQUIPSLOTS.HANDS`；吹箭/蚊子炸弹 `equipstack=true` 整叠装备整叠投掷；弹弓 `restrictedtag="slingshot_sharpshooter"` | 主模拟；OnEquip 覆盖持握符号＋`ARM_carry` | [[reference/components/equippable]]；[默认槽](../../../raw/current/scripts/components/equippable.lua#L29)、[整叠](../../../raw/current/scripts/prefabs/blowdart.lua#L106-L109)、[限制标签](../../../raw/current/scripts/prefabs/slingshot.lua#L543) |
| 组件：slingshotmods/slingshotmodder | slingshotmods 管部件三槽（band/frame/handle）与技能校验；modkit 用 slingshotmodder.StartModding 开改装界面；部件经 `containerinstalleditem` 刷新外观 | 主模拟；部件定义与替换回调在 slingshotpart_defs | [[reference/components/slingshotmods]]；[改装件入口](../../../raw/current/scripts/prefabs/slingshotmodkit.lua#L10-L22)、[部件安装事件](../../../raw/current/scripts/prefabs/slingshot.lua#L562-L564) |
| 组件：linkeditem | 弹弓有任一部件时 `LinkToOwnerUserID(主人)`，装备限定主人、检查面板 `NOT_MINE` | 主模拟；仅 slingshot 族 | [[reference/components/linkeditem]]；[绑定](../../../raw/current/scripts/prefabs/slingshot.lua#L415-L426)、[检查](../../../raw/current/scripts/prefabs/slingshot.lua#L472-L478) |
| 组件：reticule／aoecharging／aoetargeting | mosquitobomb reticule 6.5→3.5 找可落点；slingshotex aoecharging 蓄力（`SLINGSHOT_MAX_CHARGE_TIME` 0.5s）；slingshot2ex aoetargeting 长条准星＋aoespell 副手弹 | 公共端组件＋主模拟回调；蓄力需 Walter 技能激活 | [[reference/components/reticule]]、[[reference/components/aoecharging]]、[[reference/components/aoetargeting]]；[蚊弹准星](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L168-L184)、[蓄力](../../../raw/current/scripts/prefabs/slingshot.lua#L646-L655)、[AOE 弹](../../../raw/current/scripts/prefabs/slingshot.lua#L766-L784) |
| 同步：net_tinybyte×2＋net_string | slingshot `bandid`/`handleid`/`buildname`（icondirty 事件）驱动客户端分层物品图标；frameid 不联网（变体即注册名） | 公共端；皮肤改 buildname | [netvar](../../../raw/current/scripts/prefabs/slingshot.lua#L507-L510)、[图标重算](../../../raw/current/scripts/prefabs/slingshot.lua#L110-L139) |
| 同步：net_entity `houndstooth_proj._target` | 弹体飞行时 set 目标，客户端尾焰按目标距离截断（近于 1.5 不生成尾焰） | 公共端；尾焰任务仅非 dedicated | [net_entity](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L279)、[尾焰](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L127-L142) |
| API／数据：TUNING | `SLEEP_DART_SLEEPINESS` 1／`SLEEP_DART_SLEEPTIME` 15、`PIPE_DART_DAMAGE` 100、`YELLOW_DART_DAMAGE` 60、`WALRUS_DART_RANGE` 25、`HOUNDSTOOTH_BLOWPIPE_*`（34/34/×1.1/25/12/16）、`SLINGSHOT_DISTANCE(_MAX)` 10/14、`MOSQUITOBOMB_MOSQUITOS` 4、`BOMB_MOSQUITO_DAMAGE` wilson_attack×1.75、`BOMB_MOSQUITO_RANGE` 3 | 命中／发射时读取；wilson_attack=34（multiplayer_attack_modifier=1，源码推导） | [吹箭](../../../raw/current/scripts/tuning.lua#L557-L573)、[嚎弹炮](../../../raw/current/scripts/tuning.lua#L7982-L7987)、[弹弓](../../../raw/current/scripts/tuning.lua#L4858-L4874)、[蚊弹](../../../raw/current/scripts/tuning.lua#L8292-L8294) |
| API／数据：Sleeper/Grogginess | 催眠弹：有 sleeper 组件目标 `AddSleepiness(1,15)`（超过 resistance 即入睡 15s）；无 sleeper 组件（如玩家）走 `AddGrogginess(1,15)` | 主模拟；`tranquilizer` 标签另受动作门禁（见陷阱） | [[reference/components/sleeper]]、[[reference/components/grogginess]]；[AddSleepiness](../../../raw/current/scripts/components/sleeper.lua#L248-L257)、[AddGrogginess](../../../raw/current/scripts/components/grogginess.lua#L163-L178) |

### 生成宿主与消费者

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：SGwilson | ATTACK 选择：武器带 `blowdart`/`blowpipe` 标签走 `blowdart` 状态（9/14 帧 `PerformBufferedAction`）；`slingshot` 标签走 CASTAOE→`slingshot_special`（2ex 副手弹） | 服务器 SG；客户端 SG 同规则（预测） | [状态选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1408-L1413)、[blowdart 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10436-L10530)、[CASTAOE 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1317-L1331)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L683) |
| 消费者：componentactions/combat_replica | 持远程可边骑乘攻击：物品带 projectile/rangedweapon 标签即允许 ATTACK；`tranquilizer` 与无 sleeper 目标组合禁 ATTACK | 客户端动作列表＋replica GetWeapon | [动作门禁](../../../raw/current/scripts/componentactions.lua#L1982-L1992)、[replica](../../../raw/current/scripts/components/combat_replica.lua#L100-L108) |
| 关联 Prefab：`impact` | 全部吹箭 onhit 命中点：Follower 跟 hiteffectsymbol、朝向攻击者后 `animover` 自移除；非 dedicated 本地动画 | 主模拟生成；[注册](../../../raw/current/scripts/prefabs/impact.lua#L56) | [onhit](../../../raw/current/scripts/prefabs/blowdart.lua#L31-L41)、Wiki：[[reference/prefabs/impact]] |
| 关联 Prefab：`electrichitsparks` | 雷电吹箭 onattack 电火花（componentutil 公共 helper） | 主模拟 | [SpawnElectricHitSparks](../../../raw/current/scripts/componentutil.lua#L1022) |
| 关联 Prefab：`slingshotammo_*` 族 | 弹药表 18 项：17 种物品弹药（rock/gold/marble/thulecite/honey/freeze/slow/poop/stinger/moonglass/dreadstone/gunpowder/lunarplanthusk/purebrilliance/horrorfuel/gelblob/scrapfeather）带 `slingshotammo` 标签、堆叠 120；`trinket_1` 标 `no_inv_item`，只注册 `trinket_1_proj` 弹体、无物品形态 | 主模拟；效果与伤害归各弹药定义 | [弹药标签](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1293)、[堆叠](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1329-L1330)、[清单](../../../raw/current/scripts/prefabs/slingshot.lua#L58-L80)；机制页 Wiki 待建（现仅 [[reference/prefabs/slingshotammo_hitfx_rock]] FX 页） |
| 关联 Prefab：`houndstooth` | 嚎弹炮弹药：`blowpipeammo` 标签、堆叠 40；每发从弹舱拆 1 | 主模拟 | [标签](../../../raw/current/scripts/prefabs/houndstooth.lua#L21)、[堆叠](../../../raw/current/scripts/prefabs/houndstooth.lua#L40-L41) |
| 关联 Prefab：`mosquito` | 蚊子炸弹命中生成 `MOSQUITOBOMB_MOSQUITOS` 只蚊：follower 绑攻击者、`SuggestTarget` 优先可见敌对；持有 `mosquitomusk` 的目标只在**初始选敌**被排除 | 主模拟；Wurt `wurt_mosquito_craft_3` 技能 +2。**例外**：命中时的独立范围伤害 `DoImpactDamage` 不读该标签，但沿用自身的范围筛选——投掷者是玩家时其他玩家仍被排除；受击后的 `SetTarget`／共享攻击者同样不读 | [SpawnMosquitos](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L30-L73)、[范围伤害](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L88)、[[reference/prefabs/mosquitomusk|小痒痒]] 的「标签消费」；mosquito 页 Wiki 待建 |
| 消费者：walrus | `EquipBlowdart` 造内联非联网 weapon 实体（伤害＝海象 defaultdamage、射程＝attackrange、`SetProjectile("blowdart_walrus")`、nosteal） | 主模拟；本族 `blowdart_walrus` 只是被发射的弹体 | [EquipBlowdart](../../../raw/current/scripts/prefabs/walrus.lua#L88-L104) |
| 消费者：TryLuckRoll | 弹弓装 `slingshot_band_mimic` 皮筋部件后 30%（＋幸运）概率免扣弹 | 主模拟；`SLINGSHOT_MOD_FREE_AMMO_CHANCE` 0.3 | [免扣弹](../../../raw/current/scripts/prefabs/slingshot.lua#L380-L384)、[掷点](../../../raw/current/scripts/componentutil.lua#L2237-L2249) |
| 消费者：walter | `slingshot_sharpshooter` 标签是弹弓 restrictedtag 的唯一来源 | 主模拟；装备校验读 equippable.restrictedtag | [标签](../../../raw/current/scripts/prefabs/walter.lua#L1139)、[校验](../../../raw/current/scripts/prefabs/slingshot.lua#L543) |
| 皮肤 | slingshot 五变体各 4 皮肤（bandaged/nature/spiked/straws）：`OnSlingshotSkinChanged` 改 buildname＋分层图标，FX 用 OverrideItemSkinSymbol；嚎弹炮 OnEquip 有皮肤分支（OverrideItemSkinSymbol）；四吹箭与蚊子炸弹无皮肤条目 | 皮肤差异归本体页 | [slingshot 皮肤](../../../raw/current/scripts/prefabskins.lua#L1725-L1759)、[弹弓回调](../../../raw/current/scripts/prefabs/slingshot.lua#L169-L173)、[嚎弹炮分支](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L30-L37) |
| 资源：anim | `blow_dart.zip`＋`swap_blowdart(_pipe)`、`slingshot.zip`＋各弹舱 UI anim（ui_cookpot_1x2/ui_slingshot_wagpunk*/bone/gems）、`mosquitobomb.zip`＋`swap_mosquitobomb.zip`、嚎弹炮尾焰复用 `lavaarena_blowdart_attacks` | 原版资源静态声明 | [吹箭](../../../raw/current/scripts/prefabs/blowdart.lua#L1-L6)、[弹弓](../../../raw/current/scripts/prefabs/slingshot.lua#L3-L31)、[尾焰](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L112-L115) |

## 典型接入

### 调整吹箭数值并扩展命中效果

- 前提与选择：`modmain.lua`。吹箭效果回调每次攻击都从 `weapon.onattack` 取，`AddPrefabPostInit` 里包 `SetOnAttack` 可叠加自定义效果；伤害与射程在构造时写进 weapon，催眠 `1/15` 在命中回调现读 TUNING，改 TUNING 即全局生效（含海象弹射程键）。
- 生效与撤销：postinit 在实体构造时执行，weapon/projectile 组件仅主模拟存在，需 `ismastersim` 守卫；TUNING 改动停用模组不还原。
- 原版依据：onattack 由[命中回放链](../../../raw/current/scripts/components/projectile.lua#L236-L266)触发；mod 环境内置 TUNING，其余游戏全局需 `GLOBAL.` 前缀（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L286) 无 `__index=_G`）。

```lua
-- modmain.lua：环境已内置 TUNING，TheWorld 等需 GLOBAL. 前缀
TUNING.PIPE_DART_DAMAGE = 150      -- 吹箭（pipe）伤害（默认 100）
TUNING.SLEEP_DART_SLEEPTIME = 30   -- 催眠入睡时长秒（默认 15）
TUNING.SLINGSHOT_DISTANCE_MAX = 20 -- 弹弓最大射程（默认 14）

AddPrefabPostInit("blowdart_pipe", function(inst)
    if GLOBAL.TheWorld == nil or not GLOBAL.TheWorld.ismastersim then
        return -- weapon/projectile 仅主模拟
    end
    local oldattack = inst.components.weapon.onattack
    inst.components.weapon:SetOnAttack(function(inst, attacker, target)
        if oldattack ~= nil then
            oldattack(inst, attacker, target)
        end
        if target:IsValid() then
            target:PushEvent("mydart_hit", { attacker = attacker, weapon = inst })
        end
    end)
end)
```

- 调整与验证：伤害单位为生命值/发、催眠时长为秒（`AddSleepiness` 第二参）；`mydart_hit` 是自定义事件，需自行监听且先判 `IsValid`；弹弓 `SetRange` 两参是攻击距离／命中距离；雷电吹箭的电伤倍率另在 combat 电属性分支乘算，改 `YELLOW_DART_DAMAGE` 不改倍率。

## 常见陷阱与限制

- **误区：把吹箭效果写在 `projectile:SetOnHitFn` 里**：原版 onhit 只做 impact FX＋自移除；催眠/引燃/电伤都在 `weapon:SetOnAttack`，由命中链的第二次 `combat:DoAttack` 回调 `weapon:OnAttack` 触发。[onhit](../../../raw/current/scripts/prefabs/blowdart.lua#L31-L41)、[回调链](../../../raw/current/scripts/components/combat.lua#L1223-L1225)
- **边界：吹箭发射即整支脱手不可回收**：`DropItem` 后弹体 NOCLICK＋persists=false，命中 or Miss 都 `Remove`；与回旋镖的回收语义不同。[脱手](../../../raw/current/scripts/components/combat.lua#L1170-L1172)、[Miss 移除](../../../raw/current/scripts/prefabs/blowdart.lua#L96)
- **边界：催眠吹箭不能对无 sleeper 标签目标发起攻击**：玩家只有 grogginess 无 sleeper 组件，动作层 `tranquilizer`＋无 `sleeper` 组合不生成 ATTACK；`AddGrogginess` 分支供绕过动作层命中的兜底（如代码直调、AI 发射）。[门禁](../../../raw/current/scripts/componentactions.lua#L1992)、[sleeper 标签来源](../../../raw/current/scripts/components/sleeper.lua#L15)、[grogginess 分支](../../../raw/current/scripts/prefabs/blowdart.lua#L138-L140)
- **边界：弹舱武器空舱是弱近战**：嚎弹炮空舱 `UNARMED_DAMAGE`(10) 且无远程标签；弹弓空舱 0 伤；弹弓构造时就写入射程、空舱只清 `projectile`；嚎弹炮才在卸弹时清射程并移标签。[嚎弹炮空舱](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L221)、[标签增删](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80-L96)
- **边界：弹弓每发扣 1 发弹药而非整叠**：`RemoveItem(stack, false)` 对堆叠拆 1（`Stackable:Get(1)`）；拟态皮筋 30%（＋幸运）免扣，2ex 副手弹扣槽 2；999ex 的 `EnableInfiniteStackSize` 只取消堆叠上限，不是无限弹药。[拆 1](../../../raw/current/scripts/components/container.lua#L1029-L1037)、[999ex](../../../raw/current/scripts/prefabs/slingshot.lua#L735-L737)
- **边界：mosquitobomb 的 projectile 标签会误导 replica**：客户端 `GetWeapon` 要求"projectile 且非 complexprojectile"，蚊弹双标签都不满足也无 `rangedweapon`，骑乘时客户端不把它当可用武器。[replica 判定](../../../raw/current/scripts/components/combat_replica.lua#L101-L105)、[双标签](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L266-L270)
- **边界：蚊子炸弹的"PVP"按投掷者判定，不是世界开关**：`ispvp` 只由投掷者是否带 `player` 标签决定；为真时范围结算排除其他玩家、只允许投掷者自己，未开时其他玩家照常进入。范围查询单位是世界距离 3。[PVP 排除](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L78-L86)
- **边界：`blowdart_walrus` 不是海象的武器本体**：海象手持的是 `EquipBlowdart` 里 CreateEntity 的内联非联网 weapon（nosteal、掉落即删），本注册只是 `SetProjectile` 发射的弹体。[内联武器](../../../raw/current/scripts/prefabs/walrus.lua#L91-L102)
- **边界：弹弓变体不是独立配方产物**：安装对应 frame 部件触发 `ReplaceSlingshot` 整体替换注册名，部件与弹药转移、原弹弓删除；卸件回退 `slingshot`。[替换](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L116-L147)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。逐行实读 blowdart.lua、houndstooth_blowpipe.lua、slingshot.lua、mosquitobomb.lua、slingshotmodkit.lua 全文与 equippable/projectile/combat/weapon/container/stackable/sleeper/grogginess 相关函数体；核对吹箭五 return、SGwilson `blowdart` 状态与状态选择、combat 投掷/命中/CanRangedAttack 三分支、装弹扣 1 发链、slingshotpart_defs 四个 frame 替换定义、walrus EquipBlowdart、impact 注册；配方行、TUNING 各键、.po 八名称键（python 行解析）、prefabskins 弹弓皮肤、componentactions/combat_replica/SGwilson_client 消费端。2026-11-07 第四轮复检：`attacked` 重复派发、免扣弹/槽 2、蓄力 k² 与 ×2/×1.25、弹弓射程构造期写入、蚊弹玩家筛选、注册数与 `trinket_1` 弹药。复杂档理由：15 注册多变体＋跨 combat/weapon/projectile/complexprojectile/container 的整体协作链，需同页保留对照。未覆盖：slingshotammo 18 项弹药（17 物品＋trinket_1）各 `_proj` 的效果实现与 debuffs、SGwilson `slingshot_special` 帧级细节、Forge 独立注册 blowdart_lava/blowdart_lava2、mosquito 生物本体。
- 游戏运行：未运行。伤害 34/59.5/17 等为源码推导值（wilson_attack=34），非运行测量。
- 结构检查：validate_page.py 通过（149 行／32348 字节，complex 档）；validate_source_links.py --strict 通过（0 error 0 warning）；哈希基线已 record。
- 外部检视 R16（2026-11-10）：`mosquito` 关联行把「持有 `mosquitomusk` 的目标被排除」限定为炸弹刷出时的初始 `SuggestTarget`，补「命中时的 `DoImpactDamage` 不读该标签、受击后的 `SetTarget`／共享攻击者同样不读」，并链回 [[reference/prefabs/mosquitomusk]]。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 16 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
