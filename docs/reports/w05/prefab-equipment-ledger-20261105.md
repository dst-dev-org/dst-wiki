# 可装备 Prefab 扩库台账(2026-11-05)

本台账是「原版可装备物品 Prefab」扩库的工作状态来源。快照:steam-buildid **26335394**(`raw/current`)。主代理会话持续推进,逐批滚动;批次状态只记验收结论,过程细节入 `wiki/log.md`。

## 整体验收（2026-11-08）

装备 Prefab 七轮内容检视已收口。五个叶索引共 69 页，分别为工具与野外实用 22 页、武器与战斗 19 页、法杖与角色道具 11 页、穿戴防具与饰品 8 页、背包收纳与重物搬运 9 页。每页只在一个叶索引登记，返回链均包含该叶索引。逐页结论见同目录 `equipment-review-round01` 至 `equipment-review-round07`。

对 69 页和 7 个相关索引集中跑页面与严格源码链接校验，均通过。全库导航 51 个索引、1,177 个知识页零问题。6,000 处受支持的源码引用零错零警，另有 9 处非支持后缀由工具跳过。69 页的 1,873 组来源哈希全部一致。此次只汇总验收，未重新跑 Lua 示例或运行游戏。

## 扫描口径与边界

- **主口径**:`raw/current/scripts/prefabs/*.lua` 中含 `AddComponent("equippable")` 的文件共 **152** 个(2026-11-05 grep 实测)。这是可装备物品的下界:生物/建筑持有或读取装备的消费者(如 bat_boss、monkey、snowman、walrus、treasurechest 等)在批次核验时按实际内容排除,不凭文件名预判。
- **辅口径**:`equipslot`/`EQUIPSLOTS` 引用(133 文件)用于交叉定位共享工厂(MakeHat、MakeAxe、MakeFumaroleTool、staff.lua、amulet.lua 等)与 `equipslot` 未显式赋值的文件(equippable 组件默认 HANDS,见 components/equippable.lua:29)。
- **字面 `Prefab("...")` 只算候选下界**;同文件多注册(如 axe.lua 4 注册、hats.lua `MakeHat("x")`→`xhat` 约 60 注册、farm_hoe.lua 3 注册)按工厂展开。
- 已有主体页复用:`heatrock`(库存物品非装备)、`backpack`、`cane`、`yellowstaff`、`opalstaff`、`stafflight`/`staffcoldlight`(生成物)。`hats.md` 为帽子附属 FX 页,不代表帽子主体已覆盖。
- 每批建页前仍须重核:已有页面、PLAN 认领、源码实际接线。

## 状态图例

`待建` → `in-flight`(写作子代理进行中)→ `待复审`(页面已交付)→ `accepted`(通过独立复审+机械检查+索引登记)。另有 `already`(已有主体页)、`待核验`(接线/归属未确认)、`excluded`(附依据)、`blocked`(附原因)。

## 补课计划(2026-11-07 重录,用户指出流程缺位后)

**缺位事实**:批 1a/1b/1c 有独立机械检查子代理(见 log 归档卷 2026-11-06 两条);自批 2 起,因配额限流连续杀代理,独立机械检查层被"主代理收口时自跑验证器"替代,且主代理内容复审对部分页面覆盖不均、未逐页留痕。批 2 已建 20 页 + 批 3 已建 4 页(torch/lighter/nightstick/lantern)共 **24 页**缺两层或其一,全部补做。

1. **主代理内容复审(补课重点)**:每页对照源码独立复核——装备槽与持有条件、AddComponent 接线与生成/调用、关键行为与触发条件、tuning/配方/.po 官方名、服务端/客户端归属、数值与清理、示例可用性、页内一致性;逐页结论记入下方「补课复审记录」。
2. **机械检查子代理**:内容复审修复落定后,子代理**串行**分组跑全部 24 页(validate 三件套 + 锚点抽查 + 导航 + 索引登记核对),P1/P2/P3 清单修复后重跑验证器收口。

**并发纪律(2026-11-07 按用户澄清修正)**:并发上限 3,可能被限流到 2——常态按在飞 ≤2 规划,配额健康时可到 3;配额错误不反复重试,如实上报由主代理决定续派;派发前以完成通知确认在飞数量,不凭记忆推断。

**顺序**:批 3 在途代理按正常流程收口(queen_torch 已交付→主代理复审→机械检查→登记/基线;redlantern 交付后同流程)→ 24 页补课(先内容复审、后机械检查)→ 批次 4-7 恢复完整三层流程(写作子代理 → 页面固定 → 主代理内容复审 → 机械检查子代理 → 修复重验 → 索引/基线收口),批次记录必须写明两层结论,缺层不算收口。

**补课复审记录**(逐页追加:页面 | 结论 | 修复):

