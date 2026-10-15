# 装备 Wiki 第三轮检视

源码快照 `26335394`，知识仓基准 `97afb4e`。本轮复审[武器与战斗索引](../../../wiki/reference/prefabs/weapons-combat-index.md#近战长柄与海洋武器)的近战、长柄与海洋武器 13 页，核对主要注册名、战斗与耐久路径、装备状态、获取方式、示例及对应索引摘要。累计独立检视 35/69 页，另有 34 页待检视，其中本叶剩投掷、远程、防御和事件武器 6 页。

结论为需修复，共 5 项 P1、13 项 P2。本轮只记录静态复审结果。机械校验留待装备批次收口，未运行游戏。以下行号对应检视时工作树。

## P1

### R01 暗影槌破损清除击杀进度

`shadow-battleaxe.md:149` 写破损后保留 `epic_kill_count`，修复只恢复一级外观。高等级破损时，`OnBroken` 调 `SetLevel(1)`，非读档的 `SetLevel` 会把计数钳到一级阈值 0。修理重建组件，进度不会恢复。另有一个容易漏掉的边界，已在一级时 `SetLevel(1)` 直接返回，因此破损时若已有 1～2 杀，计数仍保留。同步残破章节和陷阱。 [Wiki](../../../wiki/reference/prefabs/shadow-battleaxe.md#L82-L84) · [原陷阱](../../../wiki/reference/prefabs/shadow-battleaxe.md#L149) · [破损与修复](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L475-L494) · [同级早退](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L195-L200) · [计数钳制](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L238-L245) · [阈值](../../../raw/current/scripts/tuning.lua#L8405-L8412)

### R02 暗影槌升到二级后卸下仍有饥饿消耗

`shadow-battleaxe.md:6,21` 与索引写成“装备期饥饿”，容易让玩家以为卸装能保耐久。一级初始化暂停，升到二级后恢复武器自身的饥饿周期任务。装备与卸装回调都不切换该任务，空腹回调磨耐久也不检查持有者。将饥饿限定为二级起的武器自身状态，装备期理智扣减另列。 [Wiki](../../../wiki/reference/prefabs/shadow-battleaxe.md#L18-L24) · [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L22) · [装备回调](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L46-L79) · [升等级恢复](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L217-L227) · [空腹磨损](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L376-L382) · [饥饿周期与消耗](../../../raw/current/scripts/components/hunger.lua#L75-L89) · [周期结算](../../../raw/current/scripts/components/hunger.lua#L146-L162)

### R03 三叉戟的伤害、掀飞和船漏由不同条件触发

`trident.md:22,87–89,99` 把 85 伤写成对半径内 combat 目标直接生效，把船漏写成命中船板便生效。实际先按半径搜实体并要求 `IsOnOcean(false)`，伤害分支首先要求 health，其中有 combat 的还要通过 `CanBeAttacked(owner)`。掀飞是 inventoryitem 分支，与是否可作祟无关。船漏和水柱来自另一段四次角度取样，取样点必须是海洋瓦片且非视觉地面，命中平台才尝试漏水和推船，无平台才出水柱，水柱至多四个。渔获无 complexprojectile 时还有直接掀飞回退。按这两段流程改快速判断、机制表、典型接入及索引摘要。 [Wiki](../../../wiki/reference/prefabs/trident.md#L18-L23) · [机制表](../../../wiki/reference/prefabs/trident.md#L86-L89) · [实体分派](../../../raw/current/scripts/prefabs/trident.lua#L68-L130) · [海面与四点采样](../../../raw/current/scripts/prefabs/trident.lua#L146-L186) · [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L29)

### R04 直接调用 finiteuses 不会读取玻璃刀磨损档

`glasscutter.md:90` 声称跳过 `Weapon:OnAttack` 直接 `finiteuses:Use` 会取上次残留的 `attackwear`。组件只扣 `num or 1`，按目标写入的 `attackwear` 由 weapon 组件读取。直接调用默认扣 1，调用者若要半磨损需要显式传入。同步同页示例的生效范围，伤害 PostInit 作用于构造时，`SHADOW_WEAR` 才是每次攻击现读。 [Wiki](../../../wiki/reference/prefabs/glasscutter.md#L70-L90) · [目标磨损档](../../../raw/current/scripts/prefabs/glasscutter.lua#L32-L36) · [weapon 消费](../../../raw/current/scripts/components/weapon.lua#L105-L120) · [finiteuses 扣减](../../../raw/current/scripts/components/finiteuses.lua#L81-L87)

### R05 蝙蝠棒满血时仍可出皮肤命中特效

`batbat.md:55` 把满血条件套到两个回调块。FX 分支只要求皮肤映射、有效目标和目标有 combat，之后独立判断持有者未满血及目标可汲命，才进入治疗和理智结算。索引第 18 行也应保留治疗门槛，避免“命中回血”被当成无条件效果。 [Wiki](../../../wiki/reference/prefabs/batbat.md#L50-L57) · [回调](../../../raw/current/scripts/prefabs/batbat.lua#L35-L55) · [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L18)

## P2

### R06 暗影槌示例只改耐久上限，固定伤害会被原版重写

`shadow-battleaxe.md:124–141` 的 PostInit 只设上限 250，初始当前值仍是原版 200；帽套装开关会重新按 tuning 写 weapon 基础伤害，破损修好时重挂组件也会覆盖示例的 45。把示例承诺收窄到当前实例的初始覆盖，或让数值和原套装、修复路径一致。升级阈值为全局表，宜在示例中独立于实例 PostInit 设置。 [Wiki](../../../wiki/reference/prefabs/shadow-battleaxe.md#L122-L141) · [初始化耐久](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L840-L845) · [只写上限](../../../raw/current/scripts/components/finiteuses.lua#L60-L67) · [套装](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L84-L104) · [组件重建的伤害](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L400-L419) · [修理回调](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L486-L494)

### R07 暗影槌的击杀追踪随卸装清除

`shadow-battleaxe.md:64,146` 把助攻记数概括成命中后五秒内死亡，漏了卸装会解除追踪。把第 74 行的解绑前提同步到升级章节，并改正 `smalleptic` 拼写为 `smallepic`。另外把第 70 行治疗乘区写为 `aoediminishingreturns.mult:Get()`，无组件时乘 1，理智扣减仍按未乘的汲命值。 [Wiki](../../../wiki/reference/prefabs/shadow-battleaxe.md#L62-L74) · [卸装](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L63-L79) · [目标与吸血](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L265-L305) · [死亡时间窗](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L802-L809)

### R08 铥矿棒触手接收显式目标，攻击后可提前离场

`ruins-bat.md:22–23,150–152` 将触手描述为自行索敌的群攻来源，并把 9 秒写成固定存续。召唤成功还要求找到落点并创建实例，工厂调用 `SetTarget(target)` 传入原目标，原目标无效时传 nil；触手工厂没有挂自动索敌，SG 收到目标后开始单目标攻击，动画的第 7、17 帧各尝试一次，收招动画结束即可 Remove，另有 9 秒兜底。第 146 行“只影响无幸运加成的基础概率”也应写清，正幸运仍在同一个已调整的基础值上做加法。 [Wiki](../../../wiki/reference/prefabs/ruins-bat.md#L20-L23) · [陷阱](../../../wiki/reference/prefabs/ruins-bat.md#L146-L154) · [落点和传目标](../../../raw/current/scripts/prefabs/ruins_bat.lua#L37-L58) · [触手工厂](../../../raw/current/scripts/prefabs/shadowtentacle.lua#L46-L68) · [触手状态机](../../../raw/current/scripts/stategraphs/SGshadowtentacle.lua#L75-L106) · [掷点分流](../../../raw/current/scripts/componentutil.lua#L2211-L2214)

### R09 火腿棒示例只换攻击后的快照更新

`hambat.md:90,119` 已知道本击读取上次伤害快照，但声称完整改曲线只需额外包装备回调，漏了 `inst.OnLoad`。原版装备、卸装和读档都会按原公式写回伤害。示例仅在攻击后更新 80% 下限，该更新影响下一次攻击；要让各入口一致须交代读档与穿卸两个回调，或把示例标题收窄。索引第 19 行也可加“伤害按上次更新的新鲜度快照”。 [Wiki](../../../wiki/reference/prefabs/hambat.md#L85-L110) · [源码更新点](../../../raw/current/scripts/prefabs/hambat.lua#L7-L35) · [结算顺序](../../../raw/current/scripts/components/combat.lua#L1212-L1225) · [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L19)

### R10 火腿棒的冰箱倍率是其中一项，保鲜背包也可减缓腐坏

`hambat.md:22,80,118` 把冰箱写成唯一原版延寿手段，并将 0.5 写成最终腐坏速率。保鲜背包用普通背包容器且带 `fridge` 标签，也走该档；季节温度、湿度、局部及全局倍率还会叠乘。另把“装备中腐坏会落到背包空位”改为优先交回物品栏，未找到合适位置时按 `GiveItem` 回退处理。 [Wiki](../../../wiki/reference/prefabs/hambat.md#L114-L120) · [背包容器](../../../raw/current/scripts/containers.lua#L23-L43) · [保鲜背包标签](../../../raw/current/scripts/prefabs/icepack.lua#L53-L60) · [腐坏倍率](../../../raw/current/scripts/components/perishable.lua#L78-L129) · [替换交回](../../../raw/current/scripts/components/perishable.lua#L303-L317)

### R11 亮茄剑的 200 次与命中监听需要明确适用范围

`sword_lunarplant.md:57,67–85,91` 把 `weapononattack` 当成每个目标受伤时都推送，又把 200 耐久等同 200 次命中。示例是监听 `Weapon:OnAttack` 的结算事件，通用范围副目标另走 `onareaattackother`，不逐目标推武器事件；扣耐久还可能受 attackwear 乘区、efficientuser 与忽略战斗磨损开关影响。可写默认单次武器结算扣 1，无修正时约 200 次，保留事件示例并标清作用域。 [Wiki](../../../wiki/reference/prefabs/sword_lunarplant.md#L65-L95) · [武器事件及磨损](../../../raw/current/scripts/components/weapon.lua#L105-L120) · [范围副目标](../../../raw/current/scripts/components/combat.lua#L1325-L1335)

### R12 两种鞭子的索引射程口径要一致

索引第 26–27 行把三尾猫鞭写作射程 5，公牛海带茎写作射程 2，容易看成两者攻击距离不同。两者武器加值都是 2，默认玩家在此基础上加 combat 的 3 和目标物理半径。鞭页第 51 行所引 `CalcAttackRangeSq` 是出手判定，实际命中另有 `CalcHitRangeSq`，此例的默认数值相同。索引里的“打断目标攻击”也宜写成“对筛选后已有战斗目标的单位丢目标，满足状态条件才硬直”，与正文一致。 [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L26-L27) · [鞭页](../../../wiki/reference/prefabs/whip.md#L48-L61) · [海带茎数值](../../../wiki/reference/prefabs/bullkelp_root.md#L39-L46) · [两种半径](../../../raw/current/scripts/components/combat.lua#L1014-L1035) · [命中使用点](../../../raw/current/scripts/components/combat.lua#L1067-L1092)

### R13 公牛海带茎来自海面漂流物，索引写成了漂流瓶

索引第 27 行的“漂流瓶获得”与正文的 flotsam 路径不一致。海面漂流物使用含 `bullkelp_root` 的权重表生成战利品，页面还列有海獭窝；索引改成“拔起、冲岸、海面漂流物／海獭窝”。同一行的 40 天标为基础保质期，避免被当成实际手持寿命。 [索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L27) · [获取途径](../../../wiki/reference/prefabs/bullkelp_root.md#L62-L70) · [漂流物权重表](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L18-L25) · [生成战利品](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L184-L193)

### R14 木头短剑的容器敲落与物品栏敲落有区别

`cutless.md:39–42,113–114` 把两条分支合写成同一套随机弹飞和事件行为。物品栏分支查普通物品槽、活动物品与 overflow，找到物品后传随机方向参数并推事件；容器分支单独调用 `Container:DropItem(item)`，落点从容器位置取得。容器没有可偷物时 `DropItemAt(nil)` 直接返回，`StealItem` 仍会推 `onitemstolen`（`item=nil`）再返回 false。同步概要，保留 `nosteal` 的筛选条件。 [Wiki](../../../wiki/reference/prefabs/cutless.md#L35-L42) · [两分支](../../../raw/current/scripts/components/thief.lua#L20-L54) · [物品栏搜索](../../../raw/current/scripts/components/inventory.lua#L623-L635) · [容器掉落](../../../raw/current/scripts/components/container.lua#L220-L226) · [空物品早退](../../../raw/current/scripts/components/container.lua#L246-L252)

### R15 三叉戟蓝图在赃物袋的抽取概率需分两层

`trident.md:60,141` 的“50% 概率入选”容易被读成该蓝图有 50% 掉率。第四组战利品先以 50% 概率从九项蓝图池均匀抽一项，因此单次该组抽取命中三叉戟蓝图为 `0.5 × 1/9 = 1/18`。索引可以写帝王蟹与克劳斯赃物袋两条路径。 [Wiki](../../../wiki/reference/prefabs/trident.md#L53-L61) · [池内容](../../../raw/current/scripts/components/klaussackloot.lua#L10-L21) · [抽取逻辑](../../../raw/current/scripts/components/klaussackloot.lua#L115-L124)

### R16 相关对象第一次出现时用游戏简中名

13 个主对象标题均与简中名称键相符。修正会影响玩家检索的相关对象称呼，三叉戟原料的“长鼻角”应为“一角鲸的角”，对应页已经使用该名称；三叉戟、三尾猫鞭及索引首次写 `crabking` 时用“帝王蟹”；触手尖刺的掉落表用“大触手（`tentacle_pillar`）”；铥矿棒的产物用“暗影触手（`shadowtentacle`）”；暗影槌首次介绍套装和修复物时用“虚空风帽”和“虚空修补套件”。既有简称若有检索价值，可以保留为简称。 [三叉戟配方](../../../wiki/reference/prefabs/trident.md#L55-L60) · [一角鲸的角](../../../raw/current/scripts/languages/chinese_s.po#L343340-L343342) · [帝王蟹](../../../raw/current/scripts/languages/chinese_s.po#L341780-L341782) · [大触手](../../../raw/current/scripts/languages/chinese_s.po#L350345-L350347) · [暗影触手](../../../raw/current/scripts/languages/chinese_s.po#L349175-L349177) · [帽与修补套件](../../../raw/current/scripts/languages/chinese_s.po#L351345-L351357)

### R17 蝙蝠棒的骨架布局有注册行

`batbat.md:64` 写 `skeleton_batfight` 未找到注册行。它登记在 `map/pointsofinterest.lua` 的 POI 表，检索注册键未确认具体放置调用。区分“布局文件、注册表、实际放置”三个状态。 [Wiki](../../../wiki/reference/prefabs/batbat.md#L59-L66) · [POI 注册](../../../raw/current/scripts/map/pointsofinterest.lua#L74-L78)

### R18 长矛、触手尖刺及暗夜剑示例需要区分当前实例与读档

`spear.md:74`、`tentaclespike.md:75` 将 PostInit 对“已有实例”的上限写成即时生效。它在构造时执行，读档重建也执行，之后已保存的 `uses` 可覆盖示例设置的当前值，且 `SetUses` 不按新上限钳制。移除改动后原版构造上限恢复，已写入存档的当前耐久仍可能留下。暗夜剑第 83 行也应明确该区别，并把示例中的“去掉装备理智惩罚”写为“将基础 dapperness 设为 0”，湿装备仍会叠加湿度修正。 [长矛](../../../wiki/reference/prefabs/spear.md#L71-L84) · [触手尖刺](../../../wiki/reference/prefabs/tentaclespike.md#L72-L85) · [暗夜剑](../../../wiki/reference/prefabs/nightsword.md#L80-L98) · [构造 PostInit](../../../raw/current/scripts/mainfunctions.lua#L390-L410) · [读档恢复](../../../raw/current/scripts/mainfunctions.lua#L464-L505) · [有限耐久存读](../../../raw/current/scripts/components/finiteuses.lua#L48-L77) · [装备理智](../../../raw/current/scripts/components/equippable.lua#L196-L211)

## 页面处置

| 页面 | 待处理 |
|---|---|
| 长矛 | R18 |
| 蝙蝠棒 | R05、R17 |
| 火腿棒 | R09、R10 |
| 触手尖刺 | R16、R18 |
| 铥矿棒 | R08、R16 |
| 暗影槌 | R01、R02、R06、R07、R16 |
| 暗夜剑 | R18 |
| 亮茄剑 | R11 |
| 玻璃刀 | R04 |
| 三尾猫鞭 | R12、R16 |
| 公牛海带茎 | R12、R13 |
| 木头短剑 | R14 |
| 刺耳三叉戟 | R03、R15、R16 |

13 页在本叶均有对应登记，页尾均返回本叶。待对应页面及索引摘要修正后复核。

## 处置结果（2026-11-07）

R01～R18 逐条对照当前快照复核，18 项全部成立并采纳；改 13 页与武器叶索引，另同步 voidcloth_scythe 对暗影槌饥饿的转述。

- R01：破损时先 `DisableComponents` 再 `SetLevel(1)`；等级>1 会把计数钳到 0，原本 1 级则同级早退、保留 1–2 杀。修复只重建组件与外观，不回等级。同步残破章节与陷阱。
- R02：饥饿改为“二级起武器自身持续”，装备/卸装都不切换任务；一级 `Pause()`。摘要、快速判断、数值表与索引同步。
- R03：三叉戟拆两段——实体分派（health 85，combat 还须 `CanBeAttacked`；渔获投射物；海带拔起；inventoryitem 掀飞；MINE 10）与四点采样（海洋瓦片且非视觉地面；平台漏水＋推船，否则水柱）。快速判断、机制表、典型接入与索引同步。
- R04：`finiteuses:Use` 只扣 `num or 1`，不读 `attackwear`；直接调用默认扣 1。伤害 PostInit 只在构造/读档重建执行，`SHADOW_WEAR` 才是每次攻击现读。
- R05：满血只跳过吸血与理智块；皮肤 FX 仍按皮肤与目标 combat 条件生成。索引补“未满血时”。
- R06：示例改为改 `TUNING.SHADOW_BATTLEAXE.DAMAGE/USES/LEVEL_THRESHOLDS`，使构造、套装切换、修复重建读同一组值；说明只 postinit 会被套装/修复覆盖。
- R07：补卸装 `ForgetAllTargets` 解除追踪；`smalleptic` 改 `smallepic`；回血用 `mult:Get()`（无组件按 1），理智仍用未乘 mult 的 `_lifesteal`。
- R08：触手显式 `SetTarget`、不自行索敌；攻击动画第 7、17 帧各尝试一次，收招即 Remove，9 秒只是兜底。掷点基础值改动仍会进入幸运加法（`luck×0.2`），并非只影响无幸运分支。
- R09：补 `inst.OnLoad` 也是原公式重算入口；示例只改攻击后快照，首次装备/读档后的第一击仍按原下限。索引补“按上次更新的新鲜度快照”。
- R10：冰箱不是唯一延寿手段——带 `fridge` 标签的容器（含保鲜背包）都走 ×0.5，且季节温度、湿度、局部/全局倍率叠乘；替换落点改为优先原格、`GiveItem` 回退。
- R11：`weapononattack` 只随武器结算推主目标，范围副目标走 `onareaattackother`；200 用不等于恒定 200 次命中（attackwear 乘区、efficientuser、忽略磨损都会改变）。
- R12：两种鞭子统一为“武器加值 2，叠玩家基础 3 与目标半径”；鞭页区分 `CalcAttackRangeSq`（出手）与 `CalcHitRangeSq`（命中）。索引同步“丢目标＋满足状态才硬直”。
- R13：索引改“被毁/冲岸/海面漂流物/海獭窝”，40 天标为基础保质期。
- R14：木头短剑拆物品栏与容器两分支；容器落点在容器位置，空容器仍推 `onitemstolen(item=nil)` 后返回 false。
- R15：Klaus 池先 50% 抽组、再 1/9 抽项，单次命中三叉戟蓝图为 1/18；索引写帝王蟹固定与克劳斯赃物袋两条路径。
- R16：首次称呼改为实际游戏名——一角鲸的角、帝王蟹、大触手（`tentacle_pillar`）、暗影触手（`shadowtentacle`）、虚空风帽、虚空修补套件；索引同步。
- R17：`skeleton_batfight` 注册在 `map/pointsofinterest.lua` 的 Cave POI 表，实际放置调用未确认；区分布局、注册、放置。
- R18：长矛/触手尖刺的 postinit 只在构造与读档重建执行，`SetMaxUses` 只改上限、存档 `uses` 覆盖 `SetUses`；暗夜剑湿装备仍叠加 `WET_ITEM_DAPPERNESS`，示例改“基础 dapperness 设为 0”。

校验：15 页 `validate_page.py` 通过（13 页主体＋voidcloth_scythe、voidcloth_boomerang 的同步修正；tentaclespike 12,065 B 为 simple 档内、bullkelp_root 24,559 B 最紧）；`validate_source_links.py --strict` 1,173 条引用 0 error 0 warning；`record` 14 页，2 页因新增 sources 刷新（batbat 补 pointsofinterest、cutless 补 container）；`--check-indexes` 导航通过。未运行游戏，未修改 raw，未提交或推送。

## 复检补正（2026-11-07）

- R17：`skeleton_batfight` 除布局与 Cave POI 注册外，`worldgen_main.lua` 的 `AddSingleSetPeice(level, "map/pointsofinterest")` 会从区域表随机抽取；改写成“可被洞穴世界生成按区域随机选中，实际落点与频率未验证”。sources 补 `worldgen_main.lua` 并 record。
- R14：物品栏与容器两分支都会随机弹飞，差别只在起点——物品栏从目标位置、容器由 `DropItemAt` 在容器坐标调 `OnDropped(true)`。摘要、快速判断、机制步骤与陷阱同步。
- R11：典型接入标题、前提、示例注释与数值段统一为“每次武器结算”；数值段补默认扣 1 及 `attackwear` 乘区、`efficientuser`、`IgnoresCombatDurabilityLoss` 条件。
- R04：前提说明补全键名 `TUNING.GLASSCUTTER.SHADOW_WEAR`。
- R08／R16：导航表、陷阱、摘要与索引统一为“收招可提前移除、9 秒兜底”，并用“暗影触手”。
- R18：暗夜剑撤销说明与长矛/触手尖刺一致——停用模组后新实例恢复原上限，存档 `uses` 仍可能高于原上限。

校验：7 页 `validate_page.py` 通过（glasscutter 12,275 B 为 simple 档内）；`validate_source_links.py --strict` 全批 16 文件 1,176 条引用 0 error 0 warning；`record` 刷新 batbat 1 页。未运行游戏，未修改 raw，未提交或推送。
