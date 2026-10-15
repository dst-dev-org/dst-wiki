---
title: "哑铃族（dumbbell）"
created: "2026-11-06"
updated: "2026-11-06"
type: prefab
summary: "原版沃尔夫冈专属哑铃族 7 注册：武器伤害＋finiteuses 耐久＋mightydumbbell 举铁涨体格；mighty 态运行时动态挂 complexprojectile 右键投掷（AOE＋技能树加成），wimpy 态挂 punch 标签换拳击动画；热铃复用暖石五档温差并可投喂蚁狮。"
aliases: ["dumbbell", "dumbbell_golden", "dumbbell_marble", "dumbbell_gem", "dumbbell_heat", "dumbbell_redgem", "dumbbell_bluegem", "哑铃", "黄金哑铃", "大理石哑铃", "宝石哑铃", "热铃", "火铃", "冰铃", "举重", "举铁", "投掷哑铃", "沃尔夫冈肌肉"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/dumbbells.lua", "components/mightydumbbell.lua", "components/dumbbelllifter.lua", "components/mightiness.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/complexprojectile.lua", "components/combat.lua", "components/inventory.lua", "components/inspectable.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "prefabs/wolfgang.lua", "prefabs/antlion.lua", "prefabs/skilltree_wolfgang.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 哑铃族（dumbbell）

## 快速判断

- 职责与适用：`prefabs/dumbbells.lua` 单一工厂 `MakeDumbbell(name, ...)` 注册 7 个可拾取手持装备（物品＋装备），沃尔夫冈专属：武器伤害＋finiteuses 耐久＋mightydumbbell 举铁涨肌肉；mighty 态运行时动态挂 complexprojectile（右键 TOSS 投掷），wimpy 态挂 `punch` 标签换拳击动画。同文件无 quagmire 分支、无皮肤注册。[工厂与注册](../../../raw/current/scripts/prefabs/dumbbells.lua#L499-L642)
- 优先使用：造"数值档位族"或按玩家状态动态挂组件，按[注册与构造](#注册与构造)对照复刻；只调数值查下方表；举重/投掷全链见[关联机制导航](#关联机制导航)；温度换热细节复用 [[reference/prefabs/heatrock]]。
- 重要副作用：近战与投掷都给沃尔夫冈涨体格（[OnAttack](../../../raw/current/scripts/prefabs/dumbbells.lua#L279-L283)、[onthrown](../../../raw/current/scripts/prefabs/dumbbells.lua#L97-L104)）；耐久耗尽时持有中直接移除、在地面则禁拾取并 1 秒后侵蚀；冰铃 AOE 给命中目标（含掷出者自己）加 2 冻结值，火铃落点撒 3 个 houndfire；持哑铃移动时体格自然衰减被推迟（[DoDec](../../../raw/current/scripts/components/mightiness.lua#L248-L256)）。
- 修改边界：equippable `restrictedtag="strongman"` 只允许沃尔夫冈装备（[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L162)）；complexprojectile 与 `punch`/`lifting` 是运行时形态而非构造期状态；投掷技能加成只进 AOE 路径；本页不覆盖健身房（mightygym）与皮肤（无注册）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dumbbell`（哑铃） | `MakeDumbbell` 包装工厂 → `fn` 公共构造 → `SetPristine` 端分界 → 主模拟组件；无 postinit | 基础档：效率 MED/MED/LOW；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L636) |
| `dumbbell_golden`（黄金哑铃） | 同上 | 效率 HIGH/MED/LOW；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L637) |
| `dumbbell_marble`（大理石哑铃） | 同上 | 效率 HIGH/HIGH/MED；唯一 `walkspeedmult=0.9` 减速档；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L638) |
| `dumbbell_gem`（宝石哑铃） | 同上 | 效率全 HIGH；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L639) |
| `dumbbell_heat`（热铃） | 同上＋专属温度块 | 暖石式五档温差、耐久 150、可投喂蚁狮、附属 heatrocklight 灯；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L640)、[专属块](../../../raw/current/scripts/prefabs/dumbbells.lua#L575-L611) |
| `dumbbell_redgem`（火铃） | 同上＋`lootdropper` | `isfireattack`：AOE 落点撒 3 个 houndfire；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L641)、[撒火](../../../raw/current/scripts/prefabs/dumbbells.lua#L127-L132) |
| `dumbbell_bluegem`（冰铃） | 同上 | `isiceattack`：AOE 命中 `freezable:AddColdness(2)`；[注册行](../../../raw/current/scripts/prefabs/dumbbells.lua#L642)、[冻结](../../../raw/current/scripts/prefabs/dumbbells.lua#L162-L169) |

共同初始化（[dumbbells.lua#L500-L631](../../../raw/current/scripts/prefabs/dumbbells.lua#L500-L631)）：实体组件＋`MakeInventoryPhysics`→bank/build＝注册名＋`idle`→公共端标签 `dumbbell`／`keep_equip_toss`（heat 另加 `HASHEATER`／`icebox_valid`／`heatrock`）→`MakeInventoryFloatable("small", 0.15, 0.9)`→`reticule`（[投掷准星](../../../raw/current/scripts/prefabs/dumbbells.lua#L45-L58)：3.5–6.5 半径逐 0.25 找可落点，无 `projectile` 标签时隐藏）→`SetPristine`（非主模拟返回）→主模拟依次 inventoryitem→inspectable→equippable（默认 HANDS＋`restrictedtag="strongman"`）→weapon（`SetDamage`＋`SetOnAttack`＋`attackwear=consumption×2`）→finiteuses（`SetOnFinished`）→`MakeHauntableLaunch`→mightydumbbell（`SetConsumption`＋`SetEfficiency`）；redgem 额外 lootdropper。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验（[.po](../../../raw/current/scripts/languages/chinese_s.po#L342359-L342392)）：`STRINGS.NAMES.DUMBBELL`＝哑铃、`DUMBBELL_GOLDEN`＝黄金哑铃、`DUMBBELL_MARBLE`＝大理石哑铃、`DUMBBELL_GEM`＝宝石哑铃、`DUMBBELL_HEAT`＝热铃、`DUMBBELL_REDGEM`＝火铃、`DUMBBELL_BLUEGEM`＝冰铃；动作 `LIFT_DUMBBELL`＝举起（[.po](../../../raw/current/scripts/languages/chinese_s.po#L1209-L1211)）、`STOP_LIFT_DUMBBELL`＝停止举重（[.po](../../../raw/current/scripts/languages/chinese_s.po#L2024-L2026)）。

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L38](../../../raw/current/scripts/tuning.lua#L31-L38)；效率三档 LOW/MED/HIGH＝1.5/3.0/5.0，[tuning.lua#L6487-L6489](../../../raw/current/scripts/tuning.lua#L6487-L6489)）：

| 项 | 键／来源 | 实际值 |
|---|---|---|
| 伤害 | `DUMBBELL_DAMAGE_ROCK/GOLD/MARBLE/GEM/HEAT/REDGEM/BLUEGEM`＝`wilson_attack×(0.5/0.8/1/1.25/0.5/1.25/1.25)` | 17/27.2/34/42.5/17/42.5/42.5 生命值/击（源码推导，[tuning.lua#L6472-L6479](../../../raw/current/scripts/tuning.lua#L6472-L6479)） |
| 举重消耗（每次举起） | `DUMBBELL_CONSUMPTION_ROCK/GOLD/MARBLE/GEM/BLUEGEM/REDEM/HEAT` | 0.8/0.5/0.3/0.2/0.3/0.3/0.8 耐久点（[tuning.lua#L6463-L6470](../../../raw/current/scripts/tuning.lua#L6463-L6470)、[消费](../../../raw/current/scripts/components/mightydumbbell.lua#L92-L108)） |
| 耐久上限 | finiteuses 默认；heat 用 `DUMBBELL_HEAT_MAX_USES` | 六档 100、热铃 150（[默认](../../../raw/current/scripts/components/finiteuses.lua#L12-L13)、[heat 上限](../../../raw/current/scripts/prefabs/dumbbells.lua#L577-L578)、[键值](../../../raw/current/scripts/tuning.lua#L7880)） |
| 近战磨损/击 | `attackwear`＝consumption×`DUMBBELL_ATTACK_CONSUMPTION_MULT`（默认值，另受 `attackwearmultipliers` 与 efficientuser 修正） | 默认消耗×2（[设置](../../../raw/current/scripts/prefabs/dumbbells.lua#L561)、[倍率 2](../../../raw/current/scripts/tuning.lua#L6482)、[消费](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |
| 投掷落地磨损 | `thrown_consumption`＝consumption×`DUMBBELL_THROWN_CONSUMPTION_MULT` | 消耗×10，落地结算一次（[设置](../../../raw/current/scripts/prefabs/dumbbells.lua#L625)、[消费](../../../raw/current/scripts/prefabs/dumbbells.lua#L185)、[倍率 10](../../../raw/current/scripts/tuning.lua#L6483)） |
| 攻击涨体格 | `DUMBBELL_EFFICIENCY_ATTCK_SCALE` | 效率值×0.5 体格/击或/次投掷（[tuning.lua#L6490](../../../raw/current/scripts/tuning.lua#L6490)、[DoAttackWorkout](../../../raw/current/scripts/components/mightydumbbell.lua#L84-L90)） |
| 投掷 AOE | `OnThrownHit` FindEntities 半径 2，须 `_combat`＋`_health` | 对每个可伤目标走攻击者 `combat:DoAttack`（[实现](../../../raw/current/scripts/prefabs/dumbbells.lua#L124-L170)） |
| 投掷技能加成 | `SKILLS.WOLFGANG_DUMBELL_TOSS_1/2`（键名拼写 DUMBELL） | ×1.5／×2，仅 AOE 路径（[tuning.lua#L7139-L7140](../../../raw/current/scripts/tuning.lua#L7139-L7140)、[damage 函数替换](../../../raw/current/scripts/prefabs/dumbbells.lua#L139-L149)） |
| 大理石移速 | `DUMBBELL_SLOW_MARBEL` | ×0.9（[tuning.lua#L6485](../../../raw/current/scripts/tuning.lua#L6485)、[walkspeedmult 消费](../../../raw/current/scripts/components/equippable.lua#L126-L144)） |
| 飞行参数 | complexprojectile 硬编码 | 水平速度 15、重力 −35、高弧线、发射偏移 (1,1,0)（[MakeTossable](../../../raw/current/scripts/prefabs/dumbbells.lua#L211-L223)） |

获取途径（全 scripts 字面 `SpawnPrefab("dumbbell…")` 零命中，未检索到直接生成；该检索不覆盖数据表与拼名调用，世界生成／掉落仍按未穷尽处理。七条配方可覆盖正常获取，debugcommands／scrapbook 数据非生成点）：

| 途径 | 内容与证据 |
|---|---|
| 配方（全 `TECH.NONE`＋builder 门禁） | dumbbell＝rocks4＋twigs1（`builder_tag="strongman"`）；golden＝goldnugget4＋twigs1；marble＝marble4＋twigs1；gem＝thulecite2＋purplegem1＋twigs1——四档 `builder_tag="strongman"`；heat＝rocks12＋pickaxe1＋flint4；redgem＝thulecite2＋redgem1＋twigs1；bluegem＝thulecite2＋bluegem1＋twigs1——三档 `builder_skill="wolfgang_dumbbell_crafting"`。[recipes.lua#L255-L262](../../../raw/current/scripts/recipes.lua#L255-L262) |
| 制作菜单分组 | Characters 过滤器 Wolfgang 组。[recipes_filter.lua#L222-L230](../../../raw/current/scripts/recipes_filter.lua#L222-L230) |
| 技能树 | `wolfgang_dumbbell_crafting`（training 组根技能）→`throwing_1`→`throwing_2`。[skilltree_wolfgang.lua#L106-L142](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L106-L142) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage`＋`SetOnAttack`＋`attackwear`；OnAttack 回调给持有者涨体格 | 主模拟；投掷 AOE 中通过命中检查的目标再走一次 `Weapon:OnAttack`（早退目标不扣） | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/dumbbells.lua#L558-L561)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | 上限 100（heat 150）；耗尽回调：持有中 Remove、在地面禁拾取＋1 秒后 `ErodeAway` | 主模拟；举重/近战/投掷三条消耗路径共用 | [[reference/components/finiteuses#关键接口]]；[回调](../../../raw/current/scripts/prefabs/dumbbells.lua#L563-L573) |
| 组件：equippable | `SetOnEquip`/`SetOnUnequip` 覆盖 swap_object＋ARM 显隐；`restrictedtag="strongman"`；marble walkspeedmult | 主模拟；默认 HANDS；unequip 时若 `lifting` 推 `stopliftingdumbbell{instant=true}` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/dumbbells.lua#L550-L556)、[onunequip](../../../raw/current/scripts/prefabs/dumbbells.lua#L268-L277) |
| 组件：mightydumbbell | `SetConsumption`＋`SetEfficiency(三档)`；`DoWorkout` 涨体格＋耗耐久、`DoAttackWorkout` 只涨不耗 | 主模拟；按使用者 mightiness.current 对阈值 25/75 选效率档 | [[reference/components/mightydumbbell]]；[挂载](../../../raw/current/scripts/prefabs/dumbbells.lua#L619-L621)、[选档](../../../raw/current/scripts/components/mightydumbbell.lua#L24-L42) |
| 组件：complexprojectile | **运行时动态挂/卸**：装备者 mighty 态 `MakeTossable`（speed 15、gravity −35、OnLaunch/OnHit/OnDeflect、`ismeleeweapon=true` 保持近战），非 mighty 卸 | 主模拟；装备/拾取时按态即时评估，`mightiness_statechange` 驱动 | [[reference/components/complexprojectile#关键接口]]；[MakeTossable](../../../raw/current/scripts/prefabs/dumbbells.lua#L211-L223)、[CheckMightiness](../../../raw/current/scripts/prefabs/dumbbells.lua#L239-L254) |
| 组件：reticule | 公共端投掷准星（目标函数、隐藏条件、twinstick 参数） | 公共端（SetPristine 前） | [[reference/components/reticule#关键接口]]；[接线](../../../raw/current/scripts/prefabs/dumbbells.lua#L534-L540) |
| 组件：tradable/temperature/heater（heat） | rocktribute=6 可投喂蚁狮；temperature 绝热 INSULATION_MED、忽略 `heatrock` 热源；heater 三通道共用 HeatFn（五档输出 −10/10/25/40/60、携带倍率 2.1） | 仅 `dumbbell_heat` 主模拟；≤1 档触发蚁狮 freeze、≥4 档触发 burn | [[reference/prefabs/heatrock]]（同款五档算法与灯）、[[reference/components/heater]]；[heat 块](../../../raw/current/scripts/prefabs/dumbbells.lua#L575-L611)、[蚁狮消费](../../../raw/current/scripts/prefabs/antlion.lua#L116-L129) |
| 组件：lootdropper（redgem） | AOE 落点 `FlingItem` 3 个 houndfire | 仅 `dumbbell_redgem` 主模拟 | [[reference/components/lootdropper#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/dumbbells.lua#L615-L617) |
| 组件：floater／hauntable | `MakeInventoryFloatable("small", 0.15, 0.9)`；`MakeHauntableLaunch` 作祟弹飞 | floater 公共端、hauntable 主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/dumbbells.lua#L513)、[调用](../../../raw/current/scripts/prefabs/dumbbells.lua#L613) |
| SG：SGwilson（玩家宿主，Wiki 待建） | TOSS→`throw_keep_equip`（`keep_equip_toss` 选中该状态：动作物品不在 HANDS 时先补装备再投、投出后恢复原装备；第 7 帧 `PerformBufferedAction`）；LIFT→`use_dumbbell_pre/loop/pst`（每轮 animover 调 `dumbbelllifter:Lift()`，体格满 100% 收势）；`punch` 标签→拳击动画、冷却 24 帧（持械 13 帧） | 玩家实体服务端 SG；客户端镜像同判 | [TOSS Handler](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1424-L1432)、[throw_keep_equip](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10587-L10643)、[use_dumbbell_pre](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8157-L8202)、[Lift 调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8296-L8301)、[punch 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8653-L8670) |
| API／数据：TOSS/LIFT 动作 | componentactions 按 complexprojectile 组件给 TOSS（`distance=8`，POINT 对地面／EQUIPPED 对实体）；按 mightydumbbell＋strongman＋已装备给 LIFT/STOP_LIFT | 主模拟动作、客户端提示；TOSS.fn 卸装落地→`Launch(pos, doer)` | [TOSS 定义](../../../raw/current/scripts/actions.lua#L501)、[TOSS.fn](../../../raw/current/scripts/actions.lua#L3953-L3993)、[LIFT 定义](../../../raw/current/scripts/actions.lua#L628)、[LIFT.fn](../../../raw/current/scripts/actions.lua#L5702-L5722)、[POINT 生成](../../../raw/current/scripts/componentactions.lua#L2070-L2076)、[实体目标生成](../../../raw/current/scripts/componentactions.lua#L2276-L2293) |
| API／数据：`mightiness_statechange` 事件 | 装备时监听持有者该事件→CheckMightiness 同步 tossable/punch 两形态 | 主模拟；由 mightiness:BecomeState 推送 | [[reference/components/mightiness]]；[监听](../../../raw/current/scripts/prefabs/dumbbells.lua#L265)、[推送](../../../raw/current/scripts/components/mightiness.lua#L373) |
| API／数据：`punch`/`lifting`/`dumbbell` 标签 | `punch` 只换攻击动画与节奏；`lifting` 标记举重中（LIFT/STOP 提示区分）；`dumbbell` 被 `mightiness:DoDec` 消费——持哑铃移动时衰减被 `DelayDrain(2)` 推迟 | 标签公共端可读，消费在主模拟 | [punch 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8653-L8670)、[lifting 消费](../../../raw/current/scripts/componentactions.lua#L2390-L2396)、[DoDec](../../../raw/current/scripts/components/mightiness.lua#L248-L256) |
| API／数据：wolfgang 接线 | wolfgang 主模拟挂 mightiness＋dumbbelllifter；近战命中 wolfgang `OnHitOther` 涨 0.25/0.5/1（小怪/默认/巨怪），与哑铃涨力并行；mighty 态 `externaldamagemultiplier=2` 放大武器伤害 | 阈值 25/75、上限 100（健身房可超额）；Wolfgang 专属 | [[reference/components/mightiness]]、[[reference/components/dumbbelllifter#原版训练链]]；[挂载](../../../raw/current/scripts/prefabs/wolfgang.lua#L667-L674)、[OnHitOther](../../../raw/current/scripts/prefabs/wolfgang.lua#L244-L259)、[伤害倍率](../../../raw/current/scripts/components/combat.lua#L65) |
| 关联 Prefab：`houndfire` | redgem AOE 落点生成并抛撒（`prefabs` 依赖声明） | 仅火铃、主模拟 | Wiki 待建；[生成](../../../raw/current/scripts/prefabs/dumbbells.lua#L127-L132) |
| 关联 Prefab：`heatrocklight` | heat 档 5 启 bloom＋附属灯实体（灯强随温差渐变），灯父实体沿持有链转移 | 仅热铃 | [[reference/prefabs/heatrock]]（同工厂）；[UpdateImages](../../../raw/current/scripts/prefabs/dumbbells.lua#L392-L413)、[灯归属](../../../raw/current/scripts/prefabs/dumbbells.lua#L453-L494) |
| 关联 Prefab：`round_puff_fx_sm` | 投掷落地烟尘 | 全档 | [[reference/prefabs/round_puff_fx_sm]]；[生成](../../../raw/current/scripts/prefabs/dumbbells.lua#L175) |
| 资源：anim／swap 动画包 | dumbbell*.zip（bank/build＝注册名）＋swap_dumbbell*.zip（持握）；heat 另有 dumbbell_heat1–5 分档 build 与图标 | heat 档换 build＋`ChangeImageName` | [声明](../../../raw/current/scripts/prefabs/dumbbells.lua#L1-L38)、[图标切换](../../../raw/current/scripts/prefabs/dumbbells.lua#L392-L405) |

## 典型接入

### 同步修改哑铃的消耗、磨损与耐久

- 前提与选择：服务端 `modmain.lua`。`consumption` 一个值在构造期分别写入三处（近战 attackwear、投掷 thrown_consumption、举重消耗），postinit 只改一处会失衡，须三处同步。
- 生效与撤销：postinit 只影响新实例；`SetMaxUses` 即时改上限、当前耐久按存档恢复；停用模组恢复原值。
- 原版依据：三处写入点（[attackwear](../../../raw/current/scripts/prefabs/dumbbells.lua#L561)、[thrown_consumption](../../../raw/current/scripts/prefabs/dumbbells.lua#L625)、[SetConsumption](../../../raw/current/scripts/prefabs/dumbbells.lua#L620)）；modmain 环境白名单只直挂 `TUNING` 等，`TheWorld` 须 `GLOBAL.` 前缀（[mods.lua env](../../../raw/current/scripts/mods.lua#L258-L317)）。

```lua
-- modmain.lua：石制哑铃改为 2 倍伤害，并同步磨损、消耗与耐久
AddPrefabPostInit("dumbbell", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local MULT = 2
    local weapon = inst.components.weapon
    local workout = inst.components.mightydumbbell
    local consumption = workout.consumption * MULT

    weapon:SetDamage(weapon.damage * MULT) -- 17 -> 34
    weapon.attackwear = consumption * TUNING.DUMBBELL_ATTACK_CONSUMPTION_MULT
    inst.thrown_consumption = consumption * TUNING.DUMBBELL_THROWN_CONSUMPTION_MULT
    workout:SetConsumption(consumption)

    local uses = inst.components.finiteuses
    uses:SetMaxUses(uses.total * MULT)
    uses:SetUses(uses:GetUses() * MULT)
end)
```

- 调整与验证：耐久单位为耐久点；attackwear 作用于近战与投掷 AOE 中通过命中检查的目标，thrown_consumption 只在落地结算一次。改效率数值须命中 `efficiency_rate_scale` 的 LOW/MED/HIGH 三键，见 [[reference/components/mightydumbbell]]。

## 常见陷阱与限制

- **边界：投掷磨损是"落地＋每目标"双重计数**：落地结算 `thrown_consumption`（消耗×10）一次（[L185](../../../raw/current/scripts/prefabs/dumbbells.lua#L185)），AOE 内**通过命中检查**的目标再经 `Weapon:OnAttack` 扣 `attackwear`（默认消耗×2，受磨损倍率与效率修正；`CanHitTarget` 失败的目标早退不扣）（[combat.lua#L1147-L1154](../../../raw/current/scripts/components/combat.lua#L1147-L1154)、[L1223-L1225](../../../raw/current/scripts/components/combat.lua#L1223-L1225)）；空地投掷只有落地一次。
- **边界：投掷技能加成只进 AOE 路径**：`OnThrownHit` 临时把 `weapon.damage` 换成带技能倍率的函数、结算后还原（[L139-L172](../../../raw/current/scripts/prefabs/dumbbells.lua#L139-L172)）；普通近战不吃 `WOLFGANG_DUMBELL_TOSS_1/2`（tuning 键拼写为 DUMBELL，引用时勿写成 DUMBBELL）。
- **误区：wimpy 态 `punch` 标签改变伤害**：只把攻击动画换成拳击并放慢节奏（24 帧 vs 13 帧），weapon.damage 不变；wimpy 输出减益来自 mightiness 侧 `externaldamagemultiplier=0.75`（mighty 为 2），作用于"对他人伤害"倍率。[标签分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8653-L8670)、[倍率定义](../../../raw/current/scripts/components/mightiness.lua#L10)、[伤害消费](../../../raw/current/scripts/components/combat.lua#L940)
- **边界：`dumbbell_heat` 的温度档检视状态是死代码**：本地 `GetStatus`（FROZEN/COLD/WARM/HOT）定义了但从未接到 `inspectable.getstatus`（heatrock.lua 有接线 [L230](../../../raw/current/scripts/prefabs/heatrock.lua#L230)，本文件无）；检视文本不随温度档变化。[未接线](../../../raw/current/scripts/prefabs/dumbbells.lua#L359-L369)、[消费端](../../../raw/current/scripts/components/inspectable.lua#L27-L49)
- **误区：普通角色拿到哑铃也能投掷/举重**：`restrictedtag="strongman"` 拦装备（[inventory.lua#L1270](../../../raw/current/scripts/components/inventory.lua#L1270)），LIFT 提示要求 doer 有 `strongman` 标签且哑铃已装备（[componentactions.lua#L2809-L2816](../../../raw/current/scripts/componentactions.lua#L2809-L2816)）；TOSS 依赖装备期动态挂上的 complexprojectile。
- **边界：complexprojectile 是运行时形态**：非 mighty 态装备的哑铃无该组件；强挂可绕过，但 `ReticuleShouldHideFn`、combat 投掷分支（[ismeleeweapon](../../../raw/current/scripts/components/combat.lua#L1177)）与弹道参数都会随之变行为。[切换链](../../../raw/current/scripts/prefabs/dumbbells.lua#L239-L254)
- **边界：耐久耗尽的地面哑铃不可再拾取**：`OnFinished` 在未持有时禁拾取＋NOCLICK＋1 秒后 `ErodeAway`，无残骸。[回调](../../../raw/current/scripts/prefabs/dumbbells.lua#L563-L573)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（[BUILD.txt](../../../raw/current/BUILD.txt)）。核验 prefabs/dumbbells.lua 全文（643 行）与工厂 7 注册；mightydumbbell/dumbbelllifter/mightiness 组件全文；weapon（GetDamage/OnAttack）、equippable（默认 HANDS/IsRestricted）、finiteuses（Use/SetUses）、complexprojectile（Launch/CalculateTrajectory）、combat（DoAttack/CanHitTarget/IsAlly/伤害倍率）、inventory（DropItem 卸装）、inspectable（getstatus 解析）、standardcomponents 两 helper；actions/componentactions 的 TOSS/LIFT/STOP 定义与生成；SGwilson throw_keep_equip/use_dumbbell_pre/loop/pst/attack punch 分支；wolfgang.lua 挂载与 OnHitOther；antlion 供品链；tuning 全部 DUMBBELL/HEATROCK/MIGHTINESS 键与 SKILLS.TOSS；recipes 7 行与过滤器分组；技能树 3 节点；全 scripts grep 字面无 SpawnPrefab 生成点（未覆盖数据表/拼名，按未穷尽处理）、无皮肤注册、无 quagmire 分支；名称键经 chinese_s.po `msgctxt` 逐档核验。
- 游戏运行：未运行。伤害/磨损/涨力等数值均为源码推导，非运行测量。
- 结构检查：validate_page.py 通过（135 行／24711 字节，standard 档 240/24576 内）；validate_source_links.py --strict 通过（104 处源码行锚点，0 error 0 warning）。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