- 系统性筛查(24 页):tool 标签大小写 0 残留;Lua 示例裸全局 0;canlight 语义——torch/lighter/nightstick 均显式 `canlight=false`(源行 L500-503/L374-376/L174-177 实核),与 redlantern「不覆写」不同,各页断言正确。
- TOSS 收集链勘误(3 页):torch.md「丢出背包中的火把」无收集路径(通用收集两处均排除 special_action_toss,componentactions.lua:2074/2285;火把 TOSS 仅 Wilson wilson_torch_7 经 GetPointSpecialActions 的 HANDS 装备路径,wx78 同名函数无 TOSS 已核)——已改写并补 wilson/SGwilson 锚点;keep_equip_toss 语义纠正为「选 throw_keep_equip 状态＋抑制通用自动装备(playercontroller.lua:4675),物品不在 HANDS 时先补装备、投出后恢复原装备,物品本身仍被抛出」——torch.md/queen-torch.md 措辞修正、dumbbells.md L77 补状态语义;dumbbells.md「实体目标生成」锚点 L2809-2816(实为 LIFT 收集器)→L2276-2293。
- 基础近战组 8 页(spear/batbat/hambat/tentaclespike/cutless/whip/ruins-bat/bullkelp_root):通过。配方行 5 处(recipes.lua L478-481/L809 逐字段)、tuning 16 键(含 batbat 理智 −3.4/−1.7 换算、HAMBAT_USES 死键结论)、tentacle 掉率 0.5、ruins_bat 掷点召唤、whip supercrack、cutless thief:StealItem、8 个 .po 官方名全部命中。
- 裂隙武器组 5 页(nightsword/shadow-battleaxe/sword_lunarplant/voidcloth_scythe/wathgrithr_shield):通过。NIGHTSWORD 68/100、CRAZINESS_MED=−100/day_time=−0.333/秒(注意 day_time=300=10 段×30s,非 total_day_time 480;is_magic_dapperness 全库无消费端仅注释)、SHADOW_BATTLEAXE 表逐级核验(位面 10/14/18/22、汲命 0/1.7/2.55/3.4、砍伐 1.5→2.25、饥饿 0/0.05/0.1/0.2)、LUNARPLANT/VOIDCLOTH 套装键 1.1/+5、SCYTHE 采收半径 4/锥角 165°、配方行 5 处(recipes L483/852/869/875/196)、护甲 150×4×0.7=420、招架 178°/冷却 10。
- 投掷/复杂组 7 页(boomerang/voidcloth_boomerang/trident/dumbbells/lucy/pocketwatch/gnarwail_horn):通过(dumbbells 两处修复见上)。BOOMERANG 27.2/10 用/距离 12+2、VOIDCLOTH_BOOMERANG 七键{5→27.2}/1.25/85 用/1.1 移速/影子 3、TRIDENT 表(近战 51/海洋 76.5/水爆 85/半径 2.75/50 次)、GNARWAIL 半径 20、LUCY AXE×0.5=13.6 与 LUCY_BITE_DAMAGE=5 死键、pocketwatch 八配方 builder_tag="clockmaker"＋no_deconstruction 谓词逐行、trident 配方 L491、boomerang 配方 L489。
- 照明四页(torch/lighter/nightstick/lantern):通过(torch 修复见上)。NIGHTSTICK_FUEL=night_time×6=360、NIGHTSTICK_DAMAGE=×.85=28.9、LIGHTER_FUEL=480×1.25=600、LANTERN_LIGHTTIME=(60+120)×2.6=468、REDLANTERN_LIGHTTIME=5760;canlight=false 三页源行实核。
- **补课 a 总结(2026-11-07)**:24 页全部过主代理复审。修复 4 页 6 处——torch.md(TOSS 收集方向＋keep_equip 语义＋技能门控注)、dumbbells.md(throw_keep_equip 语义＋实体目标锚点)、queen-torch.md(TOSS 收集方向,批 3 收口时)、redlantern.md(canlight 方向,批 3 收口时);其余 20 页数值/配方/官方名/换算全量核对通过。复审覆盖声明:数值层全量、机制层抽查(掷点/引燃/收集链/套装/换算/掉率/偷窃/采收/招架),页面全文语义依赖写作代理自检＋机械检查层(补课 b)兜底。
- **补课 b(2026-11-07)**:机械检查子代理×2 并行(13+13 页,含批 3 收尾两新页),0 P1、1 P2(trident.md `ACTIONS.CASTSPELL` 锚点起点偏 6 行→L3445-L3468,两处已修)、P3×10 全修(自记字节数漂移 8 页校正:batbat/shadow-battleaxe/voidcloth_scythe/boomerang/whip/trident/dumbbells/pocketwatch/gnarwail_horn/queen-torch;其中 dumbbells/queen-torch 为当日复审修改的预期漂移;tentaclespike 悬空自引用补实测 104 行/12075 字节;redlantern 引用计数回填 101/27)。锚点抽查 130/130 命中、导航 0 死链、索引 26/26 登记、基线 check 全部 unchanged。修后 12 页 validate_page 通过、trident strict 0/0、--check-indexes 全绿、git diff --check 无空白错误(仅 CRLF 提示)。**补课两层全部完成,后续批次 4-7 按完整三层流程执行。**
- **批 4(2026-11-07 完成)**:头部装备三页 accepted——hats-armor(25 注册,complex,219 行/48649B)、hats-environment(32 注册,complex,235 行/49103B 余 49B)、hats-event(28 注册,complex,163 行/39904B);85 玩法帽对账 25+32+28 吻合(机械检查硬校验逐名 grep)。主代理归属裁决(源码实核):woodcarvedhat(armor+resistance)/mask_princesshat/yoth_knighthat(armor+setbonus)补入 armor 页;spiderhat/bat_bosscorpsehat(裁决注误写 +0.333,代理实核为 −0.333/秒且有近战吸血 +6.8)/mermhat/antlionhat/蘑菇×4(insulator+waterproofer+孢子)归环境页并补写;裁决由三个写作代理分波执行(事件页差集硬校验+armor 修订+env 修订)。主代理独立抽查:ARMOR_WOODCARVED_HAT=150×2.5×0.7=262.5、mermhat/antlionhat/polly_rogershat 配方行逐字段、SPIDERHAT_PERISHTIME=120s、pumpkin 动态卸装 equippable。机械检查子代理:0 P1、1 P2(hats.md 被并行 FX 会话第 15 轮处置修订 4 行——非本批所为,按「保留他人改动」保留)、4 P3 全修(armor 自记字节、env 表B/表C 计数头、env FX 计数口径、event 陈旧覆盖缺口陈述)。三页 strict 0/0、validate_page 通过、record 3 页、--check-indexes 全绿、索引新增「头部装备」节 3 行(另更正同族补充注)、git diff --check 干净。未运行游戏、未提交推送。
- **批 5(2026-11-07 完成)**:身体装备五页 accepted——armor-classic(10 注册,complex,182 行/34505B)、clothing(12 注册,standard,134 行/24571B)、armor-rift(4 注册+3 FX,complex,168 行/38262B)、armor-boss(4 注册+1 FX,standard,176 行/24563B)、umbrella(4 注册+2 FX,standard,147 行/24574B)。写作代理勘正派发假设:armorslurper/armorskeleton/armor_carrotlure 无 armor 组件(功能装备);armor_sanity=暗夜甲(受击全额×0.10 扣理智);注册名连字符与文件名不一致三处(armordragonfly/armorwagpunk/armordreadstone);grass_umbrella.lua 不存在(伞同文件双注册);荆棘外壳为沃姆伍德专属配方非蜂后掉落、鳞甲吸收 0.7 非 0.9;umbrella 页发现酸雨 rate_modifiers 负值反号致燃料回升、winona_telebrella OnDepleted 死代码。主代理抽查:ARMORWOOD/MARBLE/RUINS/SNURTLESHELL/SANITY/DREADSTONE/DRAGONFLY 换算式逐键、RAINCOAT_PERISHTIME=10 天、UMBRELLA_PERISHTIME=6 天、minifan 分诊(weapon+equippable+heater,归批 7)。机械检查:五页全绿 0 P1/P2/P3、40 锚点全中、34 注册零重复;clothing 基线因 sources 误写 prefabs/prefabskins.lua(实际在 scripts 根)被跳过,修正后 record 成功(44 组 unchanged)。收口时父索引超字节预算(26050/24576),按 fx-index 先例拆出 [[reference/prefabs/equipment-index]] 叶级索引(手持/头部/身体三节),父索引以一行指针替代,两索引均通过 --check-indexes(父 9974B/叶 15404B)。未运行游戏、未提交推送。
- **批 6(2026-11-07 完成)**:背包与袋六页 accepted——piggyback(1 注册,减速 0.9强壮态豁免)、krampus_sack(14 格,坎普斯 1%+克劳斯袋 10% 双来源)、icepack(8 格共享 backpack 布局,fridge+nocool 保鲜不降温)、candybag(分诊独立页:14 格糖果过滤+HALLOWED_NIGHTS 科技活动解锁链)、pouch-bags(6 注册:potatosack 实为健身配重非容器、spicepack 官名厨师袋 foodpreserver 标签非组件、beard_sack 为 BEARD 槽威尔逊技能三档)、wortox-nabbag(分诊独立页:非容器强抢袋,锥形扫拾伤害按占用格数三档)。主代理抽查:PIGGYBACK_SPEED_MULT=0.9、WILSON_BEARD_BITS 1/3/9、mighty_gym 生成 2 只 potatosack、HALLOWED_NIGHTS={SCIENCE=10}、wortox_nabbag 配方行;回填 piggyback/pouch-bags 家族互链并登记 equipment-index(新增「背包与袋」节 5 行+手持工具节 wortox-nabbag 行)。机械检查:0 P1/P2、2 P3(回填互链预期字节漂移,已校正);48 锚点全中、93 链接全通、record 6 页、--check-indexes 全绿。未运行游戏、未提交推送。
- **批 7(2026-11-07 完成)**:八页 accepted——staffs(7 注册,complex 129 行/25761B;勘正 opalstaff=唤月者魔杖)、amulets(7 注册含 FX,complex 126 行/22172B;勘正护符全 BODY 槽非 NECK)、living-tools(8 注册,complex 147 行/27296B;机械检查发现注册表漏 minifan 行已补)、fishing-tools(8 主注册+46 附属清单,complex 163 行/30662B;fishingnet unused 无生成点)、event-weapons(5 注册,standard 141 行/23440B;Forge 双件分诊 blocked——equippable 在编译事件数据)、throwables(4+2FX,standard 125 行/24562B)、tools-misc(6 注册,standard 144 行/24490B;香炉引诱非驱散、罗盘 HUD 指针、木手杖与 cane 辨析、扫帚骑乘动画非飞行)、role-tools(26 注册,complex 155 行/34781B;restrictedtag 门禁全链、merm_tool 玩家不可穿、鞍角卸鞍非上鞍、bodypillow defense_callback 死键)。主代理抽查:PIGGYBACK 等换算、minifan heater 键、HALLOWED_NIGHTS tech、配方行;amulets 基线因 sources 误写 prefabs/klaussackloot.lua(实际 components/)被跳过,修正后 record 成功。机械检查:0 P1/P2、3 P3 全修(living-tools 注册表漏行、索引计数口径 2 处、4 页基线自述过时);64 锚点全中、84 导航全通、record 8 页、--check-indexes 全绿。索引:equipment-index 手持工具节 +7 行、身体装备节 +1 行(amulets)。未运行游戏、未提交推送。
- **批 8(2026-11-07 完成,终扫差额)**:152 文件终对账发现 21 个未覆盖确认候选,分诊后三页 accepted——ranged-weapons(14 注册:吹箭五变体发射脱手不可回收、弹弓 frame 改装整体替换/999ex 为堆叠上限非无限弹药、蚊弹抛物线 AoE)、staffs-more(12 注册:亮茄魔杖套装直读头部、天气风向标、王杖嵌宝石+腐化 rechargeable、滑稽马疾驰、moonstorm 静电三态——装备主体裁定仅 catcher)、misc-equipment-2(38 固名注册+veggies 巨型作物 14×3 BODY heavy 工厂:恐怖盾牌自损、伯尼 restrictedtag=bernieowner 全库仅 willow 授予、月坛部件 heavy、金库罗盘、投泥带、WX78 电刑机、水草种壳;waterplant.lua 的 equippable 命中为 boss 运行时临时武器判 excluded)。主代理分诊:books/ghostly_elixirs/abigail_flower 无 AddComponent("equippable") 排除;consumer 文件(生物/建筑/宝箱等 18 个)按排除记录归类。机械检查三页全绿(0 P1/P2/P3、64 固名注册零重复、24 锚点全中);record 3 页、--check-indexes 全绿、git diff --check 干净。索引 equipment-index 手持工具节 +3 行。未运行游戏、未提交推送。**补记(2026-11-07)**:批 8 终扫漏计 2 个含 equippable 的文件——prefabs/glass_spike.lua(5 注册)与 prefabs/oceantreenut.lua(1 注册),二者未进入 21 候选或排除表;批 9 补建 heavy-items.md 后 152/152 闭合,此前"152 文件全部完成/排除/分诊"表述以批 9 为准。misc-equipment-2 已于批 10 拆分为 giant-crops/moon-altar-pieces/event-and-carnival-gear/utility-and-throwables/character-gear-extra,历史页名仅作批次追溯。
- **批 9(2026-11-07 完成,152 文件补遗)**:两文件六注册 accepted——heavy-items(standard,104 行;glassspike_short/med/tall＋glassspike＋glassblock 五注册与 oceantreenut 一注册;BODY heavy、item/obstacle 双态、玻璃化/锤毁、撞船掉落、沉水生长)。主代理逐项对照源码复审:注册分派、heavy 两态、玻璃化与锤毁、撞船掉落与补回、沉水容器与生长判定、掉落表、读档补挂、官方名/中文名;机械检查 validate_page 通过、strict 60 引用 0 错 0 警、52 唯一目标、18 源文件,record 18 组 unchanged;索引身体装备节 +1 行。未运行游戏、未提交推送。
- **批 10(2026-11-07 完成,杂项装备拆分)**:将 misc-equipment-2 按主题拆为五页 accepted——giant-crops(80 行,veggies 14×3 巨型作物)、moon-altar-pieces(71 行,6 部件+3 岩石)、event-and-carnival-gear(76 行,恐怖盾牌/派对气球/防腐喷雾/推杆)、utility-and-throwables(79 行,瓶中信系/路标导航仪/种壳)、character-gear-extra(78 行,伯尼/投泥带/电刑机);heavy-items 保留 glassspike/oceantreenut。152 文件口径不变;索引手持工具节 -1+3 行、身体装备节 +2 行;record 5 页 98 组 unchanged;导航 46 索引/1,177 知识页零问题。未运行游戏、未提交推送。

