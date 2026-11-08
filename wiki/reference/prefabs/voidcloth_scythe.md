---
title: "暗影收割者（voidcloth_scythe）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版裂隙暗影武器 voidcloth_scythe：HANDS 槽近战＋右键 SCYTHE 群体采收植物，位面伤害、耐尽残破可修复、对影阵营持有者说话，Wortox 影契下攻击可耗灵魂触发范围打击；同文件注册持握跟随 FX，独立文件 classified 说话代理。"
aliases: ["voidcloth_scythe", "暗影收割者", "Shadow Reaper", "虚布镰刀"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/voidcloth_scythe.lua", "prefabs/voidcloth_scythe_classified.lua", "prefabs/wortox.lua", "prefabs/skilltree_wortox.lua", "prefabs/gestalt_guard.lua", "prefabs/hats.lua", "prefabs/skinprefabs.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/planardamage.lua", "components/damagetypebonus.lua", "components/talker.lua", "components/combat.lua", "components/tool.lua", "components/socket_shadow_harvester.lua", "componentactions.lua", "componentutil.lua", "bufferedaction.lua", "actions.lua", "constants.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "recipes_filter.lua", "mods.lua", "strings.lua", "stringutil.lua", "widgets/itemtile.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影收割者（voidcloth_scythe）

## 快速判断

- 职责与适用：`prefabs/voidcloth_scythe.lua` 注册的手持工具＋武器（物品＋装备）：weapon＋planardamage 位面＋tool（右键 SCYTHE"收割"群体采收植物）＋finiteuses 耐久＋talker 说话。裂隙暗影阵营武器，暗影术基座制作；同文件注册 `voidcloth_scythe_fx`，独立文件 classified；无皮肤注册、无 quagmire 分支。[工厂](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L371-L479)、[注册](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L599-L601)
- 优先使用：查数值→[数值](#数值)；改收割→[收割动作](#收割动作scythe)；改 AoE→[攻击与影契 AoE](#攻击与影契-aoe)；改伤害／耐久读[典型接入](#典型接入)；从零造手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：装备期 −`DAPPERNESS_MED` 理智/秒（magic 型）；耐久耗尽残破不消失（[OnBroken](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L350-L358)）；对 lunar_aligned ×1.1；`shadow_item` 让凝碎卫敌对（[gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L62-L65)）；Wortox 影契下攻击经灵魂跳窗口累计成本，成功后触发 AoE；灵魂在 FinishPortalHop 结算。
- 修改边界：本页覆盖主体＋同文件 FX＋classified 三注册，残破双态、说话复制与 fx 帧跟随的跨注册协作需整体保留（complex 档）；与暗影槌的差异见[同族差异](#与暗影槌的同族差异)，共通机制互链 [[reference/prefabs/shadow-battleaxe]] 不复制；talker／floater 内部机制归组件页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `voidcloth_scythe` | 无包装工厂：`ScytheFn()` 直接构造，无 postinit | 物品＋装备；残破态移除 equippable/weapon/tool，修复后重组件。[注册](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L599-L601) |
| `voidcloth_scythe_fx` | 同文件 `FollowSymbolFxFn`：网络跟随 FX 实体 | 持握/浮水视觉；`equiptoggle` netvar、`persists = false`。[工厂](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L567-L597) |
| `voidcloth_scythe_classified` | 独立文件 `fn`：隐藏网络代理实体 | 只承载说话文本/音效选择 netvar。[工厂](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L99-L142) |

共同初始化（[ScytheFn](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L371-L479)）：实体组件＋`MakeInventoryPhysics` → bank/build `scythe_voidcloth`＋`idle` → 公共端标签 `sharp`/`show_broken_ui`/`weapon`/`shadowlevel`/`shadow_item` → floater＋`isbroken` net_bool → talker（**公共端**挂载，字体 28／TALKINGFONT／暗红／符号 swap_object）→ 非专用端 `localsounds` 声音子实体 → `SetPristine`（客户端只听 `isbrokendirty`）→ 主模拟：生成 classified（父子互挂＋`SetTarget(nil)`）→ 随机帧同步＋生成 fx＋`SetFxOwner(nil)` → inspectable→inventoryitem→finiteuses→planardamage→damagetypebonus→`SetupComponents`（equippable→weapon→tool）→shadowlevel → `MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH, OnBroken, OnRepaired)` → `MakeHauntableLaunch`（以上 [L388-L469](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L388-L469)）。[equippable 默认 HANDS](../../../raw/current/scripts/components/equippable.lua#L29)、[MakeForgeRepairable](../../../raw/current/scripts/standardcomponents.lua#L1851-L1878)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.VOIDCLOTH_SCYTHE`＝暗影收割者（官方英文名 Shadow Reaper，注册名不等于官方名；`scythe_voidcloth` 只是动画资源名）；配方描述＝"从容地剔除植物群落。"；检视＝"大为简化了收割流程，简直可怕！"。[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351359-L351362)、[RECIPE_DESC](../../../raw/current/scripts/languages/chinese_s.po#L359529-L359532)、[DESCRIBE](../../../raw/current/scripts/languages/chinese_s.po#L21854-L21857)

### 与暗影槌的同族差异

（共通模式见 [[reference/prefabs/shadow-battleaxe]]，下表只列已核验差异）

| 维度 | shadow_battleaxe（暗影槌） | voidcloth_scythe（本页） |
|---|---|---|
| 成长/饥饿/汲命 | 4 级 epic 击杀成长、二级起持续饥饿（卸装不暂停）、2 级起汲命 | 均无（全文无 hunger/health 组件与成长逻辑） |
| 工具动作 | CHOP（效率 1.5–2.25） | SCYTHE 群体采收（见下） |
| 战斗 AoE | 无 | 影契＋灵魂跳窗口（至少 1 灵魂，成本可累计）→ 半径 2.5 |
| 台词表 | 饥饿/砍伐/击杀/升级等 7 表，1 级沉默 | overtime/onharvest/lucy 3 表，装备即说（影阵营限定） |
| 持握视觉 | 不覆盖 swap_object，视觉全由 FX 渲染 | 覆盖 `swap_object`（swap_scythe 符号）＋FX 帧跟随 |

### 数值

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)；键域 [L7809-L7819](../../../raw/current/scripts/tuning.lua#L7810-L7820)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `VOIDCLOTH_SCYTHE_DAMAGE`＝`wilson_attack`×2−30 | 38 生命值（源码推导；图鉴 weapondamage=38 交叉印证，[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1672)） |
| 位面伤害 | `VOIDCLOTH_SCYTHE_PLANAR_DAMAGE` | 18（图鉴同值） |
| 耐久上限 | `VOIDCLOTH_SCYTHE_USES` | 200；收割 1/次＋攻击 1/次（weapon 默认磨损） |
| 收割半径 | `VOIDCLOTH_SCYTHE_HARVEST_RADIUS` | 4（每次执行时读表） |
| 收割锥角 | `VOIDCLOTH_SCYTHE_HARVEST_ANGLE_WIDTH`＝165/RADIANS（[constants.lua#L15](../../../raw/current/scripts/constants.lua#L15)） | 165°，以持有者朝向为轴 |
| 对月相加成 | `WEAPONS_VOIDCLOTH_VS_LUNAR_BONUS` | 目标带 `lunar_aligned` 时 ×1.1 |
| 套装增益 | `WEAPONS_VOIDCLOTH_SETBONUS_DAMAGE_MULT`＝1.1、`..._SETBONUS_PLANAR_DAMAGE`＝5 | 伤害 41.8、位面 +5 |
| 装备理智 | `dapperness`＝−`DAPPERNESS_MED`（[tuning.lua#L2215](../../../raw/current/scripts/tuning.lua#L2215)） | ≈−0.056/秒（约 −26.7/游戏天）；`is_magic_dapperness`＝true |
| 影子等级 | `VOIDCLOTH_SCYTHE_SHADOW_LEVEL` | 3（[tuning.lua#L2881](../../../raw/current/scripts/tuning.lua#L2881)） |
| 说话间隔 | `TALK_INITIAL_INTERVAL`／`TALK_INTERVAL` | 3 秒首句／30 秒周期 |
| AoE 范围 | `SKILLS.WORTOX.VOIDCLOTHSCYTHE_AOE_RANGE` | 2.5（[tuning.lua#L7465](../../../raw/current/scripts/tuning.lua#L7466)） |

### 收割动作（SCYTHE）

- 动作可用：tool 挂 `SetAction(ACTIONS.SCYTHE)`（[SetupComponents](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L295-L308)）；右键动作（[ACTIONS.SCYTHE](../../../raw/current/scripts/actions.lua#L663)：rmb、distance=1.8、禁长按）只对带 `pickable` 标签且属于 `HARVESTABLE_PLANT_TARGET_TAGS`＝{plant, lichen, oceanvine, kelp} 的实体出现（[IsValidScytheTarget](../../../raw/current/scripts/componentactions.lua#L128-L130)、[标签定义](../../../raw/current/scripts/componentutil.lua#L167-L172)）。
- 执行链：[ACTIONS.SCYTHE.fn](../../../raw/current/scripts/actions.lua#L4545-L4554) 先调 `inst:DoScythe(target, doer)` 并返回 true（是否实际采到作物由 DoScythe 内部判定）→ [bufferedaction](../../../raw/current/scripts/bufferedaction.lua#L22-L37) 在动作成功后调 `OnUsedAsItem` → [finiteuses](../../../raw/current/scripts/components/finiteuses.lua#L101-L114) 按 `SetConsumption(SCYTHE, 1)` 扣 1 耐久。
- [DoScythe](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L224-L248)：先说 onharvest 台词（见[说话系统](#说话系统与网络代理voidclothscytheclassified)）→ 目标带 pickable 组件才进入收割 → 以**持有者位置**为中心 `FindEntities`（半径 4，must `pickable`，cant INLIMBO/FX/intense，mustoneof 四植物标签）→ [IsEntityInFront](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L215-L219) 过滤朝向 165° 锥角内 → 逐个 [HarvestPickable](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L200-L213)：播 picksound＋`pickable:Pick(TheWorld)`＋产物 `Launch` 弹向持有者（力度 1.5）→ 采收 ≥1 株时推 `picksomethingfromaoe`（消费者：[socket_shadow_harvester](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L63) WX-78 模块计次）。收割不走 workable/效率结算，固定 1 耐久/次。

### 攻击与影契 AoE

- [OnAttack](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L286-L293)：每次命中先 [TryToSparkOn](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L250-L257) 红色 hitsparks_fx（`colouroverride={1,0,0}`＋`black`，效果见 [[reference/prefabs/hitsparks_fx]]）；持有者激活 `wortox_allegiance_shadow`（[技能节点](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L832-L848)）且 `finishportalhoptask ~= nil`（Wortox 灵魂跳免费窗口计时，[TryToPortalHop](../../../raw/current/scripts/prefabs/wortox.lua#L614-L651) 设置）且 `TryToPortalHop(1, false)` 成功（至少 1 灵魂；成功先把 1 计入 `_soulhop_cost`，未达每魂跳跃数时本次免费并进入冷却窗口，否则立即 `FinishPortalHop` 按累计成本结算）→ [DoShadowAoE](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L274-L284)：`combat:DoAreaAttack(target, 2.5, weapon, validfn, nil, excludetags)`——半径内除目标本身外全部有效目标（[ShadowAoEValidFn](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L265-L272)：`IsAlly` 排除，其余生成火花并放行）；`hitcount == 0` 补一次 `onlyontarget=true` 只打目标本身。排除表：PVP 开启用 `NO_TAGS_PVP`，否则额外排除 player/wall。
- 伤害：[DoAreaAttack](../../../raw/current/scripts/components/combat.lua#L1312-L1340) 按攻击者 CalcDamage 结算，玩家未设 `areahitdamagepercent` → [全额伤害](../../../raw/current/scripts/components/combat.lua#L938-L946)（38＋位面 18，分别乘武器与攻击者对月倍率），不是按比例削减。

### 说话系统与网络代理（voidcloth_scythe_classified）

- 文案 `STRINGS.VOIDCLOTH_SCYTHE_TALK`（[strings.lua#L16477](../../../raw/current/scripts/strings.lua#L16477)）：overtime 15 句／onharvest 11 句／lucy 4 句，`{mortal}` 占位经 [GetMortalityStringFor](../../../raw/current/scripts/stringutil.lua#L625) 替换。**仅对 `player_shadow_aligned` 持有者说话**（影契等技能节点加同标签）；其他角色装备完全沉默。
- 触发：[OnEquip](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L171-L185) → [ToggleTalking](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L159-L169) → `classified:SetTarget(owner)` → 3 秒后首句（overtime 表），之后每 30 秒 [SayRandomLine](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L143-L157)：说传入表随机句后，Woodie（`woodcutter` 标签）30% 概率换 lucy 表再排下一轮；收割时 DoScythe 先说 onharvest 并重置周期。卸下即取消任务。
- 复制：`SayRandomLine` → [classified:Say](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L53-L62) 写 string_list/string_id/sound_override 三 netvar（音效未指定恒随机，[GetTalkSound](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L36-L38)）＋1 秒超时清空；仅 `istarget`（装备期＝持有者本人客户端）监听 `saydirty` → [OnSayDirty](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L23-L34) → `talker:Say(..., nobroadcast=true)` 本地显示不广播（[分支](../../../raw/current/scripts/components/talker.lua#L193-L200)）；`ontalk` 驱动 [localsounds](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L40-L50) 播 `rifts2/thrall_generic` 喉音。代理挂接/清理（[AttachClassified](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L14-L36)、[OnEntityReplicated](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L81-L88)）同暗影槌页模式，差异在触发点：本镰刀按装备/卸装切换，暗影槌按拾入/掉地。

### 残破与修复

- 耐久归零 → [SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) → MakeForgeRepairable 的 `SetOnFinished` 接线：先自动卸下 → [OnBroken](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L350-L358)：[DisableComponents](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L310-L314)（移除 equippable/weapon/tool，finiteuses 保留）＋`broken` 动画＋[SetIsBroken(true)](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L334-L348)（fx Hide、floater 换 `scythe_base_broken_float`）＋`broken` 标签＋检视名 `BROKEN_FORGEDITEM`（[po](../../../raw/current/scripts/languages/chinese_s.po#L9959-L9962)）。
- 修复：`voidcloth_kit`（材质 `voidcloth`，[FORGEMATERIALS](../../../raw/current/scripts/constants.lua#L1962-L1966)）→ 耐久回满并消耗 1 个 → [OnRepaired](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L360-L369)：[SetupComponents](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L295-L308) 重组件＋fx 帧复位＋idle＋摘标签。耐久不满即可修（[finiteuses watcher](../../../raw/current/scripts/components/finiteuses.lua#L1-L8) → `SetRepairable(current < total)`）。
- 读档：[finiteuses OnLoad](../../../raw/current/scripts/components/finiteuses.lua#L54-L58) `SetUses(0)` 触发同一 onfinished 链，残破态跨存档恢复（构造时 current>0，`was_positive` 成立）。

### 浮水与持握视觉（voidcloth_scythe_fx）

- floater 公共端直接挂载（[L399-L401](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L399-L401)）：正常 `SetBankSwapOnFloat(true, -20, SWAP_DATA)`／残破 `-10`＋`SWAP_DATA_BROKEN`；[OnIsBrokenDirty](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L319-L329) 两档：残破 med／offset 0.25／scale {0.8,0.4,0.8}，正常 med／0／{1.2,0.4,1.2}；客户端经 `isbrokendirty` 同步。停止浮水 [OnStopFloating](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L138-L141) 复位 fx 帧并延迟重播 idle/broken。
- 网络 fx：主模拟生成 `voidcloth_scythe_fx` 播 `swap_loop_3`（源码注释：帧 3 浮水必需，其余帧客户端本地补）；[SetFxOwner](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L108-L128)：装备挂 owner 跟 `swap_object`、卸下挂本体跟 `swap_spear`，highlightchild/colouradder 联动；`ToggleEquipped` 写 `equiptoggle` netbool → 客户端 `equiptoggledirty` → 按 [FX_DEFS](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L483-L490) 生成 4 个本地帧跟随实体（swap_loop_1 帧 0–2/_6 帧 5/_7 帧 6/_8 帧 7，[FxOnEquipToggle](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L525-L555)），卸下/移除 `FxRemoveAll` 清理；专用服务器不生成本地帧 FX。
- [OnEquip/OnUnequip](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L171-L198)：ARM 显隐＋`swap_object` 覆盖（build `scythe_voidcloth` 的 `swap_scythe`；有皮肤走 `OverrideItemSkinSymbol`——皮肤接线存在但 [skinprefabs](../../../raw/current/scripts/prefabs/skinprefabs.lua) 无本 prefab 注册，即无实际皮肤）＋SetFxOwner/SetBuffOwner/ToggleTalking。

### 虚空风帽套装与获取途径

- [SetBuffOwner](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L72-L106) 监听持有者 HEAD 槽 equip/unequip：装备 `voidclothhat` → [SetBuffEnabled](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L54-L70) 伤害 ×1.1＋位面 +5（`"setbonus"` 键）；换戴其他头饰或摘下即取消；装备瞬间已戴帽立即启用（[L100-L103](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L100-L103)）；重组件时按 `_bonusenabled` 恢复伤害（[L303](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L303)）。
- 帽侧独立机制：voidclothhat 监听 HANDS 槽，给带 planardamage＋`shadow_item` 的手持物（含本镰刀）按攻击叠位面增益（6 层最多 +24，3 秒无攻击衰减，[增益](../../../raw/current/scripts/prefabs/hats.lua#L3928-L3987)、[过滤](../../../raw/current/scripts/prefabs/hats.lua#L4008)）；细节归帽页。

| 途径 | 内容与证据 |
|---|---|
| 配方（唯一，全 scripts grep 无其他生成点） | 纯粹恐惧（horrorfuel）×3＋暗影碎布（voidcloth）×1；`TECH.SHADOWFORGING_TWO`（＝{SHADOWFORGING=2}，[constants.lua#L1334-L1335](../../../raw/current/scripts/constants.lua#L1334-L1335)），`station_tag="shadow_forge"`＋`nounlock=true`（每次都需基座）。[配方](../../../raw/current/scripts/recipes.lua#L869) |
| 制作站 | 暗影术基座：`PROTOTYPER_TREES.SHADOW_FORGE`（[tuning.lua#L1452-L1454](../../../raw/current/scripts/tuning.lua#L1452-L1454)）；配方归 SHADOW_FORGE 过滤组（[recipes_filter.lua#L458-L471](../../../raw/current/scripts/recipes_filter.lua#L458-L471)） |
| 原料名锚点 | [纯粹恐惧](../../../raw/current/scripts/languages/chinese_s.po#L343964-L343967)、[暗影碎布](../../../raw/current/scripts/languages/chinese_s.po#L351340-L351342) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(38)`＋`SetOnAttack(OnAttack)`；无射程/投射 | 主模拟；`GetDamage` 乘 damagetypebonus；`OnAttack` 先回调后按默认 1 磨损 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L302-L304)、[磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：planardamage | `SetBaseDamage(18)`；套装 `AddBonus(+5,"setbonus")`；帽侧另叠 ramping bonus | 主模拟；经 weapon 结算汇入 | [[reference/components/planardamage#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L452-L458) |
| 组件：damagetypebonus | `AddBonus("lunar_aligned", inst, 1.1)`：月相实体受伤 ×1.1 | 命中结算读取 | [[reference/components/damagetypebonus#关键接口]]；[GetBonus](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L57) |
| 组件：finiteuses | 上限＝当前＝200；`SetConsumption(SCYTHE, 1)`；耗尽进残破态（不 Remove） | 主模拟；攻击磨损走 weapon 默认 1/次 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L452-L455) |
| 组件：tool／equippable | tool 挂 SCYTHE 动作；equippable 接线 OnEquip/OnUnequip＋`dapperness`（magic 型）、槽默认 HANDS | 主模拟 | [[reference/components/tool#关键接口]]、[[reference/components/equippable#关键接口]]；[SetupComponents](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L295-L308) |
| 组件：talker | 定制字体颜色符号；客户端本地 `Say` 显示台词 | **公共端挂载**（客户端也持有组件） | [[reference/components/talker#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L403-L408) |
| 组件：shadowlevel | `SetDefaultLevel(3)`；消费端（Maxwell 影像随从）同蝙蝠棒页 | 主模拟；pristine 同名标签 | [[reference/components/shadowlevel#关键接口]]、[[reference/prefabs/batbat#关联机制导航]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L465-L466) |
| 组件：floater/hauntable/inspectable/inventoryitem | 浮水两档配置；`MakeHauntableLaunch`；残破换检视名 | floater 公共端，其余主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[挂载](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L399-L401) |
| 组件：forgerepairable | `MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH)`＋OnBroken/OnRepaired | 主模拟；修复物须 forgerepair 材质 voidcloth（voidcloth_kit） | [[reference/components/forgerepairable#关键接口]]；[调用](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L468) |
| 同步：netvar | `isbroken`（isbrokendirty→双端尺寸/贴图）；fx `equiptoggle`；classified 说话三 netvar | 公共端声明、双端消费 | [isbroken](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L400)、[fx](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L584)、[classified](../../../raw/current/scripts/prefabs/voidcloth_scythe_classified.lua#L110-L112) |
| SG／Brain | 无（不挂 SetStateGraph/SetBrain） | — | 全文核验无 |
| API／数据：标签 | `sharp`→钝/利命中音；`show_broken_ui`→物品格破损 UI；`shadow_item`→凝碎卫 L3 敌对＋帽侧过滤；`broken`→残破态 | 运行时读取 | [combat](../../../raw/current/scripts/components/combat.lua#L745)、[itemtile](../../../raw/current/scripts/widgets/itemtile.lua#L601)、[gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L62-L65)、[hats](../../../raw/current/scripts/prefabs/hats.lua#L4008) |
| API／数据：事件与条件 | `player_shadow_aligned`（说话条件，技能树授予）；`picksomethingfromaoe`（收割 ≥1 株由 doer 推送，socket_shadow_harvester 计次） | 装备期/收割时 | [判定](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L143-L145)、[推送](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L244-L246)、[消费](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L63) |
| 关联 Prefab：hitsparks_fx／voidclothhat／voidcloth_kit | 命中火花（红 override＋black）；套装判定（HEAD 槽事件）；修复材料（耗 1 个） | 命中/装备/修复时 | [[reference/prefabs/hitsparks_fx]]、[[reference/prefabs/hats]]；[火花](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L251-L257)、[套装判定](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L83-L103) |
| 资源：anim | `anim/scythe_voidcloth.zip`（bank＋build 同名）；swap 符号 `swap_scythe` | 静态声明 | [assets](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L1-L4) |

## 典型接入

### 调整伤害、耐久与收割范围

- 前提与选择：服务端 `modmain.lua`。伤害由 `TUNING` 在构造、套装切换和修复重建时重算，耐久上限/初始值只在构造时读 `USES`；要改整个预制物，应在生成前改 `TUNING.VOIDCLOTH_SCYTHE_DAMAGE`／`_USES`。收割半径/角度每次 `DoScythe` 时读 `TUNING`，可在 postinit 改。`TUNING` 是 modmain 白名单直挂对象，`TheWorld` 须经 `GLOBAL.` 前缀（[mods.lua env 构造](../../../raw/current/scripts/mods.lua#L258-L317)，写法同 [[reference/prefabs/ruins-bat]]）。
- 生效与撤销：改 `TUNING` 在模组加载时先于新实例构造；已生成实例的 weapon 伤害不会因事后改表而回算，除非触发套装切换或修复重建。停用模组后默认配置回原值（原值见[数值](#数值)）。
- 原版依据：`DoScythe`/`IsEntityInFront` 每次执行读 `TUNING.VOIDCLOTH_SCYTHE_HARVEST_RADIUS/ANGLE_WIDTH`；`SetupComponents` 与 `SetBuffEnabled` 在构造、套装切换、修复时按 `TUNING.VOIDCLOTH_SCYTHE_DAMAGE` 重设 weapon；`finiteuses` 构造时用同一 `USES` 键 `SetMaxUses`＋`SetUses`（[SetupComponents](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L295-L308)、[SetBuffEnabled](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L54-L70)、[finiteuses 初始化](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L452-L455)）。

```lua
-- modmain.lua：生成前改 tuning；套装切换与修复重建会按 tuning 重算
TUNING.VOIDCLOTH_SCYTHE_DAMAGE = 45
TUNING.VOIDCLOTH_SCYTHE_USES = 250

AddPrefabPostInit("voidcloth_scythe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    -- 收割半径/锥角每次 DoScythe 时读 TUNING，对所有实例(含已有)立即生效
    TUNING.VOIDCLOTH_SCYTHE_HARVEST_RADIUS = 6
end)
```

- 调整与验证：45 是 tuning 基础伤害，套装 ×1.1 与对 lunar_aligned ×1.1 在其上独立相乘；位面 18 独立结算（帽侧 ramping 增益再叠加）。`VOIDCLOTH_SCYTHE_USES` 同时控制上限与初始值；只改已有实例时，`SetMaxUses` 之后还要 `SetUses`。停用模组后默认配置回原值；已存实例的剩余耐久仅在存档写有 `uses` 时按存档恢复，满耐久通常不写 `uses`，会回落到本次构造/PostInit 的初始值。AoE 范围 `SKILLS.WORTOX.VOIDCLOTHSCYTHE_AOE_RANGE` 同为运行时读取。

## 常见陷阱与限制

- **误区：叫"虚空镰刀/虚布镰刀"**：官方名是暗影收割者（Shadow Reaper）；注册名与文件名才是 `voidcloth_scythe`。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L351359-L351362)
- **边界：收割目标面窄**：动作只对 `pickable`＋四植物标签（plant/lichen/oceanvine/kelp）实体出现，且实际采收要求目标带 pickable 组件、位于持有者朝向 165° 锥角内。[判定](../../../raw/current/scripts/componentactions.lua#L128-L130)
- **边界：台词条件双门槛**：需装备中（卸下即取消任务）且持有者带 `player_shadow_aligned`；文案仅持有者客户端本地显示，专用服务器无 `localsounds` 无说话声。[ToggleTalking](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L159-L169)
- **边界：耐久耗尽不消失而是残破**：与 batbat/spear 的 `inst.Remove` 不同——移除三组件进残破态，voidcloth_kit 修复回满；残破态跨存档恢复（同 [[reference/prefabs/shadow-battleaxe#常见陷阱与限制|暗影槌]]，差异见上文）。
- **边界：AoE 三条件缺一不可**：仅 `wortox_allegiance_shadow` 激活＋灵魂跳窗口（`finishportalhoptask` 未到期）＋至少 1 灵魂（成本在 FinishPortalHop 结算，可能累计）；无灵魂时普通攻击只有单目标与火花。[OnAttack](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L286-L293)
- **边界：套装增益只认 `voidclothhat` 注册名＋HEAD 槽**：监听 equip/unequip 即时切换，同 slot 换戴其他头饰即关闭。[SetBuffOwner](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L72-L106)
- **边界：卸下不重置 classified 目标**：`ToggleTalking(false)` 只取消任务不调 `SetTarget(nil)`；台词仅由说话任务触发故无可见泄漏，但自定义扩展读取 classified 目标时需自行注意。[ToggleTalking](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L159-L169)
- **边界：图鉴 specialinfo 键为源码拼写 `VOIDCLITHSCYTHE`**（非 VOIDCLOTH）：prefab 与 scrapbookdata 两侧一致，复用该键时勿"纠正"拼写。[赋值](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L425)
- **边界：影子等级无普适加成**：`shadowlevel` 只被 Maxwell 影像随从增伤与装备播报消费（同 [[reference/prefabs/batbat#常见陷阱与限制|蝙蝠棒]]核验记录）。

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/voidcloth_scythe.lua 全文（主体＋fx 双注册、收割/AoE/套装/说话/残破修复/浮水）与 voidcloth_scythe_classified.lua 全文；weapon/equippable/finiteuses/planardamage/damagetypebonus/talker/tool/combat（DoAreaAttack/CalcDamage/sharp 命中音）关键方法；componentactions `IsValidScytheTarget`、actions SCYTHE 定义与 fn、bufferedaction `OnUsedAsItem`、componentutil 植物标签表、TOOLACTIONS/FORGEMATERIALS/RADIANS/TECH 键、MakeForgeRepairable/MakeHauntableLaunch；tuning 全部 VOIDCLOTH_SCYTHE 键＋套装/对月/影子等级/dapperness/wilson_attack/SKILLS.WORTOX 范围；wortox `TryToPortalHop` 与 skilltree_wortox 影契节点；gestalt_guard `shadow_item` 消费、hats 帽侧 ramping 过滤、itemtile `show_broken_ui`、socket_shadow_harvester 事件消费；strings.lua 台词表、stringutil `GetMortalityStringFor`；名称与原料经简中 .po `msgctxt` 逐键核验；皮肤：skinprefabs.lua/prefabskin.lua 均无本 prefab 条目（皮肤接线本身存在）；全 scripts grep "voidcloth_scythe" 仅 recipes/recipes_filter/prefablist/scrapbook/skins_defs_data，无其他生成点；文件内无 quagmire 分支。未展开：ForgeRepair:OnRepair 内部流程（沿用 [[reference/prefabs/shadow-battleaxe#残破与修复|暗影槌页同链核验]]）、floater/talker 组件内部机制（归组件页）。
- 游戏运行：未运行。未覆盖：38 伤害/18 位面/200 耐久为源码推导（wilson_attack＝34）并经图鉴数据交叉印证，非运行测量；台词实机显示、fx 帧跟随视觉、AoE 实战手感未实测。
- 结构检查：validate_page.py 与 strict 均通过；本次修改的 Lua 示例经 parser_probe 语法通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
