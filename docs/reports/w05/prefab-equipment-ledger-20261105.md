# 可装备 Prefab 扩库台账(2026-11-05)

本台账是「原版可装备物品 Prefab」扩库的工作状态来源。快照:steam-buildid **26335394**(`raw/current`)。主代理会话持续推进,逐批滚动;批次状态只记验收结论,过程细节入 `wiki/log.md`。

## 扫描口径与边界

- **主口径**:`raw/current/scripts/prefabs/*.lua` 中含 `AddComponent("equippable")` 的文件共 **152** 个(2026-11-05 grep 实测)。这是可装备物品的下界:生物/建筑持有或读取装备的消费者(如 bat_boss、monkey、snowman、walrus、treasurechest 等)在批次核验时按实际内容排除,不凭文件名预判。
- **辅口径**:`equipslot`/`EQUIPSLOTS` 引用(133 文件)用于交叉定位共享工厂(MakeHat、MakeAxe、MakeFumaroleTool、staff.lua、amulet.lua 等)与 `equipslot` 未显式赋值的文件(equippable 组件默认 HANDS,见 components/equippable.lua:29)。
- **字面 `Prefab("...")` 只算候选下界**;同文件多注册(如 axe.lua 4 注册、hats.lua `MakeHat("x")`→`xhat` 约 60 注册、farm_hoe.lua 3 注册)按工厂展开。
- 已有主体页复用:`heatrock`(库存物品非装备)、`backpack`、`cane`、`yellowstaff`、`opalstaff`、`stafflight`/`staffcoldlight`(生成物)。`hats.md` 为帽子附属 FX 页,不代表帽子主体已覆盖。
- 每批建页前仍须重核:已有页面、PLAN 认领、源码实际接线。

## 状态图例

`待建` → `in-flight`(写作子代理进行中)→ `待复审`(页面已交付)→ `accepted`(通过独立复审+机械检查+索引登记)。另有 `already`(已有主体页)、`待核验`(接线/归属未确认)、`excluded`(附依据)、`blocked`(附原因)。

## 批次与候选

### 批次 1a:基础工具(核心四件)——in-flight

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| axe / goldenaxe / moonglassaxe | prefabs/axe.lua(MakeAxe 工厂) | HANDS;CHOP 工具+武器 | in-flight | reference/prefabs/axe.md |
| pickaxe / goldenpickaxe | prefabs/pickaxe.lua | HANDS;MINE 工具+武器 | in-flight | reference/prefabs/pickaxe.md |
| shovel / goldenshovel | prefabs/shovel.lua | HANDS;DIG 工具+武器 | in-flight | reference/prefabs/shovel.md |
| hammer | prefabs/hammer.lua | HANDS;HAMMER 工具+武器 | in-flight | reference/prefabs/hammer.md |

### 批次 1b:工具补充——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| multitool_axe_pickaxe | prefabs/axe_pickaxe.lua | HANDS;CHOP+MINE 双工具、shadowlevel | 待建 | reference/prefabs/multitool_axe_pickaxe.md |
| glasscutter | prefabs/glasscutter.lua | HANDS;月玻璃刀,工具+武器 | 待建 | reference/prefabs/glasscutter.md |
| farm_hoe / golden_farm_hoe | prefabs/farm_hoe.lua(MakeFarmHoe) | HANDS;HOE 农具 | 待建 | reference/prefabs/farm-hoe.md |

### 批次 1c:特殊机制工具族——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| fumaroleaxe / fumarolepickaxe / fumaroleshovel / fumarolehammer / fumarole_farm_hoe | 各工具 lua + standardcomponents.lua(MakeFumaroleTool/MakeFumaroleToolPristine) | HANDS;气孔工具:温度档、热伤害、破损/修理循环 | 待建 | reference/prefabs/fumarole-tools.md |
| pickaxe_lunarplant / shovel_lunarplant | prefabs/pickaxe_lunarplant.lua、shovel_lunarplant.lua | HANDS;月亮植物工具:套装加成(lunarplanthat)、planardamage、Forge 修理、破损循环 | 待建 | reference/prefabs/lunarplant-tools.md |

### 批次 2:手持武器(拆 2~3 小批)——待建