## 批次与候选

### 批次 1a:基础工具(核心四件)——accepted(2026-11-06)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| axe / goldenaxe / moonglassaxe | prefabs/axe.lua(MakeAxe 工厂) | HANDS;CHOP 工具+武器 | accepted | reference/prefabs/axe.md |
| pickaxe / goldenpickaxe | prefabs/pickaxe.lua | HANDS;MINE 工具+武器 | accepted | reference/prefabs/pickaxe.md |
| shovel / goldenshovel | prefabs/shovel.lua | HANDS;DIG 工具+武器 | accepted | reference/prefabs/shovel.md |
| hammer | prefabs/hammer.lua | HANDS;HAMMER 工具+武器 | accepted | reference/prefabs/hammer.md |

### 批次 1b:工具补充——accepted(2026-11-06)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| multitool_axe_pickaxe | prefabs/axe_pickaxe.lua | HANDS;CHOP+MINE 双工具、shadowlevel | accepted | reference/prefabs/multitool_axe_pickaxe.md |
| glasscutter | prefabs/glasscutter.lua | HANDS;月玻璃刀,**纯武器**(无 tool 组件) | accepted | reference/prefabs/glasscutter.md |
| farm_hoe / golden_farm_hoe | prefabs/farm_hoe.lua(MakeFarmHoe) | HANDS;锄地经 farmtiller(TILL 不走 tool) | accepted | reference/prefabs/farm-hoe.md |

