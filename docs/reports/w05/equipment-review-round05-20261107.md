# 装备 Prefab 第五轮检视（2026-11-07）

范围为「法杖与角色道具」叶索引登记的 11 页。依据 `raw/current/scripts` 与 Steam buildid 26335394，复核原版构造、动作与组件消费端、调值及示例。11 页均在该叶索引登记，均有返回入口。本轮只写检视报告，未改 Wiki 页或索引，未运行游戏及校验脚本。[叶索引](../../../wiki/reference/prefabs/staffs-character-index.md) · [BUILD](../../../raw/current/BUILD.txt)

结论为需修复，计 7 项 P1、20 项 P2。每项将同一问题的页内和索引修改面合并记录。含已核验无 P1/P2 的两页，装备初审累计 52／69 页。

## 基础法杖族（staffs）

- **R01 · P1｜懒人魔杖在实际位移前结算。** L90 写成 0.25 秒后先位移再扣精神与次数。首次校验通过后，`Blink` 安排延迟任务，随即调用 `onblinkfn` 并推 `onblink`。延迟任务重新校验落点与屏障，失败时仍可有已付出的消耗。应分开写首次校验、消耗和位移。 [页面](../../../wiki/reference/prefabs/staffs.md#L88-L90) · [位移复验](../../../raw/current/scripts/components/blinkstaff.lua#L36-L58) · [消耗时机](../../../raw/current/scripts/components/blinkstaff.lua#L61-L96) · [法杖回调](../../../raw/current/scripts/prefabs/staff_common.lua#L483-L500)
- **R02 · P1｜法杖复用案例漏掉宿主组件与清理副作用。** L110–111 把脚本资产和依赖列成前提后直接推荐 `stafffns.<gem>(inst, true)` 和 `cleargemstaff`。实际先 `require` 取 `fns`，并初始化宿主。绿分支会解引用 `itemmimicconfig`，蓝分支读冷度等宿主字段，清除函数也要求 `itemmimicconfig`，还会重建国王魔杖武器与作祟。明确此段以国王魔杖为实现参照，新宿主按所选分色核对组件、字段和撤销动作。L98 的「取出」也应写为嵌入新宝石时返还旧宝石。 [页面](../../../wiki/reference/prefabs/staffs.md#L96-L112) · [导入](../../../raw/current/scripts/prefabs/king_cane.lua#L18-L27) · [宿主换宝石](../../../raw/current/scripts/prefabs/king_cane.lua#L148-L186) · [绿分支与清除](../../../raw/current/scripts/prefabs/staff_common.lua#L770-L786) · [清除与导出](../../../raw/current/scripts/prefabs/staff_common.lua#L887-L914)
- **R03 · P2｜传送目的地与时间分支。** L82 把 `GetMinigame()` 译成「小比参与者」，把水栖与覆盖位置混进同一目标优先级。先选小游戏实体、覆盖目的实体或拴系实体。未选中且目标非水栖才找传送台，计算落点时 `GetDestPosition()` 又优先于上述实体。水栖读 locomotor 的 `IsAquatic()`。3+1 秒限定玩家路径，其他目标走即时继续和收尾。 [页面](../../../wiki/reference/prefabs/staffs.md#L80-L86) · [落点与等待](../../../raw/current/scripts/prefabs/staff_common.lua#L335-L366) · [目标选择](../../../raw/current/scripts/prefabs/staff_common.lua#L418-L449)
- **R04 · P2｜拆解产出不是简单按比例。** L94 可补每类正量材料向上取整且至少返一份。燃料／护甲百分比只用于带 `inventoryitem` 的目标，`decon_ignores_finiteuses` 可跳过有限次数。拟态目标另走变敌分支，应给修改回收逻辑的读者保留这一例外。 [页面](../../../wiki/reference/prefabs/staffs.md#L92-L94) · [拆解实现](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)

## 法杖补充族（staffs-more）

- **R05 · P1｜亮茄魔杖的 0 普通伤害是配置值。** L125 声称直接 `weapon:SetDamage` 不改变实际输出。原版在 `SetupComponents` 将武器设为 0，命中时 combat 仍走武器普通伤害及独立位面伤害计算。可用有效实例改前者，修改 TUNING 中位面参数则在构造时写入组件。L66、166 的「恒 0」也应限定原版默认。 [页面](../../../wiki/reference/prefabs/staffs-more.md#L123-L136) · [武器配置](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L87-L97) · [位面配置](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L209-L225) · [攻击结算](../../../raw/current/scripts/components/combat.lua#L1197-L1225)
- **R06 · P1｜静电式升级示例缺动作入口。** L142–159 只给两侧组件写同一自定义字符串。动作收集遍历 `UPGRADETYPES`，要求对应材料、操作者与目标标签，服务端还核对目标可升级及 `<type>_upgradeuser`。若保留可复制示例，需补类型登记、玩家授权和阶段条件，并说明成功升级由 `Upgradeable:Upgrade` 消耗材料。也可缩为原版配对说明。 [页面](../../../wiki/reference/prefabs/staffs-more.md#L140-L159) · [动作收集](../../../raw/current/scripts/componentactions.lua#L1889-L1898) · [服务端执行](../../../raw/current/scripts/actions.lua#L4056-L4071) · [操作者门禁](../../../raw/current/scripts/components/upgrader.lua#L21-L29) · [阶段与消耗](../../../raw/current/scripts/components/upgradeable.lua#L60-L85)
- **R07 · P2｜国王魔杖消耗和依赖。** L90、168 和索引 L18 的「免费」应限定无次数消耗且未腐化时不按宝石放电。精神消耗与腐化累积仍按各自事件处理。冷却列区分紫、橙、黄、绿的施法分支和红蓝的弹体攻击，避免把列在 TUNING 的两个 0 当作实际调用。L173 的「不预载 stafflight 系列」也不成立，合并依赖时只跳过 opal，yellow 仍依赖 `stafflight`。基础法杖页 L69 已能链接本页，无须保留「Wiki 待建」。 [页面](../../../wiki/reference/prefabs/staffs-more.md#L85-L97) · [陷阱](../../../wiki/reference/prefabs/staffs-more.md#L165-L173) · [国王魔杖依赖](../../../raw/current/scripts/prefabs/king_cane.lua#L20-L29) · [依赖表](../../../raw/current/scripts/prefabs/staff_common.lua#L31-L47) · [召星消耗](../../../raw/current/scripts/prefabs/staff_common.lua#L788-L805) · [腐化监听](../../../raw/current/scripts/prefabs/king_cane.lua#L349-L357)
- **R08 · P2｜龙卷风先查 PvP 门禁。** L79 只写对玩家伤害乘修正。玩家放出的风在 PvP 关闭时排除玩家和附身躯壳的伤害分支。通过该门禁后，修正只对带 `player` 标签的目标使用。工作分支是伤害分支的 `elseif`，并限定支持的工作动作。 [页面](../../../wiki/reference/prefabs/staffs-more.md#L73-L83) · [SG 消费](../../../raw/current/scripts/stategraphs/SGtornado.lua#L58-L93)

## 唤星者魔杖与唤月者魔杖（yellowstaff、opalstaff）

本轮未发现 P1/P2。核对了普通黄色／月色工厂、端分界、施法与作祟共用的召唤函数、次数与精神分支、月台转换及皮肤传递。两个物品页及其生成物入口可按原用途查找。 [唤星者魔杖](../../../wiki/reference/prefabs/yellowstaff.md) · [唤月者魔杖](../../../wiki/reference/prefabs/opalstaff.md) · [工厂](../../../raw/current/scripts/prefabs/staff.lua#L204-L229) · [月杖工厂](../../../raw/current/scripts/prefabs/staff.lua#L272-L299) · [共享回调](../../../raw/current/scripts/prefabs/staff_common.lua#L788-L823) · [月台](../../../raw/current/scripts/prefabs/moonbase.lua#L469-L488)

## 一角鲸的角（gnarwail_horn）

- **R09 · P2｜水花点和成浪点相差 1.5。** L43、70、81、108 把两侧 `wp1/wp2` 当作浪的最终落点。角传入选点横向 ±1.5，`SpawnAttackWaves` 再沿浪向前推 1.5，用最终坐标判定 `not IsPassableAtPoint`。水花在原 `wp`，目标点及水花点的判定与成浪门禁应分开写。 [页面](../../../wiki/reference/prefabs/gnarwail_horn.md#L35-L44) · [角施法](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L41-L70) · [浪位置](../../../raw/current/scripts/ocean_util.lua#L29-L75)

## 哑铃族（dumbbells）

- **R10 · P2｜投掷磨损按进入武器回调的目标结算。** L48、68、121 的「每个被 `DoAttack` 的目标」过宽，`DoAttack` 可在命中检查失败时早退，`weapon:OnAttack` 还读装备磨损修饰及持有者效率。把倍率写成默认值，并保留冰铃向投掷者施冷的独立分支。索引 L27 的等式同样限定基础值。 [页面](../../../wiki/reference/prefabs/dumbbells.md#L119-L126) · [落地结算](../../../raw/current/scripts/prefabs/dumbbells.lua#L139-L185) · [早退](../../../raw/current/scripts/components/combat.lua#L1147-L1154) · [磨损消费者](../../../raw/current/scripts/components/weapon.lua#L105-L119)
- **R11 · P2｜获取结论超出检索范围。** L56 用静态 `SpawnPrefab("dumbbell…")` 零命中断言无世界生成或掉落。该搜索不覆盖数据表及拼名调用。已确认七条合成配方足以用于查获取方式，如要排除其他来源则补数据表与动态选名证据。 [页面](../../../wiki/reference/prefabs/dumbbells.md#L55-L62)

## 旺达怀表族（pocketwatch）

- **R12 · P1｜按示例开放倒走表仍会报错。** L117、175 保证没历史只返回 `WARP_NO_POINTS_LEFT`，但 `Warp_DoCastSpell` 首行无守卫调用 `doer.components.positionalwarp:GetHistoryPosition(false)`，组件缺失会先报错。L122–149 只包不老表，应明确该示例只开放已处理的链，或同时补倒走表守卫或完整历史接线。L172 的风险说明随之更新。 [页面](../../../wiki/reference/prefabs/pocketwatch.md#L114-L149) · [陷阱](../../../wiki/reference/prefabs/pocketwatch.md#L169-L176) · [倒走实现](../../../raw/current/scripts/prefabs/pocketwatch.lua#L368-L381) · [旺达挂载](../../../raw/current/scripts/prefabs/wanda.lua#L393-L401)
- **R13 · P2｜标记不放电，开门会重启冷却。** L23 的「所有施法都 Discharge」不含首次标记和失败分支。L39、176 把两种替换的充能混在一起。裂缝表从溯源表制作或镶宝石时才经 `onPreBuilt` 复制原充能，成功开门后生成的新溯源表复制标记、重新放电 480 秒。L96–97 的「五表＋裂缝表」改为含裂缝表在内的五表。 [页面](../../../wiki/reference/prefabs/pocketwatch.md#L19-L41) · [溯源分支](../../../raw/current/scripts/prefabs/pocketwatch.lua#L226-L246) · [镶宝石](../../../raw/current/scripts/prefabs/pocketwatch.lua#L260-L274) · [开门与制作](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L42-L109)
- **R14 · P2｜乘门豁免和本族装备槽。** L40、66、88 把理智豁免写成施法者身份。入口实际按 `pocketwatchcaster` 或 `nowormholesanityloss` 标签豁免，且要求有 sanity。索引 L28 的 HANDS 限定为警钟，其余保留物品栏施法和附属实体口径。 [页面](../../../wiki/reference/prefabs/pocketwatch.md#L37-L44) · [乘门回调](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L222-L229) · [索引](../../../wiki/reference/prefabs/staffs-character-index.md#L28)
- **R15 · P2｜警钟不是每击读取伤害 TUNING。** L158–159 宣称改键会立即影响已有警钟。普通伤害保存在 weapon，构造时设满燃料档，之后按燃料档变更回写。已有空燃料实例需重新触发空档写入或直接改组件。治疗冷却则确在成功施法时读取。 [页面](../../../wiki/reference/prefabs/pocketwatch.md#L156-L165) · [警钟档位](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L63-L95) · [初始化](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L165-L180) · [不老表](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L42)

## 强抢袋（wortox-nabbag）

- **R16 · P2｜区分实时读取、构造缓存和计数容器。** L41 的磨损修饰为 0 只在初始化与首档成立，`UpdateStats` 会回写 0／0.5／1。L50 的全数值即时生效不包含 200 上限和 NET 的 10 点，二者构造时写入组件。L52、60、69 及索引 L29 的「背包」应明确指角色主物品栏占用格，额外容器不参与，鼠标手持物只补算灵魂。 [页面](../../../wiki/reference/prefabs/wortox-nabbag.md#L40-L69) · [磨损及计数](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L22-L96) · [构造](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L203-L217) · [格位迭代](../../../raw/current/scripts/components/inventory.lua#L707-L711)
- **R17 · P2｜50 点是扣耐久上限。** L22、99、102 的「每多抓一件扣 1」遗漏扣满本次上限后仍会尝试拾取，直到物件数上限或循环结束。调用 `PICKUP.fn` 后不检查返回值才递减计数，耐久用尽导致强抢袋失效会立即中断。50 由总上限 200 计算，不按当前满度取值。 [页面](../../../wiki/reference/prefabs/wortox-nabbag.md#L96-L105) · [拾取循环](../../../raw/current/scripts/components/nabbag.lua#L97-L130)

## 角色与活动专用工具（role-tools）

- **R18 · P1｜迅捷气球的扩档示例不会按预期工作。** L122–133 称 `SetSections` 写死为 3，但原版已按 `#balloon_speeds - 1` 计算。文件级局部变量持有原表，示例替换全局表不能更新已加载脚本的引用，只重设燃料分段也不会更新初始移速、图标或符号。可保留修改原表既有四档的示例，并注明初始值及后续回调的读取时点。真正加档另需处理图标／动画符号。 [页面](../../../wiki/reference/prefabs/role-tools.md#L120-L134) · [表引用](../../../raw/current/scripts/prefabs/balloonspeed.lua#L24-L28) · [档位消费](../../../raw/current/scripts/prefabs/balloonspeed.lua#L88-L107) · [构造](../../../raw/current/scripts/prefabs/balloonspeed.lua#L161-L179)
- **R19 · P2｜鱼人标签示例混淆限装与伤害。** L103–117 的猫浣示例添加 `merm_npc` 前已可通过 `IsRestricted` 的非玩家早退，移除标签也不会让 NPC 新装备被拒。该标签同时让武器回调走鱼人伤害分支，并要求宿主有 combat。若说明放行玩家，可限定具体 mod 角色并提示该标签的额外消费者。 [页面](../../../wiki/reference/prefabs/role-tools.md#L101-L117) · [装备门禁](../../../raw/current/scripts/components/equippable.lua#L147-L162) · [伤害函数](../../../raw/current/scripts/prefabs/merm_tool.lua#L40-L45)
- **R20 · P2｜清洁扫把还能去掉现有皮肤。** L61、147 用「只循环已拥有皮肤」概括施法，但允许回基础外观的对象在找不到下一款、已有皮肤时仍可通过。禁止基础外观的目标才在无候选时提前失败。 [页面](../../../wiki/reference/prefabs/role-tools.md#L138-L149) · [预检](../../../raw/current/scripts/prefabs/reskin_tool.lua#L289-L327)
- **R21 · P2｜气球掉落和飞走分支。** L23 与 L149 对燃料耗尽后掉落的说法相反，`OnDropped` 仅对未耗尽实例安排飞走。L71 与派对气球页 L23、46、65 的「被打即掉」均应加共享条件，排除重定向、无攻击者以及归属自身的攻击。 [角色工具页](../../../wiki/reference/prefabs/role-tools.md#L19-L23) · [派对气球](../../../wiki/reference/prefabs/event-and-carnival-gear.md#L44-L48) · [飞走门禁](../../../raw/current/scripts/prefabs/balloonspeed.lua#L73-L85) · [掉落监听](../../../raw/current/scripts/prefabs/balloons_common.lua#L93-L112) · [通用门禁](../../../raw/current/scripts/componentutil.lua#L111-L122)
- **R22 · P2｜26 注册含不装备的地图代理。** L77、153 将全部 26 注册算为装备，验证记录的 BODY×5／HANDS×21 也与构造表冲突。四件身体枕头和八套服装为 BODY，十三件为 HANDS，剩下一个 `balloon_mapicon` 是不装备的地图代理。同文件枕头命中特效继续归 FX 页。 [页面](../../../wiki/reference/prefabs/role-tools.md#L63-L79) · [验证记录](../../../wiki/reference/prefabs/role-tools.md#L151-L155) · [地图代理](../../../raw/current/scripts/prefabs/balloonspeed.lua#L186-L225)

## 角色专属装备补遗（character-gear-extra）

- **R23 · P1｜电刑机部署不扣 100 用。** L23 把上限写成部署成本，L59 才写了攻击事件扣次。部署先过 `IsFlyingPermittedFromPoint`，随后生成或复用无人机；收到 `ms_drone_zap_fired` 才扣控制器 2 点，技能 2 降为 1 点。配方可补 `nameoverride="wx78_drone_zap"`，方便玩家区分制作界面的「电刑机」和物品实体「电刑机控制器」。 [页面](../../../wiki/reference/prefabs/character-gear-extra.md#L19-L24) · [部署与扣次](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L259-L282) · [耐久初始化](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L378-L395) · [配方](../../../raw/current/scripts/recipes.lua#L788) · [译名](../../../raw/current/scripts/languages/chinese_s.po#L352679-L352687)
- **R24 · P2｜投泥带是施法者冷却，非世界全局。** L23、49、67 和索引 L31 需要区分配方技能、施法标签和装备。工厂未设 `restrictedtag`，施法由对应 spelltype 的用户标签授权。施法时只给持有者加 debuff，并放电其 `Inventory:FindItems` 可见的同类物品。拾取回调只会压低高于 debuff 进度的电量，`CanCast` 只拦施法。 [页面](../../../wiki/reference/prefabs/character-gear-extra.md#L47-L53) · [同步与门禁](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L46-L85) · [拾取](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L147-L163) · [装备初始化](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L203-L231) · [技能授权](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L455-L468) · [搜索范围](../../../raw/current/scripts/components/inventory.lua#L638-L664)
- **R25 · P2｜投泥带附属注册名和月态条件。** L33 的 `_cast_debuff`、`_chargedfx` 排在 `wurt_terraform_projectile` 后会被读成同前缀。写出实际的 `wurt_terraform_cast_debuff` 及影、月两条 `wurt_swampitem_*_chargedfx`。L51 明确月态对有效持有者的范围内鱼人跟随者上 buff，有 `DoLunarMutation` 方法才调用突变。 [页面](../../../wiki/reference/prefabs/character-gear-extra.md#L28-L35) · [注册](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L463-L468) · [月态判定](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L261-L289)

## Boss 与狂欢节装备（event-and-carnival-gear）

派对气球掉落门禁与迅捷气球共用，见 R21。

- **R26 · P2｜盾自损和推杆清理分别写实际触发。** 恐怖盾牌 L23、40 应写匹配武器的 `onattackother` 事件扣护甲，事件发生在实际伤害结算前，伤害是否扣到目标并非扣盾条件。推杆 L58 的不存档源自 tee 设置的 `persists=false`，小游戏结束由设施清理链 Remove 球杆。两条机制分别说明。 [盾牌段](../../../wiki/reference/prefabs/event-and-carnival-gear.md#L38-L42) · [自损回调](../../../raw/current/scripts/prefabs/shieldofterror.lua#L89-L93) · [事件时机](../../../raw/current/scripts/components/combat.lua#L1147-L1175) · [推杆段](../../../wiki/reference/prefabs/event-and-carnival-gear.md#L56-L60) · [设施清理](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L136-L149)
- **R27 · P2｜英文别名漏了三个实际显示名。** L8 的 `Balloon Party`、`Bug Repellent Spray`、`Carnival Golf Club` 不对应此快照的 NAMES。补 `Party Balloon`、`Embalming Spritz`、`Putter`。如保留历史搜索词，保持与官方英文名的区别。中文三项已与简中词条一致。 [页面](../../../wiki/reference/prefabs/event-and-carnival-gear.md#L6-L11) · [派对气球](../../../raw/current/scripts/languages/chinese_s.po#L339000-L339002) · [防腐喷雾](../../../raw/current/scripts/languages/chinese_s.po#L339365-L339367) · [推杆](../../../raw/current/scripts/languages/chinese_s.po#L340300-L340302)

## 处置结果（2026-11-07）

27 项逐条对照 raw/current 源码复核，全部成立并修复 9 页＋法杖叶索引（10 文件）。

- **R01 采纳**：闪现改为“先校验→立即扣精神/次数→0.25 秒后 OnBlinked 再校验落点与屏障，失败只留消耗”。
- **R02 采纳**：复用节补 `require("prefabs/staff_common")`、`staff_common.fns`、宿主字段/`itemmimicconfig` 前提，并说明 cleargemstaff 会重建国王魔杖武器、重挂 hauntable；“取出”改为“嵌入新宝石时返还旧宝石”。
- **R03 采纳**：目标选择与落点选择拆开写（小游戏参与者/覆盖目标/套索实体→非水栖才找 telebase；落点按 GetDestPosition/随机/teletopos/实体位置），补 `locomotor:IsAquatic()` 与玩家 3+1 秒、非玩家即时。
- **R04 采纳**：拆解补 `max(1, ceil(amount×percent))`、fueled/armor 的 `inventoryitem` 条件、`decon_ignores_finiteuses`、拟态 TurnEvil 例外。
- **R05 采纳**：亮茄改“原版 SetupComponents 配置 0；weapon 与 planardamage 在 combat 分别结算，SetDamage 会生效；TUNING 位面键只影响新实例”，陷阱同步“配置为 0”。
- **R06 采纳**：示例补 `UPGRADETYPES` 登记、`_upgrader/_upgradeuser/_upgradeable` 三组标签、`canupgradefn` 与玩家授权；文字补 CanUpgrade/Upgrade 消耗链。
- **R07 采纳**：国王魔杖改“无次数/充能消耗（精神与腐化照算）”，红蓝 0 标注占位、不施法；依赖说明改为只跳过 staffcoldlight、yellow 的 stafflight 仍预载。
- **R08 采纳**：龙卷风行补 PvP 关闭时排除 player/possessedbody、仅 player 吃 PVP 倍率、工作分支是 `elseif` 且要求 CanBeWorked＋支持动作。
- **R09 采纳**：数值表改“水花在 ±1.5，浪再前推 1.5 的最终点判海面”，接入与陷阱同步。
- **R10 采纳**：磨损限定“通过命中检查的目标”，并写默认消耗×2 受倍率/效率修正；导航行同步。
- **R11 采纳**：获取结论改为“字面 SpawnPrefab 未命中，不覆盖数据表/拼名，按未穷尽处理；七条配方覆盖正常获取”。
- **R12 采纳**：示例补倒走表 `positionalwarp` 守卫，正文与陷阱改为“无组件会先报错”；“不崩”撤回。
- **R13 采纳**：门禁段改“仅成功分支 Discharge，首次标记/失败原因不进冷却”；裂缝表两种替换分开（开门新表立即 480 秒；喂宝石走 onPreBuilt 复制未满充能）。
- **R14 采纳**：入口理智改“有 sanity 且无 pocketwatchcaster/nowormholesanityloss 才扣”，索引改“警钟 HANDS、五表物品栏施法”。
- **R15 采纳**：警钟伤害只在构造与 percentusedchange 写入，已有实例不重读 TUNING；冷却仍在施法时现读。
- **R16 采纳**：磨损修饰改“构造 0、UpdateStats 按桶写 0/0.5/1”；200 耐久与 NET 10 点标注构造期；满度限定主物品栏格、额外容器不计、手持只补灵魂。
- **R17 采纳**：连抓改为“每处理一个候选目标扣 1（不看 PICKUP 返回值），扣满 total×0.25 后仍继续尝试，直到物件上限或耐久耗尽”。
- **R18 采纳**：扩档示例改为原地修改既有四档，说明替换整表不生效、SetSections 按 #-1、真加档需图标/符号。
- **R19 采纳**：鱼人标签改为“NPC 本就不受 IsRestricted 约束；标签是 getDamage 分支开关；放行玩家用 AddPlayerPostInit”。
- **R20 采纳**：扫把补“找不到下一款时，允许回基础外观且有现有皮肤仍可通过；仅 SHOULD_NOT_SELECT 目标无候选才失败”。
- **R21 采纳**：飞走仅未耗尽；被打掉落加 `ShouldProcOnAttackedOrBlocked` 共享门禁，活动页同步。
- **R22 采纳**：26 注册改为 BODY 12／HANDS 13／mapicon 1，快速判断、装备行、验证记录同步。
- **R23 采纳**：电刑机部署不扣 100（上限），扣次在 ms_drone_zap_fired（2/技能 1）。
- **R24 采纳**：投泥带冷却限定持有者 `Inventory:FindItems`，拾取只在更高时压平；补 spelltype 授权与 CanCast 只拦施法。
- **R25 采纳**：注册名改为 `wurt_terraform_cast_debuff`＋`wurt_swampitem_*_chargedfx`；月态只对跟随者、有 DoLunarMutation 才突变再挂 wurt_merm_planar。
- **R26 采纳**：盾自损按 `onattackother`（伤害结算前，扣没扣到目标无关）；推杆补 tee 的 persists=false 与 ClearGolfClub 清理链。
- **R27 采纳**：英文别名改 Party Balloon／Embalming Spritz／Putter。

校验：10 文件 validate_page 通过（新 32KiB standard 档），strict 0 error 0 warning，record 9 页已记录，--check-indexes 通过；staffs-more/pocketwatch/role-tools 六个 Lua 示例 parser_probe 通过。未运行游戏、未改 raw、未提交推送。
## 复核补正 2（2026-11-07）

- 投泥带：正文/陷阱/索引统一 `Inventory:FindItems` 范围（主物品栏＋装备槽＋手持 activeitem＋overflow 装备背包），并补“`SPELLTYPES.SHADOW_SWAMP_BOMB` 用户标签限制的是施法，不是装备”。
- 怀表：倒走表 `WARP_NO_POINTS_LEFT` 以守卫或 positionalwarp 为前提；裂缝表陷阱改“只复制标记并 `Discharge(480)`，不继承充能进度”；索引改为“警钟 HANDS；复活表点选幽灵，其余四表物品栏施法”。
- 强抢袋：摘要与索引改“扣满单次上限仍继续尝试拾取，只有耐久耗尽/袋子销毁才中断”。
- 静电升级：示例删恒真 `canupgradefn`；正文写明默认 stage 1→3、每阶 5 次，`canupgradefn` 是附加条件、不能绕过阶段上限。
- 基础法杖：拆解“至少返一份”限定正量材料（0 量仍返 0）；拟态目标只走 `TurnEvil`，不进入普通材料/清空/移除分支；`cleargemstaff` 标为国王魔杖专用清理参照，自建宿主需按自身组件与标签配对撤销。
- 同步遗漏：一角鲸角导航区分水花点与浪前推点；哑铃示例改“通过命中检查的目标”；恐怖盾牌正文删“每次命中自损”旧句，保留 onattackother 时点说明。

校验：8 文件 validate_page 与 strict 0 错 0 警、record 7 页已记录、导航通过；升级示例 parser_probe 通过。
补记：投泥带的范围修正同步补到 `character-gear-extra` 正文与陷阱，最终为 8 页＋索引全绿。

补记 2：投泥带补月态 `LUNAR_SWAMP_BOMB.."_spelluser"`；基础法杖页 kingcane 改链 staffs-more 小节、`dodischarge` 统一为“进入腐化态”。