| 候选注册名 | 源码入口 | 预计用途 | 状态 | 页面 |
|---|---|---|---|---|
| spear | prefabs/spear.lua | HANDS 基础长矛 | 待建 | reference/prefabs/spear.md |
| batbat | prefabs/batbat.lua | HANDS 吸血蝙蝠棒 | 待建 | reference/prefabs/batbat.md |
| hambat | prefabs/hambat.lua | HANDS 新鲜度驱动火腿棒 | 待建 | reference/prefabs/hambat.md |
| nightsword | prefabs/nightsword.lua | HANDS 暗夜剑 | 待建 | reference/prefabs/nightsword.md |
| tentaclespike | prefabs/tentaclespike.lua | HANDS 触手尖刺 | 待建 | reference/prefabs/tentaclespike.md |
| ruins_bat(图勒棍棒) | prefabs/ruins_bat.lua | HANDS 蓄力群攻 | 待建 | reference/prefabs/ruins-bat.md |
| shadow_battleaxe | prefabs/shadow_battleaxe.lua(+classified) | HANDS 暗影战斧,位面/吸血 | 待建 | reference/prefabs/shadow-battleaxe.md |
| sword_lunarplant | prefabs/sword_lunarplant.lua | HANDS 月亮植物剑,套装加成 | 待建 | reference/prefabs/sword_lunarplant.md |
| voidcloth_scythe | prefabs/voidcloth_scythe.lua(+classified) | HANDS 虚布镰刀 | 待建 | reference/prefabs/voidcloth_scythe.md |
| boomerang | prefabs/boomerang.lua | HANDS 回旋镖,projectile 接线 | 待建 | reference/prefabs/boomerang.md |
| voidcloth_boomerang | prefabs/voidcloth_boomerang.lua | HANDS 虚布回旋镖 | 待建 | reference/prefabs/voidcloth_boomerang.md |
| blowdart | prefabs/blowdart.lua | HANDS 吹箭(睡眠/火焰/减速变体待核验) | 待建 | reference/prefabs/blowdart.md |
| houndstooth_blowpipe | prefabs/houndstooth_blowpipe.lua | HANDS 犬齿吹管 | 待建 | reference/prefabs/houndstooth_blowpipe.md |
| slingshot(+slingshotmodkit) | prefabs/slingshot.lua、slingshotmodkit.lua | HANDS 弹弓,弹药系统 | 待建 | reference/prefabs/slingshot.md |
| whip | prefabs/whip.lua | HANDS 鞭子 | 待建 | reference/prefabs/whip.md |
| trident | prefabs/trident.lua | HANDS 三叉戟(取水/武器) | 待建 | reference/prefabs/trident.md |
| dumbbells | prefabs/dumbbells.lua | HANDS 沃尔夫冈哑铃(普通/黄金/宝石) | 待建 | reference/prefabs/dumbbells.md |
| wathgrithr_shield | prefabs/wathgrithr_shield.lua | HANDS 女武神盾 | 待建 | reference/prefabs/wathgrithr_shield.md |
| lucy | prefabs/lucy.lua(+lucy_classified) | HANDS 伍迪专属斧 | 待建 | reference/prefabs/lucy.md |
| pocketwatch 族 | prefabs/pocketwatch*.lua | HANDS 旺达怀表族 | 待建 | 核验后定(可能拆数页) |
| spear_wathgrithr / spear_lance / spear_gungnir / rabbitkingspear / yoth_lance | 各自 lua | HANDS 事件/角色武器 | 待建 | 核验后定 |
| cutless | prefabs/cutless.lua | HANDS 弯刀 | 待建 | 核验后定 |
| gnarwail_horn | prefabs/gnarwail_horn.lua | HANDS 号角武器 | 待建 | 核验后定 |
| bullkelp_root | prefabs/bullkelp_root.lua | HANDS 牛海带根武器 | 待建 | 核验后定 |
| sleepbomb / pumpkin_bomb / bomb_lunarplant / snowball_item | 各自 lua | 投掷物,持握接线待核验 | 待核验 | 核验后定 |
| glasscutter(武器侧) | prefabs/glasscutter.lua | 批次 1b 覆盖主体,武器章节同页 | — | reference/prefabs/glasscutter.md |