### 批次 1c:特殊机制工具族——accepted(2026-11-06)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| fumaroleaxe / fumarolepickaxe / fumaroleshovel / fumarolehammer / fumarole_farm_hoe | 各工具 lua + standardcomponents.lua(MakeFumaroleTool/MakeFumaroleToolPristine) | HANDS;气孔工具:使用降温、冷却自毁、重热自修、分档效率外观、对冻结目标增伤;配方统一树枝4+螨腺2+硝石3 | accepted | reference/prefabs/fumarole-tools.md |
| pickaxe_lunarplant / shovel_lunarplant | prefabs/pickaxe_lunarplant.lua、shovel_lunarplant.lua | HANDS;亮茄工具:套装加成(lunarplanthat 手工监听)、planardamage、Forge 修理、破损循环、幸运触手 | accepted | reference/prefabs/lunarplant-tools.md |

### 批次 2:手持武器(拆 2~3 小批)——accepted(2026-11-06)

| 候选注册名 | 源码入口 | 预计用途 | 状态 | 页面 |
|---|---|---|---|---|
| spear | prefabs/spear.lua | HANDS 基础长矛 | accepted(2026-11-06,索引已登记) | reference/prefabs/spear.md |
| batbat | prefabs/batbat.lua | HANDS 吸血蝙蝠棒（回血6.8/理智-3.4,蝙蝠帽减半;配方3翅膀+2活木+1紫宝石） | accepted(2026-11-06) | reference/prefabs/batbat.md |
| hambat | prefabs/hambat.lua | HANDS 新鲜度驱动火腿棒（无耐久,伤害59.5→29.75,腐坏替换spoiled_food,配方肉2非火腿） | accepted(2026-11-06) | reference/prefabs/hambat.md |
| nightsword | prefabs/nightsword.lua | HANDS 暗夜剑（伤害68、装备期-0.333理智/秒,Maxwell免疫;5皮肤跟随FX） | accepted(2026-11-06) | reference/prefabs/nightsword.md |
| tentaclespike | prefabs/tentaclespike.lua | HANDS 触手尖刺（掉落型无配方,tuning 键 SPIKE_DAMAGE/SPIKE_USES,51/100） | accepted(2026-11-06) | reference/prefabs/tentaclespike.md |
| ruins_bat(铥矿棒) | prefabs/ruins_bat.lua | HANDS 命中 20%+幸运掷点生成影子触手;无蓄力 AoE(派发前认知已纠) | accepted(2026-11-06) | reference/prefabs/ruins-bat.md |
| shadow_battleaxe | prefabs/shadow_battleaxe.lua(+同文件 fx、classified 文件) | HANDS 暗影槌(Shadow Maul,非"战斧"):四级 epic 击杀成长、汲命、饥饿、说话;残破可修 | accepted(2026-11-06) | reference/prefabs/shadow-battleaxe.md |
| sword_lunarplant | prefabs/sword_lunarplant.lua(+同文件 blade_fx) | HANDS 亮茄纯武器(38+位面30,套装监听,破损可修,200用) | accepted(2026-11-06) | reference/prefabs/sword_lunarplant.md |
| voidcloth_scythe | prefabs/voidcloth_scythe.lua(+同文件 fx、classified 文件) | HANDS 暗影收割者(Shadow Reaper):SCYTHE 群体采收、38+位面18、影阵营说话、沃拓克斯影契 AoE;残破可修 | accepted(2026-11-06) | reference/prefabs/voidcloth_scythe.md |
| boomerang | prefabs/boomerang.lua | HANDS 回旋镖(本体即投射物,接回链完整;27.2/10用;配方炼金引擎档) | accepted(2026-11-06) | reference/prefabs/boomerang.md |
| voidcloth_boomerang | prefabs/voidcloth_boomerang.lua(+同文件 fx/proj) | HANDS 阴郁回旋镖(Gloomerang):充能代理投射、伤害随尺寸 5→27.2、不可接回、放电退化近战、85 用掷出即磨 | accepted(2026-11-06) | reference/prefabs/voidcloth_boomerang.md |
| blowdart | prefabs/blowdart.lua | HANDS 吹箭五变体(睡眠/火焰/吹管/雷电/海象) | accepted(2026-11-07) | reference/prefabs/ranged-weapons.md |
| houndstooth_blowpipe | prefabs/houndstooth_blowpipe.lua | HANDS 犬齿吹管 | accepted(2026-11-07) | reference/prefabs/ranged-weapons.md |
| slingshot(+slingshotmodkit) | prefabs/slingshot.lua、slingshotmodkit.lua | HANDS 弹弓,弹药系统 | accepted(2026-11-07) | reference/prefabs/ranged-weapons.md |
| whip | prefabs/whip.lua | HANDS 三尾猫鞭(加长近战 5 射程;超响鞭打断无伤害;27.2/175 用) | accepted(2026-11-06) | reference/prefabs/whip.md |
| trident | prefabs/trident.lua | HANDS 刺耳三叉戟(近战 51/76.5 按攻击者位置;水爆 85/4 耐久;非取水工具;蓝图链蟹王/Klaus) | accepted(2026-11-06) | reference/prefabs/trident.md |
| dumbbells | prefabs/dumbbells.lua | HANDS 哑铃族 7 注册(石/金/大理石/宝石/热/火/冰):举重涨体格+mighty 投掷 AOE;strongman/技能门 | accepted(2026-11-06) | reference/prefabs/dumbbells.md |
| wathgrithr_shield | prefabs/wathgrithr_shield.lua | HANDS 战斗圆盾(armor 420/85%,招架链,双层角色门禁) | accepted(2026-11-06) | reference/prefabs/wathgrithr_shield.md |
| lucy | prefabs/lucy.lua(+lucy_classified) | HANDS 露西斧:无限耐久、CHOP 效率2、会说话;非伍迪掉落;三条还原路径 | accepted(2026-11-06) | reference/prefabs/lucy.md |
| pocketwatch 族 | prefabs/pocketwatch*.lua(6 文件 16 注册) | HANDS 旺达怀表族:本页主责 11 注册(五表+警钟+零件+工具+裂缝口+跨片媒介);5 个 FX 注册归既有 warp_marker/weapon_fx 页 | accepted(2026-11-06,complex 189 行) | reference/prefabs/pocketwatch.md |
| spear_wathgrithr / spear_lance / spear_gungnir / rabbitkingspear / yoth_lance | 各自 lua | HANDS 事件/角色武器(Forge 两件 blocked) | accepted(2026-11-07,event-weapons.md;Forge 两件 blocked) | reference/prefabs/event-weapons.md |
| cutless | prefabs/cutless.lua | HANDS 木头短剑(官方名,非"弯刀"):粉猴生态获取无配方、命中偷窃敲落、27.2/150 用 | accepted(2026-11-06) | reference/prefabs/cutless.md |
| gnarwail_horn | prefabs/gnarwail_horn.lua | 施法器非武器(官方名一角鲸的角):点海召唤双浪+农田照料;耐久10;三叉戟原料 | accepted(2026-11-06) | reference/prefabs/gnarwail_horn.md |
| bullkelp_root | prefabs/bullkelp_root.lua | HANDS 公牛海带茎(官方名):新鲜度即寿命40天、每击掷点折断(幸运反向)、whip 同族;植物被毁获得 | accepted(2026-11-06) | reference/prefabs/bullkelp_root.md |
| sleepbomb / pumpkin_bomb / bomb_lunarplant / snowball_item | 各自 lua | 投掷物,持握接线已核 | accepted(2026-11-07) | reference/prefabs/throwables.md |
| glasscutter(武器侧) | prefabs/glasscutter.lua | 批次 1b 覆盖主体,武器章节同页 | — | reference/prefabs/glasscutter.md |

