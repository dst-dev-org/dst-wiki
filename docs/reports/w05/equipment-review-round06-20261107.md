# 装备 Prefab 第六轮检视（2026-11-07）

范围为「穿戴防具与饰品」叶索引登记的 8 页。以 `raw/current/scripts` 的 Steam buildid 26335394 核对物品构造、装备回调、数值和关键消费端。8 页均登记在该叶索引，且页尾均有返回入口。新增报告，未改 Wiki 页或索引，未运行游戏及校验脚本。装备初审累计 60／69 页。 [叶索引](../../../wiki/reference/prefabs/wearables-index.md) · [BUILD](../../../raw/current/BUILD.txt)

结论为需修复，计 11 项 P1、18 项 P2。涉及多页的同一问题合并为一项，正文、示例和索引的修改位置一并列出。

## 跨页问题

- **R01 · P1｜普通护甲扣的是分摊后的吸收量。** 防护帽、传统甲、Boss 甲及裂隙甲的「按受击全额扣耐久／与吸收率无关」容易被读成每件按原始来伤扣。实际先乘伤害类型抗性，再取各装备最高有效吸收率得到总吸收量，按各自吸收率占比分配，另加弱点与该件吸收的特殊伤害。最后 `Armor:TakeDamage` 才乘该件损耗倍率并调用 `ontakedamage`。暗夜甲的理智扣减和铥矿皇冠 proc 的理智扣减都基于各自回调收到的损耗量。可在 armor 组件页已有入口处对照，四页各留一句，并清理索引里的「全额扣耐久」。 [防护帽 L23、85、188](../../../wiki/reference/prefabs/hats-armor.md#L185-L189) · [传统甲 L23、38、49、155](../../../wiki/reference/prefabs/armor-classic.md#L152-L156) · [Boss 甲 L22、77、152](../../../wiki/reference/prefabs/armor-boss.md#L149-L154) · [裂隙甲 L123、140](../../../wiki/reference/prefabs/armor-rift.md#L137-L142) · [索引](../../../wiki/reference/prefabs/wearables-index.md#L23-L28) · [伤害分摊](../../../raw/current/scripts/components/inventory.lua#L408-L499) · [耐久回调](../../../raw/current/scripts/components/armor.lua#L156-L164) · [暗夜甲](../../../raw/current/scripts/prefabs/armor_sanity.lua#L32-L41) · [铥矿皇冠](../../../raw/current/scripts/prefabs/hats.lua#L455-L475)
- **R02 · P1｜原版两个案例把全局函数当作模块成员。** 防护帽与 Boss 甲案例的 `local util = require("componentutil")` 后访问 `util.ShouldProcOnAttackedOrBlocked` 等无效，脚本以全局方式定义这些函数和表，没有返回工具表。`modmain` 示例应使用 `GLOBAL.ShouldProcOnAttackedOrBlocked`、`GLOBAL.TryLuckRoll`、`GLOBAL.LuckFormulas` 与 `GLOBAL.IsRangedWeapon`。 [防护帽案例](../../../wiki/reference/prefabs/hats-armor.md#L119-L176) · [鳞甲案例](../../../wiki/reference/prefabs/armor-boss.md#L100-L145) · [启动载入](../../../raw/current/scripts/main.lua#L260) · [函数定义](../../../raw/current/scripts/componentutil.lua#L103-L122) · [运势判定](../../../raw/current/scripts/componentutil.lua#L2129-L2247) · [模组环境](../../../raw/current/scripts/mods.lua#L258-L317)
- **R03 · P2｜防雷属于命中结算，气球帽仍可成为闪电目标。** 事件帽 L91 写成闪电不选中玩家。默认命中回调才读取装备槽的 `IsInsulated`，改发 `lightningdamageavoided`，`DoStrike` 仍发 `playerlightningtargeted`；也有 `force_no_insulation` 优先门禁。环境帽、衣物和 Boss 甲可沿用现有「命中免伤」口径。 [气球帽标签表](../../../wiki/reference/prefabs/hats-event.md#L90-L92) · [防雷读取](../../../raw/current/scripts/components/inventory.lua#L759-L770) · [命中和选中事件](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L37)
- **R04 · P2｜统一气球帽／背心骑乘态说明。** 帽页 L23、74 的「受击即爆」以及衣物页 L112 的「四条路都会引爆 Remove」都要带分支。帽和背心都只监听 `attacked`，先经过共同 proc 门禁。帽表把 `blocked` 也列为触发，应删去。真的调用 `DoPop` 时先执行 `DeactiveBalloon`，若有有效骑乘持有者则该分支既不播爆裂 FX、不打 AoE，也不立即 Remove；放在地上另走延迟 AoE。装备态 AoE 有显式排除标签，避免写成纯默认集合。 [气球帽](../../../wiki/reference/prefabs/hats-event.md#L73-L75) · [背心](../../../wiki/reference/prefabs/clothing.md#L109-L115) · [帽监听](../../../raw/current/scripts/prefabs/hats.lua#L1365-L1380) · [背心监听](../../../raw/current/scripts/prefabs/balloonvest.lua#L14-L42) · [共同爆裂](../../../raw/current/scripts/prefabs/balloons_common.lua#L19-L85)

## 防护与战斗帽族（hats-armor）

- **R05 · P1｜拾荒尖帽有已确认的 Boss 掉落路径。** L53、208 写脚本域找不到，甚至留下账户物品推测。拾荒疯猪在击败与相关撤场路径把仍装备的尖刺按 `_loot_fx` 动态拼名，`daywalker2_spike_loot_fx` 动画结束或读档后生成 `scraphat`，耐久为 6%–20%。清掉获取缺口与推测，并链接已有的碎裂 FX 页。 [帽表](../../../wiki/reference/prefabs/hats-armor.md#L51-L54) · [已有 FX 页](../../../wiki/reference/prefabs/cavein_dust_low.md#L44-L46) · [拼名](../../../raw/current/scripts/prefabs/daywalker2.lua#L365-L395) · [击败分支](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2285-L2298) · [撤场分支](../../../raw/current/scripts/prefabs/daywalker2.lua#L751-L763) · [掉落工厂](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L10-L17) · [注册与读档](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L73)
- **R06 · P1｜虚空风帽单戴就能叠加位面伤害。** L51 把武器叠层放在「套装 VOIDCLOTH」之后，容易误以为要穿长袍。装备帽即设置持有者与监听，HAND 槽 `shadow_item` 且有 `planardamage` 的物品才接收叠层。套装回调只增减暗影抗性。可同时注明 6 层最多 +24、3 秒不继续攻击清零、特定 `attacked` 情况也重置。 [帽表](../../../wiki/reference/prefabs/hats-armor.md#L49-L52) · [叠层与清零](../../../raw/current/scripts/prefabs/hats.lua#L3928-L4038) · [装备即接线](../../../raw/current/scripts/prefabs/hats.lua#L4042-L4064) · [套装回调](../../../raw/current/scripts/prefabs/hats.lua#L4105-L4111) · [调值](../../../raw/current/scripts/tuning.lua#L1912-L1914)
- **R07 · P1｜铥矿皇冠示例卸帽漏还原吸收率。** 即使修复 R02，proc 期间卸帽会取消唯一还原吸收率的定时任务，重新佩戴仍为 100%。解绑后成对复位 `forcefield` 和进入 proc 前的吸收率；案例当前直接覆盖两个装备回调，需限定由示例拥有回调的自定义帽或保留原装配回调。 [案例](../../../wiki/reference/prefabs/hats-armor.md#L131-L172) · [原版卸帽与复位](../../../raw/current/scripts/prefabs/hats.lua#L425-L453) · [原版接线](../../../raw/current/scripts/prefabs/hats.lua#L486-L555)
- **R08 · P2｜启迪之冠召唤的是攻击弹体。** L65、208 把触发写成通过 petleash 召唤 `gestalt_guard_evolved` 守卫。实际仅激活且 `onhitother.data.from_doattack` 时进入有效目标判定，排除 `gestalt_guard_evolved` 以及建筑等目标，生成 `alterguardianhat_projectile` 并通过 `follower:SetLeader(owner)` 归属，另扣 1 理智。85% 是比例阈值，佩戴初查与理智事件复查用的 getter 不同。 [帽表](../../../wiki/reference/prefabs/hats-armor.md#L61-L65) · [验证记录](../../../wiki/reference/prefabs/hats-armor.md#L205-L208) · [目标与归属](../../../raw/current/scripts/prefabs/hats.lua#L3515-L3585) · [阈值](../../../raw/current/scripts/tuning.lua#L2188)
- **R09 · P2｜绝望石头盔自愈表的百分比差了百倍。** L49 的 `lerp(1/6,1/10)÷1.5` 漏 0.01，并把实际的倒数插值写成直接插值。每秒修复约最大耐久的 0.0667%–0.1111%，套装再乘 1.5；穿上已受损的帽也能起任务，卸下停。理智损耗只有普通疯狂模式且相关 regen 任务存在时才有，戴同族身体甲时两件各减半。 [头盔表](../../../wiki/reference/prefabs/hats-armor.md#L47-L50) · [任务与理智](../../../raw/current/scripts/prefabs/hats.lua#L3705-L3766) · [调值](../../../raw/current/scripts/tuning.lua#L1885-L1888)
- **R10 · P2｜公主面具缺少启动套装的身体件入口。** L58 列了收编与运势副作用，却没有点明搭配 `costume_princess_body` 才走 `YOTH_PRINCESS` 套装回调。与镀金骑士胸甲的 `YOTH_KNIGHT` 分开写，查穿搭时可直接定位身体件。叶索引可把铥矿皇冠的官方名作为主检索词，保留「铥矿帽」为别名。 [公主面具](../../../wiki/reference/prefabs/hats-armor.md#L56-L59) · [套装检查](../../../raw/current/scripts/components/setbonus.lua#L52-L64) · [公主身体装注册](../../../raw/current/scripts/prefabs/costumes.lua#L101-L119) · [索引](../../../wiki/reference/prefabs/wearables-index.md#L17) · [铥矿皇冠译名](../../../raw/current/scripts/languages/chinese_s.po#L348815-L348817)

## 环境与功能帽族（hats-environment）

- **R11 · P1｜刮地皮头盔可空槽工作。** L68、126、209 把每格都消耗一块地皮写成必然。代码区分直接铺、还原底层、没有底层时铺入槽地皮或置为泥地。只有放入槽中地皮才扣那件物品，完成操作仍扣一次使用次数。槽空时，符合 `CanTerraformAtPoint` 的格子仍可刮。另将「蚁狮帽（`turfhat`）」中的注册名纠正为 `antlionhat`，`turfhat` 是标签。 [注册表](../../../wiki/reference/prefabs/hats-environment.md#L66-L68) · [地皮链](../../../wiki/reference/prefabs/hats-environment.md#L124-L128) · [陷阱](../../../wiki/reference/prefabs/hats-environment.md#L209) · [格子分支](../../../raw/current/scripts/components/autoterraformer.lua#L57-L104) · [标签](../../../raw/current/scripts/prefabs/hats.lua#L2405-L2413)
- **R12 · P1｜蝙蝠帽佩戴时的周期任务恢复帽子耐久。** L78 的「约 2.9 倍速衰减」与代码方向相反。满足吸血门禁且有 health 的普通穿戴者使其暂停常规腐坏，每 0.6 秒给帽加 0.06% 的腐坏时间，同时持有者周期损血 3。首次 10 点走延后的一次 combat 攻击，读档装备时跳过。吸血 6.8 是基础值，要求持有者已受伤、目标存活可吸血、非远程且非骑乘，并受 AoE 递减倍率影响。 [蝙蝠帽表](../../../wiki/reference/prefabs/hats-environment.md#L76-L78) · [吸血](../../../raw/current/scripts/prefabs/hats.lua#L6511-L6531) · [装备门禁](../../../raw/current/scripts/prefabs/hats.lua#L6570-L6590) · [加时封顶](../../../raw/current/scripts/components/perishable.lua#L180-L186) · [调值](../../../raw/current/scripts/tuning.lua#L9646-L9650)
- **R13 · P2｜睡帽保留工厂基础组件，风沙护目标签有跨族来源。** L75 的「无任何组件」应限定为没有额外玩法组件，`simple` 仍安装 HEAD 装备、物品等基础组件，且调整 floater。L200 「防沙尘暴的是」可限定「本页中的」，亮茄头盔与南瓜帽也会写 `goggles`。 [睡帽](../../../wiki/reference/prefabs/hats-environment.md#L74-L76) · [护目镜陷阱](../../../wiki/reference/prefabs/hats-environment.md#L200) · [睡帽工厂](../../../raw/current/scripts/prefabs/hats.lua#L1074-L1089) · [基础组件](../../../raw/current/scripts/prefabs/hats.lua#L188-L234) · [亮茄头盔](../../../raw/current/scripts/prefabs/hats.lua#L3879-L3882) · [南瓜帽](../../../raw/current/scripts/prefabs/hats.lua#L6100-L6104)
- **R14 · P2｜月亮蘑菇帽受击排队的是生成尝试。** L52「反击 3 个孢子」应补 `ShouldProcOnAttackedOrBlocked` 门禁，重复命中将同一队列的剩余尝试数重置为 3，已有任务时不另起一组。每次走 `TrySpawn`，按尝试次数说明更准确。 [蘑菇帽表](../../../wiki/reference/prefabs/hats-environment.md#L52) · [生成与续队](../../../raw/current/scripts/prefabs/hats.lua#L1841-L1886)

## 事件装扮与角色帽族（hats-event）

- **R15 · P2｜身份标签示例含未定义调用，且未演示接线。** L121 的 `DoEquip_ExchangeSkinnySymbolHack` 是占位且源码无该方法，复制会失败。删掉占位；若保留为协议摘录，只展示真实的加删标签回调并交代装备与卸下的挂载位置，以及模型路径按需清理。L23 的 `ancient_reader` 属面具本体标签、`spook_protection` 由装备扫描、`elixir_drinker` 写在花冠上，应与持有者身份标签分开。 [快速判断](../../../wiki/reference/prefabs/hats-event.md#L21-L24) · [示例](../../../wiki/reference/prefabs/hats-event.md#L112-L132) · [通用帽接线](../../../raw/current/scripts/prefabs/hats.lua#L218-L234) · [南瓜帽标签](../../../raw/current/scripts/prefabs/hats.lua#L6100-L6105)

## 传统身体护甲族（armor-classic）

- **R16 · P2｜骨头盔甲无 armor 但能免伤，示例须区分原版与简化版。** L154 不应把骨甲和另外两件一起写成「伤害全额穿透」。L127–146 的简化版永久登记 resistance，窗口后只恢复回调，因此它始终能挡匹配攻击。原版窗口结束会移除标签，充能完成才重加；`forcefield` 门禁是拒绝本件免伤，不是只跳过扣费。L61 的 1/3 秒是免伤窗口，不能直接用来断言 FX 寿命。 [导航](../../../wiki/reference/prefabs/armor-classic.md#L49-L61) · [示例及陷阱](../../../wiki/reference/prefabs/armor-classic.md#L114-L155) · [原版登记和清理](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L49-L93)
- **R17 · P2｜胡萝卜外套把兔人交给穿戴者。** L41、55 把 `leader` 写成挂在外套本体。搜索、容量和加忠诚都走 `owner.components.leader`。已有兔随从参与计数，进入装备时不替换它们。L28 也可去掉三个功能装备不装配 BODY／inventoryitem／inspectable／equippable 的误读，它们是机制不同，各自仍装配同槽骨架。 [注册说明](../../../wiki/reference/prefabs/armor-classic.md#L26-L41) · [导航](../../../wiki/reference/prefabs/armor-classic.md#L53-L55) · [收编和续忠诚](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L12-L39) · [外套装备组件](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L117-L135) · [腰带](../../../raw/current/scripts/prefabs/armor_slurper.lua#L60-L70) · [骨甲](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L225-L245)
- **R18 · P2｜区分腰带的无效护甲调值和有效减饥饿调值。** L43 指向 L1829–1830 并列作未使用，实际 L1830 是使用中的 `ARMORSLURPER_SLOW_HUNGER`；未用的护甲配置是 L1829、1831，后一条含 0.7 乘数，按本快照为 420。L63 的 `slurtlehat` 名称可用已核实的「背壳头盔」。 [传统甲表后](../../../wiki/reference/prefabs/armor-classic.md#L41-L43) · [陷阱](../../../wiki/reference/prefabs/armor-classic.md#L154) · [腰带调值](../../../raw/current/scripts/tuning.lua#L1829-L1831) · [装备消费](../../../raw/current/scripts/prefabs/armor_slurper.lua#L6-L12) · [头盔译名](../../../raw/current/scripts/languages/chinese_s.po#L349645-L349647)

## Boss 护甲族（armor-boss）

- **R19 · P2｜荆棘外壳的角色门禁只覆盖主动叠击。** 索引「双路径反伤（沃姆伍德专属）」会把通用受击尖刺也读成角色专属。装备时给所有穿戴者监听受击／格挡；`plantkin` 才额外监听 `onattackother`，回调核技能、冷却再累加。计数未到 3 时保留，进门失败或释放尖刺才归零；`onattackother` 在伤害前发出，避免写成已经造成伤害的 3 次连击。裂隙甲页的荆棘茄甲同类表述一起对齐。 [外壳机制](../../../wiki/reference/prefabs/armor-boss.md#L40-L44) · [茄甲机制](../../../wiki/reference/prefabs/armor-rift.md#L131-L135) · [索引](../../../wiki/reference/prefabs/wearables-index.md#L28) · [外壳分支](../../../raw/current/scripts/prefabs/armor_bramble.lua#L15-L66) · [茄甲分支](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L255-L288) · [攻击事件](../../../raw/current/scripts/components/combat.lua#L1159-L1175)
- **R20 · P2｜鳞甲案例少了原版反燃目标门禁。** L118–124 的示例未检查 `thorny` 和已死亡目标，修完 R02 后会比原版多点燃两类目标。若例子继续称为复刻，补门禁；并保留该免火伤修饰符只覆盖使用火伤倍率的结算。 [案例](../../../wiki/reference/prefabs/armor-boss.md#L118-L127) · [原版反燃](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L6-L18) · [火伤入口](../../../raw/current/scripts/components/health.lua#L190-L229)

## 裂隙护甲族（armor-rift）

- **R21 · P1｜沃拓克斯的亮茄免伤需要灵魂回响窗口。** L54、147 只有「装备＋激活阵营技能」，遗漏 `finishportalhoptask` 存在且 `TryToPortalHop(1, false)` 成功。后者先查灵魂数，再参与正常跳跃计数／结算；不等于常驻免伤。正常亮茄盔甲和头盔接此回调，荆棘茄甲的 `husk_master_postinit` 不接。索引也补窗口限定。 [机制表](../../../wiki/reference/prefabs/armor-rift.md#L50-L55) · [陷阱](../../../wiki/reference/prefabs/armor-rift.md#L145-L148) · [索引](../../../wiki/reference/prefabs/wearables-index.md#L27) · [免伤门禁](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L113-L125) · [跳跃结算](../../../raw/current/scripts/prefabs/wortox.lua#L614-L650) · [盔甲和茄甲接线](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L239-L247) · [茄甲专用接线](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L291-L305) · [头盔接线](../../../raw/current/scripts/prefabs/hats.lua#L3913-L3923)
- **R22 · P1｜0.75 是承伤倍率。** L47 的计算正确，括注「只承 25%」把承受与减免倒置。改成阵营抗性乘区承受 75%、减免 25%，之后还走普通吸收和位面防御。 [抗性表](../../../wiki/reference/prefabs/armor-rift.md#L45-L48) · [抗性调值](../../../raw/current/scripts/tuning.lua#L1897-L1910) · [乘区](../../../raw/current/scripts/components/inventory.lua#L421-L461)
- **R23 · P1｜绝望石自愈案例把装备限定写丢了。** L101–120 只看 `inventoryitem.owner`，护甲放回人物背包后仍有 owner，周期任务还能继续修；留在别的容器时任务也不会因失去合格持有者自动取消。原版受击要求已装备，穿上受损件也启动，卸下立即取消。补装备态和成对清理，或把案例缩为原版实现入口。 [案例](../../../wiki/reference/prefabs/armor-rift.md#L91-L123) · [原版任务启停](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L15-L76)
- **R24 · P2｜技能核验时机、装备限制和魔法理智消费端分别写。** L133 写成 onequip 查技能，实际当时只查 `plantkin` 决定是否监听，事件触发时才查技能。L129 漏恢复 `restrictedtag` 的后果写反，漏写会丢限制；L21 的「4 个注册＋茄甲变身态」可改成「4 个注册，含独立注册的茄甲」。L146 的「无消费者」漏了 Walter 的 `GetEquippableDapperness`，同批护符页已有正确锚点。 [范围](../../../wiki/reference/prefabs/armor-rift.md#L19-L24) · [案例](../../../wiki/reference/prefabs/armor-rift.md#L125-L135) · [陷阱](../../../wiki/reference/prefabs/armor-rift.md#L144-L147) · [事件门禁](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L26-L40) · [装备监听](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L66-L80) · [修复后恢复限制](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L120-L150) · [Walter 消费端](../../../raw/current/scripts/prefabs/walter.lua#L754-L760)

## 衣物与背心族（clothing）

- **R25 · P2｜独奏乐器招募动物，照料作物。** L84 把作物写进招募范围，实为同次扫描的两条分支。猪／鱼人需有 follower、无首领、通过 `CanBeRollCalled` 且未满额才招募；作物走 `farmplanttendable:TendTo`，随从满额仍能照料。续忠诚另外遍历符合条件的现有随从。 [独奏乐器](../../../wiki/reference/prefabs/clothing.md#L81-L86) · [组件消费](../../../raw/current/scripts/components/leaderrollcall.lua#L116-L149)
- **R26 · P2｜索引应区分腐坏与装备燃料。** 索引 L26 的「衬衫／背心腐坏双注册」容易被读成背心也走 perishable。该页只有花衬衫持续腐坏，透气／松软两件考拉象背心是 fueled USAGE 的两个注册。 [衣物页耐久链](../../../wiki/reference/prefabs/clothing.md#L68-L72) · [索引](../../../wiki/reference/prefabs/wearables-index.md#L26) · [花衬衫](../../../raw/current/scripts/prefabs/hawaiianshirt.lua#L51-L64) · [考拉象背心](../../../raw/current/scripts/prefabs/trunkvest.lua#L95-L127)

## 宝石护符族（amulets）

- **R27 · P1｜原版重生护符复活后被消耗。** L76、112 写作祟复活不限次、复活后自动穿回。实际玩家先临时装备来源物，进入 `amulet_rebirth` 时取下并销毁；20 次是佩戴期间的换血上限，跟复活无关。自定义 prefab 复用 `HAUNT_INSTANT_REZ` 不自动得到原版物品销毁分支，若示例要限次，应显式接复活成功后的消耗而非只写「改用 onhaunt 返回控制」。索引可以补「作祟复活消耗」。 [案例与陷阱](../../../wiki/reference/prefabs/amulets.md#L73-L112) · [换血](../../../raw/current/scripts/prefabs/amulet.lua#L11-L17) · [作祟事件](../../../raw/current/scripts/components/hauntable.lua#L82-L104) · [临时装备](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L358-L370) · [状态内销毁](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15218-L15239)
- **R28 · P2｜建造护符的折扣阈值和案例前提。** L114 可直接说明触发判断，避免读者用 `amount>1` 自行判定，建议写「计算后所需数量小于原配方才标记折扣」。示例应说明佩戴者要有 builder，物品须初始化 finiteuses，卸下／入模型复位和解绑；缺 `discounted` 字段会扣次属向后兼容，别让注释说成一定实际打折才扣。 [示例与陷阱](../../../wiki/reference/prefabs/amulets.md#L89-L115) · [取量比较](../../../raw/current/scripts/components/builder.lua#L500-L516) · [原版接线](../../../raw/current/scripts/prefabs/amulet.lua#L146-L185)
- **R29 · P2｜魔光护符和灯 FX 的归属分开。** 保留护符页的生成、挂接和清理说明，把 L7 的 `yellowamuletlight` 注册别名留在已登记的 FX 主页面，护符页注册表改为关系入口。L54 也要区分卸下／入模型会回退 bloom 与掉落回调只关灯。 [护符别名](../../../wiki/reference/prefabs/amulets.md#L7) · [灯与 bloom 表述](../../../wiki/reference/prefabs/amulets.md#L52-L55) · [FX 主页面](../../../wiki/reference/prefabs/armor_lunarplant_glow_fx.md#L35-L42) · [原版清理](../../../raw/current/scripts/prefabs/amulet.lua#L282-L323) · [掉落接线](../../../raw/current/scripts/prefabs/amulet.lua#L570-L589)

## 处置结果（2026-11-07）

29 项逐条对照 `raw/current` 源码复核，全部成立（其中 2 项按源码收窄表述），改 8 页＋穿戴叶索引（9 文件）。

- **R01 采纳**：五页＋索引统一为“armor 耐久扣的是 `Inventory:ApplyDamage` 分摊到该件的吸收量”：先乘伤害类型抗性、取各件最高有效吸收率算总吸收量、按各自吸收率占比分配，另加该件弱点与它吸收的特殊伤害，最后 `Armor:TakeDamage` 乘 `conditionlossmultipliers` 才扣 `condition`；暗夜甲理智＝（分摊量×损耗倍率）×0.10，不是原始来伤。
- **R02 采纳**：防护帽与鳞甲示例删 `require("componentutil")`，改 `GLOBAL.ShouldProcOnAttackedOrBlocked`／`GLOBAL.TryLuckRoll`／`GLOBAL.LuckFormulas`／`GLOBAL.IsRangedWeapon`，并写明该文件把这些定义成全局、不 return 模块表。
- **R03 采纳**：防雷改“命中时由 `DefaultOnStrike` 扫装备槽改发 `lightningdamageavoided`”，`DoStrike` 仍推 `playerlightningtargeted`，`ForceNoInsulated` 优先于装备；“不被闪电选中”删除。
- **R04 采纳**：气球帽/背心统一为“只监听 `attacked`，过 `ShouldProcOnAttackedOrBlocked` 后 `Pop`”；`DoPop` 先 `DeactiveBalloon`，骑乘分支不播 FX／不打 AoE／不 Remove，落地走延迟 AoE，装备态 AoE 有显式排除标签；帽表“blocked 触发”删去。
- **R05 采纳**：拾荒尖帽获取改为“疯猪击败/撤场把仍装备的尖刺按 `_loot_fx` 拼名，`daywalker2_spike_loot_fx` 的 animover 或读档 OnLoad 生成，耐久 6%–20%”，删“脚本域找不到/账户物品”推测，并链到拾荒疯猪碎装 FX 页。
- **R06 采纳**：虚空风帽改“单戴即接线”：装备时登记 HANDS 槽 `shadow_item`＋`planardamage` 物品为叠层目标，6 层＝+24、3 秒不攻击清零、非自身装备的 `attacked` 也清零；套装回调只增减暗影抗性。
- **R07 采纳（收窄）**：示例补 `inst._absorption` 与 `endproc`，卸下成对复位 `forcefield` 与进入 proc 前的吸收率，并注明示例覆盖了 `SetOnEquip/SetOnUnequip` 只适用自建帽；原文“原版卸帽会漏复位”按源码不成立——原版 `ruins_onunequip → ondetach → ruinshat_unproc` 本来就复位，故只保留“自建帽必须自己补全”。
- **R08 采纳**：启迪之冠改“激活且 `onhitother.data.from_doattack` 时生成 `alterguardianhat_projectile` 攻击弹体”，排除 `gestalt_guard_evolved` 与 structure/wall/balloon/smashable/deck_of_cards，`follower:SetLeader(owner)`、另扣 1 理智；85% 阈值初查用 `GetPercent`、复查用 `GetPercentWithPenalty`。
- **R09 采纳**：绝望石头盔自愈速率补 0.01，写为“每秒最大耐久的 0.0667%–0.1111%（理智越高越慢），套装 ×1.5”；补“戴上已受损件或 armor 受击才启动、装满/卸下即停、只在普通疯狂模式下真回血”，理智损耗同模式且戴同族身体甲各减半。
- **R10 采纳**：公主面具点明“须配 `costume_princess_body` 才启用 YOTH_PRINCESS”，与 YOTH_KNIGHT 的 `armor_yoth_knight` 分开；索引把铥矿皇冠作主检索词、保留铥矿帽为别名。
- **R11 采纳**：蚁狮帽改四分支（同格同地皮直接返回；可铺则耗地皮；有 `undertile` 还原且不耗地皮；无底层耗地皮或置泥地），明确“槽空也能刮、`CanTerraformAtPoint` 不过则不耗”；`turfhat` 标为标签，注册名改回 `antlionhat`。
- **R12 采纳**：蝙蝠帽改“满足吸血门禁时 `StopPerishing` 并每 0.6 秒 `AddTime`＝每秒 0.1% 腐坏时间（0.06%/tick），同时 −3HP/0.6 秒”；首次 −10HP 走延后一帧的 combat 攻击、读档装备跳过；吸血 6.8 需已受伤/可吸血/非远程/非骑乘并乘 AoE 递减。
- **R13 采纳**：睡帽改“无额外玩法组件（`simple()` 仍装 inventoryitem/inspectable/tradable/equippable＋floater）”；护目镜陷阱限定“本页中”，并补亮茄头盔与无面南瓜也写 `goggles`。
- **R14 采纳**：蘑菇帽改“`ShouldProcOnAttackedOrBlocked` 门禁后排 3 次 `TrySpawn`（0.25 秒间隔），重复命中把剩余次数重置为 3、已有任务不另起一组”。
- **R15 采纳**：删不存在的 `DoEquip_ExchangeSkinnySymbolHack`，改为真实的三回调标签协议（`SetOnEquip/SetOnUnequip/SetOnEquipToModel`）并说明模型路径为何要撤标签；快速判断区分“加给持有者”与“加在帽本体由装备扫描消费”的两类标签（`ancient_reader`/`spook_protection`/`elixir_drinker`）。
- **R16 采纳**：骨头盔甲改“无 armor 但有 `resistance`，充能窗口内整击归零”，与腰带/外套的无减伤分开；示例标注简化版永久登记 resistance、原版窗口结束 `OnShieldOver` 撤走七类登记、`OnChargedFn` 才重加；`forcefield` 门禁改“本件不承担这一击”；1/3 秒改称免伤窗口而非 FX 寿命。
- **R17 采纳**：胡萝卜外套随从改挂 `owner.components.leader`（搜索/容量/加忠诚都在持有者侧，本件虽也装 leader 却不收编），已有兔随从计入上限、不被替换；删“三件不装 BODY/inventoryitem/equippable”的误读，改为“10 件都同槽骨架，差别在玩法组件”。
- **R18 采纳**：区分腰带调值——`ARMORSLURPER_SLOW_HUNGER`（L1830）在用，未用护甲配置是 L1829 与 L1831（后者含 0.7 乘数＝420）；`slurtlehat` 名称改“背壳头盔”。
- **R19 采纳**：荆棘外壳改“通用受击反伤（任何穿戴者）＋沃姆伍德专属 `onattackother` 3 连击”，计数未满 3 保留、门禁失败或释放才清零，`onattackother` 在伤害前发出；索引删“双路径反伤（沃姆伍德专属）”，裂隙甲页荆棘茄甲同步。
- **R20 采纳**：鳞甲示例补 `thorny` 与已死亡攻击者两个门禁，并在正文注明“要与原版逐条对齐就不能省”；火免限定为“走 `GetFireDamageScale` 的结算”。
- **R21 采纳**：Wortox 免伤改四道门禁（装备中＋`wortox_allegiance_lunar`＋`finishportalhoptask` 存在＋`TryToPortalHop(1, false)` 成功），说明每次抵挡是一次真实灵魂跳跃、不是常驻免伤，husk 不接；索引补窗口限定。
- **R22 采纳**：0.75 改称承伤倍率（该阵营伤害按 75% 结算、减免 25%），删“只承 25%”。
- **R23 采纳**：绝望石自愈示例补装备态判定与 `startregen/stopregen` 成对清理（含覆盖原版回调的写法），并说明只读 `inventoryitem.owner` 会在入包后继续修、留容器不自动取消。
- **R24 采纳**：技能改“onequip 只查 plantkin，事件触发时才查技能”；`restrictedtag` 漏恢复的后果改为“限制消失、谁都能穿”；`is_magic_dapperness` 改“有消费者（Walter `GetEquippableDapperness`，Wanda/Waxwell/WX78 同钩子）”；注册计数改为“4 个注册，含独立注册的茄甲”。
- **R25 采纳**：独奏乐器改为同一次扫描分两支——猪/鱼人需 `follower`、无首领、`CanBeRollCalled` 且未满额；作物走 `farmplanttendable:TendTo`，不受随从上限；另遍历既有随从 +5 忠诚。
- **R26 采纳**：索引改“花衬衫常驻腐坏 vs 两件考拉象背心 fueled USAGE”，衣物页补“两件背心不是 perishable”。
- **R27 采纳**：重生护符改为“复活后消耗”（临时装备进 `amulet_rebirth` 再取下销毁），20 次是换血上限、与复活无关；示例与陷阱都写明自定义 prefab 复用 `HAUNT_INSTANT_REZ` 不会自动得到销毁分支，限次须显式接。
- **R28 采纳**：建造护符改“只有算出来比原配方少才标记 `discounted`”，并写明缺字段仍扣次属向后兼容；示例补佩戴者需有 builder、本件须初始化 finiteuses、卸下/入模型都要复位与解绑。
- **R29 采纳**：`yellowamuletlight` 注册别名从护符页移除，注册表行改为关系入口（工厂与端边界归装备跟随光效页）；新增区分“卸下/入模型做 Pop bloom＋停耗＋关灯”与“掉落回调只关灯”。

校验：9 文件 `validate_page.py --check-indexes` 通过（最紧 hats-armor 53,232 B／64 KiB），`validate_source_links.py --strict` 1,094 条引用 / 215 源文件 / 0 error 0 warning，8 页 `record` 全部 already_recorded（0 changed），11 段 Lua 经 `parser_probe --syntax-only` 通过，导航 0 问题。未运行游戏、未改 raw、未提交推送。累计 60／69 页。

## 复核补正（2026-11-07，用户复核 7 组）

- **R06 导航**：`hats-armor` 导航行改「voidcloth **单戴即**用 `attacked`/`onattackother` 叠武器位面伤害（与套装无关，套装回调只改 `shadow_aligned` 抗性）」，与注册表一致。
- **R08 速查**：`alterguardianhat` 从“攻击召唤守卫”改为“攻击时生成 `alterguardianhat_projectile` 攻击弹体”。
- **R09 理智**：绝望石头盔行补 `dapperfn`——不成套时只有本件有 regen 任务才扣一份 `CRAZINESS_MED`；戴同族 `armordreadstone` 成套后只要任一件有自愈任务，两件就各扣半份（合计仍是一份）。
- **R21 位移**：Wortox 陷阱改“只借跳跃的记账、不产生位移”：`TryToPortalHop` 只查灵魂数并计入 `_freesoulhop_counter`/`_soulhop_cost`（未达上限记为免费并起 `finishportalhoptask`，否则立刻 `FinishPortalHop` 结算）；真正的位移是 `ACTIONS.BLINK` → `portal_jumpin` 状态，`FinishPortalHop` 本身不移动玩家。
- **R04 气球**：`hats-event` 陷阱删掉 `blocked` 入口（改为“除 `attacked` 之外…”），并补“`DoPop` 在骑乘时只 `DeactiveBalloon`，不放 FX、不打 AoE、不立即 Remove”；`clothing` 速查同步“被打/溺水即爆（骑乘时只停用不炸）”。
- **R25 招募**：`clothing` 速查改“招募猪／鱼人、照料作物（作物不受随从上限）”；独奏乐器段删“提前 break”，改为“`can_tend_crop` 为真时循环不会 break，随从满员照样扫到列表结束继续照料”。
- **R16 骨甲示例**：窗口结束改为**无条件**恢复回调，并写明装备态交给 `ShouldResistFn`——条件恢复会在窗口内卸甲/转移后永久丢掉回调（仍免伤、却不再触发）。
- **R28 示例**：`SetsMaxUses` 改 `SetMaxUses`，并补“`SetUses` 初始化”。
- **R29 导航**：bloomer 行改“卸下／入模型 Pop 回退 bloom；掉落回调只 `turnoff_yellow` 关灯，不动 bloom”。

校验：9 文件 `validate_page.py --check-indexes` 与 `validate_source_links.py --strict` 仍全绿（1,095 条引用 / 215 源文件 / 0 error 0 warning，较处置段 +1 条即新增 BLINK 锚点），8 页 `record` 全部已记录。未运行游戏、未改 raw、未提交推送。

## 复核补正 2（2026-11-07）

- **R09 数值符号**：绝望石头盔的理智损耗改 `CRAZINESS_MED`（= −100/`day_time`，day_time＝300 秒 → −0.333 理智/秒，不再写"−100/天"以避开 480 秒 `total_day_time`）；成套时半份写 `CRAZINESS_MED / 2`≈−0.167 理智/秒，不再前置负号（常量本身为负，`−CRAZINESS_MED/2` 会变成回理智）。
- **R06 事件拆分**：导航行把两个事件分开写——`onattackother` 加层（HANDS 槽符合条件的武器），非自身装备来源的 `attacked` 清层。

校验：9 文件 `validate_page.py --check-indexes` 与 `strict` 全绿（1,095 条引用 / 215 源文件 / 0 error 0 warning），8 页 `record` 全部已记录。

补记 3：同一处"−100/天"口径在 `hats-event` 的单位换算行也一并改为 `CRAZINESS_MED`＝−100/`day_time`（300 秒 → −0.333 理智/秒；480 秒全天合计 −160，别按 `total_day_time` 换算）；全库 `reference/` 已无"−100/天"写法。
