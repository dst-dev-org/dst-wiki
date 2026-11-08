---
title: "战斗圆盾（wathgrithr_shield）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版女武神技能树专属手持盾牌 wathgrithr_shield：装备时 85% 被动吸收＋51 伤害，右键主动招架（178° 扇区、10 秒冷却，传入 1 秒窗口在 pre 后计时）成功免伤并回充冷却；耐久在 armor 组件、耗尽整体移除，可借鉴'主动格挡＋冷却'武器组装。"
aliases: ["wathgrithr_shield", "战斗圆盾", "Battle Rönd", "盾牌格挡", "女武神盾牌"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wathgrithr_shield.lua", "components/weapon.lua", "components/armor.lua", "components/rechargeable.lua", "components/parryweapon.lua", "components/aoespell.lua", "components/aoetargeting.lua", "components/equippable.lua", "components/equippable_replica.lua", "components/inventory.lua", "components/combat.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/builder.lua", "components/playeractionpicker.lua", "stategraphs/SGwilson.lua", "actions.lua", "componentactions.lua", "standardcomponents.lua", "mods.lua", "tuning.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "prefabs/skilltree_wathgrithr.lua", "prefabs/reticulearc.lua", "scrapbook_prefabs.lua", "screens/redux/scrapbookdata.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 战斗圆盾（wathgrithr_shield）

## 快速判断

- 职责与适用：`prefabs/wathgrithr_shield.lua` 顶层 `fn` 注册的单注册可拾取手持盾牌（物品＋装备），女武神技能树专属：weapon 伤害＋armor 被动吸收与耐久＋equippable HANDS 槽＋aoespell/aoetargeting 右键施放＋parryweapon 招架＋rechargeable 冷却。**没有 finiteuses**——磨损走 armor，与 spear／batbat 的 finiteuses 模式不同。同文件无 quagmire 分支、快照无已注册皮肤。[工厂](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L138-L218)、[注册](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L220)
- 优先使用：想做"主动格挡＋冷却"武器，按[注册与构造](#注册与构造)复刻四件套（aoespell＋aoetargeting＋parryweapon＋rechargeable）并读[典型接入](#典型接入)；招架角度/冷却/成功效果与双端 SG 接线细节查 [[reference/components/parryweapon#原版动作与清理链]]。
- 重要副作用：装备态提供 85% 吸收（多件护甲共穿取最高吸收率、按各自吸收率分摊磨损）；右键施放进入招架窗口，成功格挡本击完全免伤（护甲分支整个跳过）且冷却低于 70% 时才推进到 70%；每次攻击磨盾 3 点；耐久耗尽整个实体移除。[免伤分支](../../../raw/current/scripts/components/combat.lua#L615)、[回充](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L99-L101)、[耗尽移除](../../../raw/current/scripts/components/armor.lua#L101-L111)
- 修改边界：aoetargeting 挂公共端（客户端要用它建瞄准环，[分界](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L167-L184)），其余组件挂主模拟；角色门禁分两层——配方 `builder_skill`＋装备 `restrictedtag`（见[获取与门禁](#获取与门禁)）；玩家 SG 的招架状态本身**不女武神限定**。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wathgrithr_shield` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备＋施法；快照 `prefabskin.lua`／`skinprefabs.lua` 未注册本 prefab 皮肤（grep 无命中），但装备回调内已预留皮肤接线。[注册](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L220)、[皮肤分支](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L53-L57) |

共同初始化（[wathgrithr_shield.lua#L138-L218](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L138-L218)）：实体组件＋`MakeInventoryPhysics` → bank／build 均为 `wathgrithr_shield`＋`idle` → 公共端标签 `toolpunch`／`battleshield`／`shield`／`parryweapon`／`weapon`／`rechargeable`（pristine 镜像注释）→ `MakeInventoryFloatable(nil, 0.2, {1.1, 0.6, 1.1})`（size 缺省落 `"small"`，[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1774)）→ **aoetargeting（挂公共端，`SetPristine` 之前）** → `SetPristine`（非主模拟返回）→ 主模拟依次 inspectable → inventoryitem → weapon → armor → equippable → aoespell → parryweapon → rechargeable → `MakeHauntableLaunch`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L654-L660)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.WATHGRITHR_SHIELD`＝战斗圆盾（Battle Rönd）；施放动作 `STRINGS.ACTIONS.CASTAOE.WATHGRITHR_SHIELD`＝格挡（Block）。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L351890-L351892)、[.po 动作](../../../raw/current/scripts/languages/chinese_s.po#L464-L467)

### 数值

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1、`wilson_health`＝150、护甲吸收倍率 1／耐久倍率 0.7，[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.WATHGRITHR_SHIELD_DAMAGE`＝`wilson_attack`×1.5 | 51 生命值（源码推导，[tuning.lua#L8075](../../../raw/current/scripts/tuning.lua#L8076)） |
| 护甲吸收 | `TUNING.WATHGRITHR_SHIELD_ABSORPTION`＝0.85×1 | 85%（[tuning.lua#L8076](../../../raw/current/scripts/tuning.lua#L8077)） |
| 护甲耐久上限 | `TUNING.WATHGRITHR_SHIELD_ARMOR`＝150×4×0.7 | 420 耐久点（[tuning.lua#L8077](../../../raw/current/scripts/tuning.lua#L8078)） |
| 每次攻击磨损 | `TUNING.WATHGRITHR_SHIELD_USEDAMAGE` | 3 耐久点/次，`armor:TakeDamage`（[tuning.lua#L8078](../../../raw/current/scripts/tuning.lua#L8079)、[OnAttackFn](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L121-L126)） |
| 招架扇区角 | `TUNING.WATHGRITHR_SHIELD_PARRY_ARC` | 178 度（判定阈 89°，[tuning.lua#L8080](../../../raw/current/scripts/tuning.lua#L8081)） |
| 施法冷却 | `TUNING.WATHGRITHR_SHIELD_COOLDOWN` | 10 秒（施法 `Discharge`，[tuning.lua#L8082](../../../raw/current/scripts/tuning.lua#L8083)） |
| 装备冷却下限 | `TUNING.WATHGRITHR_SHIELD_COOLDOWN_ONEQUIP` | 2 秒（[tuning.lua#L8083](../../../raw/current/scripts/tuning.lua#L8084)） |
| 招架窗口时长 | `TUNING.WATHGRITHR_SHIELD_PARRY_DURATION` | 传入 1 秒（`parry_pre` 结束才交 `parry_idle` 计时）；技能 2 ×2.5＝2.5 秒（[tuning.lua#L8084](../../../raw/current/scripts/tuning.lua#L8085)、[倍率](../../../raw/current/scripts/tuning.lua#L7218)） |
| 招架成功回充 | `TUNING.WATHGRITHR_SHIELD_COOLDOWN_ONPARRY_REDUCTION` | 0.7：不足 70% 充能时推进到 70%，剩余冷却 3 秒（[tuning.lua#L8085](../../../raw/current/scripts/tuning.lua#L8086)、[OnParry](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L99-L101)） |
| 招架反击加成 | `SKILLS.WATHGRITHR.SHIELD_PARRY_BONUS_DAMAGE_SCALE`／`..._BONUS_DAMAGE` | clamp(来袭伤害×0.5, 15, 30)，持续 5 秒（[tuning.lua#L7219-L7221](../../../raw/current/scripts/tuning.lua#L7220-L7222)） |

### 招架流程（施放 → 判定 → 免伤）

1. **施放**：盾装备在 HANDS 时右键点地，动作选择器对装备槽物品收集 POINT 动作并先查 `aoetargeting:IsEnabled()`（[playeractionpicker.lua#L455-L467](../../../raw/current/scripts/components/playeractionpicker.lua#L455-L467)）；`componentactions` 的 `aoespell` 分支再查落点可施放后给 `CASTAOE`（[componentactions.lua#L2026-L2056](../../../raw/current/scripts/componentactions.lua#L2026-L2056)）。充能中 `OnDischarged` 已禁用 aoetargeting（[wathgrithr_shield.lua#L128-L130](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L128-L130)），服务端 `aoespell:CanCast` 亦拒绝（[aoespell.lua#L53-L57](../../../raw/current/scripts/components/aoespell.lua#L53-L57)）。
2. **入状态与下发窗口**：`CASTAOE` 按武器 `parryweapon` 标签选玩家 SG 状态 `parry_pre`（[SGwilson.lua#L1326](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1326)）；状态内同步执行动作 → `aoespell:CastSpell` → 盾 `SpellFn`：按技能 2 换算时长后 `EnterParryState`（时长先存 SG、前摇结束才交 `parry_idle`）并 `Discharge(10)` 开始冷却（[SpellFn](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L85-L94)、[EnterParryState](../../../raw/current/scripts/components/parryweapon.lua#L35-L37)、[动作执行](../../../raw/current/scripts/actions.lua#L4535-L4542)）。盾动画变体由公共端 `shield` 标签选 `shieldparry_*`（[SGwilson.lua#L5757](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5759)）。
3. **判定**：`parry_pre` 在监听窗口内给持有者装 `combat.redirectdamagefn` 闭包，每次来袭调 `TryParry`——拒绝非空且非 `"stun"` 的 stimuli 与地刺攻击者（`nil` 普通物攻照常判角度），按 178° 扇区判防御者朝向（失败且攻击者有 locomotor 时再判其对向），成功调 `OnParryFn` 并返回 true（[闭包](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5776-L5786)、[TryParry](../../../raw/current/scripts/components/parryweapon.lua#L39-L67)）。`OnParry` 播镜头震动、回充冷却，技能 3 时记录反击加成（[OnParry](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L96-L111)）。
4. **免伤**：招架成功时 `combat` 整体跳过护甲吸收与扣血分支（盾自身无 combat 组件，重定向不做二次受击结算），仍推带 `redirected` 的 `attacked` 事件 → SG 转 `parry_hit`；招架失败或窗口外走普通受击，盾作为 HANDS 护甲参与吸收（单盾 85%，多件取最高吸收率并按各件吸收率分摊磨损）（[免伤分支](../../../raw/current/scripts/components/combat.lua#L615-L691)、[事件](../../../raw/current/scripts/components/combat.lua#L716)、[parry_hit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1896-L1904)、[吸收收集](../../../raw/current/scripts/components/inventory.lua#L419-L424)、[磨损](../../../raw/current/scripts/components/inventory.lua#L494-L497)）。
5. **收尾**：前摇到点转 `parry_idle` 续窗口，退出链清理 `redirectdamagefn`，详见 [[reference/components/parryweapon#原版动作与清理链]]。

### 获取与门禁

| 途径 | 内容与证据 |
|---|---|
| 配方（唯一获取路径） | 金块 4＋牛毛 3，`TECH.NONE` 无需站台；`builder_skill="wathgrithr_arsenal_shield_1"`，未激活该技能时 `KnowsRecipe`／`CanLearn` 拒绝（[配方](../../../raw/current/scripts/recipes.lua#L196)、[KnowsRecipe 检查](../../../raw/current/scripts/components/builder.lua#L876-L878)、[CanLearn 检查](../../../raw/current/scripts/components/builder.lua#L964-L966)）。全 scripts grep 除配方、图鉴与列表数据外无其他生成点。 |
| 配方过滤 | crafting 菜单归 CHARACTER（Wathgrithr 分组）、WEAPONS、ARMOUR 三类（[CHARACTER](../../../raw/current/scripts/recipes_filter.lua#L182)、[WEAPONS](../../../raw/current/scripts/recipes_filter.lua#L1019)、[ARMOUR](../../../raw/current/scripts/recipes_filter.lua#L1059)） |
| 图鉴数据（非生成点） | scrapbook_prefabs 收录；`scrapbookdata` 展示 weapondamage=51／armor=420／absorb 0.85，与 tuning 推导一致（[收录](../../../raw/current/scripts/scrapbook_prefabs.lua#L1677)、[展示](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1747)） |

角色限制两层，都落在技能层而非 prefab 硬编码：

- **配方**：`builder_skill` 指向技能树节点 `wathgrithr_arsenal_shield_1`（[节点](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L307-L318)），经 `skilltreeupdater:IsActivated` 判定（[builder.lua#L876](../../../raw/current/scripts/components/builder.lua#L876)）。
- **装备**：`equippable.restrictedtag = "wathgrithrshielduser"`（[wathgrithr_shield.lua#L198-L201](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L198-L201)）；服务端 `Inventory:Equip` 调 `Equippable:IsRestricted`，目标无该标签即拒绝（[inventory.lua#L1270](../../../raw/current/scripts/components/inventory.lua#L1270)、[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L157)）；标签由同一技能节点 `onactivate` 添加、`ondeactivate` 移除（[skilltree_wathgrithr.lua#L316-L317](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L316-L317)）。读档宽容：`IsRestricted_FromLoad` 结合 `SKILLTREE_EQUIPPABLE_RESTRICTED_TAGS` 认女武神存档天然持有该标签（[equippable.lua#L164-L173](../../../raw/current/scripts/components/equippable.lua#L164-L173)、[constants.lua#L2157](../../../raw/current/scripts/constants.lua#L2157)）。
- **SG 不限定**：`parry_pre`／`parry_hit` 是玩家公共 SG 的 `CASTAOE` 分支，任何角色可用；女武神限定完全由配方与装备门禁承担。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(DamageFn)`（读 TUNING＋反击加成）＋`SetOnAttack(OnAttackFn)`（清加成、armor 磨 3 点） | 主模拟；命中先回调后磨损，无 finiteuses 即无 `usesdepleted` 链 | [[reference/components/weapon#关键接口]]；[接线](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L191-L193)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：armor | `InitCondition(420, 0.85)`；无 `SetOnFinished`／`SetKeepOnFinished`，耗尽走默认 Remove | 主模拟；只对装备持有者生效（inventory 遍历全部装备槽，HANDS 在内） | [[reference/components/armor#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L195-L196)、[耗尽移除](../../../raw/current/scripts/components/armor.lua#L101-L111) |
| 组件：rechargeable | `Discharge(10)` 施法冷却、`SetPercent(0.7)` 招笔回充、装备下限 2 秒；`OnDischarged/OnCharged` 切 aoetargeting | 主模拟；`total`＝默认 180、`chargetime` 每次 Discharge 重写；冷却进度随存档保存 | [[reference/components/rechargeable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L211-L213)、[Discharge](../../../raw/current/scripts/components/rechargeable.lua#L129-L132)、[存档](../../../raw/current/scripts/components/rechargeable.lua#L150-L164) |
| 组件：parryweapon | `SetParryArc(178)`＋`SetOnParryFn(OnParry)`；`EnterParryState` 由 SpellFn 调 | 主模拟判定；`parryweapon` 标签 pristine 预置 | [[reference/components/parryweapon#关键接口]]、[[reference/components/parryweapon#原版动作与清理链]]；[接线](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L206-L209) |
| 组件：aoespell／aoetargeting | `SetSpellFn(SpellFn)`；aoetargeting 配弧形瞄准环（`reticulearc`，玩家前方 6.5 距离、鼠标跟随） | **aoetargeting 挂公共端**，含 net_bool 开关同步禁用态；aoespell 主模拟 | [[reference/components/aoespell#关键接口]]、[[reference/components/aoetargeting#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L167-L178)、[netvar](../../../raw/current/scripts/components/aoetargeting.lua#L40-L53) |
| 组件：equippable | `restrictedtag`＋装备/卸装覆盖 `lantern_overlay`／`swap_shield` 符号、ARM 显隐 | 主模拟；槽默认 HANDS；皮肤走 `OverrideItemSkinSymbol`（快照无注册皮肤） | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L198-L201)、[OnEquip](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L47-L66) |
| 组件：inspectable／inventoryitem | 检视与拾取，无定制 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L188-L189) |
| 组件：floater／hauntable | `MakeInventoryFloatable`（"small"）；`MakeHauntableLaunch` 作祟弹飞 | 公共端／主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[floater](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L165)、[hauntable helper](../../../raw/current/scripts/standardcomponents.lua#L1053) |
| API／数据：`combat.redirectdamagefn` | parry_pre 监听期间安装的招架闭包，是免伤的真正执行者 | 持有者 combat 的单槽回调；状态退出链负责清理 | [[reference/components/parryweapon#原版动作与清理链]]、[[reference/components/combat#关键接口]]；[安装](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5776-L5786)、[消费](../../../raw/current/scripts/components/combat.lua#L585) |
| API／数据：技能树节点 | `wathgrithr_arsenal_shield_1`（配方＋装备标签）、`_2`（窗口×2.5）、`_3`（反击加成） | 女武神技能树 arsenal 组；经 `skilltreeupdater` 查询 | [[reference/components/skilltreeupdater#关键接口]]；[节点](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L307-L330)、[倍率](../../../raw/current/scripts/tuning.lua#L7218-L7222) |
| API／数据：`toolpunch` 标签 | 装备态攻击播 `toolpunch` 拳击动画（与 shieldofterror 同标签） | 攻击状态时读取 | [SGwilson.lua#L10845-L10855](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10845-L10855) |
| API／数据：`battleshield` 标签 | 公共端添加；快照全 scripts grep 无消费者 | — | [添加](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L153) |
| 关联 Prefab：reticulearc／reticulearcping | aoetargeting 的瞄准环与落点 ping | 客户端表现；同文件注册两个 prefab | [声明](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L7-L11)、[注册](../../../raw/current/scripts/prefabs/reticulearc.lua#L65-L66) |
| 资源：anim／swap 动画包 | `wathgrithr_shield.zip`（bank＋build）／`swap_wathgrithr_shield.zip`（持握符号 `swap_shield`） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L1-L5)、[符号覆盖](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L59-L61) |

## 典型接入

### 收窄招架角并延长招架窗口

- 前提与选择：服务端 `modmain.lua`。招架角在盾构造时读 `TUNING.WATHGRITHR_SHIELD_PARRY_ARC` 一次性存进组件（[工厂](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L207)），改它必须 `AddPrefabPostInit` 重设；招架窗口在每次施法时读 `TUNING.WATHGRITHR_SHIELD_PARRY_DURATION`（[SpellFn](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L85-L94)），直接改键即时全局生效。modmain 环境直挂 `TUNING` 白名单，`TheWorld`／`SpawnPrefab` 等必须 `GLOBAL.` 前缀（[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)、[TUNING 直挂](../../../raw/current/scripts/mods.lua#L278)）。
- 生效与撤销：postinit 只影响新实例；TUNING 改键即时影响所有实例（含已有盾的下一次施法）。停用模组恢复原值。
- 原版依据：角度与时长消费点如上；判定阈＝arc/2，技能 2 的 2.5 倍率会乘在修改后的基础时长上。

```lua
-- modmain.lua：战斗圆盾招架角收窄到 90 度、招架窗口延长到 1.5 秒
AddPrefabPostInit("wathgrithr_shield", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.parryweapon ~= nil then
        inst.components.parryweapon:SetParryArc(90) -- 完整扇区度数，判定阈为其一半
    end
end)

-- 招架窗口：每次施法时读取，改键即时生效（技能 wathgrithr_arsenal_shield_2 仍会 ×2.5）
TUNING.WATHGRITHR_SHIELD_PARRY_DURATION = 1.5
```

- 调整与验证：角度单位度、窗口单位秒；冷却（10 秒）、回充（70%）与反击加成可按[数值](#数值)同法处理——冷却时长随施法 `Discharge` 写入组件，改 `TUNING.WATHGRITHR_SHIELD_COOLDOWN` 只影响之后的施法。示例未进游戏运行。

## 常见陷阱与限制

- **边界：耐久在 armor 不在 finiteuses**：磨损两条路径——攻击磨 3 点（OnAttackFn）、吸收伤害按吸收份额磨（inventory:ApplyDamage）；招架成功的本击两者都不发生。耐久到 0 无 `onfinished` 且未 `SetKeepOnFinished` → 整个实体 Remove，无掉落。[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L162)、[Remove](../../../raw/current/scripts/components/armor.lua#L101-L111)
- **边界：装备即至少 2 秒冷却**：OnEquip 检查剩余充能 <2 秒就 `Discharge(2)`——满充能状态重新装备也要等 2 秒才能施放；期间 aoetargeting 禁用、右键无格挡动作。[OnEquip](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L63-L65)
- **边界：招架要主动施放且有窗口**：不是装备即自动格挡；10 秒冷却＋前摇后 1 秒窗口（技能 2.5 秒），窗口内还要求 178° 朝向；非空且非 `"stun"` 的 stimuli 与地刺攻击不可招架（`nil` 普通物攻可招架）。[TryParry](../../../raw/current/scripts/components/parryweapon.lua#L39-L67)
- **边界：女武神限定在配方与装备层**：SG 招架状态全角色通用；未激活技能的女武神就算拿到盾也装备不上。模组反向接法：给角色 `AddTag("wathgrithrshielduser")` 即可绕过装备门禁，但配方门禁独立于该标签，需另处理（`no_builder_skill` 无对应豁免，需改 recipe 或放行 builder）。[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L157)
- **边界：反击加成只覆盖下一次攻击，且需技能 3**：激活 `wathgrithr_arsenal_shield_3` 时招架才记录 `_lastparrytime` 与 `_bonusdamage`，5 秒内用盾攻击附加 clamp(0.5×该次招架所受伤害, 15, 30)；清空发生在 weapon 的 onattack 回调（`OnAttackFn`），不叠加。[OnAttackFn](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L121-L126)
- **边界：`battleshield` 标签无消费者**：快照全 scripts grep 仅自身添加行，勿据此推断有专属机制；`shield` 标签的可见消费只有 SG 招架动画变体选择。[battleshield](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L153)、[shield 变体](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5759)
- **边界：冷却进度随存档保存**：冷却中途存档时 `rechargeable` 存 `chargetime`＋`charge`，读档继续冷却；而装备门禁在读档时走 `IsRestricted_FromLoad` 宽容路径。[OnSave](../../../raw/current/scripts/components/rechargeable.lua#L150-L164)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394。核验 prefabs/wathgrithr_shield.lua 全文（单注册、端分界、SpellFn/OnParry/DamageFn/OnAttackFn）；weapon/armor/rechargeable/parryweapon/aoespell/aoetargeting/equippable（含 replica）/inventory/combat 关键方法；招架链 playeractionpicker→componentactions→CASTAOE→SGwilson parry_pre→EnterParryState/TryParry→combat 重定向免伤→parry_hit；配方 builder_skill、装备 restrictedtag 与读档宽容、技能树三节点；tuning 十键；recipes_filter；皮肤零注册；标签消费端；reticulearc；.po 三处名称；全 scripts grep 其余命中均非生成点。2026-11-07 第四轮复检：`nil` stimuli 可招架、1 秒在前摇后计时、85% 为单盾吸收率与多件分摊、反击需技能 3。
- 游戏运行：未运行。未覆盖：51 伤害／420 耐久／85% 吸收为源码推导非实测；实机招架手感（角度与时序）、骑乘禁用表现、客户端预测延迟未验证；客户端 SG 镜像未逐行核验，招架一致性沿用 [[reference/components/parryweapon#原版动作与清理链]]。
- 结构检查：validate_page.py 0 error（134 行／24529 字节，standard 档）与 validate_source_links.py --strict 0 error 0 warning（26 个源文件）。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 13 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