### 批次 3:照明装备——accepted(2026-11-07)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| torch | prefabs/torch.lua(torchfire 族) | HANDS 火炬:75 秒燃尽、攻击引燃、右键投掷 | accepted(2026-11-06) | reference/prefabs/torch.md |
| lighter | prefabs/lighter.lua | HANDS 薇洛打火机:600 秒仅装备燃烧、低理智起火死键、光半径两端不对称 | accepted(2026-11-06) | reference/prefabs/lighter.md |
| nightstick | prefabs/nightstick.lua | HANDS 晨星锤(官方名):电击武器 360 秒照明、电池/紫宝石充电 | accepted(2026-11-07) | reference/prefabs/nightstick.md |
| lantern | prefabs/mininglantern.lua(注册名 lantern) | HANDS 提灯:machine 开关、468 秒可充电耗尽不销毁、无武器组件、配方不含萤火虫 | accepted(2026-11-07) | reference/prefabs/lantern.md |
| redlantern / yots_redlantern | prefabs/redlantern.lua | HANDS 一次性 MAGIC 燃料 12 游戏日、常亮不可开关不可补;可被任何火源点燃 3 秒烧毁;幸运金块×3 供品配方;蛇年变体假灯笼诱蠕虫 | accepted(2026-11-07) | reference/prefabs/redlantern.md |
| queen_torch | prefabs/queen_torch.lua | HANDS 女王火炬:永燃不耗尽、驱影门控 16²、独立联网光源 queen_torchlight、生成点为 Charlie 战败仪式(非蜂后掉落) | accepted(2026-11-07) | reference/prefabs/queen-torch.md |
| yots_worm_lantern | prefabs/yots_worm_lantern.lua | 无 equippable(世界物件) | excluded(批内复核后) | — |
| floatinglantern | prefabs/floatinglantern.lua | 世界物件 | excluded | — |
| moonstorm_static | prefabs/moonstorm_static.lua | 装备主体仅 catcher;item/roamer/nowag 为世界/附属 | accepted(2026-11-07) | reference/prefabs/staffs-more.md |

### 批次 4:头部装备(hats.lua 85 玩法帽注册,拆 3 页)——accepted(2026-11-07)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| 防护/战斗帽 25 注册:footballhat、ruinshat、slurtlehat、skeletonhat、beehat、hivehat、eyemaskhat、wathgrithrhat、wathgrithr_improvedhat、cookiecutterhat、dreadstonehat、lunarplanthat、voidclothhat、wagpunkhat、scraphat、mermarmorhat、mermarmorupgradedhat、shadow_thrall_parasitehat、mask_halfwit/sage/toadyhat、woodcarvedhat、mask_princesshat、yoth_knighthat、alterguardianhat | prefabs/hats.lua(MakeHat→`xhat`) | HEAD armor/战斗 | accepted(2026-11-07) | reference/prefabs/hats-armor.md |
| 环境/功能帽 32 注册:straw、earmuffs、winter、catcoon、walrus、beefalo、flower、kelp、watermelon、ice、rain、eyebrella、miner、mole、desert、goggles、moonstorm_goggles、plantregistry、nutrientsgoggles、inspectacles、roseglasses、scrap_monocle、bush、nightcap、spider、merm、antlion、bat_bosscorpse、蘑菇×4 | prefabs/hats.lua | HEAD 保温/防水/视觉/伪装/改造 | accepted(2026-11-07) | reference/prefabs/hats-environment.md |
| 事件/装扮帽 28 注册:top、feather、walter、ghostflower、dragonhead/body/tail、pumpkin、mask_doll 系×9、mask_ancient×3、mask_charlie_boss、monkey×2、polly_rogers、salty_dog、batnose、balloon、rabbit | prefabs/hats.lua | HEAD 事件/装饰/角色 | accepted(2026-11-07) | reference/prefabs/hats-event.md |
| 附属 FX(hats.md) | 既有 FX 页 | FX 会话第 15 轮处置自行修订(保留),三帽页互链回补 | already | reference/prefabs/hats.md |

