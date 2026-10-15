---
title: "装备·武器与战斗"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# 装备·武器与战斗

近战、长柄、海洋、投掷、远程、盾牌与事件武器的叶级索引；以下均为 HANDS 槽（战斗圆盾也占 HANDS）。按主要用途唯一登记，工具／照明等其他用法从文末“跨用途入口”进入。返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。

## 近战、长柄与海洋武器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/spear|长矛（spear）]] | HANDS 槽；基础近战武器模板：伤害 34、耐久 150 仅攻击磨损、耗尽即移除；sharp/pointy 标签消费端与获取途径 | spear、长矛、Spear、基础武器 / active |
| [[reference/prefabs/batbat|蝙蝠棒（batbat）]] | HANDS 槽；吸血武器：命中回血（未满血时）6.8＋扣理智（蝙蝠帽减半）、伤害 42.5、耐久 75、影子等级 2；皮肤命中 FX 代理同文件注册 | batbat、蝙蝠棒、Bat Bat、吸血武器、武器吸血 / active |
| [[reference/prefabs/hambat|火腿棒（hambat）]] | HANDS 槽；新鲜度驱动武器：无耐久、伤害 59.5→29.75 随新鲜度线性衰减（按上次更新的新鲜度快照）、腐坏整体替换 spoiled_food、不可食用 | hambat、火腿棒、Ham Bat、火腿武器、新鲜度武器、火腿棒伤害 / active |
| [[reference/prefabs/tentaclespike|触手尖刺（tentaclespike）]] | HANDS 槽；掉落型武器（无配方）：触手／大触手 0.5 掉率、伤害 51、耐久 100 仅攻击磨损；无皮肤、无 pointy 标签 | tentaclespike、触手尖刺、Tentacle Spike、触手掉落武器 / active |
| [[reference/prefabs/ruins-bat|铥矿棒（ruins_bat）]] | HANDS 槽；远古武器：伤害 59.5、耐久 200、装备移速 +10%；命中 20%（幸运加成）掷点生成暗影触手；无蓄力 AoE；远古站配方 nounlock | ruins_bat、铥矿棒、Thulecite Club、图勒棍棒 / active |
| [[reference/prefabs/shadow-battleaxe|暗影槌（shadow_battleaxe）]] | HANDS 槽；裂隙暗影武器：伤害 38+位面、汲命随 epic 击杀升 4 级（0→3.4）、二级起持续饥饿（卸装不暂停）、会说话；残破可修复；虚空风帽套装 ×1.1 | shadow_battleaxe、暗影槌、Shadow Maul、暗影战斧、会说话的武器 / active |
| [[reference/prefabs/nightsword|暗夜剑（nightsword）]] | HANDS 槽；暗影操控器武器：伤害 68、耐久 100、装备期 −0.333 理智/秒（Maxwell 免疫/Wanda 按年龄）、影子等级 2；5 皮肤各配跟随 FX | nightsword、暗夜剑、Dark Sword、暗影武器 / active |
| [[reference/prefabs/sword_lunarplant|亮茄剑（sword_lunarplant）]] | HANDS 槽；亮茄纯武器：命中 38+位面 30、对影增伤 ×1.1、亮茄头盔套装监听；破损保留实体可修理；双刃跟随 FX；200 用仅命中磨损 | sword_lunarplant、亮茄剑、Brightshade Sword、亮茄武器 / active |
| [[reference/prefabs/glasscutter|玻璃刀（glasscutter）]] | HANDS 槽；月岛科技纯武器：伤害 68、对 shadowsubmissive ×1.25、对纯影目标磨损 0.5；无工具功能 | glasscutter、玻璃刀、Glass Cutter、对暗影生物增伤的武器、玻璃刀耐久 / active |
| [[reference/prefabs/whip|三尾猫鞭（whip）]] | HANDS 槽；加长近战（武器加值 2，叠基础 3 与目标半径）：伤害 27.2、耐久 175；命中掷点“超响鞭”半径 14，对已有战斗目标的单位丢目标、满足状态才硬直（无伤害）；帝王蟹免疫 | whip、三尾猫鞭、Tail o' Three Cats、鞭子、超响鞭 / active |
| [[reference/prefabs/bullkelp_root|公牛海带茎（bullkelp_root）]] | HANDS 槽；鞭族武器（无耐久组件，新鲜度即寿命，40 天为基础保质期）：伤害 27.2/武器加值 2（叠基础 3）、每击掷点折断（幸运反向压低折断概率）；被毁/冲岸/海面漂流物/海獭窝获得 | bullkelp_root、公牛海带茎、牛海带根、海带茎、Bull Kelp / active |
| [[reference/prefabs/cutless|木头短剑（cutless）]] | HANDS 槽；粉猴武器（无配方）：命中偷窃敲落目标物品（thief 组件）；伤害 27.2、耐久 150、fuel 45 可当燃料 | cutless、木头短剑、弯刀、Cutlass、粉猴剑 / active |
| [[reference/prefabs/trident|刺耳三叉戟（trident）]] | HANDS 槽；海洋武器：近战按攻击者位置 51/76.5、右键点海面水爆（health 85/掀飞/拔海带/砸矿；四点采样船漏或水柱；照料农田；4 耐久/次）；蓝图配方（帝王蟹固定、克劳斯赃物袋 1/18）；非取水工具 | trident、刺耳三叉戟、三叉戟、Strident Trident / active |

