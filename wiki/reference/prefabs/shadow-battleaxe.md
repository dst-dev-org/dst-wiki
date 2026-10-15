---
title: "暗影槌（shadow_battleaxe）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版裂隙暗影武器 shadow_battleaxe：HANDS 槽近战＋CHOP 双用，位面伤害/汲命/砍伐效率随 epic 巨兽击杀升 4 级，二级起持续饥饿（卸装不暂停）、耗尽残破可修复，会说话；同文件注册持握跟随 FX。"
aliases: ["shadow_battleaxe", "暗影槌", "Shadow Maul", "暗影战斧", "会说话的武器"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadow_battleaxe.lua", "prefabs/shadow_battleaxe_classified.lua", "prefabs/forge_repair_kits.lua", "prefabs/gestalt_guard.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/planardamage.lua", "components/damagetypebonus.lua", "components/hunger.lua", "components/talker.lua", "components/sanity.lua", "components/shadowlevel.lua", "components/tool.lua", "components/forgerepairable.lua", "components/forgerepair.lua", "components/aoediminishingreturns.lua", "components/spdamageutil.lua", "componentutil.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "strings.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影槌（shadow_battleaxe）

## 快速判断

- 职责与适用：`prefabs/shadow_battleaxe.lua` 注册的手持武器（物品＋装备）：weapon＋planardamage 位面＋tool（CHOP）＋equippable HANDS；击杀 epic 巨兽升 4 级、二级起持续饥饿（卸装不暂停）、对持有者说话。裂隙暗影阵营武器，暗影术基座制作；同文件注册 `shadow_battleaxe_fx`、独立文件 classified；无皮肤注册与 quagmire 分支。[工厂](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L699-L865)
- 优先使用：查数值→[数值表](#数值)；改成长/汲命→[等级与升级](#等级与升级epic-击杀)、[命中结算与汲命](#命中结算与汲命)；复刻"击杀成长武器"或"会说话的物品"读[典型接入](#典型接入)；从零造手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：装备期约 −0.056 理智/秒（magic 型，[dapperness](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L400-L405)）；二级起饥饿空每秒磨 1 耐久并说 starving 台词（与是否装备无关）；对 lunar_aligned 目标 ×1.1；`shadow_item` 让凝碎卫直接敌对（[gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L62-L65)）。
- 修改边界：本页覆盖主体＋同文件 FX＋classified 三注册；三注册与四级成长/残破修复/说话复制协作需整体保留（complex 档）；皮肤未注册（skinprefabs 无条目、无皮肤接线）；talker widget／floater 内部机制归组件页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_battleaxe` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；残破态移除四组件，修复后重组件。[注册](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1035) |
| `shadow_battleaxe_fx` | 同文件 `FollowSymbolFxFn`：网络跟随 FX 实体 | 持握/浮水视觉与等级同步 netvar；`persists = false`。[工厂](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L988-L1031) |
| `shadow_battleaxe_classified` | 独立文件 `fn`：隐藏网络代理实体 | 只承载说话文本/音效选择；主体生成、父子挂接。[工厂](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L124-L170) |

共同初始化（[fn](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L699-L865)）：实体组件＋`MakeInventoryPhysics` → bank/build `nightmare_axe`＋`idle_level1` → 公共端标签 `sharp`/`show_broken_ui`/`weapon`/`shadowlevel`/`shadow_item` → floater＋`isbroken` net_bool → talker（定制字体/颜色/偏移）→ 非专用端 `localsounds` 声音子实体 → `SetPristine`（非主模拟只听 `isbrokendirty`）→ 主模拟：生成 classified → FX＋`SetFxOwner(nil)` → `SetupComponents`（equippable→weapon→tool→hunger）→ inspectable→inventoryitem→planardamage→damagetypebonus→finiteuses→shadowlevel → `MakeHauntableLaunch` → `MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH, OnBroken, OnRepaired)`。[equippable 默认 HANDS](../../../raw/current/scripts/components/equippable.lua#L29)、[MakeForgeRepairable](../../../raw/current/scripts/standardcomponents.lua#L1851-L1878)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.SHADOW_BATTLEAXE`＝暗影槌（官方英文名 Shadow Maul，非 "Shadow Battleaxe"；nightmare_axe 是动画资源名）；配方描述＝"用这把大锤砍倒一切。"。[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L349244-L349247)、[RECIPE_DESC](../../../raw/current/scripts/languages/chinese_s.po#L358639-L358642)

### 数值

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `SHADOW_BATTLEAXE.DAMAGE`＝`wilson_attack`×2−30 | 38 生命值（源码推导；图鉴 weapondamage=38 交叉印证，[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1311)） |
| 耐久上限 | `SHADOW_BATTLEAXE.USES` | 200；攻击 1/次＋CHOP 1/次＋二级起饥饿 1/秒（不随卸装暂停） |
| 饥饿上限 | `SHADOW_BATTLEAXE.MAX_HUNGER` | 500；击杀回 `HUNGER_GAIN_ONKILL`＝50 |
| 套装增益 | `WEAPONS_VOIDCLOTH_SETBONUS_DAMAGE_MULT`＝1.1、`..._PLANAR_DAMAGE`＝5 | 伤害 41.8、位面 +5（[tuning.lua#L7818-L7819](../../../raw/current/scripts/tuning.lua#L7818-L7819)） |
| 对月相加成 | `WEAPONS_NIGHTMARE_VS_LUNAR_BONUS` | 目标带 `lunar_aligned` 时 ×1.1（[tuning.lua#L8455](../../../raw/current/scripts/tuning.lua#L8455)） |
| 装备理智 | `dapperness`＝−`DAPPERNESS_MED`（[tuning.lua#L2215](../../../raw/current/scripts/tuning.lua#L2215)） | ≈−0.056/秒（约 −26.7/游戏天）；`is_magic_dapperness`＝true（Walter 也受影响） |
| 影子等级 | `SHADOW_BATTLEAXE_SHADOW_LEVEL` | 3（[tuning.lua#L2883](../../../raw/current/scripts/tuning.lua#L2883)） |
| 语音冷却 | `TALK_INTERVAL`：OVERTIME 30／CHOPPING 10／CHOPPING_WOODIE 10／STARVING 5／CREATURE_KILLED 5 | 秒（GetTime 计时） |

分级表 `TUNING.SHADOW_BATTLEAXE.LEVEL[1..4]`（[tuning.lua#L8392-L8438](../../../raw/current/scripts/tuning.lua#L8392-L8438)）：

| 级 | PLANAR_DAMAGE | CHOPPING_EFFICIENCY | LIFE_STEAL（汲命） | HUNGER_RATE（饥饿/秒） |
|---|---|---|---|---|
| 1 | 10 | 1.50 | 0 | 0（hunger `Pause()`，不会饿） |
| 2 | 14 | 1.75 | `wilson_attack`×.05＝1.7 | 0.05（≈20.8 天耗空） |
| 3 | 18 | 2.00 | `wilson_attack`×.075＝2.55 | 0.1（≈10.4 天） |
| 4 | 22 | 2.25 | `wilson_attack`×.1＝3.4 | 0.2（≈5.2 天） |

### 等级与升级（epic 击杀）

- 计数与升级：[OnAttack](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L309-L337) 结算；目标带 `epic`、非 `smallepic`、非 alterguardian_phase1/2 才算（[IsEpicCreature](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L265-L273)）。直接击杀或命中后 5 秒内死亡（`RECENT_TARGET_TIME`，[TrackTarget](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L339-L357) 监听 death/onremove）都计数，按 tracked 去重；卸装时 `ForgetAllTargets` 解除全部 death/onremove 监听，卸装后的击杀不再计数；计数达 `LEVEL_THRESHOLDS[level+1]` 即升级（[TryLevelingUp](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L243-L251)）。
- 阈值 `{0, 3, 6, 9}`：累计 3/6/9 杀升到 2/3/4 级（与图鉴文案"每级 3 只巨兽"一致）。
- [SetLevel](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L195-L241)：换 `idle_levelN` 动画与 `shadow_battleaxe_lN` 图标、fx 同步、重设 `_lifesteal`/位面伤害/砍伐效率/饥饿率（level>1 时 hunger `Resume()`）、切说话开关与声音档；非读档时把计数 clamp 到当前级阈值；破损时先卸四组件再 `SetLevel(1)`，等级>1 会把计数钳到 0，原本 1 级则同级早退、保留 1–2 杀。`OnSave/OnLoad` 只存 `epic_kill_count`，等级按阈值重推（[L499-L519](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L499-L519)）。

### 命中结算与汲命

服务端每次命中：红色黑焰 hitsparks_fx（`colouroverride={1,0,0}`＋`black`，[DoAttackEffects](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L255-L263)）→ epic 追踪/计数（见上）→ 击杀时 hunger +50 → [DoLifeSteal](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L295-L307)：`_lifesteal > 0` 且 owner 未满血且 `IsLifeDrainable(target)` 时回血 `_lifesteal × mult:Get()`（无 `aoediminishingreturns` 组件按 1；WX-78 旋身等 AOE 效率递减写入该倍率），并按 `LIFE_STEAL_SANITY_LOSS_SCALE`＝0.5 扣理智（理智用未乘 mult 的 `_lifesteal`）（2 级 −0.85/击，4 级 −1.7/击）。`IsLifeDrainable` 与 `NON_LIFEFORM_TARGET_TAGS` 判定同蝙蝠棒：[[reference/prefabs/batbat#命中吸血与理智消耗|同函数核验]]、[定义](../../../raw/current/scripts/componentutil.lua#L185-L188)。

### 装备副作用与虚空风帽套装

[OnEquip/OnUnequip](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L46-L80)：ARM 显隐（不设 `swap_object` 贴图覆盖，持握视觉全由 FX 渲染）、`SetBuffOwner`/`SetFxOwner`、说话开关（level>1）、按级待机循环音、监听 owner `working`（[OnOwnerWorking](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L384-L396)：woodcutter 标签（伍迪）砍树额外 −`SANITY_MED`＝15 理智并换台词）、卸下时 `ForgetAllTargets`。[SetBuffOwner](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L107-L151) 监听持有者 HEAD 槽 equip/unequip：装备虚空风帽（`voidclothhat`）→ [SetBuffEnabled](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L84-L105) 伤害 ×1.1＋位面 +5（`"setbonus"` 键）；换戴其他头饰或摘下即取消。

### 说话系统与网络代理（shadow_battleaxe_classified）

- 文案源 `STRINGS.SHADOW_BATTLEAXE_TALK`（[strings.lua#L16251](../../../raw/current/scripts/strings.lua#L16251)）：overtime/hungry/starving/chopping(_woodie)/creature_killed/epic_killed/level_up 各表均从 l2 起，**1 级完全沉默**；饥饿 ≤50% 时闲聊换 hungry 表（[GetOvertimeChatLine](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L586-L597)）。
- 服务端 [SayRegularChatLine](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L609-L637) 按表冷却选行，经 `classified:Say(list, id, sound)` 写三个 netvar（1 秒超时清空，[classified](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L71-L83)）；升级台词首选"第 N 级第 N 杀"条目、满级后随机（[SayEpicKilledLine](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L639-L655)）。
- 代理角色：主体 fn 生成并 `SetParent` 挂本体、`_parent` 回写（[L778-L781](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L778-L781)）；拾入背包 `SetTarget(owner)`、掉地 `SetTarget(nil)`（[L685-L695](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L685-L695)），内部 `Network:SetClassifiedTarget` 限定复制目标（同模式见 [attunable_classified](../../../raw/current/scripts/prefabs/attunable_classified.lua#L18)）。仅持有者客户端（掉地后全员）收到 `saydirty` → [OnSayDirty](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L40-L52) 用本地 talker widget 显示（`nobroadcast=true` 不再广播）；`ontalk` 驱动 `localsounds` 播分级说话声（[GetTalkSound](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L54-L56)）。清理：主体 `OnRemoveEntity` 双端解挂/移除（[L31-L42](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L31-L42)）。

### 残破与修复

耐久归零经 MakeForgeRepairable 接线：先卸下再 [OnBroken](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L475-L484)——移除 equippable/weapon/tool/hunger、强制 1 级、播 `broken` 动画、加 `broken` 标签、检视名换 `BROKEN_FORGEDITEM`。用虚空修补套件（`voidcloth_kit`）（[forgerepair 材质同为 voidcloth](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L57)）修复：[ForgeRepairable:Repair](../../../raw/current/scripts/components/forgerepairable.lua#L58-L73) 校验材质后 [ForgeRepair:OnRepair](../../../raw/current/scripts/components/forgerepair.lua#L28-L56) 把耐久 `SetPercent(1)` 回满并消耗 kit 1 个 → [OnRepaired](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L486-L495) 重组件、复显 FX、摘标签。耐久不满即可修（`SetRepairable(current < total)`）；修复只重建组件与外观，不回等级或击杀计数。

### 浮水与持握视觉（shadow_battleaxe_fx）

- 主体直接 `AddComponent("floater")`（无 MakeInventoryFloatable），`SetIsBroken(false)` 设 `SetBankSwapOnFloat(true, -20, SWAP_DATA)`；残破/正常两档尺寸偏移（[OnIsBrokenDirty](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L432-L448)）。浮水时 floater 用 `swap_spear` 符号覆盖贴图（[floater.lua#L123-L143](../../../raw/current/scripts/components/floater.lua#L123-L143)），未装备的网络 FX 跟随该符号对齐浮水画面；停止浮水复位帧并重播 idle（[OnStopFloating](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L188-L191)）。
- `shadow_battleaxe_fx`（同文件）：网络实体渲染武器帧 3（注释：浮水必需，其余帧客户端本地补），`_level`/`equiptoggle` netvar 同步（[FollowSymbolFxFn](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L988-L1031)）。`SetFxOwner`：装备时挂 owner、跟随其 `swap_object`；卸下时挂本体、跟随 `swap_spear`（[L155-L178](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L155-L178)）。客户端 `equiptoggledirty` 后按 [FX_DEFS](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L869-L877) 生成本地逐帧跟随实体（f1 帧 0–2/f4 帧 3 强制 1 级/f6–f8 帧 5–7，[FxOnEquipToggle](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L917-L947)），卸装/移除时 `FxRemoveAll` 清理；专用服务器不生成本地帧 FX。

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 配方（唯一，全 scripts grep 无其他生成点） | 附身暗影心房（shadowheart_infused）×1＋纯粹恐惧（horrorfuel）×5＋绝望石（dreadstone）×3＋暗影碎布（voidcloth）×2；`TECH.SHADOWFORGING_TWO`（＝{SHADOWFORGING=2}，[constants.lua#L1335](../../../raw/current/scripts/constants.lua#L1335)），`station_tag="shadow_forge"`＋`nounlock=true`（每次都需基座）。[配方](../../../raw/current/scripts/recipes.lua#L875) |
| 制作站 | 暗影术基座：prototyper 树 `PROTOTYPER_TREES.SHADOW_FORGE`（[tuning.lua#L1452-L1454](../../../raw/current/scripts/tuning.lua#L1452-L1454)）、`shadow_forge` 标签（[shadow_forge.lua#L125-L152](../../../raw/current/scripts/prefabs/shadow_forge.lua#L125-L152)）；配方归 CRAFTING_STATION 过滤（[recipes_filter.lua#L398](../../../raw/current/scripts/recipes_filter.lua#L398)、[L469](../../../raw/current/scripts/recipes_filter.lua#L469)） |
| 原料名锚点 | [附身暗影心房](../../../raw/current/scripts/languages/chinese_s.po#L349150-L349152)、[纯粹恐惧](../../../raw/current/scripts/languages/chinese_s.po#L343965-L343967)、[绝望石](../../../raw/current/scripts/languages/chinese_s.po#L342260-L342262)、[暗影碎布](../../../raw/current/scripts/languages/chinese_s.po#L351340-L351342)、[暗影术基座](../../../raw/current/scripts/languages/chinese_s.po#L349260-L349262) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(38)`＋`SetOnAttack(OnAttack)`；无射程/投射 | 主模拟；`GetDamage` 乘 damagetypebonus；`OnAttack` 先回调后按默认 1 磨损 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L407-L409)、[GetDamage](../../../raw/current/scripts/components/weapon.lua#L94-L103) |
| 组件：planardamage | 按级 `SetBaseDamage`（10–22）；套装 `AddBonus(+5,"setbonus")` | 主模拟；经 SpDamageUtil 汇入 weapon 结算 | [[reference/components/planardamage#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L835-L836)、[汇入](../../../raw/current/scripts/components/spdamageutil.lua#L24) |
| 组件：damagetypebonus | `AddBonus("lunar_aligned", 1.1)`：lunarplanttentacle、alterguardian_phase3、gestalt_guard 等月相实体受伤 ×1.1 | 命中结算读取 | [[reference/components/damagetypebonus#关键接口]]；[AddBonus](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L838-L839)、[GetBonus](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L57) |
| 组件：finiteuses | 上限＝当前 200；`SetConsumption(CHOP, 1)`；耗尽进残破态（不 Remove） | 主模拟；攻击磨损走 weapon 默认 1/次 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L841-L844)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：tool | `SetAction(ACTIONS.CHOP, 1.5–2.25)`＋`EnableToughWork(true)`；效率经 worker 侧结算（本页未展开） | 主模拟；CHOP 动作消耗 1 耐久 | [[reference/components/tool#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L411-L413)、[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L44) |
| 组件：equippable | OnEquip/OnUnequip 接线＋`dapperness`（magic 型）；槽默认 HANDS | 主模拟；dapperness 由 sanity 汇总 | [[reference/components/equippable#关键接口]]；[汇总](../../../raw/current/scripts/components/sanity.lua#L512-L518)、[Walter magic 限定](../../../raw/current/scripts/prefabs/walter.lua#L754-L760) |
| 组件：hunger | 上限 500、按级 rate、`OverrideStarveFn`（饥饿期每秒 `finiteuses:Use(dt)`）＋击杀 `DoDelta(+50)` | 主模拟；1 级 `Pause()` 不掉饥饿 | [[reference/components/hunger#关键接口]]；[starving 调用](../../../raw/current/scripts/components/hunger.lua#L157-L158)、[OnStarving](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L376-L382) |
| 组件：shadowlevel | `SetDefaultLevel(3)`；消费端（Maxwell 影像随从）同蝙蝠棒页 | 主模拟；pristine 同名标签 | [[reference/components/shadowlevel#关键接口]]、[[reference/prefabs/batbat#关联机制导航]]；[挂载](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L846-L847) |
| 组件：floater/hauntable/talker/inspectable/inventoryitem | 浮水两档配置；`MakeHauntableLaunch`；talker 定制字体颜色偏移；残破换名；掉落/拾取回调切 classified 目标 | floater 公共端，其余主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]、[[reference/components/talker#关键接口]]；[调用](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L729-L738)、[MakeHauntableLaunch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 组件：forgerepairable | `MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH)`＋OnBroken/OnRepaired | 主模拟；修复物须 forgerepair 材质 voidcloth（voidcloth_kit） | [[reference/components/forgerepairable#关键接口]]；[调用](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L862)、[FORGEMATERIALS](../../../raw/current/scripts/constants.lua#L1962-L1966) |
| 同步：netvar | `isbroken`（isbrokendirty→客户端尺寸/贴图）；fx `_level`+`equiptoggle`；classified 说话三 netvar | 公共端声明、双端消费 | [isbroken](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L730)、[fx netvar](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1012-L1015)、[classified](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L138-L140) |
| SG／Brain | 无（不挂 SetStateGraph/SetBrain） | — | 全文核验无 |
| API／数据：标签 | `sharp`→combat 命中音；`show_broken_ui`→物品栏破损 UI；`shadow_item`→凝碎卫 L3 敌对；`broken`→残破态 | 运行时读取 | [combat 命中音](../../../raw/current/scripts/components/combat.lua#L745)、[itemtile](../../../raw/current/scripts/widgets/itemtile.lua#L601)、[gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L62-L65) |
| 关联 Prefab：hitsparks_fx | 命中火花（红色 override＋black 档） | 每次命中生成 | [[reference/prefabs/hitsparks_fx]]；[生成](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L257-L262) |
| 关联 Prefab：虚空风帽（voidclothhat）／虚空修补套件（voidcloth_kit） | 套装判定对象（HEAD 槽事件）；修复材料（消耗 1 个） | 装备事件/修复流程 | [[reference/prefabs/hats]]；[判定](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L127-L135)、[MakeKit](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L57) |
| 资源：anim/图标/音效 | `nightmare_axe.zip`（bank＋build）；`INV_IMAGE shadow_battleaxe_l1..l4` 四级图标；`rifts4/nightmare_axe/*` 分级 idle/talk/levelup/chop 音 | 静态声明 | [assets](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1-L14) |

## 典型接入

### 调整暗影槌的成长与数值

- 前提与选择：服务端 `modmain.lua`。伤害/耐久虽在实例上写入，但套装切换（`SetBuffEnabled`）与修复重建（`SetupComponents`）都会按 `TUNING.SHADOW_BATTLEAXE.DAMAGE` 重算；只 postinit 改组件会被下一个入口覆盖。要让构造、套装、修复三处一致，改全局基础值。
- 生效与撤销：`TUNING` 改动对之后构造的实例、下一次套装切换/修复生效；已有实例的当前伤害要到这些入口才重算。停用模组后需自行还原（原值见[数值](#数值)）。
- 原版依据：`TryLevelingUp`/`CheckForEpicCreatureKilled` 每次击杀读 `LEVEL_THRESHOLDS`；`_lifesteal`/位面伤害/砍伐效率/饥饿率只在 `SetLevel`（升级、修复、读档）时从 `LEVEL` 表刷新（[SetLevel](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L195-L241)）。

```lua
-- modmain.lua：改全局表，构造/套装/修复读同一组值
local B = GLOBAL.TUNING.SHADOW_BATTLEAXE
B.DAMAGE = 45
B.USES = 250
B.LEVEL_THRESHOLDS = { 0, 2, 4, 6 }
```

- 调整与验证：45 是 weapon 基础伤害，结算还会乘对 lunar_aligned ×1.1 与套装 ×1.1；改表后套装切换与修复重建也读 45，不再被 38 覆盖。位面伤害独立（要改每级位面需同步 `planardamage:SetBaseDamage`，已有实例等下一次 `SetLevel` 才按 `LEVEL` 表刷新）。耐久三路消耗不变；`USES` 改 250 同时改初始当前值。

## 常见陷阱与限制

- **误区：叫"暗影斧/Shadow Battleaxe"**：官方名是暗影槌（Shadow Maul）；`battleaxe` 只是注册名与文件名。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L349244-L349247)
- **边界：升级只认 epic 巨兽**：非 `smallepic` 且非 alterguardian_phase1/2；辅助击杀需最后一次命中后 5 秒内死亡，按 tracked 去重防重复计数；卸装 `ForgetAllTargets` 解除全部追踪，卸装后的击杀不计。[判定](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L265-L293)
- **边界：1 级无汲命、不说话、不会饿**：`_lifesteal` 为 0 直接跳过汲命；台词表从 l2 起；hunger `Pause()`。[OnAttack](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L332-L334)、[台词表](../../../raw/current/scripts/strings.lua#L16251)
- **边界：汲命三条件缺一不可**：owner 未满血＋目标 `IsLifeDrainable`＋`_lifesteal > 0`；对建筑/墙/气球等 `NON_LIFEFORM_TARGET_TAGS` 目标不回血（同 [[reference/prefabs/batbat#常见陷阱与限制|蝙蝠棒]]）。[DoLifeSteal](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L295-L307)
- **边界：耐久耗尽不消失而是残破，但成长会被压回 1 级**：与 batbat/spear 的 `inst.Remove` 不同——先卸四组件进残破态，再 `SetLevel(1)`；若破损前等级>1，计数被钳到 0（等级记录不再保留），修复只重建组件与外观、不回等级。只有破损时本来就是 1 级，`SetLevel(1)` 同级早退，1–2 个击杀计数才留下。[OnBroken](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L475-L484)
- **边界：套装增益只认虚空风帽（`voidclothhat`）＋HEAD 槽**：监听 equip/unequip 事件即时切换，同 slot 换戴其他头饰即关闭。[SetBuffOwner](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L107-L151)
- **边界：说话显示范围受限**：仅持有者客户端（掉地后全员）本地显示，不经 `TheNet:Talker` 广播；专用服务器无 `localsounds`，无说话声。[classified](../../../raw/current/scripts/prefabs/shadow_battleaxe_classified.lua#L40-L52)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/shadow_battleaxe.lua 全文（主体＋fx 双注册、成长/套装/说话/残破修复/浮水）与 shadow_battleaxe_classified.lua 全文；weapon/equippable/finiteuses/planardamage/damagetypebonus/hunger/talker/sanity/shadowlevel/tool/forgerepairable/forgerepair/aoediminishingreturns/spdamageutil 关键方法；componentutil `IsLifeDrainable`（沿用 batbat 页同函数核验）；tuning SHADOW_BATTLEAXE 全键＋套装/对月/影子等级/dapperness/wilson_attack；配方、TECH/PROTOTYPER_TREES、shadow_forge 站标签、forge_repair_kits、strings.lua 台词键域、scrapbookdata 数值交叉印证、`shadow_item`/`show_broken_ui`/`sharp` 消费端；皮肤：skinprefabs.lua 无本 prefab 条目且文件无皮肤接线；全 scripts grep "shadow_battleaxe" 仅 recipes/recipes_filter/prefablist/scrapbook 数据/skins_defs_data，无其他生成点。
- 游戏运行：未运行。未覆盖：38 伤害、汲命 1.7–3.4、饥毁灭档换算（20.8/10.4/5.2 天）均为源码推导；四级实机成长观感、FX 帧视觉、台词实机显示与音效未实测。
- 结构检查：validate_page.py（0 error，159 行／26234 字节，complex 档）与 validate_source_links.py --strict（103 条源码引用、29 个源文件，0 error 0 warning）实际执行通过；lua 围栏为完整可解析片段。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