### 批次 5:身体装备——accepted(2026-11-07)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| armorgrass/armorwood/armormarble/armorruins/armorsnurtleshell/armor_sanity(暗夜甲)/armorslurper(饥饿腰带,无 armor)/armorskeleton(无 armor,resistance 免伤)/armor_rocky/armor_carrotlure(无 armor,引兔) | armor_grass/marble/ruins/snurtleshell/sanity/skeleton/rocky/carrotlure.lua、armorwood.lua | BODY 传统甲+功能装备 10 注册 | accepted(2026-11-07) | reference/prefabs/armor-classic.md |
| raincoat/hawaiianshirt/sweatervest/reflectivevest/trunkvest_summer/winter/beargervest/balloonvest/carnival_vest_a/b/c/onemanband(BODY) | raincoat.lua 等 9 文件 | BODY 衣物/背心 12 注册 | accepted(2026-11-07) | reference/prefabs/clothing.md |
| armordreadstone/armor_voidcloth(+fx)/armor_lunarplant/armor_lunarplant_husk(plantkin) | armor_dreadstone/voidcloth/lunarplant.lua | BODY 裂隙甲 4 注册 | accepted(2026-11-07) | reference/prefabs/armor-rift.md |
| armor_bramble(荆棘外壳)/armordragonfly/armorwagpunk(+fx)/armor_yoth_knight | armor_bramble/dragonfly/wagpunk/yoth_knight.lua | BODY Boss/特殊甲 4 注册 | accepted(2026-11-07) | reference/prefabs/armor-boss.md |
| umbrella/grass_umbrella(同文件)/voidcloth_umbrella(+fx)/winona_telebrella(+fx) | umbrella.lua、voidcloth_umbrella.lua、winona_telebrella.lua | HANDS 伞 4 注册 | accepted(2026-11-07) | reference/prefabs/umbrella.md |
| minifan | prefabs/minifan.lua | weapon+equippable+heater,HANDS | accepted(2026-11-07) | reference/prefabs/living-tools.md |

### 批次 6:背包与背部装备——accepted(2026-11-07)

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| backpack | 已有 | BODY 八格 | already | reference/prefabs/backpack.md |
| piggyback | prefabs/piggyback.lua | BODY 12 格减速 0.9 无耐久不可燃 | accepted(2026-11-07) | reference/prefabs/piggyback.md |
| krampus_sack | prefabs/krampus_sack.lua | BODY 14 格,坎普斯 1%+克劳斯袋 10% | accepted(2026-11-07) | reference/prefabs/krampus_sack.md |
| icepack | prefabs/icepack.lua | BODY 8 格 fridge+nocool 保鲜不降温 | accepted(2026-11-07) | reference/prefabs/icepack.md |
| candybag | prefabs/candybag.lua | BODY 14 格糖果过滤,HALLOWED_NIGHTS 科技(分诊结论:独立成页) | accepted(2026-11-07) | reference/prefabs/candybag.md |
| seedpouch / spicepack / potatosack(健身配重)/ beard_sack_1/2/3(BEARD 槽) | 各自 lua | 小袋族 6 注册 | accepted(2026-11-07) | reference/prefabs/pouch-bags.md |
| wortox_nabbag | prefabs/wortox_nabbag.lua | HANDS 强抢袋(非容器,分诊:独立成页) | accepted(2026-11-07) | reference/prefabs/wortox-nabbag.md |

### 批次 8:152 文件终扫差额(2026-11-07 终对账发现,拆 3 页;批 9 补遗、批 10 拆分后闭合)——accepted(2026-11-07)

**落页**:ranged-weapons.md(14 注册,complex,148 行/31181B)、staffs-more.md(12 注册,complex,181 行/38430B)、misc-equipment-2.md(38 固名注册+veggies 巨型作物 14×3 工厂,complex,252 行/49151B 仅 1B 余量;已于批 10 拆分为五页)。waterplant 分诊:waterplant.lua 命中为 boss 运行时临时武器(不注册 prefab)excluded;waterplant_seed 的 waterplant_bomb 种壳玩家可拾取投掷 included。moonstorm_static 装备主体裁定:仅 moonstorm_static_catcher 可装备。

终对账(152 equippable 文件 vs 已覆盖/排除)发现的未覆盖确认候选,主代理 grep 分诊:

