---
title: "环境与功能帽族（strawhat 等 32 注册）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "hats.lua 中 32 顶环境／功能帽（HEAD 槽）的工厂、组件组合与消费端：保温／防水／降温、矿工帽照明、鼹鼠帽夜视与护目镜族视觉标签、灌木丛帽隐身与牛角帽／蜘蛛帽／鱼人帽／蝙蝠帽伪装、蚁狮帽地形改造、蘑菇帽孢子温控、耕作与信号眼镜；含 fueled 装备期消耗链与配方、官方名。"
aliases: ["strawhat", "earmuffshat", "winterhat", "catcoonhat", "walrushat", "beefalohat", "flowerhat", "kelphat", "watermelonhat", "icehat", "rainhat", "eyebrellahat", "minerhat", "molehat", "deserthat", "goggleshat", "moonstorm_goggleshat", "nightcaphat", "bushhat", "plantregistryhat", "nutrientsgoggleshat", "inspectacleshat", "roseglasseshat", "scrap_monoclehat", "spiderhat", "mermhat", "antlionhat", "bat_bosscorpsehat", "red_mushroomhat", "green_mushroomhat", "blue_mushroomhat", "moon_mushroomhat", "蜘蛛帽", "聪明的伪装", "刮地皮头盔", "蝙蝠帽", "红蘑菇帽", "绿蘑菇帽", "蓝蘑菇帽", "月亮蘑菇帽", "蘑菇帽", "草帽", "兔耳罩", "冬帽", "猫帽", "贝雷帽", "牛角帽", "花环", "海花环", "西瓜帽", "冰帽", "雨帽", "眼球伞", "矿工帽", "鼹鼠帽", "沙漠护目镜", "时髦的护目镜", "星象护目镜", "睡帽", "灌木丛帽", "耕作先驱帽", "高级耕作先驱帽", "检查专用镜", "玫瑰色眼镜", "视界扩展器", "保温帽", "防水帽", "夜视帽", "护目镜", "防沙尘暴"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hats.lua", "recipes.lua", "tuning.lua", "componentactions.lua", "constants.lua", "simutil.lua", "languages/chinese_s.po", "components/playervision.lua", "components/fueled.lua", "components/equippable.lua", "components/inventory.lua", "components/playerlightningtarget.lua", "components/sandstormwatcher.lua", "components/moonstormwatcher.lua", "components/miasmawatcher.lua", "components/sleepingbaguser.lua", "components/combat.lua", "components/heater.lua", "components/temperature.lua", "components/moisture.lua", "components/moonsparkchargeable.lua", "components/farmplantstress.lua", "components/grue.lua", "components/sanity.lua", "stategraphs/SGwilson.lua", "prefabs/beefalo.lua", "prefabs/walrus.lua", "prefabs/moonstorm_spark.lua", "prefabs/moonstorm_static.lua", "prefabs/wagstaff_npc.lua", "prefabs/wormlight.lua", "prefabs/lightbulb.lua", "prefabs/fireflies.lua", "prefabs/slurtleslime.lua", "prefabs/daywalker2.lua", "prefabs/yotr_rabbitshrine.lua", "prefabs/bundle.lua", "components/autoterraformer.lua", "components/periodicspawner.lua", "components/perishable.lua", "components/klaussackloot.lua", "components/playercontroller.lua", "componentutil.lua", "containers.lua", "prefabs/antlion.lua", "prefabs/bat_boss.lua", "prefabs/batbat.lua", "prefabs/gridplacer.lua", "prefabs/junk_pile_common.lua", "prefabs/leafpile.lua", "prefabs/merm.lua", "prefabs/mosquitomermsalve.lua", "prefabs/mushroom_farm.lua", "prefabs/mushroom_light.lua", "prefabs/mushtree_moonspore.lua", "prefabs/player_common.lua", "prefabs/spider.lua", "prefabs/spiderqueen.lua", "prefabs/toadstool.lua", "prefabs/tumbleweed.lua", "widgets/bloodover.lua", "widgets/healthbadge.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 环境与功能帽族（strawhat 等 32 注册）

## 快速判断

- 职责与适用：`prefabs/hats.lua` 经 `MakeHat` 注册 85 顶玩法帽，本页收其中 **32 顶环境／功能帽**（全部 HEAD 槽装备，纯装饰与战斗装区分页）：保温／防水／降温（草帽到眼球伞、冰帽、蘑菇帽 ×4）、照明与视觉（矿工帽、鼹鼠帽、护目镜族）、地形改造（蚁狮帽）、隐身／伪装／睡眠等状态功能（灌木丛帽、睡帽、蜘蛛帽、鱼人帽、蝙蝠帽）、耕作与信号眼镜（耕作先驱帽、检查专用镜等）。工厂与跟随层 FX 见 [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]。
- 优先使用：查"某顶帽有什么组件／耐久／配方"用[分族注册表](#注册与构造)；查"夜视／护目标签如何生效"用[视觉系统](#视觉标签与玩家视野)；查"燃料怎么补"用[燃料与耐久链](#燃料与耐久链)；查"伪装／隐身对 AI 的实际效果"用[伪装与隐身消费端](#伪装与隐身消费端)；蚁狮帽改地皮／蘑菇帽孢子用[地形改造与孢子链](#地形改造与孢子链)。
- 重要副作用：fueled 帽**装备期间**持续耗燃料、耗尽即移除并推 `torchranout`／`umbrellaranout` 等事件；视觉帽靠装备标签改玩家视野（客户端换色表／沙尘暴移速）；伪装帽装备期给持有者加 `monster` 与对应伪装标签并收编跟随者（卸下即撤并可能反目）；灌木丛帽使持有者进入 `hiding` 态、全 `Combat:SetTarget` 拒绝锁定。
- 修改边界：只覆盖帽本体组件组合、配方、官方名与消费端标签；帽的跟随层／帽灯／地面脸等 FX、皮肤 build 资源、动画包内容不在本页（见 hats.md 与[未核验项](#验证记录)）。数值均为快照源码推导值，未运行测量。

## 注册与构造

### 工厂与共同骨架

`MakeHat(name)`：`fname = "hat_"..name`（动画 build）、`symname = name.."hat"`（bank 与**注册名**）（[hats.lua#L16-L20](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)）。85 顶帽共用 `simple()` 骨架：Transform／AnimState／Network＋`hat` 标签＋snowmandecor，主模拟装 inventoryitem／inspectable／tradable／equippable（**equipslot 一律 `EQUIPSLOTS.HEAD`**）＋MakeInventoryFloatable＋MakeHauntableLaunch（[hats.lua#L188-L235](../../../raw/current/scripts/prefabs/hats.lua#L188-L235)）。装备表现三路：`_onequip`／`opentop_onequip`／`fullhelm_onequip` 把 build 的 `swap_hat` 符号盖到持有者（[hats.lua#L31-L171](../../../raw/current/scripts/prefabs/hats.lua#L31-L171)）。**凡带 fueled 的帽，装备回调经 `_base_onequip` 自动 `StartConsuming`、卸下／入袋自动 `StopConsuming`**（[hats.lua#L40-L42](../../../raw/current/scripts/prefabs/hats.lua#L40-L42)、[L101-L103](../../../raw/current/scripts/prefabs/hats.lua#L101-L103)）——燃料只在装备期消耗。皮肤走 `_base_onequip` 的 `GetSkinBuild`→`OverrideItemSkinSymbol`＋`skin_equip_sound`（[hats.lua#L31-L47](../../../raw/current/scripts/prefabs/hats.lua#L31-L47)），逐帽皮肤清单未核验。

### 本页注册分族表

单位换算（[tuning.lua#L19-L29](../../../raw/current/scripts/tuning.lua#L19-L29)、[L2213-L2217](../../../raw/current/scripts/tuning.lua#L2213-L2217)、[L2260-L2263](../../../raw/current/scripts/tuning.lua#L2260-L2263)、[L2949-L2954](../../../raw/current/scripts/tuning.lua#L2949-L2954)）：`total_day_time` = 480 秒（16 段×30）；insulator 保温值单位是**秒**（SMALL 60／MED 120／LARGE 240）；waterproofer 效果是比例（SMALL 20%／SMALLMED 35%／LARGE 70%／ABSOLUTE 100%）；dapperness 单位是**理智/秒**（TINY ≈10.7／SMALL ≈16／MED ≈26.7／LARGE ≈53.3 每天合计）。

**表A 保温／降温／防水（16 注册）**

| 注册名（官方名） | 组件与关键数值 | 获取与证据 |
|---|---|---|
| `strawhat`（草帽） | 夏季保温 SMALL＋防水 SMALL＋fuel（LARGE_FUEL，可点燃可燃烧）＋fueled USAGE 5 天耗尽自毁 | 配方 12 cutgrass NONE（[recipes.lua#L530](../../../raw/current/scripts/recipes.lua#L530)）；[hats.lua#L237-L271](../../../raw/current/scripts/prefabs/hats.lua#L237-L271) |
| `earmuffshat`（兔耳罩） | 冬季保温 SMALL＋fueled USAGE 5 天；opentop 装备（露发） | 配方 2 rabbit＋1 twigs NONE（[recipes.lua#L533](../../../raw/current/scripts/recipes.lua#L533)）；[hats.lua#L302-L329](../../../raw/current/scripts/prefabs/hats.lua#L302-L329) |
| `winterhat`（冬帽） | 冬季保温 MED＋dapperness TINY＋fueled USAGE 10 天 | 配方 4 牛毛＋4 蛛丝 SCIENCE_TWO（[recipes.lua#L535](../../../raw/current/scripts/recipes.lua#L535)）；[hats.lua#L331-L352](../../../raw/current/scripts/prefabs/hats.lua#L331-L352) |
| `catcoonhat`（猫帽） | 冬季保温 SMALL＋dapperness MED＋fueled USAGE 10 天 | 配方 1 浣猫尾＋4 蛛丝 SCIENCE_TWO（[recipes.lua#L536](../../../raw/current/scripts/recipes.lua#L536)）；[hats.lua#L1714-L1736](../../../raw/current/scripts/prefabs/hats.lua#L1714-L1736) |
| `walrushat`（贝雷帽） | 冬季保温 MED＋dapperness LARGE＋fueled USAGE 25 天 | 海象（MacTusk）掉落 25%（[walrus.lua#L23-L29](../../../raw/current/scripts/prefabs/walrus.lua#L23-L29)）；[hats.lua#L654-L675](../../../raw/current/scripts/prefabs/hats.lua#L654-L675) |
| `beefalohat`（牛角帽） | 冬季保温 LARGE＋防水 SMALL＋fueled USAGE 10 天＋**装备给持有者加 `beefalo` 标签**（[hats.lua#L605-L613](../../../raw/current/scripts/prefabs/hats.lua#L605-L613)） | 配方 8 牛毛＋1 牛角 SCIENCE_ONE（[recipes.lua#L534](../../../raw/current/scripts/recipes.lua#L534)）；[hats.lua#L625-L651](../../../raw/current/scripts/prefabs/hats.lua#L625-L651)；消费端见[伪装](#伪装与隐身消费端) |
| `flowerhat`（花环） | dapperness TINY＋`flipdapperonmerms`＋perishable 6 天耗尽自毁＋可堆肥 | 配方 12 petals NONE（[recipes.lua#L529](../../../raw/current/scripts/recipes.lua#L529)）；[hats.lua#L1142-L1172](../../../raw/current/scripts/prefabs/hats.lua#L1142-L1172) |
| `kelphat`（海花环） | dapperness **−TINY**＋`flipdapperonmerms`（鱼人反转收益）＋perishable 6 天＋可堆肥 | 配方 6 kelp NONE（[recipes.lua#L537](../../../raw/current/scripts/recipes.lua#L537)）；[hats.lua#L1174-L1203](../../../raw/current/scripts/prefabs/hats.lua#L1174-L1203) |
| `watermelonhat`（西瓜帽） | 装备降温目标 55 度（heater endo）＋夏季保温 MED＋装备期潮湿增速上限 32＋perishable 3 天→变 `spoiled_food`＋防水 SMALL＋dapperness −SMALL | 配方 1 西瓜＋3 twigs SCIENCE_ONE（[recipes.lua#L541](../../../raw/current/scripts/recipes.lua#L541)）；[hats.lua#L1738-L1783](../../../raw/current/scripts/prefabs/hats.lua#L1738-L1783) |
| `icehat`（冰帽） | 装备降温目标 40 度＋夏季保温 LARGE＋移速 ×0.9＋装备期潮湿增速上限 49＋perishable 8 天（耗尽给持有者 +30 潮湿）＋repairable(ICE)（修复材料见 [[reference/prefabs/ice]]）＋waterproofer **effectiveness=0** | 配方 2 绳＋10 冰 SCIENCE_TWO（[recipes.lua#L542](../../../raw/current/scripts/recipes.lua#L542)）；[hats.lua#L1660-L1712](../../../raw/current/scripts/prefabs/hats.lua#L1660-L1712) |
| `rainhat`（雨帽） | 防水 LARGE＋fueled USAGE 10 天＋`equippable.insulated = true`（**雷电绝缘**） | 配方 2 鼹鼠＋1 草帽＋1 骨片 SCIENCE_TWO（[recipes.lua#L532](../../../raw/current/scripts/recipes.lua#L532)）；[hats.lua#L1267-L1288](../../../raw/current/scripts/prefabs/hats.lua#L1267-L1288) |
| `eyebrellahat`（眼球伞） | 防水 **ABSOLUTE(100%)**＋夏季保温 LARGE＋雷电绝缘＋fueled USAGE 9 天＋pristine `umbrella` 标签＋装备放大持有者 DynamicShadow(2.2,1.4)；耗尽推 `umbrellaranout` | 配方 1 独眼鹿眼球＋15 twigs＋4 骨片 SCIENCE_TWO（[recipes.lua#L554](../../../raw/current/scripts/recipes.lua#L554)）；[hats.lua#L1290-L1363](../../../raw/current/scripts/prefabs/hats.lua#L1290-L1363) |
| `red_mushroomhat`／`green_mushroomhat`／`blue_mushroomhat`／`moon_mushroomhat`（红／绿／蓝／月亮蘑菇帽） | 共用 `common_mushroom` 工厂：perishable 6 天耗尽自毁＋forcecompostable＋装备期 periodicspawner 产孢子（red/green/blue→spore_medium/small/tall，60–61 秒/个；moon→spore_moon，1.5–1.75 秒/个且落近即爆）＋夏季保温 SMALL＋防水 SMALL＋装备期饥饿消耗 ×0.75；装备加 `spoiler`（背包食物腐坏 ×1.5），moon 版另加 `moon_spore_protection`＋受击经 `ShouldProcOnAttackedOrBlocked` 门禁排队 3 次 `periodicspawner:TrySpawn`（每次 0.25 秒间隔；重复命中把剩余尝试数重置为 3，已有任务时不另起一组） | 红/绿/蓝：6 对应色蘑菇 TECH.LOST（[recipes.lua#L555-L557](../../../raw/current/scripts/recipes.lua#L555-L557)），蓝图出自毒菌蟾蜍必掉其一（[toadstool.lua#L68-L71](../../../raw/current/scripts/prefabs/toadstool.lua#L68-L71)）与克劳斯布袋池（[klaussackloot.lua#L10-L21](../../../raw/current/scripts/components/klaussackloot.lua#L10-L21)）；月亮版 4 moon_cap＋1 红蘑菇帽 TECH.CELESTIAL_ONE `nounlock`（[recipes.lua#L834](../../../raw/current/scripts/recipes.lua#L834)）；[hats.lua#L1871-L2045](../../../raw/current/scripts/prefabs/hats.lua#L1871-L2045) |

**表B 照明／视觉／护目（11 注册）**

| 注册名（官方名） | 组件与关键数值 | 获取与证据 |
|---|---|---|
| `minerhat`（矿工帽） | fueled **FUELTYPE.CAVE**（吃荧光果／萤火虫／蜗牛粘液）燃时 (night+dusk)×2.6=468 秒＋防水 SMALL＋装备开关生成 `minerhatlight`（光参归 hats.md）＋耗尽推 `torchranout` | 配方 1 草帽＋1 金＋1 萤火虫 SCIENCE_TWO（[recipes.lua#L431](../../../raw/current/scripts/recipes.lua#L431)）；[hats.lua#L677-L787](../../../raw/current/scripts/prefabs/hats.lua#L677-L787) |
| `molehat`（鼹鼠帽） | pristine `nightvision` 标签＋fueled **FUELTYPE.WORMLIGHT** 1.5 天＋装备／卸下播 moggles 音 | 配方 2 鼹鼠＋2 电子件＋1 发光浆果 SCIENCE_TWO（[recipes.lua#L432](../../../raw/current/scripts/recipes.lua#L432)）；[hats.lua#L1785-L1839](../../../raw/current/scripts/prefabs/hats.lua#L1785-L1839) |
| `deserthat`（沙漠护目镜） | pristine `goggles` 标签（防沙尘暴）＋夏季保温 MED＋dapperness MED＋防水 SMALL＋fueled USAGE 10 天；**TECH.LOST 蓝图** | 湿布袋掉 `deserthat_blueprint`（[bundle.lua#L611-L628](../../../raw/current/scripts/prefabs/bundle.lua#L611-L628)）；配方 goggleshat＋猪皮（[recipes.lua#L539](../../../raw/current/scripts/recipes.lua#L539)）；[hats.lua#L2208-L2240](../../../raw/current/scripts/prefabs/hats.lua#L2208-L2240) |
| `goggleshat`（时髦的护目镜） | **仅 dapperness MED＋fueled USAGE 10 天＋opentop**——无 `goggles` 标签，源码注明"只为准头不防沙尘暴" | 配方 1 金＋1 猪皮 SCIENCE_ONE（[recipes.lua#L538](../../../raw/current/scripts/recipes.lua#L538)）；[hats.lua#L2242-L2267](../../../raw/current/scripts/prefabs/hats.lua#L2242-L2267) |
| `moonstorm_goggleshat`（星象护目镜） | pristine `goggles`＋装备加 `moonstormevent_detector`＋`moonsparkchargeable`（月熠火花充能 +10%/次）＋dapperness MED＋防水 SMALL＋fueled USAGE 1 天；TECH.LOST 蓝图 | 月风暴工作站蓝图（[recipes.lua#L1012](../../../raw/current/scripts/recipes.lua#L1012)），配方 2 月玻璃＋1 土豆（[recipes.lua#L540](../../../raw/current/scripts/recipes.lua#L540)）；[hats.lua#L2269-L2312](../../../raw/current/scripts/prefabs/hats.lua#L2269-L2312) |
| `plantregistryhat`（耕作先驱帽） | pristine `plantinspector` 标签＋useableitem→`plantregistry_open` 态＋植物注册表弹窗＋夏季保温 SMALL | 配方 1 肥料＋3 种子＋1 电子件 SCIENCE_TWO（[recipes.lua#L448](../../../raw/current/scripts/recipes.lua#L448)）；[hats.lua#L3065-L3116](../../../raw/current/scripts/prefabs/hats.lua#L3065-L3116) |
| `nutrientsgoggleshat`（高级耕作先驱帽） | 耕作先驱帽全量＋pristine `nutrientsvision`＋`detailedplanthappiness`＋shadowlevel 1；配方 `nounlock` | 配方 1 耕作先驱帽＋4 铥矿碎片＋1 紫宝石 ANCIENT_TWO（[recipes.lua#L806](../../../raw/current/scripts/recipes.lua#L806)）；[hats.lua#L3118-L3159](../../../raw/current/scripts/prefabs/hats.lua#L3118-L3159) |
| `inspectacleshat`（检查专用镜） | pristine `inspectaclesvision`＋`cannotuse`＋signalpulse／signallevel netvar＋useableitem 小游戏态＋`restrictedtag="inspectacleshatuser"`（Winona 技能树授予）；信号 FX 归 hats.md | 配方 1 goggleshat＋2 电子件＋builder_skill winona_wagstaff_1（[recipes.lua#L317](../../../raw/current/scripts/recipes.lua#L317)）；[hats.lua#L5043-L5081](../../../raw/current/scripts/prefabs/hats.lua#L5043-L5081) |
| `roseglasseshat`（玫瑰色眼镜） | pristine `roseglassesvision`＋dapperness TINY＋`restrictedtag="handyperson"`（仅 Winona 可装）＋技能 winona_charlie_1 动态挂 closeinspector、charlie_2 给 `wormholetracker` | 配方 12 petals＋3 twigs＋builder_skill winona_charlie_1（[recipes.lua#L318](../../../raw/current/scripts/recipes.lua#L318)）；[hats.lua#L5085-L5177](../../../raw/current/scripts/prefabs/hats.lua#L5085-L5177) |
| `scrap_monoclehat`（视界扩展器） | pristine `scrapmonolevision`＋装备 `AddCameraExtraDistance`(+20 视距)＋分时段 dapperness（昼 +SMALL／黄昏 0／夜 −TINY）＋fueled USAGE 12 天且 `no_sewing`（不可缝补） | Daywalker 掉落（[daywalker2.lua#L56](../../../raw/current/scripts/prefabs/daywalker2.lua#L56)）；[hats.lua#L4664-L4717](../../../raw/current/scripts/prefabs/hats.lua#L4664-L4717) |
| `antlionhat`（刮地皮头盔） | finiteuses 400 次＋container 1 槽（只收 `groundtile` 地皮）＋autoterraformer 装备期改造脚下地块（四条分支：同格同地皮直接返回；有槽内地皮且可铺 → 消耗它并铺上；有底层地皮 → 还原底层、不耗地皮；无底层 → 耗槽内地皮铺入或置泥地——**槽空也能刮**；同格重复改造延迟 0.25 秒，每完成一格 finiteuses −1＋喷 `turf_smoke_fx`）＋防水 SMALL＋shadowlevel 1＋pristine `turfhat` 标签（见[地形改造与孢子链](#地形改造与孢子链)） | 蚁狮被杀必掉蓝图（[antlion.lua#L39-L41](../../../raw/current/scripts/prefabs/antlion.lua#L39-L41)）或上贡 antliontrinket 随机换取（[antlion.lua#L136](../../../raw/current/scripts/prefabs/antlion.lua#L136)）；配方 5 牛毛＋3 猪皮＋1 townportaltalisman TECH.LOST（[recipes.lua#L587](../../../raw/current/scripts/recipes.lua#L587)）；[hats.lua#L2371-L2446](../../../raw/current/scripts/prefabs/hats.lua#L2371-L2446) |

**表C 伪装／状态功能（5 注册）**

| 注册名（官方名） | 组件与关键数值 | 获取与证据 |
|---|---|---|
| `bushhat`（灌木丛帽） | pristine `hide` 标签＋useableitem→`owner.sg:GoToState("hide")`；持有者离开 hide 态即自动停用；无 fuel／无保温 | 配方 1 草帽＋1 绳＋1 挖起的浆果丛 SCIENCE_TWO（[recipes.lua#L545](../../../raw/current/scripts/recipes.lua#L545)）；[hats.lua#L1091-L1140](../../../raw/current/scripts/prefabs/hats.lua#L1091-L1140) |
| `nightcaphat`（睡帽） | **无额外玩法组件**（走 `simple()` 工厂，仍装 inventoryitem／inspectable／tradable／equippable＋floater 缩放 0.65），只有 pristine `good_sleep_aid` 标签（睡袋理智恢复 ×1.3）；兔年活动卧兔生成并装备 | [yotr_rabbitshrine.lua#L129-L132](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L129-L132)；[hats.lua#L1074-L1089](../../../raw/current/scripts/prefabs/hats.lua#L1074-L1089) |
| `spiderhat`（蜘蛛帽） | 装备加 `monster`＋`spiderdisguise`、踢掉猪跟随者，0.5 秒周期收编 12 格内无主蜘蛛（上限 10 个）；卸下／落地／腐坏撤标签并令已收编蜘蛛 `SuggestTarget` 反咬（Webber 只清忠诚 0 的蜘蛛）；fueled USAGE 燃时 120 秒耗尽自毁，**`no_sewing` 不可缝补**＋防水 SMALL＋dapperness −SMALL | 蜘蛛女王掉落（[spiderqueen.lua#L33](../../../raw/current/scripts/prefabs/spiderqueen.lua#L33)）＋克劳斯布袋池（[klaussackloot.lua#L29](../../../raw/current/scripts/components/klaussackloot.lua#L29)）；无制作配方，拆解 4 蛛丝＋2 蛛腺体＋1 怪兽肉（[recipes.lua#L1674](../../../raw/current/scripts/recipes.lua#L1674)）；[hats.lua#L789-L900](../../../raw/current/scripts/prefabs/hats.lua#L789-L900) |
| `mermhat`（聪明的伪装） | 装备加 `merm`＋`mermdisguise`（原生 `merm` 者不加标记）、踢掉猪跟随者；卸下撤标签、延迟一帧移除 merm 跟随者，原有 `monster` 标签还原；perishable 15 天耗尽自毁＋dapperness −TINY＋opentop 露发＋无 armor | 配方 1 pondfish＋1 cutreeds＋2 twigs，`builder_tag=merm_builder`（Wurt 限定）（[recipes.lua#L111](../../../raw/current/scripts/recipes.lua#L111)）；[hats.lua#L2912-L3004](../../../raw/current/scripts/prefabs/hats.lua#L2912-L3004) |
| `bat_bosscorpsehat`（蝙蝠帽） | 装备加 `monster`＋`batdisguise`，pristine `bathat`（batbat 套装）／`icebox_valid`／`acidrainimmune` 等；dapperness **CRAZINESS_MED＝−0.333 理智/秒**（480 秒天 −160）；perishable 6 天耗尽变 `spoiled_food`；装备期若持有者满足吸血门禁（`IsLifeDrainable` 且有 health、非 equipmentmodel）则 `StopPerishing` 并每 0.6 秒 `AddTime(PERISH_FAST×0.0006)`＝**每秒回 0.1% 腐坏时间**（0.06%/tick），同时持续 −3 HP/0.6 秒；防水 SMALL；装备即 −10 HP（延后一帧的一次 combat 攻击，读档装备时跳过），近战命中回 6.8 HP（wilson_attack×0.2；要求持有者已受伤、目标可吸血且存活、非远程武器、非骑乘，并乘 AoE 递减倍率）；无 armor 无武器组件 | 猎犬蝙蝠 bat_boss 及酸蚀变体必掉（[bat_boss.lua#L40-L48](../../../raw/current/scripts/prefabs/bat_boss.lua#L40-L48)）；无制作配方，拆解 2 batwing＋4 houndstooth＋1 怪兽肉（[recipes.lua#L1688](../../../raw/current/scripts/recipes.lua#L1688)）；[hats.lua#L6511-L6674](../../../raw/current/scripts/prefabs/hats.lua#L6511-L6674) |
| `minerhat`／`molehat`／`plantregistryhat` 等 fueled 开关细节 | 见[燃料与耐久链](#燃料与耐久链) | — |

### 其余 53 注册的分派

| 分派 | 注册名 | 依据 |
|---|---|---|
| 归 [[reference/prefabs/hats-armor|防护与战斗帽族]]（25 注册） | beehat、footballhat、woodcarvedhat、ruinshat、cookiecutterhat、slurtlehat、wathgrithrhat、wathgrithr_improvedhat、hivehat、eyemaskhat、shadow_thrall_parasitehat、**skeletonhat（骨头头盔，有 armor 勿当装饰帽）**、scraphat、dreadstonehat、lunarplanthat、voidclothhat、wagpunkhat、mermarmorhat、mermarmorupgradedhat、mask_princesshat、yoth_knighthat（21 顶 `AddComponent("armor")`）＋宿主面具 mask_halfwithat／mask_sagehat／mask_toadyhat＋alterguardianhat（无 armor） | 逐帽构造、配方与消费端见 armor 页 |
| 归 [[reference/prefabs/hats-event|事件装扮与角色帽族]]（28 注册） | tophat（高礼帽）、featherhat、walterhat、ghostflowerhat、dragonheadhat／dragonbodyhat／dragontailhat、pumpkinhat（无面南瓜）、mask_doll 系 ×12、mask_charlie_boss、monkey_mediumhat／monkey_smallhat、polly_rogershat、salty_doghat、batnosehat、balloonhat、rabbithat | tophat [hats.lua#L1044-L1071](../../../raw/current/scripts/prefabs/hats.lua#L1044-L1071)、幸运兽三件套 [L2183-L2213](../../../raw/current/scripts/prefabs/hats.lua#L2183-L2213)、波莉·罗杰的帽子 [L2613-L2700](../../../raw/current/scripts/prefabs/hats.lua#L2613-L2700) |

## 关联机制导航

### 视觉标签与玩家视野

装备上的标签由玩家 `playervision` 组件在 equip/unequip（客户端含 inventoryclosed）时统一刷新（[playervision.lua#L41-L80](../../../raw/current/scripts/components/playervision.lua#L41-L80)）：

| 装备标签 | 来源帽 | 生效链与消费端 |
|---|---|---|
| `nightvision` | molehat（pristine） | → `CanEntitySeeInDark` 为真（[simutil.lua#L502-L515](../../../raw/current/scripts/simutil.lua#L502-L515)）；换鼹鼠色表（黄昏/夜晚 mole_vision_on，[playervision.lua#L9-L15](../../../raw/current/scripts/components/playervision.lua#L9-L15)、[L175-L194](../../../raw/current/scripts/components/playervision.lua#L175-L194)）；Grue 查克启动被跳过（[grue.lua#L111-L116](../../../raw/current/scripts/components/grue.lua#L111-L116)）；理智黑暗判定按 0.9 光值（[sanity.lua#L535](../../../raw/current/scripts/components/sanity.lua#L535)） |
| `goggles` | deserthat、moonstorm_goggleshat（pristine） | → 推 `gogglevision` 事件；沙尘暴／月风暴／瘴气移速恢复（[sandstormwatcher.lua#L22-L26](../../../raw/current/scripts/components/sandstormwatcher.lua#L22-L26)、[moonstormwatcher.lua#L18-L25](../../../raw/current/scripts/components/moonstormwatcher.lua#L18-L25)、[miasmawatcher.lua#L50-L54](../../../raw/current/scripts/components/miasmawatcher.lua#L50-L54)）；`CanEntitySeePoint` 风暴分支（[simutil.lua#L541-L550](../../../raw/current/scripts/simutil.lua#L541-L550)） |
| `nutrientsvision`＋`detailedplanthappiness` | nutrientsgoggleshat | → 推世界级 `nutrientsvision`（仅本地玩家）；植物压力描述给具体压力源（[farmplantstress.lua#L108](../../../raw/current/scripts/components/farmplantstress.lua#L108)） |
| `plantinspector` | plantregistryhat、nutrientsgoggleshat | → componentactions 给植物/肥料加 PLANTREGISTRY_RESEARCH、ASSESSPLANTHAPPINESS 动作（[componentactions.lua#L60-L82](../../../raw/current/scripts/componentactions.lua#L60-L82)） |
| `inspectaclesvision` | inspectacleshat | → 推 `inspectaclesvision` 事件；小游戏参与走持有者 `inspectaclesparticipant` 组件与 netvar（[hats.lua#L4746-L4821](../../../raw/current/scripts/prefabs/hats.lua#L4746-L4821)） |
| `roseglassesvision` | roseglasseshat | → 推事件；近距检查走 closeinspector（技能门禁） |
| `scrapmonolevision` | scrap_monoclehat | → 推事件；视距扩大走 `AddCameraExtraDistance`（[hats.lua#L4664-L4670](../../../raw/current/scripts/prefabs/hats.lua#L4664-L4670)） |

`moonstormevent_detector`（moonstorm_goggleshat 装备期加／卸下移除）：月风暴游走者与 Wagstaff 的地图指示追踪门禁（[moonstorm_static.lua#L473-L479](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L473-L479)、[wagstaff_npc.lua#L388-L394](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L388-L394)）。`moonsparkchargeable` 挂星象护目镜，月熠火花（moonstorm_spark）命中时 `DoSpark` 充能 fueled +10%（[moonstorm_spark.lua#L42-L46](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L42-L46)、组件 [moonsparkchargeable.lua](../../../raw/current/scripts/components/moonsparkchargeable.lua)）。

### 燃料与耐久链

- **fueled 类型决定补给方式**（[fueled.lua#L5-L18](../../../raw/current/scripts/components/fueled.lua#L5-L18)）：`FUELTYPE.USAGE` 帽（本页大部分）打 `needssewing` 标签→用**缝纫包**补（蜘蛛帽 USAGE＋`no_sewing` 例外，不可缝补，[fueled.lua#L63](../../../raw/current/scripts/components/fueled.lua#L63)）；`FUELTYPE.CAVE`（矿工帽）与 `FUELTYPE.WORMLIGHT`（鼹鼠帽）打 `<fueltype>_fueled` 标签→直接吃对应燃料物品（荧光果/萤火虫/蜗牛粘液 [lightbulb.lua#L58](../../../raw/current/scripts/prefabs/lightbulb.lua#L58)、[fireflies.lua#L235](../../../raw/current/scripts/prefabs/fireflies.lua#L235)、[slurtleslime.lua#L65](../../../raw/current/scripts/prefabs/slurtleslime.lua#L65)；发光浆果 [wormlight.lua#L105](../../../raw/current/scripts/prefabs/wormlight.lua#L105)）。
- **消耗窗口**：装备即耗（`_base_onequip` StartConsuming），卸下／入袋／掉地（矿工帽另挂 SetOnDroppedFn）即停（[hats.lua#L40-L42](../../../raw/current/scripts/prefabs/hats.lua#L40-L42)、[L101-L103](../../../raw/current/scripts/prefabs/hats.lua#L101-L103)、[L767](../../../raw/current/scripts/prefabs/hats.lua#L767)）。
- **耗尽分支**：多数帽 `inst.Remove` 自毁；矿工帽先关灯推 `torchranout`（[hats.lua#L721-L737](../../../raw/current/scripts/prefabs/hats.lua#L721-L737)）；眼球伞推 `umbrellaranout`（[hats.lua#L1306-L1325](../../../raw/current/scripts/prefabs/hats.lua#L1306-L1325)）； perishable 帽（花环/海花环/西瓜帽/冰帽/蘑菇帽 ×4/鱼人帽）走 perishable 耗尽替换或自毁（背包／箱内也照常计时，不同于 fueled 装备期窗口），蝙蝠帽耗尽变 `spoiled_food`（`onperishreplacement`，可借 `icebox_valid` 入冰箱，[hats.lua#L6665-L6668](../../../raw/current/scripts/prefabs/hats.lua#L6665-L6668)）。
- 降温帽的 heater：`SetThermics(false, true)`=仅 endothermic；temperature 组件把装备换热并入 delta，把持有者体温向 40（冰帽）/55（西瓜帽）度拉（[temperature.lua#L372-L380](../../../raw/current/scripts/components/temperature.lua#L372-L380)、[heater.lua#L24-L26](../../../raw/current/scripts/components/heater.lua#L24-L26)）。
- 装备期潮湿：`equippedmoisture`/`maxequippedmoisture` 由 moisture 组件并入潮湿增速（[moisture.lua#L107-L112](../../../raw/current/scripts/components/moisture.lua#L107-L112)）——冰帽/西瓜帽戴着会"出汗"变湿。
- 雷电绝缘：`equippable.insulated` → `Equippable:IsInsulated`（[equippable.lua#L64-L66](../../../raw/current/scripts/components/equippable.lua#L64-L66)）→ `Inventory:IsInsulated`（[inventory.lua#L759-L770](../../../raw/current/scripts/components/inventory.lua#L759-L770)）→ 闪电命中改推 `lightningdamageavoided`（[playerlightningtarget.lua#L3-L10](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L10)）；月风暴火花伤害也被跳过（[moonstorm_spark.lua#L27](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L27)）。

### 伪装与隐身消费端

- **牛角帽**：装备给持有者加 `beefalo` 标签；牛群 retarget 的 CANT 过滤含 `beefalo`，找不到戴帽玩家（[beefalo.lua#L312-L313](../../../raw/current/scripts/prefabs/beefalo.lua#L312-L313)＋[hats.lua#L605-L613](../../../raw/current/scripts/prefabs/hats.lua#L605-L613)）。
- **灌木丛帽**：`hide` 玩家状态带 `hiding` statetag（[SGwilson.lua#L5618-L5650](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5618-L5650)）；`Combat:SetTarget` 对 `hiding` 态玩家一律拒绝设为目标（[combat.lua#L473-L482](../../../raw/current/scripts/components/combat.lua#L473-L482)），个别 AI（兔人、大影触手等）另有同类过滤。
- **蜘蛛帽**：`spiderdisguise`（＋`monster`）进蜘蛛 retarget CANT（[spider.lua#L208-L209](../../../raw/current/scripts/prefabs/spider.lua#L208-L209)），蜘蛛女王同（[spiderqueen.lua#L39](../../../raw/current/scripts/prefabs/spiderqueen.lua#L39)）；危险感判定不把蜘蛛算威胁（[player_common.lua#L27](../../../raw/current/scripts/prefabs/player_common.lua#L27)）；翻 junk_pile／叶堆／风滚草不触发蜘蛛敌意（[junk_pile_common.lua#L48](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L48)、[leafpile.lua#L258](../../../raw/current/scripts/prefabs/leafpile.lua#L258)、[tumbleweed.lua#L133](../../../raw/current/scripts/prefabs/tumbleweed.lua#L133)）。
- **鱼人帽**：`merm`＋`mermdisguise`——普通鱼人按 `merm` 标签接受跟随与赠礼（[merm.lua#L975-L977](../../../raw/current/scripts/prefabs/merm.lua#L981-L983)），但 **mermguard 明确拒绝伪装者**（[merm.lua#L973](../../../raw/current/scripts/prefabs/merm.lua#L979)）；蚊子女工膏不能用在伪装玩家（[mosquitomermsalve.lua#L14](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L14)）。
- **蝙蝠帽**：`bathat` 与 batbat 组套装——蝙蝠杖命中吸血时的理智扣减系数 0.5×→0.25×（[batbat.lua#L47-L54](../../../raw/current/scripts/prefabs/batbat.lua#L47-L54)）；healthbadge 把戴帽计入血条告警态、bloodover 常开（[healthbadge.lua#L378-L396](../../../raw/current/scripts/widgets/healthbadge.lua#L378-L396)、[bloodover.lua#L71-L76](../../../raw/current/scripts/widgets/bloodover.lua#L71-L76)）。
- `umbrella` 标签（眼球伞 pristine）：雨声切伞声（[weather.lua#L991](../../../raw/current/scripts/components/weather.lua#L991)、[caveweather.lua#L693](../../../raw/current/scripts/components/caveweather.lua#L693)）。

### 地形改造与孢子链

- **蚁狮帽（注册名 `antlionhat`；`turfhat` 是标签，不是注册名）**：装备自动开 1 槽地皮容器并 `StartTerraforming`。`AutoTerraformer:DoTerraform` 分四支（[L57-L104](../../../raw/current/scripts/components/autoterraformer.lua#L57-L104)）：①槽内地皮＝目标格当前地皮 → 直接返回，不耗地皮也不计次数；②有槽内地皮且 `CanPlaceTurfAtPoint` → 消耗该地皮、`SetTile` 后计一次使用；③过了 `CanTerraformAtPoint` 且有 `undertile` 底层 → 还原底层，**不耗槽内地皮**；④无底层 → 有槽内地皮就消耗并铺它，没有就置 `WORLD_TILES.DIRT`——**槽空仍能刮**，随后 `HandleDugGround`＋土块 `collapsesoil`＋计一次使用。同格重复改造有 0.25 秒冷却，`FinishTerraforming` 每次都扣 1 点 finiteuses 并喷 `turf_smoke_fx`，400 次耗尽自毁（[autoterraformer.lua#L60-L109](../../../raw/current/scripts/components/autoterraformer.lua#L60-L109)、[hats.lua#L2395-L2431](../../../raw/current/scripts/prefabs/hats.lua#L2395-L2431)）。`turfhat` 标签令 playercontroller 装备期换用 `gridplacer_turfhat` 网格放置器（[playercontroller.lua#L44-L50](../../../raw/current/scripts/components/playercontroller.lua#L44-L50)、[gridplacer.lua#L92-L125](../../../raw/current/scripts/prefabs/gridplacer.lua#L92-L125)）。
- **蘑菇帽孢子**：装备期 periodicspawner 按色产孢子（普通三色 60–61 秒/个；月版 1.5–1.75 秒/个、`_alwaysinstantpops` 落近即爆，[periodicspawner.lua#L39-L50](../../../raw/current/scripts/components/periodicspawner.lua#L39-L50)、[mushtree_moonspore.lua#L68-L79](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L68-L79)）；孢子是菇农场种源，`spore_to_cap` 映射 tall→blue_cap／medium→red_cap／small→green_cap（[mushroom_farm.lua#L33-L40](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L33-L40)），也可投入蘑菇灯（`spore` 标签，[mushroom_light.lua#L115](../../../raw/current/scripts/prefabs/mushroom_light.lua#L115)）。`spoiler` 标签使持有者背包内食物按地面速率 ×1.5 腐坏（[perishable.lua#L93-L95](../../../raw/current/scripts/components/perishable.lua#L93-L95)）；`moon_spore_protection` 把穿戴者排除出月孢子范围攻击（[mushtree_moonspore.lua#L49](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L49)）。月亮蘑菇帽是 `spore_moon` 的三条入口之一（另两条见孢子页：砍过的月亮树、蘑菇地精）：装备期 `periodicspawner:Start()`（[hats.lua#L1871-L1889](../../../raw/current/scripts/prefabs/hats.lua#L1871-L1889)），生成回调置 `_alwaysinstantpops`（[hats.lua#L1937-L1973](../../../raw/current/scripts/prefabs/hats.lua#L1937-L1973)），三条入口与帽产分支的对照见 [[reference/prefabs/mushtree-spores#月亮孢子差异]]。

### 关联 Prefab 与技能门禁

| 对象 | 关系 | 证据 |
|---|---|---|
| `minerhatlight` | 矿工帽开关回调生成/移除的联网帽灯 | [hats.lua#L677-L709](../../../raw/current/scripts/prefabs/hats.lua#L677-L709)；机制归 [[reference/prefabs/hats|帽子跟随层与帽灯特效族]] |
| `inspectacleshat_fx` | 检查专用镜的 LED 信号跟随层 | 归 [[reference/prefabs/hats|帽子跟随层与帽灯特效族]] |
| `deserthat_blueprint`／`moonstorm_goggleshat_blueprint` | 两顶 TECH.LOST 帽的蓝图实体 | [bundle.lua#L611-L628](../../../raw/current/scripts/prefabs/bundle.lua#L611-L628)、[recipes.lua#L1012](../../../raw/current/scripts/recipes.lua#L1012) |
| `SKILLTREE_EQUIPPABLE_RESTRICTED_TAGS` | inspectacleshatuser→winona 的全局映射（restrictedtag 的授予方在技能树侧） | [constants.lua#L2153-L2157](../../../raw/current/scripts/constants.lua#L2153-L2157) |

## 典型接入

### 调整原版环境帽的数值

- 前提与选择：modmain 里 `AddPrefabPostInit` 是唯一不动原版文件的入口；只调数值时无需处理客户端，组件配置在主模拟侧已同步。
- 生效与撤销：postinit 在 prefab 构造完成后执行，改动即存档随实体；卸载 mod 后组件残留字段按实体重建消失。
- 原版依据：组件组合见[表A](#本页注册分族表)锚点。

```lua
-- modmain.lua：把冬帽保温提到 LARGE、去掉移除耐久
AddPrefabPostInit("winterhat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.insulator:SetInsulation(GLOBAL.TUNING.INSULATION_LARGE)
    inst.components.fueled:SetDepletedFn(nil)   -- 不再耗尽自毁
end)
```

- 调整与验证：`TUNING` 在 mod 环境白名单内可直用，`TheWorld` 等游戏全局必须 `GLOBAL.` 前缀（mod 环境无 `__index=_G`，见 mods.lua `CreateEnvironment` 白名单机制）；`SetDepletedFn(nil)` 仅示例用途，fueled 组件耗尽时对 nil 回调的容错未核验。

### 新建一顶环境帽（保温＋防水＋夜视）

- 前提与选择：自建帽复刻 `simple()` 最小集（inventoryitem＋inspectable＋equippable HEAD）＋组件；`nightvision` 等视觉标签**放在 SetPristine 前**与原版一致（pristine 优化），标签在装备实体上即可被 playervision 消费。
- 生效与撤销：装备期标签随装备网络同步生效；卸下自动失效，无需自写清理。
- 原版依据：骨架 [hats.lua#L188-L235](../../../raw/current/scripts/prefabs/hats.lua#L188-L235)、鼹鼠帽标签与 fueled [hats.lua#L1785-L1839](../../../raw/current/scripts/prefabs/hats.lua#L1785-L1839)。

```lua
-- mod 的 scripts/prefabs/myhat.lua（PrefabFiles 登记）；需自备 anim/hat_myhat.zip
local assets = { Asset("ANIM", "anim/hat_myhat.zip") }
local function fn()
    local inst = GLOBAL.CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()
    GLOBAL.MakeInventoryPhysics(inst)
    inst.AnimState:SetBank("hat_myhat")
    inst.AnimState:SetBuild("hat_myhat")
    inst.AnimState:PlayAnimation("anim")
    inst:AddTag("hat")
    inst:AddTag("nightvision")            -- pristine：夜视消费端见 playervision
    inst.entity:SetPristine()
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("inventoryitem")
    inst:AddComponent("inspectable")
    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = GLOBAL.EQUIPSLOTS.HEAD
    inst:AddComponent("insulator")
    inst.components.insulator:SetInsulation(GLOBAL.TUNING.INSULATION_MED)
    inst:AddComponent("waterproofer")
    inst.components.waterproofer:SetEffectiveness(GLOBAL.TUNING.WATERPROOFNESS_SMALL)
    return inst
end
return GLOBAL.Prefab("myhat", fn, assets)
```

- 调整与验证：`GLOBAL.` 覆盖全部游戏全局（CreateEntity／MakeInventoryPhysics／EQUIPSLOTS／Prefab），TUNING 同样建议显式取；bank/build 名须与实际动画资源一致。示例仅语法自检，未在游戏内运行。

## 常见陷阱与限制

- **误区：时髦的护目镜（goggleshat）能防沙尘暴。** 源码注释明确它"不做在眼睛上、仅为装饰"（[hats.lua#L2242-L2243](../../../raw/current/scripts/prefabs/hats.lua#L2242-L2243)）：它只有 dapperness 和 fueled，无 `goggles` 标签。**本页中**写 `goggles` 的是沙漠护目镜（deserthat）与星象护目镜；该标签并非本页独占——亮茄头盔（[hats.lua#L3879-L3882](../../../raw/current/scripts/prefabs/hats.lua#L3879-L3882)）与无面南瓜（[hats.lua#L6100-L6104](../../../raw/current/scripts/prefabs/hats.lua#L6100-L6104)）也会加，所以判断能不能防沙尘暴只认标签、不认"是不是护目镜"。
- **边界：冰帽的防水是 0%。** pristine 带 `waterproofer` 标签（优化用），但组件 `effectiveness = 0`（[hats.lua#L1660-L1669](../../../raw/current/scripts/prefabs/hats.lua#L1660-L1669)、[L1693-L1694](../../../raw/current/scripts/prefabs/hats.lua#L1693-L1694)），且装备期给潮湿增速上限 49——"有 waterproofer 组件/标签"不等于有效防水。
- **边界：fueled USAGE 帽不能喂燃料物品。** 缝纫包是唯一补给（`needssewing` 标签链，[fueled.lua#L5-L18](../../../raw/current/scripts/components/fueled.lua#L5-L18)）；矿工帽（CAVE）与鼹鼠帽（WORMLIGHT）是例外。蜘蛛帽／视界扩展器 `no_sewing` 连缝纫包也不收（[hats.lua#L4714](../../../raw/current/scripts/prefabs/hats.lua#L4714)、[L894](../../../raw/current/scripts/prefabs/hats.lua#L894)）。
- **边界：燃料只在装备期消耗。** 放背包／箱子不掉耐久；但耗尽是自毁（多数帽 `Remove`），缝纫包补满 ≠ 永久。
- **边界：保温值与理智值的单位不同。** insulator 保温是秒（一段 30 秒，冬帽 MED=120 秒）；dapperness 是理智/秒（冬帽 TINY≈10.7/天）；fueled 天数按 480 秒/天换算，别按 300 秒的 day_time 换算。
- **边界：牛角帽只防"被牛主动锁定"。** `beefalo` 标签进的是牛 retarget 的 CANT 过滤；主动攻击牛仍触发 OnAttacked 反击链（[beefalo.lua#L360-L368](../../../raw/current/scripts/prefabs/beefalo.lua#L360-L368)），发情期牛群追打不受帽保护。
- **边界：蜘蛛帽撤装即反目。** 卸帽／落地／耗尽先撤 `spiderdisguise`、再逐个 `SuggestTarget`、最后清跟随——前蜘蛛立刻攻击原主人；Webber（`spiderwhisperer`）只清忠诚 0 的蜘蛛（[hats.lua#L796-L818](../../../raw/current/scripts/prefabs/hats.lua#L796-L818)）。
- **边界：蘑菇帽的 `spoiler` 常驻加速背包腐坏。** ×1.5（PERISH_GROUND_MULT）作用于持有者全部容器，不摘不掉。
- **边界：蝙蝠帽穿戴是持续放血。** 理智侧 CRAZINESS_MED 按公式 −100/day_time（day_time=300）＝−0.333/秒，480 秒天合计 −160，别按 total_day_time 换算；叠加持续扣血，正收益只有近战吸血与 batbat 套装理智减半。
- **边界：蚁狮帽不是「每格一块地皮」。** finiteuses 每完成一格必 −1；槽内地皮只在「直接铺」与「无底层时铺入」两支被消耗，还原底层那支不耗（[格子分支](../../../raw/current/scripts/components/autoterraformer.lua#L57-L104)）；槽空时只要 `CanTerraformAtPoint` 通过就能继续刮（无底层则置泥地）。耗尽自毁时容器物品全部掉地（[hats.lua#L2400-L2403](../../../raw/current/scripts/prefabs/hats.lua#L2400-L2403)）。
- **边界：灌木丛帽的隐身覆盖所有 `Combat:SetTarget` 调用。** 包括离开灌木瞬间的目标选择；但已锁定你的目标在你藏起来后是否立即丢仇恨由其 KeepTarget 决定，逐 AI 差异未核验。
- **边界：睡帽离开睡袋无任何效果。** `good_sleep_aid` 的唯一消费端是 sleepingbaguser 的理智 tick ×1.3 与起身 `goodsleep` 标记（[sleepingbaguser.lua#L65](../../../raw/current/scripts/components/sleepingbaguser.lua#L65)、[L81-L85](../../../raw/current/scripts/components/sleepingbaguser.lua#L81-L85)）；兔年活动获取（卧兔装备）。
- **边界：帽注册名与官方名不能互推。** 兔耳罩=earmuffshat（Rabbit Earmuffs）、贝雷帽=walrushat（Tam o' Shanter）、冰帽=icehat（Ice Cube）、骨头头盔=skeletonhat（且带 armor）；名称键核验见[验证记录](#验证记录)。
- **边界：`waterproofer` 标签在 pristine 只为客户端动作过滤优化。** 实际防水由主模拟组件 `effectiveness` 决定；给帽追加防水时两处都要动（或只用组件，接受无 pristine 优化）。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/hats.lua` 工厂头（L16-L235）、本页 32 注册的 custom_init 与 fn（各表锚点）、名字分派块（L6752-L6987）与 return 注册清单（L7860-L7953）；`recipes.lua` 本页 29 条配方行逐行实读；消费端 playervision 全文，simutil（CanEntitySeeInDark/SeePoint）、combat（SetTarget）、fueled（fueltype 标签）、grue、sanity、sleepingbaguser、sandstorm/moonstorm/miasmawatcher、temperature（装备换热）、moisture（equippedmoisture）、componentactions（plantinspector）、farmplantstress、playerlightningtarget、moonstorm_spark、moonstorm_static、wagstaff_npc、beefalo（retarget）、walrus（掉落）、daywalker2（掉落）、yotr_rabbitshrine（睡帽生成）、bundle（湿布袋蓝图）、constants（restrictedtag 映射）按锚点实读。补入 8 注册增读：hats.lua 蜘蛛（L789-L900）／蘑菇（L1841-L2045）／蚁狮（L2369-L2446）／蝙蝠帽（L6511-L6674）段与分派分支（L6881-L6886）；autoterraformer 全文、periodicspawner、fueled（no_sewing）、perishable（spoiler）、componentutil（IsLifeDrainable）、containers（antlionhat）、playercontroller/gridplacer（turfhat）、spider/spiderqueen/merm/mosquitomermsalve/junk_pile_common/leafpile/tumbleweed/player_common（伪装）、batbat/healthbadge/bloodover（套装与 UI）、bat_boss/antlion/toadstool/klaussackloot（获取）、mushroom_farm/mushroom_light/mushtree_moonspore（孢子）按锚点实读。
- 穷举复核：`grep -n 'MakeHat("' hats.lua` 得 **85 条注册**，与 hats.md"85 帽"一致；`grep -n 'AddComponent("armor")'` 命中 21 处，全部划归 hats-armor，本页 32 注册（含补入 8 注册）fn 实读均无 armor 组件。
- 名称核验：`languages/chinese_s.po` 按 `STRINGS.NAMES.<KEY>` 逐键检索——STRAWHAT 草帽 [L350090](../../../raw/current/scripts/languages/chinese_s.po#L350090)、EARMUFFSHAT 兔耳罩 [L342435](../../../raw/current/scripts/languages/chinese_s.po#L342435)、WINTERHAT 冬帽 [L352205](../../../raw/current/scripts/languages/chinese_s.po#L352205)、CATCOONHAT 猫帽 [L340700](../../../raw/current/scripts/languages/chinese_s.po#L340700)、WALRUSHAT 贝雷帽 [L351705](../../../raw/current/scripts/languages/chinese_s.po#L351705)、BEEFALOHAT 牛角帽 [L339275](../../../raw/current/scripts/languages/chinese_s.po#L339275)、FLOWERHAT 花环 [L342910](../../../raw/current/scripts/languages/chinese_s.po#L342910)、KELPHAT 海花环 [L344190](../../../raw/current/scripts/languages/chinese_s.po#L344190)、WATERMELONHAT 西瓜帽 [L351805](../../../raw/current/scripts/languages/chinese_s.po#L351805)、ICEHAT 冰帽 [L344080](../../../raw/current/scripts/languages/chinese_s.po#L344080)、RAINHAT 雨帽 [L348480](../../../raw/current/scripts/languages/chinese_s.po#L348480)、EYEBRELLAHAT 眼球伞 [L342500](../../../raw/current/scripts/languages/chinese_s.po#L342500)、MINERHAT 矿工帽 [L345440](../../../raw/current/scripts/languages/chinese_s.po#L345440)、MOLEHAT 鼹鼠帽 [L345515](../../../raw/current/scripts/languages/chinese_s.po#L345515)、DESERTHAT 沙漠护目镜 [L342115](../../../raw/current/scripts/languages/chinese_s.po#L342115)、GOGGLESHAT 时髦的护目镜 [L343350](../../../raw/current/scripts/languages/chinese_s.po#L343350)、MOONSTORM_GOGGLESHAT 星象护目镜 [L345705](../../../raw/current/scripts/languages/chinese_s.po#L345705)、NIGHTCAPHAT 睡帽 [L346135](../../../raw/current/scripts/languages/chinese_s.po#L346135)、BUSHHAT 灌木丛帽 [L340015](../../../raw/current/scripts/languages/chinese_s.po#L340015)、PLANTREGISTRYHAT 耕作先驱帽 [L346995](../../../raw/current/scripts/languages/chinese_s.po#L346995)、NUTRIENTSGOGGLESHAT 高级耕作先驱帽 [L346215](../../../raw/current/scripts/languages/chinese_s.po#L346215)、INSPECTACLESHAT 检查专用镜 [L344130](../../../raw/current/scripts/languages/chinese_s.po#L344130)、ROSEGLASSESHAT 玫瑰色眼镜 [L348795](../../../raw/current/scripts/languages/chinese_s.po#L348795)、SCRAP_MONOCLEHAT 视界扩展器 [L349000](../../../raw/current/scripts/languages/chinese_s.po#L349000)；补入 8 注册：SPIDERHAT 蜘蛛帽 [L349847](../../../raw/current/scripts/languages/chinese_s.po#L349847)、MERMHAT 聪明的伪装 [L345327](../../../raw/current/scripts/languages/chinese_s.po#L345327)、ANTLIONHAT 刮地皮头盔 [L338697](../../../raw/current/scripts/languages/chinese_s.po#L338697)、BAT_BOSSCORPSEHAT 蝙蝠帽 [L339202](../../../raw/current/scripts/languages/chinese_s.po#L339202)、RED_MUSHROOMHAT 红蘑菇帽 [L348562](../../../raw/current/scripts/languages/chinese_s.po#L348562)、GREEN_MUSHROOMHAT 绿蘑菇帽 [L343467](../../../raw/current/scripts/languages/chinese_s.po#L343467)、BLUE_MUSHROOMHAT 蓝蘑菇帽 [L339582](../../../raw/current/scripts/languages/chinese_s.po#L339582)、MOON_MUSHROOMHAT 月亮蘑菇帽 [L345867](../../../raw/current/scripts/languages/chinese_s.po#L345867)（四帽通用检查名 MUSHROOMHAT 蘑菇帽 [L346007](../../../raw/current/scripts/languages/chinese_s.po#L346007)）。
- 主代理裁决执行：2026-11-07 按归属裁决把 spiderhat、mermhat、antlionhat、bat_bosscorpsehat、蘑菇帽 ×4 自事件/待定夺行转入本页（24→32 注册）；woodcarvedhat／mask_princesshat／yoth_knighthat 去向行改指 [[reference/prefabs/hats-armor|防护与战斗帽族]]。蝙蝠帽两处按源码实测修正：dapperness 实为 CRAZINESS_MED＝−0.333 理智/秒（裁决注 "+0.333" 符号有误）；"无近战吸血"不成立——吸血监听注册于装备回调（hats.lua#L6511-L6573），每击 +6.8 HP。
- 分档说明：`size_class: complex`——32 注册共享一个参数化工厂但逐帽组件组合各异；视觉标签系统（6 种装备标签→playervision→三类消费端）、燃料链（三种 fueltype＋perishable）、伪装收编与反目链、地形改造与孢子链需要整体对照说明，超出 standard 单页容量。
- 未核验项：①孢子实体（spore_*）的漂浮、捕获与过期细节（归 mushtree 系页面）；②克劳斯布袋三池的抽取概率与组合规则（仅核验池成员）；③bat_boss 战斗流程与酸蚀变体差异（仅核验掉落表）；④逐帽皮肤清单与皮肤 FX、动画/音效资源内容、`SetDepletedFn(nil)` 容错；⑤睡觉帽的玩家获取全路径（仅核验到卧兔生成装备）。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（235 行／51098 字节，complex 档上限 480 行／65536 字节，0 issue）；`validate_source_links.py --strict` 通过（201 条引用、62 个源文件，0 error 0 warning）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]（hats.lua 内 11 个 FX＋1 个技术注册、矿工帽灯与检查专用镜信号层）
- [[reference/prefabs/hats-armor|防护与战斗帽族]]、[[reference/prefabs/hats-event|事件装扮与角色帽族]]（85 顶玩法帽的另两个分族页）
- [[reference/components/insulator|insulator 组件]]、[[reference/components/waterproofer|waterproofer 组件]]（保温／防水的通用消费端）
- [[reference/components/fueled|fueled 组件]]（fueltype、needssewing 与装备期消耗）
- [[reference/components/playervision|playervision 组件]]（夜视／护目／营养视野的统一入口）
- [[reference/components/equippable|equippable 组件]]（equipslot、dapperness、insulated、restrictedtag）
- [[reference/components/heater|heater 组件]]、[[reference/components/temperature|temperature 组件]]（降温帽的换热链）
- 返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|原版 Prefab 索引]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