### 批次 3:照明装备——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| torch | prefabs/torch.lua(torchfire 族) | HANDS 火把 | 待建 | reference/prefabs/torch.md |
| lighter | prefabs/lighter.lua | HANDS 薇洛打火机 | 待建 | reference/prefabs/lighter.md |
| nightstick | prefabs/nightstick.lua | HANDS 光杖,燃料 | 待建 | reference/prefabs/nightstick.md |
| lantern | prefabs/mininglantern.lua(注册名 lantern) | HANDS 提灯,燃料 | 待建 | reference/prefabs/lantern.md |
| redlantern / yots_redlantern | prefabs/redlantern.lua | 槽位待核验 | 待核验 | 核验后定 |
| yots_worm_lantern | prefabs/yots_worm_lantern.lua | 待核验 | 待核验 | 核验后定 |
| queen_torch | prefabs/queen_torch.lua | 待核验(持握?) | 待核验 | 核验后定 |
| miniboatlantern | prefabs/miniboatlantern.lua | 船用挂灯? | 待核验 | — |
| floatinglantern | prefabs/floatinglantern.lua | 世界物件? | 待核验 | — |
| moonstorm_static | prefabs/moonstorm_static.lua | 用途待核验 | 待核验 | — |

### 批次 4:头部装备(hats.lua 约 60 注册,拆 3 页)——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| 防护/战斗帽:footballhat、ruinshat、slurtlehat、skeletonhat、mermarmor(?)、dreadstone、lunarplant、voidcloth、dragonhead/body/tail、beehat、spiderhat、icehat、eyemask 等 | prefabs/hats.lua(MakeHat→`xhat`) | HEAD 护甲 | 待建 | reference/prefabs/hats-armor.md |
| 环境/功能帽:straw、winter、earmuffs、catcoon、walrus、beefalo、flower、watermelon、desert、bush、rain、kelp、nightcap、miner、eyebrella、goggles、moonstorm_goggles、mole(夜视)、nutrientsgoggles、inspectacles、roseglasses、scrap_monocle 等 | prefabs/hats.lua | HEAD 环境/照明/查看 | 待建 | reference/prefabs/hats-environment.md |
| 事件/角色帽:mask_*、monkey_*、polly_rogers、salty_dog、woodcarved、pumpkin、ghostflower、bat_bosscorpse、alterguardian、yoth_knight、wagpunk、蘑菇帽、top、rabbit、merm、cookiecutter、batnose、wathgrithr(假发)、walter、antlion、mask_charlie_boss、shadow_thrall_parasite 等 | prefabs/hats.lua | HEAD 事件/装饰/角色 | 待建 | reference/prefabs/hats-event.md |
| 附属 FX(hats.md) | 既有 FX 页 | 保持不动,主体页建后回补互链 | already | reference/prefabs/hats.md |

### 批次 5:身体装备(拆 3~4 小批)——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| armorgrass / armorwood / armormarble / armorruins | armor_grass/wood/marble/ruins.lua | BODY 传统护甲 | 待建 | 合一页 armor-classic.md |
| armorslurper / armorsnurtleshell / armor_sanity / armorskeleton / armor_rocky / armor_carrotlure | 各自 lua | BODY 特殊护甲,具体待核验 | 待建 | 合入 armor-classic.md 或独立小节 |
| armor_bramble / armor_dragonfly / armor_lunarplant / armor_voidcloth / armor_wagpunk / armor_dreadstone / armor_yoth_knight | 各自 lua | BODY Boss/裂隙护甲(套装、反射等) | 待建 | 每件一页或小族页,批内核验定 |
| raincoat / hawaiianshirt / sweatervest / reflectivevest / trunkvest / beargervest / balloonvest / carnivalvest | 各自 lua | BODY 衣服与背心 | 待建 | 合一页 clothing.md |
| onemanband | prefabs/onemanband.lua | BODY 一人乐队 | 待建 | 并入 clothing.md |
| minifan | prefabs/minifan.lua | 槽位待核验 | 待核验 | 核验后定 |
| umbrella / grass_umbrella / voidcloth_umbrella / winona_telebrella | umbrella.lua、voidcloth_umbrella.lua、winona_telebrella.lua | HANDS?伞类 | 待建 | 合一页 umbrella.md(winona_telebrella 角色专属核验) |