> 批 10 已把 misc-equipment-2 按主题拆分;下表「页面」列的 misc-equipment-2 是批 8 历史归属,当前页见批次 10 节。

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| blowdart_sleep/fire/pipe/yellow/walrus(+impact) | prefabs/blowdart.lua | HANDS 吹箭 5+ 变体 | accepted(2026-11-07,ranged-weapons.md) | reference/prefabs/ranged-weapons.md |
| houndstooth_blowpipe(+proj) | prefabs/houndstooth_blowpipe.lua | HANDS 犬齿吹管 | accepted(2026-11-07,ranged-weapons.md) | reference/prefabs/ranged-weapons.md |
| slingshot(+slingshotmodkit 另文件) | prefabs/slingshot.lua | HANDS 弹弓 | accepted(2026-11-07,ranged-weapons.md) | reference/prefabs/ranged-weapons.md |
| mosquitobomb(=mosquito 爆炸态) | prefabs/mosquitobomb.lua | 投掷爆炸物 | accepted(2026-11-07,ranged-weapons.md) | reference/prefabs/ranged-weapons.md |
| staff_lunarplant(+fx) | prefabs/staff_lunarplant.lua | HANDS 亮茄法杖 | accepted(2026-11-07,staffs-more.md) | reference/prefabs/staffs-more.md |
| staff_tornado(+tornado) | prefabs/staff_tornado.lua | HANDS 龙卷风魔杖 | accepted(2026-11-07,staffs-more.md) | reference/prefabs/staffs-more.md |
| king_cane(+fx) | prefabs/king_cane.lua | HANDS 王杖(Charlie 遗物,与 queen_torch 同组) | accepted(2026-11-07,staffs-more.md) | reference/prefabs/staffs-more.md |
| yoth_knightstick | prefabs/yoth_knightstick.lua | HANDS 骑士杖(YOTH) | accepted(2026-11-07,staffs-more.md) | reference/prefabs/staffs-more.md |
| moonstorm_static(+item/catcher/roamer/nowag) | prefabs/moonstorm_static.lua | 静电杆系统(月暴);装备主体仅 catcher | accepted(2026-11-07,staffs-more.md) | reference/prefabs/staffs-more.md |
| shieldofterror | prefabs/shieldofterror.lua | HANDS 恐惧盾 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| balloonparty(+confetti 系) | prefabs/balloonparty.lua | Wes 气球派对机 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| beeswax_spray | prefabs/beeswax_spray.lua | 蜂蜡喷雾 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| bernie_inactive(+bernie_active) | prefabs/bernie_inactive.lua | 伯尼(equippable 仅 inactive;active 为活体) | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| carnival_golfclub | prefabs/carnival_golfclub.lua | 狂欢节高尔夫球杆 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| messagebottle(+empty,2 equippable) | prefabs/messagebottle.lua | 漂流瓶 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| moon_altar_pieces | prefabs/moon_altar_pieces.lua | 月坛部件(可持有) | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| vault_compass(+marker/visual) | prefabs/vault_compass.lua | 金库罗盘 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| veggies(巨型作物 heavy 持握,3 equippable) | prefabs/veggies.lua | BODY heavy | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| wurt_terraform_item(+projectile 系) | prefabs/wurt_terraform_item.lua | Wurt 地形改造物 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| wx78_drone_zap(+remote) | prefabs/wx78_drone_zap.lua | WX78 扫描模块 | accepted(2026-11-07,misc-equipment-2.md) | reference/prefabs/misc-equipment-2.md |
| waterplant / waterplant_seed(+burr/bomb 系) | 各自 lua | 海草/种子;waterplant 命中为 boss 运行时武器排除,waterplant_seed 种壳玩家可持有 | 分诊(2026-11-07) | reference/prefabs/misc-equipment-2.md |

### 批次 9:152 文件终扫补遗(2026-11-07,拆 1 页)——accepted(2026-11-07)

**落页**:heavy-items.md(standard,104 行;glassspike 五注册＋oceantreenut 一注册,BODY heavy)。批 8 的 21 候选表未包含这两个文件;补建后 152 文件口径闭合。

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| glassspike_short/med/tall、glassspike、glassblock | prefabs/glass_spike.lua | BODY heavy 玻璃化障碍;item/obstacle 双态、锤毁无掉落、沉水可绞盘取回 | accepted(2026-11-07) | reference/prefabs/heavy-items.md |
| oceantreenut | prefabs/oceantreenut.lua | BODY heavy 疙瘩树果;大树干撞船掉落、沉水计时长成 oceantree_short | accepted(2026-11-07) | reference/prefabs/heavy-items.md |

### 批次 10:杂项装备按主题拆分(2026-11-07,拆 5 页)——accepted(2026-11-07)

**落页**:giant-crops.md(80 行,veggies 14×3 巨型作物)、moon-altar-pieces.md(71 行,6 部件+3 岩石)、event-and-carnival-gear.md(76 行,恐怖盾牌/派对气球/防腐喷雾/推杆)、utility-and-throwables.md(79 行,瓶中信系/路标导航仪/种壳)、character-gear-extra.md(78 行,伯尼/投泥带/电刑机)。misc-equipment-2.md 退役;heavy-items.md 保留 glassspike/oceantreenut。

| 原 misc-equipment-2 对象 | 源码入口 | 现页面 |
|---|---|---|
| glassspike_short/med/tall、glassspike、glassblock、oceantreenut | prefabs/glass_spike.lua、oceantreenut.lua | reference/prefabs/heavy-items.md(批 9 保留) |
| veggies 巨型作物三变体 | prefabs/veggies.lua | reference/prefabs/giant-crops.md |
| moon_altar_pieces 部件与岩石 | prefabs/moon_altar_pieces.lua | reference/prefabs/moon-altar-pieces.md |
| shieldofterror、balloonparty、beeswax_spray、carnivalgame_golfclub | 各自 lua | reference/prefabs/event-and-carnival-gear.md |
| messagebottle 系、vault_compass、waterplant_bomb | 各自 lua | reference/prefabs/utility-and-throwables.md |
| bernie 系、wurt 投泥带、wx78 电刑机 | 各自 lua | reference/prefabs/character-gear-extra.md |

**排除(终对账复核)**:books/ghostly_elixirs/abigail_flower 无 `AddComponent("equippable")`(books 为 reader 消费、elixirs 为药剂、flower 非装备);bat_boss/malbatross/monkey/spider/walrus/snowman/spat/slurper/eyeturret/chesspieces/cavein_boulder/propsign/sculpture_pieces/shell_cluster/treasurechest/gestalt_cage/fence_rotator/carnivalgame_feedchicks/waterprojectiles 为生物/建筑/消费者命中;hats 为 FX 页已建。

### 批次 7:法杖、护符与角色专属/杂项——accepted(2026-11-07)

| 候选注册名 | 状态 | 页面 |
|---|---|---|
| staff.lua 七注册(icestaff/icestaff2/icestaff3/firestaff/telestaff/orangestaff/greenstaff) | accepted | reference/prefabs/staffs.md |
| amulet 六注册+yellowamuletlight(BODY 槽非 NECK,勘正) | accepted | reference/prefabs/amulets.md |
| bugnet/thulecitebugnet/brush/wateringcan/premiumwateringcan/pitchfork/goldenpitchfork/minifan | accepted | reference/prefabs/living-tools.md |
| oar 五桨/fishingrod/oceanfishingrod/fishingnet(unused) | accepted | reference/prefabs/fishing-tools.md |
| rabbitkingspear/yoth_lance/spear_wathgrithr 三件套 | accepted | reference/prefabs/event-weapons.md |
| sleepbomb/pumpkin_bomb/bomb_lunarplant/snowball_item(+2FX) | accepted | reference/prefabs/throwables.md |
| thurible/compass/diviningrod/walking_stick/witchbroom(+fx) | accepted | reference/prefabs/tools-misc.md |
| pillow 系 8/costume 8/firepen/merm_tool 2/reskin_tool/saddlehorn/chum/bootleg/balloonspeed/balloon(26 注册) | accepted | reference/prefabs/role-tools.md |
| spear_gungnir/spear_lance(Forge) | blocked(equippable 在编译事件数据内,快照不可读,证据不足) | 分诊结论记于 event-weapons.md |
| books/ghostly_elixirs/abigail_flower | excluded(无 AddComponent("equippable")) | — |
| nightmareamulet | excluded(源码零出现) | — |

