# 装备页第四轮独立检视（2026-11-07）

按 Steam buildid `26335394` 的本地 Lua 快照检视。知识仓基准 `97afb4e`，正文采用检视时的工作区版本。范围为武器与战斗索引剩余 6 页，覆盖投掷、远程、防御与事件武器；优先核对发射与命中、伤害／耐久、动作门禁、存档及示例，并对照索引摘要。累计已独立检视 41/69 页，余 28 页。

**结论：需修订，P1 7 项、P2 17 项。** P1 涉及反向机制结论、玩家操作和接入示例；同一问题在各处的复述合并记录。页行号以本次快照为准。

## 回旋镖（boomerang）

- **R01 · P2｜回程与接回。** 摘要、L20–22、L61 和索引 L35 将回程接回写得过于确定。去程 Miss 也会尝试返航，命中时掷出者已是鬼魂则走落地分支；去程 `Throw` 也会对具有 catcher 的目标登记观察。动作提示同时要求弹体 `catchable` 和操作者 `cancatch`；真正接取停弹后，只有物品栏打开才走自动装备／入包，且 HANDS 空闲才尝试自动装备。 [页面](../../../wiki/reference/prefabs/boomerang.md#L18-L23) · [捕捉与返航](../../../raw/current/scripts/prefabs/boomerang.lua#L47-L87) · [观察注册](../../../raw/current/scripts/components/projectile.lua#L184-L201) · [动作门禁](../../../raw/current/scripts/componentactions.lua#L687-L690)
- **R02 · P2｜脱靶门禁。** L43、L99 把 `hitdist=1` 写成实际半径，并混用卡滞、离屏和失去目标三条路径。追踪判定叠加目标物理半径，且有越过目标的判定；卡滞与休眠计时各自触发 Miss。失去目标后，持有人 combat 与武器射程数据齐备才走射程兜底。 `CheckTarget` 不直接读死亡状态。 [页面](../../../wiki/reference/prefabs/boomerang.md#L94-L100) · [目标门禁](../../../raw/current/scripts/components/projectile.lua#L293-L301) · [追踪与 Miss](../../../raw/current/scripts/components/projectile.lua#L396-L455) · [休眠计时](../../../raw/current/scripts/components/projectile.lua#L268-L289)
- **R03 · P2｜世界生成证据。** L45 的字面 `SpawnPrefab` 搜索不能支持「无世界生成点」。L50 的 WeaponBoon 确实可经世界生成随机抽取并登记到相应任务区，所选布局的物品区再从四个候选抽一件。补实际消费者，保留落点与频率未核验。 [页面](../../../wiki/reference/prefabs/boomerang.md#L45-L50) · [布局](../../../raw/current/scripts/map/boons.lua#L150-L155) · [抽取与任务区](../../../raw/current/scripts/worldgen_main.lua#L300-L317) · [配置门禁](../../../raw/current/scripts/worldgen_main.lua#L368-L373)

## 阴郁回旋镖（voidcloth_boomerang）

- **R04 · P1｜弹体上限示例。** L103–121 声称已有实例改 TUNING 即生效、任一弹体注销就读新上限。实际是构造时和套装启用状态改变时读取 TUNING 并缓存为 `inst.max_projectiles`，计数回调只比较缓存。套装退回普通上限后，即使删掉一枚也可能仍达上限，须降到阈值以下才回充。同步 L22、82、127 和示例生效说明。 [页面](../../../wiki/reference/prefabs/voidcloth_boomerang.md#L99-L127) · [缓存写入](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L38-L46) · [计数](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220) · [初始化](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L279-L285)
- **R05 · P1｜近战仍耗耐久。** L84 写磨损只走掷出链，L126 把 85 耐久等同于总能掷 85 次。放电近战同样由本体 `weapon:OnAttack` 扣 finiteuses；远程由 OnThrown 扣本体，弹体命中不再扣本体。两条路径的 1 点均为默认值，受通用攻击磨损修正影响。同步数值、陷阱及索引 L36。 [页面](../../../wiki/reference/prefabs/voidcloth_boomerang.md#L123-L128) · [放电](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L202-L209) · [发射磨损](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L442-L457) · [近战回调](../../../raw/current/scripts/components/combat.lua#L1212-L1225) · [磨损计算](../../../raw/current/scripts/components/weapon.lua#L105-L119)
- **R06 · P2｜返航缩放用错常量。** L121 的 `10/RETURN_SPEED` 应为普通回旋镖的 `TUNING.BOOMERANG_DISTANCE / RETURN_SPEED`，即默认 `12/7.5=1.6s`。去程才是 `VOIDCLOTH_BOOMERANG_ATTACK_DIST / LAUNCH_SPEED`。返航公式给的是尺寸插值时长，抵达、越过或目标失效可提前移除。 [页面](../../../wiki/reference/prefabs/voidcloth_boomerang.md#L119-L121) · [返航](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L347-L358) · [去程](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L515-L520) · [调值](../../../raw/current/scripts/tuning.lua#L173)
- **R07 · P2｜玩家动作入口。** L131 称与普通回旋镖共用 `throw`，但本体无 `thrown` 标签。普通分流按标签进 `throw`，阴郁版走常规攻击状态再由 combat 发射独立弹体。L69 两版分流也应说清是 combat 查组件／武器能力，SG 查标签。 [页面](../../../wiki/reference/prefabs/voidcloth_boomerang.md#L125-L131) · [本体标签](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L247-L261) · [SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1393-L1416) · [combat](../../../raw/current/scripts/components/combat.lua#L1168-L1188)

## 投掷炸弹与雪球（throwables）

- **R08 · P1｜睡袋没有动态爆炸组件。** 摘要、L21–24、57、87、110 和索引 L37 把三枚炸弹归为动态挂 `explosive`。南瓜和亮茄才走该链，也可由虫 Boss 啃食入口触发；睡袋 OnHit 移除整叠，只生成一朵 sleepcloud 和一个爆裂 FX。睡袋在熔炉分支也不挂 weapon，普通世界的 combat 投掷链须限定适用模式。 [页面](../../../wiki/reference/prefabs/throwables.md#L19-L35) · [示例前提](../../../wiki/reference/prefabs/throwables.md#L83-L89) · [睡袋命中](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L21) · [熔炉／普通分支](../../../raw/current/scripts/prefabs/sleepbomb.lua#L95-L128) · [南瓜啃食](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L23-L45)
- **R09 · P1｜雪球确实走冻结链。** 摘要、L23、114 和索引 L37 的「不冻结」与组件消费者相反。以有效命中目标为中心扩散，范围内可用的 freezable 会收到 `AddColdness(0.2)`，累积达到抗性后可冻结。玩家标签排除的是扩散效果，前面的唤醒／0 伤事件按各自条件处理。组件未设置正的 `addwetness`；建议写降温、冷度和灭火，将融化增湿留在腐败段。 [页面](../../../wiki/reference/prefabs/throwables.md#L108-L115) · [命中分支](../../../raw/current/scripts/prefabs/snowball_item.lua#L28-L49) · [配置](../../../raw/current/scripts/prefabs/snowball_item.lua#L231-L237) · [组件消费](../../../raw/current/scripts/components/wateryprotection.lua#L21-L46) · [冻结阈值](../../../raw/current/scripts/components/freezable.lua#L137-L157)
- **R10 · P2｜获取条件。** L43 的 100% 蓝图限定悲惨的毒菌蟾蜍 `toadstool_dark`，普通共享表没有该项。L44 先按实际吐出的糖果数量（可含装扮加成）计算余数，再对大于零的余数掷一次，成功才生成对应数量。 [页面](../../../wiki/reference/prefabs/throwables.md#L39-L46) · [毒菌蟾蜍两表](../../../raw/current/scripts/prefabs/toadstool.lua#L88-L140) · [游戏名](../../../raw/current/scripts/languages/chinese_s.po#L350435-L350437) · [猪王](../../../raw/current/scripts/prefabs/pigking.lua#L95-L119) · [鱼人王](../../../raw/current/scripts/prefabs/mermking.lua#L161-L185)
- **R11 · P2｜爆炸与击退半径读取时机。** L106 宣称改 `PUMPKIN_BOMB_RANGE` 会同时修改两个半径。模块加载时先缓存击退事件的 radius，爆炸查询范围才在 ConfigureExplosive 现读。模块已经加载后改 TUNING 只影响查询范围。 [页面](../../../wiki/reference/prefabs/throwables.md#L104-L106) · [缓存与现读](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L15-L40) · [半径消费者](../../../raw/current/scripts/components/explosive.lua#L116-L142)
- **R12 · P2｜玩家投掷的伤害与击退分流。** L23、57、112 的范围伤害描述缺玩家例外。有效 player／possessedbody 投掷者设置 `pvpattacker`，其他玩家跳过伤害；南瓜的 `forcepvp` 仍让他们进击退分支，前提是进入范围并通过对应实体判定。亮茄也有同类玩家伤害门禁；其「物理伤害为 0」限定生命值结算，仍会走建筑工作量。 [页面](../../../wiki/reference/prefabs/throwables.md#L54-L58) · [南瓜设置](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L15-L41) · [亮茄设置](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L15-L34) · [目标与伤害分流](../../../raw/current/scripts/components/explosive.lua#L61-L120)

## 远程手持武器族（ranged-weapons）

- **R13 · P1｜吹箭示例重复派发受击。** L114–125 给 100 伤害吹箭额外推 `attacked(damage=0)`，命中链已调用 `GetAttacked_Internal`，普通有效受击会推原事件，再推一遍可能重复触发监听，示例也没在旧回调后确认目标仍有效。用自定义事件或明确的单次效果展示包装，保留原回调。L104–105 同时区分构造期写入的伤害／射程和命中时读取的催眠参数。 [页面](../../../wiki/reference/prefabs/ranged-weapons.md#L100-L128) · [吹箭伤害赋值](../../../raw/current/scripts/prefabs/blowdart.lua#L235-L250) · [命中及回调顺序](../../../raw/current/scripts/components/combat.lua#L1212-L1225) · [原事件](../../../raw/current/scripts/components/combat.lua#L715-L723)
- **R14 · P2｜耗弹例外。** L39、49、73、136 和索引 L38 的「每发扣 1」需与 L95 统一。拟态皮筋成功掷点可免耗弹，普通耗弹拆一发，槽位默认 1，特殊副手弹用槽 2。999ex 的改动是取消普通堆叠限制。 [页面](../../../wiki/reference/prefabs/ranged-weapons.md#L130-L140) · [免耗与扣弹槽](../../../raw/current/scripts/prefabs/slingshot.lua#L361-L395) · [副手分流](../../../raw/current/scripts/prefabs/slingshot.lua#L785-L815)
- **R15 · P2｜射程与蓄力。** L38 的 `k²` 是传给弹体的归一化系数，弹体据此插值伤害和速度倍率（默认最大分别 ×2、×1.25），不是直接把伤害乘 `k²`。L128、135 将 `SetRange(10,14)` 写成区间和装弹取得的射程；弹弓构造时写入攻击／命中距离，空舱只清投射物，嚎弹炮才在卸弹时清距离。 [页面](../../../wiki/reference/prefabs/ranged-weapons.md#L128-L136) · [蓄力系数](../../../raw/current/scripts/prefabs/slingshot.lua#L656-L672) · [弹体消费](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1143-L1155) · [弹弓卸弹](../../../raw/current/scripts/prefabs/slingshot.lua#L397-L412) · [弹弓构造](../../../raw/current/scripts/prefabs/slingshot.lua#L547-L552) · [嚎弹炮](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80-L96)
- **R16 · P2｜蚊子炸弹玩家筛选。** L43、138 的 PVP 容易被读成世界 PvP 开关。这里的布尔量只由有效投掷者是否有 `player` 标签决定，开启该分支时其他玩家排除、自身仍是候选；生成蚊子的索敌另走盟友与麝香筛选。 [页面](../../../wiki/reference/prefabs/ranged-weapons.md#L133-L140) · [范围查询](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L99) · [分支入口](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L154-L165)
- **R17 · P2｜明确注册计数与弹药口径。** L6–7、28–43、60、144 及索引 L38 用「十四注册」，但 `slingshotmodkit` 已列在 aliases 并写了配方，只有其构造入口缺在注册表。明确它是第 15 个主要覆盖注册，或把它作为外部关联、从本页负责清单移出。L91 的 18 类包含独立的 `trinket_1`，它不以 slingshotammo 物品工厂注册、仍有 `trinket_1_proj`，应补全枚举并分别说明。 [页面](../../../wiki/reference/prefabs/ranged-weapons.md#L26-L45) · [作业包工厂](../../../raw/current/scripts/prefabs/slingshotmodkit.lua) · [弹药依赖](../../../raw/current/scripts/prefabs/slingshot.lua#L58-L80) · [弹体工厂](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1557-L1585)

## 战斗圆盾（wathgrithr_shield）

- **R18 · P1｜常规攻击可招架。** L56、122 的「非 stun 不可招架」漏掉 `nil`。只有非空且不等于 `stun` 的 stimuli 或 groundspike 才在首个门禁被拒绝；`nil` 的普通物理攻击继续进角度判定。同步流程与陷阱。 [页面](../../../wiki/reference/prefabs/wathgrithr_shield.md#L54-L58) · [招架门禁](../../../raw/current/scripts/components/parryweapon.lua#L39-L61)
- **R19 · P2｜1 秒是传入时长。** L6、48、55–58、122 把 1 秒写成总窗口；实际 `parry_pre` 同步执行动作时已装重定向闭包，前摇正常结束才把完整的 duration 交给 `parry_idle` 启动计时。技能倍率作用在这项传入时长。回充也应写为进度低于 70% 才推进至 70%，避免摘要暗示回落。 [页面](../../../wiki/reference/prefabs/wathgrithr_shield.md#L45-L58) · [传入时长与回充](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L85-L101) · [前摇与交接](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5767-L5789) · [计时](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5816-L5848)
- **R20 · P2｜85% 是该盾的护甲吸收率。** L6、22、57 和索引 L44 可区分单盾与并穿护甲。多件共穿时取最高吸收率，再按各件吸收率比例分摊磨损；也不应泛称所有来袭伤害都按 85% 吸收。L124 的反击说明再补技能 3 前提，清空发生在 weapon 的 onattack 回调。 [页面](../../../wiki/reference/prefabs/wathgrithr_shield.md#L18-L23) · [护甲分摊](../../../raw/current/scripts/components/inventory.lua#L408-L452) · [反击](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L103-L125)

## 事件武器族（event-weapons）

- **R21 · P1｜冲锋骑枪的骑乘描述写反。** L21 的「骑乘冲锋」会误导玩家在坐骑上使用，JOUST 点动作明确排除骑乘并要求地面落点。L48 的「全角色可右键」应改为无专属角色限制、满足动作门禁才可发动。命中也区分双方对冲碰撞与普通攻击／击退。 [页面](../../../wiki/reference/prefabs/event-weapons.md#L18-L23) · [动作收集](../../../raw/current/scripts/componentactions.lua#L2128-L2132) · [动作执行](../../../raw/current/scripts/actions.lua#L6683-L6697) · [碰撞分支](../../../raw/current/scripts/components/joustsource.lua#L103-L121)
- **R22 · P2｜区分奔雷矛普攻与突刺。** L32、43、49 写的 59.5 是普通攻击基值。突刺期间暂把 weapon 伤害切成 `34×2=68`、`attackwear=0`，以 electric stimuli 走命中后恢复原配置；充能版仍有自身位面组件。普攻的电属性由湿度／突刺窗口决定，非免疫目标按 `1+0.5×GetWetMultiplier()` 结算，「湿就 ×1.5」只对应满湿倍率。 [页面](../../../wiki/reference/prefabs/event-weapons.md#L41-L50) · [电属性门禁](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L83-L85) · [突刺配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L474-L498) · [临时覆盖](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L53-L68) · [恢复](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L119-L127) · [湿度系数](../../../raw/current/scripts/components/combat.lua#L1198-L1212)
- **R23 · P2｜充能矛的施放与修复门禁。** L45、49、58 和摘要应说明技能 4 与满充共同控制准星，装备时可重新进入冷却。修复要求充能形态并通过持有者 `IsValidVictim`，每次有效命中回 4、单次突刺最多计两次。L22、59、120 所谓「不能直接装备」限定普通玩家路径，读档有技能标签对应角色的宽容逻辑。 [页面](../../../wiki/reference/prefabs/event-weapons.md#L43-L59) · [准星门禁](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L62-L64) · [装备冷却](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L100-L109) · [修复门禁](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L184-L195) · [读档门禁](../../../raw/current/scripts/components/equippable.lua#L147-L173)
- **R24 · P2｜Forge 证据边界。** L86–88、132 把 `requireeventfile` 入口写成已确认的编译数据及 equippable 所在位置。公开脚本只能证明初始化委托给事件文件加载器，本快照不含对应实现。可保留角色起始物品、解锁映射和未核验的组件缺口，主世界取得途径也限定为可读脚本检索结论。 [页面](../../../wiki/reference/prefabs/event-weapons.md#L82-L88) · [加载器](../../../raw/current/scripts/entityscript.lua#L76-L87) · [长矛入口](../../../raw/current/scripts/prefabs/spear_lance.lua#L76)

## 索引与覆盖

六页在武器叶索引均有登记，页尾均返回该索引。索引 L35–45 的机制摘要随上面对应条目同步。主要可见对象已按同版本简中键定向核对，未发现需单独修订的主要名称。此前已建的阴郁回旋镖关联 FX 页仍负责附属注册，本轮按主体页的跨页断言比对，不展开新一轮 FX 检视。 [武器叶索引](../../../wiki/reference/prefabs/weapons-combat-index.md#L31-L45)

本次只做静态内容复审，未运行游戏或重跑校验脚本。实机投掷、招架时序、Forge 事件实现及世界布点频率仍按相应缺口处理。下一批可转到法杖与角色道具 11 页。

## 处置结果（2026-11-07）

24 项逐条对照 raw/current 源码复核，全部成立并修复 6 页＋武器叶索引（7 文件）。

- **R01 采纳**：摘要/L20–22/导航/索引改“命中或脱靶都返航、掷出者已死则落地”；CATCH 补双标签与“仅背包开启才 Equip/GiveItem、HANDS 空闲才装备”。
- **R02 采纳**：数值表改“判定半径＝目标物理半径＋hitdist”，陷阱拆三条脱靶路径（距离、越过、卡滞/休眠），射程兜底补“combat 与 weapon.attackrange 都在”的前提，注明 CheckTarget 不查死亡。
- **R03 采纳**：删“无世界生成点”，改“字面检索只覆盖直接生成”；获取表补 worldgen Sandbox 抽取与任务区登记，保留落点/频率未核验。
- **R04 采纳**：改“构造与套装切换时缓存 max_projectiles，计数只比缓存”；示例与调整说明改为只影响新实例、已有实例须改字段并手动调计数回调。
- **R05 采纳**：数值/导航/陷阱/索引统一“掷出与放电近战各磨 1”，注明默认值受磨损与效率修正；撤回“85 耐久＝85 次掷出”。
- **R06 采纳**：返航尺寸时长改 `BOOMERANG_DISTANCE/RETURN_SPEED`（默认约 1.6 秒），去程用 `ATTACK_DIST/LAUNCH_SPEED`，并注明提前抵达/越过/失效即移除。
- **R07 采纳**：对照表补“SG 按 thrown 标签分流、combat 按组件分流”；附注与陷阱改“本体无 thrown 标签，落普通 attack 状态”。
- **R08 采纳**：摘要/优先使用/副作用/边界/陷阱/索引统一“只有南瓜与亮茄动态挂 explosive”；睡袋只生成 burst＋sleepcloud，熔炉分支不挂 weapon 并限定普通世界投掷链。
- **R09 采纳**：删“不冻结”，改“命中经 wateryprotection 以目标为中心扩散，非 player 且有 freezable 者 AddColdness(0.2)，累积到抗性冻结”；唤醒/0 伤事件与融化增湿分链说明。
- **R10 采纳**：蓝图限定“悲惨的毒菌蟾蜍（toadstool_dark）”；糖果按实际数量（含装扮加成）取 %8 余数，余数>0 才掷一次 20% 并生成该数量。
- **R11 采纳**：调整说明区分“爆炸查询半径命中时现读”与“击退 radius 模块加载时缓存”，加载后改键只影响前者。
- **R12 采纳**：explosive 行与陷阱补玩家分流：ispvp 由投掷者标签决定，南瓜 forcepvp 仍进击退、亮茄无击退数据时其他玩家不结算；建筑工作量不受 0 物理伤害影响。
- **R13 采纳**：示例改推自定义事件 `mydart_hit` 并先判 IsValid，避免与命中链已推的 attacked 重复；前提补“伤害/射程构造期写入、催眠参数命中时现读”。
- **R14 采纳**：999ex 行改“默认每发扣 1、拟态皮筋可免”；容器行与陷阱补“槽位默认 1、2ex 副手弹扣槽 2”。
- **R15 采纳**：slingshotex 行改“k 为 0–1 蓄力，SetChargedMultiplier(k*k) 后按 ×2 伤害/×1.25 速度上限插值”；陷阱改“弹弓构造期写入射程、空舱只清 projectile，嚎弹炮卸弹才清射程”。
- **R16 采纳**：蚊弹行与陷阱改“ispvp 只看投掷者是否带 player 标签，不是世界 PvP 开关；为真时排除其他玩家、自身仍可被结算”。
- **R17 采纳（按第 15 个注册处理）**：摘要改十五注册，注册表增 slingshotmodkit 行；弹药改为“表 18 项：17 种物品弹药＋trinket_1（no_inv_item，只注册 trinket_1_proj）”。
- **R18 采纳**：流程与陷阱改“非空且非 stun 的 stimuli 与地刺攻击不可招架，nil 普通物攻照常判角度”。
- **R19 采纳**：摘要/数值/流程改“传入 1 秒，parry_pre 结束才交 parry_idle 计时”；回充保持“低于 70% 才推进至 70%”。
- **R20 采纳**：摘要/副作用/流程/陷阱区分单盾 85% 与多件共穿（取最高吸收率、按各件吸收率分摊磨损）；反击陷阱补技能 3 前提与 OnAttackFn 清空时机。
- **R21 采纳**：优先使用与 joustsource 行改“地面落点冲锋，JOUST 明确排除骑乘”，右键条件改为 right＋未骑乘＋地面点＋CanJoust。
- **R22 采纳**：奔雷矛行改“电属性＝1+0.5×目标湿度，满湿才 ×1.5，突刺窗口亦电”；数值表注明 59.5 是普攻基值、突刺期间 weapon 临时 68 且 attackwear=0，命中后恢复。
- **R23 采纳**：aoetargeting 门禁补“满充＋skill_4”，装备时冷却不足会重新 Discharge；修复行补 charged 与 IsValidVictim；装备陷阱补读档宽容（IsRestricted_FromLoad）。
- **R24 部分采纳**：保留委托链与可确证部分，把“确证编译数据”降为“公开脚本只能证明这次委托，文件不在快照内”，结论改为“只能由委托推断 equippable 在编译事件文件内”。

校验：6 页＋索引 validate_page 通过（阴郁回旋镖 24563B、投掷族 24545B 接近 standard 上限），validate_source_links --strict 0 error 0 warning，validate_source_hashes record 6 页，--check-indexes 通过；未运行游戏、未改 raw、未提交推送。

## 复检补正（2026-11-07）

收口复检补两处同源残句：阴郁回旋镖的把“弹体注销即回充”改为“计数低于缓存 `max_projectiles` 才回充”，并写明套装上限回退后须降到新阈值以下；投掷族补“投掷者为玩家时其他玩家跳过生命值伤害，南瓜 `forcepvp` 仍进击退、亮茄无击退数据则整体不结算”。7 文件 validate_page 与 strict 仍 0 错 0 警。

## 用户复核补正（2026-11-07）

按复核意见修 6 页：回旋镖的「鬼魂落地」限定到命中分支、接取主体改为被登记观察的目标；阴郁回旋镖补套装切换时重读上限、回充条件改为降到当前上限以下；投掷族补睡袋熔炉不挂 weapon、湿冷改降温/灭火/冷度扩散、南瓜玩家击退与伤害分流；远程族示例补 IsValid、弹舱链补拟态皮筋免扣、射程口径与 15 注册/18 项弹药；圆盾 70% 推进与前摇后窗口；事件武器 skill_4 满充门禁与 Forge 待核验措辞。7 文件校验全绿。
