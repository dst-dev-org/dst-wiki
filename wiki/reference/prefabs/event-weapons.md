---
title: "事件武器族（rabbitkingspear、yoth_lance、spear_wathgrithr 等 5 注册）"
created: 2026-11-07
updated: 2026-11-07
type: prefab
summary: "三份武器 prefab 文件的 5 个 HANDS 槽武器注册：兔王棍（兔王 aggressive 形态掉落，对兔人加成并掉理智）、冲锋骑枪（镀金骑士掉落，joustsource 线段冲锋碰撞）、战斗长矛三件套（薇格弗德初始矛、奔雷矛、充能奔雷矛，技能树门禁与能量静电升级）；覆盖伤害与耐久推导值、配方与掉落、官方名、皮肤与消费端标签；Forge 双件（spear_gungnir／spear_lance）分诊结论见正文。"
aliases: ["rabbitkingspear", "兔王棍", "yoth_lance", "冲锋骑枪", "骑士枪", "spear_wathgrithr", "战斗长矛", "女武神长矛", "spear_wathgrithr_lightning", "奔雷矛", "spear_wathgrithr_lightning_charged", "充能奔雷矛", "事件武器", "活动武器", "Wigfrid 长矛"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rabbitkingspear.lua", "prefabs/yoth_lance.lua", "prefabs/spear_wathgrithr.lua", "prefabs/rabbitking.lua", "prefabs/knight.lua", "prefabs/moonstorm_static.lua", "prefabs/skilltree_wathgrithr.lua", "prefabs/spear_gungnir.lua", "prefabs/spear_lance.lua", "prefabskins.lua", "components/equippable.lua", "components/joustsource.lua", "stategraphs/SGwilson.lua", "tuning.lua", "constants.lua", "recipes.lua", "entityscript.lua", "characterutil.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 事件武器族（rabbitkingspear、yoth_lance、spear_wathgrithr 等 5 注册）

## 快速判断

- 职责与适用：三份 prefab 文件共 5 个注册，全部为可装备手持武器（equipslot 均为默认 `EQUIPSLOTS.HANDS`，本页无一显式改槽，[默认值](../../../raw/current/scripts/components/equippable.lua#L29)）：兔王棍（兔王族掉落武器）、冲锋骑枪（YOTH 事件掉落武器）、战斗长矛三件套（薇格弗德专属武器链）。
- 优先使用：条件伤害＋攻击回调参照兔王棍的 `DamageCalculator`＋`weapon.onattack` 模式（[伤害函数](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L35-L41)）；地面点冲锋与线段碰撞参照冲锋骑枪（JOUST 排除骑乘） [[reference/components/joustsource]]；技能树门禁与"基础→强化→充能"武器升级链参照奔雷矛系。
- 重要副作用：兔王棍装备期持续掉理智且攻击兔人再扣 5 点（[回调](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L27-L33)）；充能奔雷矛带 restrictedtag，普通装备路径只允许习得"二级奔雷矛加强"的薇格弗德（读档按技能标签宽容）（[限制](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L536)）；冲锋骑枪命中推开目标（[knockback](../../../raw/current/scripts/components/joustsource.lua#L120)）。
- 修改边界：本页不含 Forge 双件 spear_gungnir／spear_lance（equippable 挂载点不可读，见[Forge 分诊](#forge-分诊)）；SGwilson 冲锋状态逐帧与充电全程未展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rabbitkingspear` | 单注册 [Prefab](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L105)：pristine 标签（shadow_item/sharp/manrabbitscarer/weapon/shadowlevel）→ 端分界 → weapon 伤害函数、finiteuses、equippable＋魔法掉理智、shadowlevel | 手持武器，无配方（仅拆解），兔王 aggressive 形态固定掉落；[构造](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L43-L103) |
| `yoth_lance` | 单注册 [Prefab](../../../raw/current/scripts/prefabs/yoth_lance.lua#L92)：nopunch/sharp/pointy/lancejab/weapon 标签 → finiteuses、weapon（伤害＋攻击距离 2）、joustsource、fencerotator、equippable | 手持骑枪，镀金骑士 25% 掉落；[构造](../../../raw/current/scripts/prefabs/yoth_lance.lua#L31-L90) |
| `spear_wathgrithr` | `CommonFn`（[共用工厂](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L343-L407)）＋ `BasicSpearFn` 数据（[L409-L419](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L409-L419)），注册 [L692](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L691-L696) | 战斗长矛：battlespear/sharp/pointy 标签，无 planar；薇格弗德出生自带＋专属配方 |
| `spear_wathgrithr_lightning` | `CommonFn`＋`LightningSpearCommonFn_Normal`＋`LightningSpearPostInitFn_Base/Normal`（[工厂](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L548-L560)），注册 [L693](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L691-L696) | 奔雷矛：电属性（`1+0.5×目标湿度`，满湿才 ×1.5；突刺窗口亦电）、突刺、可被能量静电升级；skilltree 配方 |
| `spear_wathgrithr_lightning_charged` | `CommonFn`＋`LightningSpearCommonFn_Charged`＋`LightningSpearPostInitFn_Charged`（[工厂](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L562-L574)），注册 [L694](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L691-L696) | 充能奔雷矛：+20 位面伤害、移速 ×1.2、restrictedtag 门禁、跟随电光 FX |

附属 FX：`spear_wathgrithr_lightning_lunge_fx`、`spear_wathgrithr_lightning_fx` 已由 [[reference/prefabs/spear_gungnir_lungefx]] 主要负责，本页不重复登记。官方名核验：`STRINGS.NAMES.RABBITKINGSPEAR`＝兔王棍（[po L348450-L348452](../../../raw/current/scripts/languages/chinese_s.po#L348450-L348452)）、`YOTH_LANCE`＝冲锋骑枪（[L353025-L353027](../../../raw/current/scripts/languages/chinese_s.po#L353025-L353027)）、`SPEAR_WATHGRITHR`＝战斗长矛（[L349760-L349762](../../../raw/current/scripts/languages/chinese_s.po#L349760-L349762)）、`SPEAR_WATHGRITHR_LIGHTNING`＝奔雷矛、`SPEAR_WATHGRITHR_LIGHTNING_CHARGED`＝充能奔雷矛（[L349765-L349772](../../../raw/current/scripts/languages/chinese_s.po#L349765-L349772)）。

## 关联机制导航

### 组件与数值

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：weapon` | 兔王棍 SetDamage(伤害函数)＋onattack；骑枪伤害＋SetRange(2)；长矛系 SetDamage，奔雷矛 SetElectric(1, 0.5)＋SetOverrideStimuliFn | 服务端；`wilson_attack=34`（[tuning L37](../../../raw/current/scripts/tuning.lua#L37)）：兔王棍 34×1.5＝51、对 manrabbit ×1.4；骑枪 51；战斗长矛 34×1.25＝42.5；奔雷矛系普攻 34×1.75＝59.5（突刺期间 weapon 临时 68、attackwear=0） | [weapon 配置](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L77-L79)、[骑枪](../../../raw/current/scripts/prefabs/yoth_lance.lua#L69-L71)、[电属性](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L474-L477)、tuning [L8511-L8515](../../../raw/current/scripts/tuning.lua#L8511-L8515)/[L9272-L9276](../../../raw/current/scripts/tuning.lua#L9272-L9276)/[L357](../../../raw/current/scripts/tuning.lua#L357)/[L8058](../../../raw/current/scripts/tuning.lua#L8058) |
| `组件：planardamage` | 充能奔雷矛 `SetBaseDamage(20)`，其余注册未挂载 | 仅 charged 工厂链 | [挂载条件](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L386-L389)、[数值](../../../raw/current/scripts/tuning.lua#L8066) |
| `组件：finiteuses` | 耐久 125（兔王棍）/200（骑枪、战斗长矛、充能矛）/150（奔雷矛）；兔王棍耗尽 `inst.Remove`，骑枪耗尽推 `toolbroke` 给持有者再移除 | 服务端；突刺命中修复充能矛 4 点、每次突刺上限 2 次（仅 charged 且 `doer.IsValidVictim(target)`） | [兔王棍](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L83-L86)、[骑枪](../../../raw/current/scripts/prefabs/yoth_lance.lua#L17-L24)、[修复](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L184-L195)、tuning [L8515](../../../raw/current/scripts/tuning.lua#L8515)/[L9274](../../../raw/current/scripts/tuning.lua#L9274)/[L168](../../../raw/current/scripts/tuning.lua#L168)/[L8057/L8064](../../../raw/current/scripts/tuning.lua#L8057-L8070) |
| `组件：equippable` | 全部 HANDS 槽；兔王棍 dapperness＝`CRAZINESS_SMALL`（魔法掉理智，[tuning L2224](../../../raw/current/scripts/tuning.lua#L2224)）；充能矛 `restrictedtag`＋`walkspeedmult=1.2` | 装备/卸下换持握外观并（长矛系）刷新技能联动 | [兔王棍](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L91-L95)、[充能矛](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L536-L537)、[限制判定](../../../raw/current/scripts/components/equippable.lua#L159-L161) |
| `组件：shadowlevel` | 兔王棍 SetDefaultLevel(2)＋pristine `shadowlevel` 标签 | 见[标签消费端](#标签消费端) | [L97-L98](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L97-L98)、[tuning L2866](../../../raw/current/scripts/tuning.lua#L2866) |
| `组件：joustsource`＋`fencerotator` | 骑枪专属：冲锋速度 10、枪长 2、跑动循环 2 圈、命中回调生成 puff FX；右键 [[reference/components/joustuser]] 发动 JOUST | 无专属角色限制，但须同时满足 right、未骑乘、地面落点、`joustuser:CanJoust`（[动作收集](../../../raw/current/scripts/componentactions.lua#L2128-L2132)、[执行](../../../raw/current/scripts/actions.lua#L6683-L6697)）；PVP 关闭时碰撞排除 player；线段命中 `combat:DoAttack` 并推 `knockback`（半径 6.5 强制落地） | [joustsource 配置](../../../raw/current/scripts/prefabs/yoth_lance.lua#L73-L77)、[碰撞几何与击退](../../../raw/current/scripts/components/joustsource.lua#L43-L130)、[动作注册](../../../raw/current/scripts/actions.lua#L6683-L6697)、SG [joust_start](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21761)、`jousting` 标签 [L21849](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21849) |
| `组件：aoeweapon_lunge／aoespell／rechargeable／aoetargeting` | 奔雷矛系专属：右键突刺（CASTAOE 事件 `combat_lunge`）、充能冷却 3 秒（充能矛 1.5 秒）、施法距离 6.5 | 需 `rechargeable:IsCharged()` 且习得 skill_4 才启用 aoetargeting；装备时剩余冷却不足 `_cooldown` 会重新 `Discharge`；放电由突刺触发 | [基础配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L464-L498)、[施法](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L157-L159)、[启用门禁](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L62-L64)、组件见 [[reference/components/aoeweapon-lunge]] |
| `组件：upgradeable` | 仅奔雷矛（未充能态）：`UPGRADETYPES.SPEAR_LIGHTNING`，装备中不可升级 | 充能矛由升级物直接生成并继承耐久/充能百分比 | [升级链](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L500-L507)、[升级物](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L312)、[升级回调](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L244-L270)、[[reference/components/upgradeable]] |

### 技能树与配方（API／数据）

| 数据 | 作用 | 证据 |
|---|---|---|
| `wathgrithr_arsenal_spear_1/2`（一级/二级布拉基的祝福） | 装备战斗长矛系时经 `RefreshAttunedSkills` 给 `singinginspiration` 挂 `arsenal_spear` 修正：灵感获得 ×1.25/×1.5 | 节点定义 [skilltree_wathgrithr L215-L232](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L215-L232)、倍率 [tuning L7201-L7204](../../../raw/current/scripts/tuning.lua#L7201-L7204)、刷新 [L46-L65](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L46-L65)、[[reference/components/skilltreeupdater]] |
| `wathgrithr_arsenal_spear_3`（奔雷矛） | 解锁配方 `spear_wathgrithr_lightning`（树枝 2＋闪电羊角 2＋牛毛 1，TECH.NONE） | builder_skill [recipes L192](../../../raw/current/scripts/recipes.lua#L192)、节点 [L233-L243](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L233-L243) |
| `wathgrithr_arsenal_spear_4`（一级奔雷矛加强） | 启用奔雷矛/充能矛右键突刺（aoetargeting，两者都需满充＋skill_4）；突刺命中还会修复充能矛 | 节点 [L246-L251](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L246-L251)、启用判定见上表 |
| `wathgrithr_arsenal_spear_5`（二级奔雷矛加强） | 给薇格弗德加 `spear_lightning_upgradeuser` 标签，才可装备充能奔雷矛 | 节点 [L252-L257](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L252-L257)、标签映射 [constants L2158](../../../raw/current/scripts/constants.lua#L2158)、[[reference/components/equippable]] |
| 战斗长矛配方 | 树枝 2＋燧石 2＋金块 2，TECH.NONE，`builder_tag="valkyrie"`（角色门禁，非技能） | [recipes L178](../../../raw/current/scripts/recipes.lua#L178)；出生自带 [tuning L3461](../../../raw/current/scripts/tuning.lua#L3461) |

### 标签消费端

| 标签 | 消费者与效果 | 证据 |
|---|---|---|
| `manrabbitscarer`（兔王棍） | 兔人脑 4 格内回避该持有人；兔人个体再查装备标签 | [bunnymanbrain L34-L35/L158](../../../raw/current/scripts/brains/bunnymanbrain.lua#L34-L35)、[bunnyman L269](../../../raw/current/scripts/prefabs/bunnyman.lua#L269)、半径 [tuning L8511](../../../raw/current/scripts/tuning.lua#L8511) |
| `shadow_item`（兔王棍） | 温蒂 Gestalt 卫兵把持有者按"暗影目标"处理（等级 3＝攻击） | [gestalt_guard_evolved L115-L118](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L115-L118) |
| `shadowlevel`＋组件（兔王棍，等级 2） | 装备时角色播报暗影等级台词；麦斯威尔（shadowmagic）查看时附"暗影等级"注释；本页检索范围未发现战斗数值消费端 | [waxwell L186-L214](../../../raw/current/scripts/prefabs/waxwell.lua#L186-L214)、[stringutil L343-L345](../../../raw/current/scripts/stringutil.lua#L343-L345) |
| `lancejab`（骑枪） | 攻击前摇改用八向骑枪 jab 动画（预测朝向），冷却 21 帧 | [SGwilson L8646-L8652](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8646-L8652) |
| `battlespear`（战斗长矛系） | 本页全量检索（scripts 除 speech）未发现消费端；技能树实际经 `inspirationgain` 技能计数生效，不读此标签 | 定义 [spear_wathgrithr L358](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L356-L361) |

### 关联 Prefab 与皮肤

| 对象 | 关系 | 证据 |
|---|---|---|
| `rabbitking_aggressive`（兔王·敌对形态，Wiki 待建） | 掉落兔王棍（loot_aggressive 固定项）；兔王由世界常驻组件 rabbitkingmanager 管理（喂 12 根胡萝卜或杀兔孽值触发生成），与兔王族装备（胡萝卜外套、挖洞兔号角）同属 recipes.lua "Rabbit King" 分组 | [loot](../../../raw/current/scripts/prefabs/rabbitking.lua#L161-L166)、[SetLoot](../../../raw/current/scripts/prefabs/rabbitking.lua#L281)、[注册](../../../raw/current/scripts/prefabs/rabbitking.lua#L608-L612)、[manager 挂载](../../../raw/current/scripts/prefabs/forest.lua#L679)、[胡萝卜/孽值触发](../../../raw/current/scripts/components/rabbitkingmanager.lua#L354-L400)、[tuning L8471-L8475](../../../raw/current/scripts/tuning.lua#L8471-L8475)、拆解分组 [recipes L1614-L1619](../../../raw/current/scripts/recipes.lua#L1614-L1619)、同族装备见 [[reference/prefabs/armor-classic]] |
| `knight_yoth`（镀金骑士，Wiki 待建） | YOTH（发条骑士之年，[constants L856](../../../raw/current/scripts/constants.lua#L856)）事件期由 yoth_knightmanager 管理（[事件门禁](../../../raw/current/scripts/components/yoth_knightmanager.lua#L36)）；共享掉落表 `knight_gilded` 含冲锋骑枪 25%、幸运马蹄铁、红色红包；四骑士中最后倒下者追加掉镀金骑士头盔＋胸甲（100%） | [loot 表](../../../raw/current/scripts/prefabs/knight.lua#L72-L80)、[末骑掉落](../../../raw/current/scripts/prefabs/knight.lua#L262-L288)、[注册](../../../raw/current/scripts/prefabs/knight.lua#L632-L634)、帽/甲与套装见 [[reference/prefabs/hats-armor]] |
| `moonstorm_static`（能量静电，Wiki 待建） | 升级物：对奔雷矛使用生成充能奔雷矛（保留耐久与充能百分比） | [upgrader](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L312)、[升级回调](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L248-L270) |
| `round_puff_fx_lg/sm`、`reticuleline/ping` | 骑枪命中烟圈（[[reference/prefabs/round_puff_fx_sm]]）；奔雷矛施法指示线（声明即依赖，施法时由组件系统使用） | [OnHitOther](../../../raw/current/scripts/prefabs/yoth_lance.lua#L26-L29)、[reticule](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L428-L438) |
| 皮肤 | 战斗长矛/奔雷矛/充能矛各 4 款（lunar/northern/valkyrie/wrestle）；充能矛光层按皮肤偏移（FX_OFFSETS）；兔王棍、冲锋骑枪无皮肤注册 | [prefabskins L1773-L1792](../../../raw/current/scripts/prefabskins.lua#L1773-L1792)、[FX_OFFSETS](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L274-L280) |

## Forge 分诊

两文件全文实读：`AddComponent("equippable")` 确为 0 命中（spear_gungnir.lua 全 152 行、spear_lance.lua 全 82 行）。分诊结论：

- **不存在可读的共享 Forge 工厂。** scripts 快照内没有 forge_common／forge_equipment 之类可读文件；两件的服务端初始化转交 `event_server_data("lavaarena", "prefabs/<名>").master_postinit(inst)`（[gungnir](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L95)、[lance](../../../raw/current/scripts/prefabs/spear_lance.lua#L76)），该函数由引擎 `requireeventfile` 按 `lavaarena_event_server/prefabs/<名>` 路径加载**编译事件文件**（[entityscript L77-L87](../../../raw/current/scripts/entityscript.lua#L77-L87)）；公开脚本只能证明这次委托，文件本身不在本快照（raw/current 仅 scripts/）内。
- **可确证的部分：** shell 文件 pristine 标签注释证明编译数据会加 weapon／aoeweapon_lunge（gungnir）／aoeweapon_leap（lance）／rechargeable（[gungnir L68-L75](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L68-L75)、[lance L50-L57](../../../raw/current/scripts/prefabs/spear_lance.lua#L50-L57)）；Forge 模式（LAVAARENA）薇格弗德出生自带 spear_gungnir（[tuning L3473-L3484](../../../raw/current/scripts/tuning.lua#L3473-L3484)，消费端 [characterutil L129](../../../raw/current/scripts/characterutil.lua#L129)），spear_lance 为 Forge 进度解锁（[communityprogression L17](../../../raw/current/scripts/lavaarena_communityprogression.lua#L17)）；主世界无配方、无掉落（仅 [prefablist L1188-L1189](../../../raw/current/scripts/prefablist.lua#L1188-L1189) 注册）。
- **结论：** equippable 挂载点只能由这次委托推断在编译事件文件内，本快照不可读，"是否仅熔炉模式可装备"无法按本快照确证；两件**不纳入本页注册表**，列待核验问题交主代理（见[验证记录](#验证记录)）。

## 典型接入

### 参照兔王棍给武器加"条件伤害＋攻击回调"

- 前提与选择：`modmain.lua`（服务端逻辑走 `AddPrefabPostInit`，仅 master sim 生效）；目标武器已有 weapon 组件，直接替换伤害计算器与 onattack 即可，不新增资源。
- 生效与撤销：`weapon:SetDamage(fn)` 每次攻击取伤害、`weapon.onattack` 命中后回调；`SetDamage` 会整体覆盖原计算器，需自行实现完整条件分支（含原本的默认伤害）。
- 原版依据：兔王棍工厂 [DamageCalculator＋onattack](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L27-L41)、[组件挂载](../../../raw/current/scripts/prefabs/rabbitkingspear.lua#L77-L79)；mod 环境变量按 mods.lua：`TUNING` 直用、`TheWorld` 需 `GLOBAL.`。

```lua
-- modmain.lua：把兔王棍对兔人的伤害加成固定为 2 倍（覆盖原 1.4 倍计算器）
local function DamageCalculator(inst, attacker, target)
    local damage = TUNING.RABBITKINGSPEAR_DAMAGE
    if target and target:HasTag("manrabbit") then
        damage = damage * 2
    end
    return damage
end

AddPrefabPostInit("rabbitkingspear", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetDamage(DamageCalculator)
end)
```

- 调整与验证：伤害单位与原版一致（weapon damage，兔王棍基准 51）；客户端不装本 mod 时依赖 pristine 标签的预测不受影响，伤害只在服务端结算；用 `c_spawn("rabbitkingspear")`＋攻击兔人实测数值。

## 常见陷阱与限制

- **充能奔雷矛不能直接装备（普通路径）。** `equippable.restrictedtag` 要求持有 `spear_lightning_upgradeuser` 标签（只有薇格弗德激活"二级奔雷矛加强"）；未满足时装备判定直接拒绝。读档例外：`IsRestricted_FromLoad` 对技能树标签按角色 prefab 宽容，女武神存档可直接恢复装备。[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L159-L161)、[读档宽容](../../../raw/current/scripts/components/equippable.lua#L164-L173)
- **CRAZINESS_SMALL 是"每秒"速率。** 值为 `-100/(day_time*2)`，day_time＝300 秒（[tuning L2224](../../../raw/current/scripts/tuning.lua#L2224)），即约 −0.167 理智/秒；不要用 total_day_time＝480 秒折算每秒值，也不要把它当一次性扣减。
- **兔王棍的理智惩罚有两层。** 装备期持续魔法掉理智（dapperness）＋每次攻击到 manrabbit 再 `DoDelta(-5)`（[tuning L8514](../../../raw/current/scripts/tuning.lua#L8514)）；对非兔人目标只有第一层。
- **`battlespear` 标签不产生任何效果（边界）。** 检索范围内无消费者，灵感加成实际由技能树 `inspirationgain` 计数驱动；复制该标签不会带来战斗或灵感收益（见[标签消费端](#标签消费端)）。
- **`YOTH_LANCE_CHARGE_DAMAGE`（34×2）在本快照无消费端。** 冲锋命中实际走 `combat:DoAttack`，按手持武器伤害（51）结算（[joustsource L117-L119](../../../raw/current/scripts/components/joustsource.lua#L103-L121)）；引用该常量做"冲锋伤害"会与实现不符。
- **YOTH 套装（头盔＋胸甲）与骑枪无组件关联。** 骑枪不参与 setbonus；套装效果（幸运）归 [[reference/prefabs/hats-armor]]。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394。实读三主文件全文并核验 5 个注册名与工厂分界；核验 tuning 推导值、equippable 默认槽位、技能树 5 节点与两条配方、掉落链、皮肤、消费端标签。2026-11-07 第四轮复检：JOUST 排除骑乘、59.5 为普攻基值/突刺 68、充能+skill_4 双门禁与读档宽容、Forge 委托边界。Forge 双件按[分诊](#forge-分诊)实读全文并追至 entityscript 加载器。未覆盖：SGwilson joust 状态逐帧时序、moonstorm_static 充电交互全程、YOTR 兔王生成条件中的 `ShouldStopActions` 分支。
- 游戏运行：未运行。
- 结构检查：validate_page.py／validate_source_links.py --strict 实测：本页共 143 行、24277 字节（standard 档），0 error／0 warning（2026-11-07）。
- 待核验问题：spear_gungnir／spear_lance 的组件面只能确认初始化委托与加载路径（`lavaarena_event_server/`），具体挂载本快照不可读；需带 data 包的快照或运行时核验后方可为两件建页或并入本页。

## 相关链接

- [[reference/prefabs/spear_gungnir_lungefx]]（奔雷矛系突刺/跟随 FX 与尖齿矛突刺烟）
- [[reference/prefabs/hats-armor]]（镀金骑士头盔＋胸甲套装与幸运系统）
- [[reference/prefabs/armor-classic]]（兔王族装备：胡萝卜外套等）
- [[reference/prefabs/wathgrithr_shield]]（薇格弗德战斗圆盾，同技能树）
- [[reference/prefabs/spear]]（基础长矛，同族手持武器参照）
- [[reference/prefabs/weapons-combat-index|武器与战斗]]（装备页索引）

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
