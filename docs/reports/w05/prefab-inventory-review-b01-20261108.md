# 非装备物品栏 B01 独立检视

复检 Steam build `26335394`。范围为 B01 交付的 9 页、30 个主体注册及 Prefab 暂行索引。对照了注册与主要组件、获得和结算路径、示例语义、中文名称及前次四项 P1 的修订。九页均有暂行登记并能返回该索引。30 个主体的简中名称键均能在当前快照对应。

结论为**修订后复检**。本次发现 P1 一项、P2 十二项、P3 一项。保留写作方机械验收记录。本次检查文件内容和源码，未重新运行 Wiki 校验器、Lua 示例或游戏。

| 编号 | 等级 | 页面位置 | 发现与建议 | 源码依据 |
|---|---|---|---|---|
| R01 | P1 | [催长剂 L82–L101](../../../wiki/reference/prefabs/soil-amender.md#修改发酵时长) | 示例把构造后初始化与读档顺序说反了。存档重建先生成实例并执行 prefab postinit，随后 `SetPersistData` 调组件的 `OnLoad`，把保存的剩余秒数和暂停状态覆盖回去。当前代码会改变发酵上限，无法保证已存档的进度百分比按新时长缩放。在 postinit 当场改用 `SetNewMaxPerishTime` 也不能解决顺序。写清新做物品和存量存档的区别；若演示保持比例，应在组件读回后按明确的旧上限换算。 | [构造后回调](../../../raw/current/scripts/mainfunctions.lua#L394-L412)、[存档生成与恢复](../../../raw/current/scripts/mainfunctions.lua#L464-L503)、[加载组件](../../../raw/current/scripts/entityscript.lua#L2002-L2008)、[保质组件读档](../../../raw/current/scripts/components/perishable.lua#L329-L348) |
| R02 | P2 | [催长剂 L20、L22、L75、L106–L108](../../../wiki/reference/prefabs/soil-amender.md#快速判断) | 快速判断把用途限成“只经部署”。部署负责地块养分，物品仍可由普通 `FERTILIZE` 动作进入符合条件的旧目标或自肥者，例如沃姆伍德。最后一次施用的空瓶优先尝试交给有库存组件的使用者，无该组件时在目标或物品位置生成。同步快速判断和陷阱。 | [动作收集](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[FERTILIZE](../../../raw/current/scripts/actions.lua#L1812-L1842)、[沃姆伍德自肥组件](../../../raw/current/scripts/prefabs/wormwood.lua#L841-L846)、[还瓶](../../../raw/current/scripts/prefabs/soil_amender.lua#L41-L52) |
| R03 | P2 | [催长剂 L6、L40–L43、L107](../../../wiki/reference/prefabs/soil-amender.md#三态发酵链) | “原位／原持有格”应写成尝试。地面按旧实例坐标生成，库存态把旧槽传给对应库存或容器的 `GiveItem`，结果由其收纳规则决定。腐坏条只属于有 `show_spoilage` 和保质阶段标签的起子，超级催长剂接的是耐久。 | [替换](../../../raw/current/scripts/components/perishable.lua#L293-L318)、[起子显示与标签](../../../raw/current/scripts/prefabs/soil_amender.lua#L150-L162)、[超级催长剂耐久](../../../raw/current/scripts/prefabs/soil_amender.lua#L233-L244)、[腐坏条判断](../../../raw/current/scripts/widgets/itemtile.lua#L732-L747) |
| R04 | P2 | [月亮碎片 L69](../../../wiki/reference/prefabs/moonglass-materials.md#常见陷阱与限制) | “替换只换实体，堆叠位置不保留”容易让人误以为一堆只替换一件或必定离开原格。两边有 stackable 时源码继承原数量，地面继承坐标，持有时尝试按原槽插入。换实体后原槽可能保留。 | [注能态替换配置](../../../raw/current/scripts/prefabs/moonglass.lua#L139-L146)、[数量与槽位](../../../raw/current/scripts/components/perishable.lua#L293-L318) |
| R05 | P2 | [月亮碎片 L39、L71](../../../wiki/reference/prefabs/moonglass-materials.md#获得路径) | 月相仪两条退晶路径需分开。初始化或睡眠的即时分支会把数量加到第一个未满 `moonglass_piece` 候选，候选可以是注能态；无合适候选才弹出普通碎片。清醒分支走 SG，在第 10 帧直接弹出普通碎片。 | [睡眠与清醒分支](../../../raw/current/scripts/prefabs/moondial.lua#L51-L70)、[退晶状态](../../../raw/current/scripts/stategraphs/SGmoondial.lua#L148-L159) |
| R06 | P2 | [羽毛扇与幸运扇 L22、L54–L59、L107](../../../wiki/reference/prefabs/cooling-fans.md#扇风结算) | 降温只作用于标签筛选后的候选，组件类型和数值需再判断。两个回调都读当前温度，以无隔热且初温 30 度为例，原回调降 27.5 度，第二段读到 2.5 度后降 0。文中的合计 −55 度错误。玩家的 `Temperature:DoDelta` 还有隔热换算，−50／−80 是两段请求值的夹限。 | [筛选和夹限](../../../raw/current/scripts/prefabs/fans.lua#L16-L33)、[温度分流](../../../raw/current/scripts/componentutil.lua#L2416-L2438)、[隔热换算](../../../raw/current/scripts/components/temperature.lua#L71-L83) |
| R07 | P2 | [羽毛扇与幸运扇 L59、L63–L69](../../../wiki/reference/prefabs/cooling-fans.md#幸运扇吟唱) | 9 次耐久拆成 3 次动作、龙卷风 2＋1＋0，只在各次达到门槛的吟唱都找到落点、普通 FAN 都执行成功时成立。无落点不扣 2；普通动作还有物品拟态提前返回。写成示例轨迹，保留每次上限和各自条件。 | [吟唱门禁与扣费](../../../raw/current/scripts/prefabs/fans.lua#L39-L74)、[FAN 动作](../../../raw/current/scripts/actions.lua#L3943-L3951) |
| R08 | P2 | [排箫 L60–L73](../../../wiki/reference/prefabs/panflute.md#使用流程与听觉分流) | 入睡不是“达到抗性即睡 20 秒”。等于抗性时先排延迟，真正入睡还有有效性条件及持续时间倍率。遗忘还要求 `AddSleepiness` 返回后已在睡眠中并有 `combat`。灵魂召唤环带半径为 3.95–7.9，8 是吸引范围调参基值。 | [听觉和灵魂](../../../raw/current/scripts/prefabs/panflute.lua#L28-L85)、[入睡门槛与倍率](../../../raw/current/scripts/components/sleeper.lua#L248-L307)、[范围基值](../../../raw/current/scripts/tuning.lua#L4542) |
| R09 | P2 | [排箫 L80](../../../wiki/reference/prefabs/panflute.md#调整催眠时长与半径)、[羽毛扇与幸运扇 L76](../../../wiki/reference/prefabs/cooling-fans.md#调整扇风降温强度与目标) | 两页的“postinit 对新旧实例都跑”应收窄到新构造及读档重建。排箫示例直接修改全局 TUNING，与只改构造的半径具有不同生效范围。 | [构造时回调](../../../raw/current/scripts/mainfunctions.lua#L394-L412)、[读档重建](../../../raw/current/scripts/mainfunctions.lua#L464-L466) |
| R10 | P2 | [宝石 L32、L44–L45](../../../wiki/reference/prefabs/gems.md#特色消费蟹王嵌彩与档案馆开关) | 蟹王的三颗彩虹宝石只会在珍珠未嵌、且寄居蟹已交出珍珠时采用珍珠款冬季饰品。L32 的“工厂外部无任何按颜色的分支”应收窄到本工厂。档案馆存档恢复生成的彩虹宝石交给开关 trader，用于恢复嵌宝状态。 | [蟹王饰品门禁](../../../raw/current/scripts/prefabs/crabking.lua#L1497-L1505)、[恢复开关](../../../raw/current/scripts/prefabs/archive_props.lua#L886-L891) |
| R11 | P2 | [船补丁 L23、L39–L47、L79](../../../wiki/reference/prefabs/boat-patches.md#修补动作与消费链) | 船体回血须由 `Repairable:Repair` 返回成功，材料匹配只是门槛之一，满血不会回血，补漏仍消耗一件补丁。L47 的“修船扣船体血”应写回血。首次生成缓存的是配方计算出的海带数量，用于修复量和食用数值；“改配方不生效”应限定为这几个取缓存的参数。 | [补漏消费](../../../raw/current/scripts/components/boatleak.lua#L62-L94)、[满血与修复门禁](../../../raw/current/scripts/components/repairable.lua#L106-L114)、[缓存取用](../../../raw/current/scripts/prefabs/boatpatch.lua#L105-L129) |
| R12 | P2 | [常用采集材料 L40–L45](../../../wiki/reference/prefabs/basic-gathered-materials.md#获取途径与入栏路径)、[索引 L40](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) | 从草丛、树苗和芦苇采集走 `PICK → Pickable:Pick → SpawnProductLoot`，通常直接尝试 `GiveItem`。地上的物品才经 `PICKUP` 拾取。分开写两条入口，更新索引的“SetUp 产品经 PICKUP 入栏”。 | [PICK](../../../raw/current/scripts/actions.lua#L1933-L1945)、[产物和入栏](../../../raw/current/scripts/components/pickable.lua#L492-L515)、[拾地面物品](../../../raw/current/scripts/actions.lua#L954-L965) |
| R13 | P2 | [基础精炼材料 L22、L60、L85–L87](../../../wiki/reference/prefabs/basic-refined-materials.md#借鉴对特定目标使用的材料接线)、[矿物原料 L48](../../../wiki/reference/prefabs/basic-mineral-materials.md#获取途径与入栏路径) | 绳子的动作是 `USEITEMON`，`TIE_ONTO` 是界面文案键。绳子燃烧 30 秒小于燃料值 45 秒，两者应分开解释。猪王小游戏只按猪年 YOTP 切换普通金块和幸运金块，其他 YOT 活动不能套这个切换条件；幸运金兑换也受玩家门禁。 | [绳子文案](../../../raw/current/scripts/prefabs/rope.lua#L25-L31)、[动作结算](../../../raw/current/scripts/actions.lua#L3351-L3362)、[绳子燃料](../../../raw/current/scripts/prefabs/rope.lua#L64-L71)、[猪王小游戏](../../../raw/current/scripts/prefabs/pigking.lua#L538-L541)、[猪王接受测试](../../../raw/current/scripts/prefabs/pigking.lua#L598-L605) |
| R14 | P3 | [羽毛扇与幸运扇 L36](../../../wiki/reference/prefabs/cooling-fans.md#注册与构造)、总台账 B01 行 | `perdshrine` 的简中名为“火鸡神龛”，正文目前写作“火鸡祭坛”。台账 gems 括号可改为“六色＋彩虹宝石”，与 7 个注册和全批 30 名一致。 | [PERDSHRINE 名称](../../../raw/current/scripts/languages/chinese_s.po#L346810-L346812)、[宝石七注册](../../../raw/current/scripts/prefabs/gem.lua#L95-L101) |

前次复审修订中，木板技能门禁、堆肥桶拒收 ELEMENTAL、排箫骑乘后继续处理听众自身、扇子示例先定义再注册均已在对应页核对。待上述内容修订后，定向复检涉及的正文与索引，再将 B01 转为已收口。

## 修订后复检（2026-11-08）

R01 的催长剂读档顺序已对齐源码。注能碎片到期继承堆叠并尝试回填、扇子两段重新读取体温及 9 次耐久轨迹的前提也已修正。B01 仍有以下局部表述待同步。

| 原编号 | 位置 | 剩余修订 |
|---|---|---|
| R05 | [月亮碎片 L39、L81](../../../wiki/reference/prefabs/moonglass-materials.md#获得路径) | 原报告“第一个未满候选”的措辞也需勘正。源码先取半径内搜索结果的 `[1]`，再判断这一堆是否已满。第一堆满时直接弹出普通碎片，后面的堆即使有空位也不会继续找。[月相仪即时分支](../../../raw/current/scripts/prefabs/moondial.lua#L59-L67) |
| R06 | [羽毛扇与幸运扇 L22](../../../wiki/reference/prefabs/cooling-fans.md#快速判断) | 快速判断仍写“所有带温度的实体”。改成“通过标签筛选且能读取温度的目标”，与同页目标集表对齐。折算发生在主模拟的玩家实体温度路径。[扇风目标筛选](../../../raw/current/scripts/prefabs/fans.lua#L16-L31) |
| R10 | [宝石 L44、L95](../../../wiki/reference/prefabs/gems.md#特色消费蟹王嵌彩与档案馆开关) | 现文“否则普通款”会排掉已经嵌珍珠的路径。已嵌珍珠直接选珍珠款。未嵌珍珠才检查彩虹宝石至少三颗，以及寄居蟹已经交出珍珠。[冬季饰品选择](../../../raw/current/scripts/prefabs/crabking.lua#L1497-L1505) |
| R12 | [常用采集材料 L45、L66](../../../wiki/reference/prefabs/basic-gathered-materials.md#获取途径与入栏路径)、[索引 L40](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) | PICK 和 PICKUP 已分开。把“有 inventory 时产物不落地”收窄为“采集时直接调用 `GiveItem` 尝试入栏”，物品栏无法接收时仍可能掉地。[直接交付](../../../raw/current/scripts/components/pickable.lua#L492-L515)、[物品栏满时的掉落分支](../../../raw/current/scripts/components/inventory.lua#L1167-L1215) |
| R13 | [矿物原料 L83](../../../wiki/reference/prefabs/basic-mineral-materials.md#关联机制导航) | 获得表已明确 YOTP，导航表中普通金块的“非活动期”也改成“非 YOTP”，与两种金块的标签门禁一致。[普通金块](../../../raw/current/scripts/prefabs/goldnugget.lua#L42-L47)、[幸运金块](../../../raw/current/scripts/prefabs/goldnugget.lua#L128-L132) |

本轮复检只读页面和源码，机械验收结果沿用写作方修订后的记录。

## 收口（2026-11-08）

R05、R06、R10、R12、R13 均已对照源码复验。月相仪只检查搜索结果的第一堆，蟹王已嵌珍珠优先选珍珠款，扇风限定标签筛选与可读取温度，采集入栏条件已同步正文和索引，普通金块标签明确限定非 YOTP。B01 的 9 页、30 个注册已收口。采用修订方针对五页及索引的校验结果，本次未重跑脚本或游戏。