## 排除/阻塞记录

| 对象 | 依据 | 日期 |
|---|---|---|
| nightmareamulet | `raw/current/scripts` 全目录 grep `nightmareamulet` 零命中;为单机版内容 | 2026-11-05 |
| *_classified(inventory、shadow_battleaxe、voidcloth_scythe、lucy 等) | 分类代理实体,非装备主体 | 2026-11-05 |
| player_common / wilson / willow / winona / woodie / wolfgang / wormwood / wx78 等角色工厂文件 | 角色工厂,非物品 | 2026-11-05 |
| 生物/建筑消费者文件(bat_boss、bearger、bunnyman、merm、pigman、walrus、spider、snowman、spat、treasurechest、eyeturret、punchingbag、propsign、chesspieces、sculpture_pieces、shell_cluster、cavein_boulder、dropperweb、spiderden、spiderhole、moonspiderden、monkey、slurper、hermitcrab、gelblob、storage_robot、wx78_possessedbody、vault_security_desk、daywalker_pillar、charlie_stage、stalker、worm、rocky_boss、malbatross 等) | 持有/读取装备的消费者或生物实体,非玩家可装备物品;逐批按内容确认后从候选划出,若有反例(如 slurper 帽)按实际处理 | 逐批 |

## 批次记录

- **批 1a+1b(2026-11-06 完成)**:axe/pickaxe/shovel/hammer/multitool_axe_pickaxe/glasscutter/farm-hoe 共 7 页 14 注册 accepted。主代理独立复审抽查全部关键外部证据(woodie Lucy 消费、PURE_SHADOW_TARGET_TAGS、tuning 值、配方、.po 名称键、glasscutter 纯武器结论、farm_hoe 工厂细节);机械检查子代理 6 页全绿,farm-hoe 由主代理补验(76 引用 0 错 0 警)。修复:tool 标签大小写跨页更正(`CHOP_tool`/`MINE_tool`/`DIG_tool`/`HAMMER_tool`,大写,4 页 7 处,消费端 componentactions/SGwilson_client/wx78_common 实证)、farmtiller.md 候选锚点 L2110-2113→L2104-2108、hammer sources 补 pickaxe_lunarplant.lua、axe/multitool 自记字节数校正、axe↔glasscutter 互链。收口:索引"手持工具"节 7 行登记、validate_page --check-indexes 通过(34 索引/1119 知识页,导航 0 问题)、record 7 页 check 0 漂移、git diff --check 干净。未运行游戏、未提交推送。
- **批 1c(2026-11-06 完成)**:fumarole-tools(complex,190 行,5 注册,共享 MakeFumaroleTool 协议:使用降温/冷却自毁/重热自修/工作攻击不磨损/heatrock 换热排除/WX-78 双重降温)与 lunarplant-tools(standard,141 行,2 注册,手工套装监听≠setbonus 组件/破损循环+亮茄修补套件/幸运触手门槛)accepted。主代理复审核实 HEAT_ON_USE 五键、WX-78 双 OnUsed、工厂协议、套装判定、获取途径(月亮锻造台)。机械检查 1 P2(lunarplant sources 漏 lunarrift_crystals.lua)已修,3 P3(.po 锚点/字节数/节选注明)已修。连带勘误两个组件页:fumaroletool.md 气孔陷阱加热表述纠偏(trapfumaroleburning 3°/s 才是工具升温链)、lunarplant_tentacle_weapon.md"镐/斧/铲"→"镐/铲/剑"(三宿主实查)。五基础工具页"待建气孔族页"注记全部替换为真实互链。索引"手持工具"节 +2 行(共 9 行),record 2 页 check 0 漂移,git diff --check 干净;--check-indexes 时点导航 false 仅因批次 2 的 spear.md 在建未登记。未运行游戏、未提交推送。
- **批 2(2026-11-06 完成)**:手持武器族 20 页 accepted(spear/batbat/hambat/nightsword/tentaclespike/ruins-bat/shadow-battleaxe/sword_lunarplant/voidcloth_scythe/boomerang/voidcloth_boomerang/whip/trident/dumbbells/wathgrithr_shield/lucy/pocketwatch/cutless/gnarwail_horn/bullkelp_root),索引「手持工具」节滚动登记、基线已记、--check-indexes 全绿;跨页修复 modmain 裸全局(GLOBAL. 前缀,8 页)与 tool 标签大小写遗留。**缺位注记**:本批因配额限流,独立机械检查子代理缺位、主代理内容复审未逐页留痕,按「补课计划」补做。
- **批 3(2026-11-07 收口)**:六页 accepted(torch/lighter/nightstick/lantern 2026-11-06/07;queen-torch/redlantern 2026-11-07)。收尾两页主代理独立复审:queen-torch 修复 P1×1(TOSS 收集表述方向反——componentactions 两处通用收集均排除 `special_action_toss`(L2074/L2285),投掷仅经 wilson.lua GetPointSpecialActions 的 `wilson_torch_7` 门控;补 keep_equip_toss 投掷不卸装机制),核过 TORCH_DAMAGE=17、QUEEN_TORCH_RADIUS_SQ=16²、7 Brain 行号逐一命中、atrium_gate 生成链、.po 四组键;redlantern 修复 P1×1(canlight 断言反向——burnable 默认 `canlight=true` 且镜像 `canlight` 标签(burnable.lua:6-8/50),MakeSmallBurnable 不覆写,LIGHT 动作(componentactions.lua:1481)、lighter 执行(lighter.lua:18-21)、闪电(componentutil.lua:1080)均可点燃,3 秒 TINY_BURNTIME 烧毁成灰;改写四处并留痕验证记录)+lootsetfn 锚点纠偏(L378-380→L260-264)+补 lighter/componentactions 两 sources,核过掉落表 50%、retarget HANDS 标签引诱、tuning 六键、配方行、PERDSHRINE=3＋九神龛各 1、torchranout 零监听、.po 两键。互链:torch.md/lantern.md 尾行回补两新页,lighter.md 仅 4 字节余量暂缺待补。索引「手持工具」节 +2 行(35 行),两页 strict 0/0、validate_page 通过,record 2 页 67 组 0 漂移,--check-indexes 导航 0 问题。未运行游戏、未提交推送。