### 批次 6:背包与背部装备——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| backpack | 已有 | BODY 八格 | already | reference/prefabs/backpack.md |
| piggyback | prefabs/piggyback.lua | BODY 猪皮包 | 待建 | reference/prefabs/piggyback.md |
| krampus_sack | prefabs/krampus_sack.lua | BODY 小偷背包 | 待建 | reference/prefabs/krampus_sack.md |
| candybag | prefabs/candybag.lua | BODY 糖果袋 | 待建 | 核验后定(独立或并入 piggyback) |
| icepack | prefabs/icepack.lua | BODY 保鲜包 | 待建 | reference/prefabs/icepack.md |
| seedpouch / spicepack / potatosack | 各自 lua | BODY 小袋(过滤) | 待建 | 合一页 pouch-bags.md |
| beard_sack | prefabs/beard_sack.lua | BODY 待核验 | 待建 | 核验后定 |
| wortox_nabbag | prefabs/wortox_nabbag.lua | 沃拓克斯角色袋 | 待建 | 核验后定 |

### 批次 7:法杖、护符与角色专属/杂项——待建

| 候选注册名 | 源码入口 | 预计槽位/用途 | 状态 | 页面 |
|---|---|---|---|---|
| icestaff / icestaff2 / icestaff3 / firestaff / telestaff / orangestaff / greenstaff | prefabs/staff.lua | HANDS 施法武器 | 待建 | 合一页 staffs.md(yellowstaff/opalstaff 已有页互链) |
| amulet / blueamulet / purpleamulet / orangeamulet / greenamulet / yellowamulet | prefabs/amulet.lua | NECK 护符 | 待建 | 合一页 amulets.md |
| thurible | prefabs/thurible.lua | 槽位待核验 | 待核验 | 核验后定 |
| compass | prefabs/compass.lua | 装备槽待核验 | 待核验 | 核验后定 |
| diviningrod | prefabs/diviningrod.lua | HANDS 探测杖 | 待建 | reference/prefabs/diviningrod.md |
| bugnet / brush / wateringcan / pitchfork | 各自 lua | HANDS 生活工具 | 待建 | 合页或各一页,批内定 |
| abigail_flower | prefabs/abigail_flower.lua | equippable 组件零命中,接线待核验 | 待核验 | 核验后定 |
| ghostly_elixirs | prefabs/ghostly_elixirs.lua | 非装备? | 待核验 | — |
| costumes | prefabs/costumes.lua | YotC 服装,接线待核验 | 待核验 | 核验后定 |
| pillow_equipment | prefabs/pillow_equipment.lua | 活动装备 | 待建 | 核验后定 |
| witchbroom | prefabs/witchbroom.lua | 待核验 | 待核验 | 核验后定 |
| firepen / merm_tool / reskin_tool / saddlehorn / chum / bootleg / balloonspeed / balloon 等 | 各自 lua | 杂项/事件,接线待核验 | 待核验 | 逐个核验后分批 |
| oar / fishingrod / oceanfishingrod / fishingnet | 各自 lua | HANDS 水上工具 | 待建 | 各一页 |
| walking_stick | prefabs/walking_stick.lua | 待核验 | 待核验 | 核验后定 |
| books | prefabs/books.lua | 非装备(reader 消耗)? | 待核验 | — |
| nightmareamulet | 无 | **源码零出现**(prefabs/components 全 grep 无命中),单机版内容 | excluded | — |

## 排除/阻塞记录

| 对象 | 依据 | 日期 |
|---|---|---|
| nightmareamulet | `raw/current/scripts` 全目录 grep `nightmareamulet` 零命中;为单机版内容 | 2026-11-05 |
| *_classified(inventory、shadow_battleaxe、voidcloth_scythe、lucy 等) | 分类代理实体,非装备主体 | 2026-11-05 |
| player_common / wilson / willow / winona / woodie / wolfgang / wormwood / wx78 等角色工厂文件 | 角色工厂,非物品 | 2026-11-05 |
| 生物/建筑消费者文件(bat_boss、bearger、bunnyman、merm、pigman、walrus、spider、snowman、spat、treasurechest、eyeturret、punchingbag、propsign、chesspieces、sculpture_pieces、shell_cluster、cavein_boulder、dropperweb、spiderden、spiderhole、moonspiderden、monkey、slurper、hermitcrab、gelblob、storage_robot、wx78_possessedbody、vault_security_desk、daywalker_pillar、charlie_stage、stalker、worm、rocky_boss、malbatross 等) | 持有/读取装备的消费者或生物实体,非玩家可装备物品;逐批按内容确认后从候选划出,若有反例(如 slurper 帽)按实际处理 | 逐批 |

## 批次记录

- **批 1a(2026-11-05)**:axe/pickaxe/shovel/hammer 4 页派发写作子代理,in-flight。其余批次未开工。