## 投掷与远程

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/boomerang|回旋镖（boomerang）]] | HANDS 槽；投掷武器：本体即投射物，命中/脱靶均返航（CATCH 需双标签、背包开启才自动装备/入包）；伤害 27.2、耐久 10、未接住回程自伤；炼金引擎配方与 WeaponBoon 恩惠 | boomerang、回旋镖、Boomerang、飞去来器、投掷武器 / active |
| [[reference/prefabs/voidcloth_boomerang|阴郁回旋镖（voidcloth_boomerang）]] | HANDS 槽；充能代理投射：本体不离手、伤害/位面 5→27.2 随尺寸缩放、对月 ×1.25、不可接回；在场弹体 3（套装 4）放电退化近战；耐久 85（掷出/近战各磨 1） | voidcloth_boomerang、阴郁回旋镖、Gloomerang、虚布回旋镖、投掷武器 / active |
| [[reference/prefabs/throwables|投掷武器族（sleepbomb 等 4 注册＋2 伴生 FX）]] | HANDS 槽；三炸弹 complexprojectile 抛物线，仅南瓜/亮茄命中动态挂 explosive（50 堆叠/200 位面）、睡袋只生成 sleepcloud；雪球追踪、降温灭火＋冷度扩散（可冻结）、推雪人 | sleepbomb、pumpkin_bomb、bomb_lunarplant、snowball_item、睡袋、南瓜炸弹、亮茄炸弹、雪球 / active |
| [[reference/prefabs/ranged-weapons|远程武器族（blowdart 等 15 注册）]] | HANDS 槽；吹箭五变体（发射脱手不可回收、催眠/引燃/减速）、嚎弹炮、弹弓五变体＋FX/作业包（frame 改装整体替换、999ex 只取消堆叠上限、拟态皮筋可免弹）、蚊子炸弹抛物线 AoE（按投掷者玩家标签筛人） | blowdart、blowdart_sleep、blowdart_fire、houndstooth_blowpipe、slingshot、slingshotex、mosquitobomb、吹箭、催眠吹箭、嚎弹炮、弹弓、蚊子炸弹 / active |

## 防御与事件武器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/wathgrithr_shield|战斗圆盾（wathgrithr_shield）]] | HANDS 槽；女武神盾：armor 420 耐久/85% 吸收（单盾；多件取最高并分摊，无 finiteuses）、右键招架（178° 扇区/10 秒冷却，1 秒在 pre 后计时，nil 物攻可招架，技能树强化）；配方与装备双层角色门禁 | wathgrithr_shield、战斗圆盾、Battle Rönd、女武神盾 / active |
| [[reference/prefabs/event-weapons|事件武器族（rabbitkingspear 等 5 注册）]] | HANDS 槽；兔王棍对 manrabbit ×1.4、冲锋骑枪 joustsource 地面点冲锋（排除骑乘）、战斗长矛三件套（奔雷矛普攻 59.5/突刺 68，充能＋技能 4 门禁）；Forge 双件分诊留档 | rabbitkingspear、yoth_lance、spear_wathgrithr、spear_wathgrithr_lightning、奔雷矛、冲锋骑枪、兔王棍 / active |

## 跨用途入口

- 斧、鹤嘴锄、铲、锤、多用斧镐、气孔工具、亮茄工具、露西斧、暗影收割者、火炬、晨星锤、女王火炬的工具／照明主入口在 [[reference/prefabs/tools-field-index|工具与野外实用]]，本叶不重复登记；其武器侧数值见对应页战斗章节。
- 战斗圆盾同时提供 armor 吸收与耐久；护甲换算、身体护甲与饰品见 [[reference/prefabs/wearables-index|穿戴防具与饰品]] 及 [[reference/components/armor|armor 组件]]。
- 事件武器族的角色技能、活动门禁与该族其他活动装备见 [[reference/prefabs/staffs-character-index|法杖与角色道具]]。
- 三叉戟的水爆兼有砸矿、掀鱼、拔海带、船漏水与照料农田等工具用途；野外工具入口见 [[reference/prefabs/tools-field-index|工具与野外实用]]。
- 实用投掷（瓶中信、路标导航仪、种壳）不属战斗武器，登记在 [[reference/prefabs/utility-and-throwables|实用与投掷装备]]。

返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
