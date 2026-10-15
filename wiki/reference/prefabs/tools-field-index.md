---
title: "装备·工具与野外实用"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# 装备·工具与野外实用

采集、制作、照明、雨具、移动、钓鱼与野外实用装备的叶级索引；除特别标注外均为 HANDS 槽。按主要用途唯一登记，武器／战斗等其他用法从文末“跨用途入口”进入。返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。

## 采集、制作与维修工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/axe|斧头族（axe、goldenaxe、moonglassaxe）]] | HANDS 槽；MakeAxe 三注册：CHOP 工具＋武器、耗尽即移除；黄金斧消耗 1/4 折算、月光玻璃斧对纯影目标低磨损增伤；Woodie 砍树 Lucy 替换与获取途径 | axe、goldenaxe、moonglassaxe、斧头、斧子、黄金斧头、黄金斧、月光玻璃斧、Luxury Axe、Moon Glass Axe、砍树工具、斧头耐久 / active |
| [[reference/prefabs/pickaxe|鹤嘴锄（pickaxe）]] | HANDS 槽；MakePickaxe 两注册：MINE 工具＋武器、耐久 33、黄金鹤嘴锄消耗 1/4 折算与金装备音 | pickaxe、goldenpickaxe、鹤嘴锄、黄金鹤嘴锄、Opulent Pickaxe、镐、挖矿工具 / active |
| [[reference/prefabs/shovel|铲子与黄金铲子（shovel／goldenshovel）]] | HANDS 槽；MakeShovel 两注册：DIG 工具＋弱武器、耐久 25、黄金铲子 1/4 折算、quagmire 分支与 AddInherentAction 边界 | shovel、goldenshovel、铲子、黄金铲子、Regal Shovel、金铲、挖掘工具、铲子挖坑 / active |
| [[reference/prefabs/hammer|锤子（hammer）]] | HANDS 槽；MakeHammer 单注册：HAMMER 拆解工具＋武器、耐久 75、耗尽即移除；hammer 标签消费端（鼹鼠眩晕掉物） | hammer、锤子、Hammer、拆除建筑的工具、拆建筑 / active |
| [[reference/prefabs/multitool_axe_pickaxe|多用斧镐（multitool_axe_pickaxe）]] | HANDS 槽；一工具登记 CHOP+MINE 双动作分档扣次（1/3）、可通过默认 tough work 门禁、shadowlevel 静态 1 级；远古祭坛配方与宝箱生成点 | multitool_axe_pickaxe、多用斧镐、Pick/Axe、多功能工具斧镐、斧镐 / active |
| [[reference/prefabs/farm-hoe|园艺锄（farm_hoe、golden_farm_hoe）]] | HANDS 槽；MakeFarmHoe 两注册：farmtiller 右键锄地生成 farm_soil、TILL 消耗、黄金园艺锄 1/4 折算；锄地候选不经 tool 组件 | farm_hoe、golden_farm_hoe、园艺锄、黄金园艺锄、Garden Hoe、Splendid Garden Hoe、金锄、锄头、锄地工具、翻耕土壤 / active |
| [[reference/prefabs/fumarole-tools|气孔工具族（fumaroleaxe、fumarolepickaxe、fumaroleshovel、fumarolehammer、fumarole_farm_hoe）]] | HANDS 槽；五件温度工具共享 MakeFumaroleTool 协议：使用降温、冷却自毁、重热自修、分档外观效率、对冻结目标增伤；工作攻击不磨损，4 次破损即毁 | fumaroleaxe、fumarolepickaxe、fumaroleshovel、fumarolehammer、fumarole_farm_hoe、炽热斧、烈焰鹤嘴锄、呲啦铲、炎头锤、烧灼园艺锄、气孔工具、气孔斧、气孔镐 / active |
| [[reference/prefabs/lunarplant-tools|亮茄工具族（pickaxe_lunarplant、shovel_lunarplant）]] | HANDS 槽；亮茄镐与锄铲：亮茄头盔套装加成（手工监听）、破损循环＋修补套件修理、幸运触手协战；月亮锻造台配方 | pickaxe_lunarplant、shovel_lunarplant、亮茄粉碎者、亮茄锄铲、Brightshade Smasher、Brightshade Shoevel、亮茄工具、月晶工具 / active |
| [[reference/prefabs/lucy|露西斧（lucy）]] | HANDS 槽；伍迪专属活体斧：无限耐久、CHOP 效率 2、会说话（classified 定向广播）；非伍迪掉落；仅三条还原路径、丢弃不飞回；LUCY_BITE_DAMAGE 为死键 | lucy、露西斧、Lucy the Axe、露西尔、伍迪的斧头 / active |
| [[reference/prefabs/voidcloth_scythe|暗影收割者（voidcloth_scythe）]] | HANDS 槽；裂隙暗影工具+武器：右键群体采收（半径 4/165° 锥角）、命中 38+位面 18、影阵营持有者会说话、沃拓克斯影契耗灵魂 AoE；残破可修复 | voidcloth_scythe、暗影收割者、Shadow Reaper、虚布镰刀 / active |

## 照明与火源

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/torch|火炬（torch）]] | HANDS 槽；燃料照明装备：75 秒燃尽（雨中加速）、攻击掷点引燃、右键投掷落地持续照明（可被外部灭火）；不可续燃料/被点燃；威尔逊技能档 | torch、火炬、火把、Torch、火把耐久、点火 / active |
| [[reference/prefabs/lighter|薇洛的打火机（lighter）]] | HANDS 槽；薇洛照明装备：600 秒燃尽（装备燃烧计时；烹饪另扣 1%/5%）、攻击引燃（薇洛技能必燃/增伤）、可烤食物（expertchef）、吸火收 ember（不回燃料）；低理智起火不存在 | lighter、薇洛的打火机、打火机、幸运打火机、吸火 / active |
| [[reference/prefabs/nightstick|晨星锤（nightstick）]] | HANDS 槽；电击照明武器：360 秒燃尽（仅装备燃烧）、伤害 28.9 干燥 ×1.5/湿最高 ×2.5、零攻击磨损；Winona 电池/紫宝石 G.E.M.erator 充电（batteryuser 链） | nightstick、晨星锤、Morning Star、光杖、夜棍、电击武器 / active |
| [[reference/prefabs/lantern|提灯（lantern）]] | HANDS 槽；可充能照明装备：machine 开关（装备/落地可切换）、468 秒/次、光随燃料渐变；荧光果/萤火虫/黏浆充电、耗尽不销毁；无武器组件；配方不含萤火虫 | lantern、提灯、灯笼、Lantern、mininglantern、开关提灯 / active |
| [[reference/prefabs/queen-torch|女王火炬（queen_torch）]] | HANDS 槽；永燃照明武器：近伤 17 与火炬同源、右键投掷（Charlie 战域内会被弹回）、独立联网光源 queen_torchlight 蓝白光、装备态半径 5×威尔逊技能档修正；16 单位驱退/门控暗影类 AI；无配方仅 Charlie 战败仪式生成；不可点燃、不防水（标签无组件） | queen_torch、女王火炬、Queen's Torch、queen_torchlight、驱影火炬、投掷照明火炬、暗花火炬 / active |
| [[reference/prefabs/redlantern|红灯笼（redlantern、yots_redlantern）]] | HANDS 槽；一次性 MAGIC 燃料照明：12 游戏日、常亮不可开关不可补燃料、光挂 redlanternlight、装备外观 redlanternbody（骑乘/悬挂切换）；入袋即熄；可被任何火源点燃 3 秒烧毁；幸运金块×3 供品科技配方；蛇年变体假灯笼诱蠕虫 | redlantern、红灯笼、Red Lantern、yots_redlantern、redlanternlight、redlanternbody、蛇年红灯笼 / active |

## 雨具、移动与野外探索

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/umbrella|伞族（雨伞、花伞、暗影伞、传送伞）]] | HANDS 槽手持防雨与传送伞族：防水保温、暗影伞雨罩、传送伞技能门禁与 pad 供电、酸雨回充陷阱；四把伞都占 HANDS 槽 | 雨伞、花伞、暗影伞、传送伞、手持防雨 / active |
| [[reference/prefabs/cane|步行手杖（cane）]] | HANDS 槽；装备加速、持握，七皮肤雪花／花瓣／彩纸／紫烟／闪光／影子／棋子与 FX 入口 | 手杖、加速手杖、Walking Cane、手杖皮肤特效；远古手杖、拐杖糖、城堡手杖、愚人手杖、玫瑰红手杖、腐败手杖、山羊头手杖 / active |
| [[reference/prefabs/tools-misc|探具与仪式工具族（thurible 等 6 注册）]] | HANDS 槽；暗影香炉引诱影随从、6 格光环影响复活的骨架选敌、复活的骨架／远古织影者的缠绕（非驱散）、罗盘 HUD 指针弹簧与扰动、探测杖四档测距 teleportato、木手杖与 cane 辨析（1.15 速/1920s 磨损/露西雕刻配方）、女巫扫帚骑乘动画非飞行 | thurible、compass、diviningrod、walking_stick、witchbroom、暗影香炉、指南针、探测杖、木手杖、女巫扫帚 / active |
| [[reference/prefabs/living-tools|生活工具族（bugnet 等 8 注册）]] | HANDS 槽；捕虫网对（NET_tool 标签）、刷子 brushable、浇水壶 wateryprotection 与 FILL 拒海水、干草叉 TERRAFORM 地块置换、小风扇新实例首次装备即吸冷（卸装后再装备要等前进）、移动才耗油 90 秒 | bugnet、thulecitebugnet、brush、wateringcan、premiumwateringcan、pitchfork、goldenpitchfork、minifan、捕虫网、刷子、浇水壶、空鸟嘴壶、干草叉、旋转的风扇 / active |

## 钓鱼与水上工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/fishing-tools|钓鱼与水上工具族（oar 等 8 注册）]] | HANDS 槽；五桨同厂分档（master_crewman 折半）、淡水钓竿 9 耐久、海钓竿容器饵具+张力断线、渔网无常规生成点（仅调试上架）；浮子/鱼饵 46 附属注册列清单 | oar、oar_driftwood、malbatross_beak、oar_monkey、yotd_oar、fishingrod、oceanfishingrod、fishingnet、船桨、钓竿、海钓竿、渔网 / active |

## 实用道具与投掷

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/utility-and-throwables|实用与投掷装备（瓶中信、路标导航仪、种壳）]] | HANDS 槽；瓶中信显影/空瓶装恶液、罗盘三分支指向、种壳整叠投掷 AOE（PvP 未命中落船板才漏水）；waterplant 运行时武器 excluded | messagebottle、messagebottleempty、gelblob_bottle、vault_compass、waterplant_bomb、瓶中信、空瓶子、恶液罐、路标导航仪、种壳 / active |

## 跨用途入口

- 斧、鹤嘴锄、铲、锤、多用斧镐、气孔工具、亮茄工具、露西斧、暗影收割者均带武器／攻击侧；战斗选型与投掷入口见 [[reference/prefabs/weapons-combat-index|武器与战斗]]，具体数值与磨损见各页战斗章节。
- 火炬、打火机、晨星锤、女王火炬的引燃、投掷、电击与驱影属于战斗用途；见各页战斗／投掷章节，武器叶不重复登记。
- 伞族四把均为 HANDS 槽；暗影伞雨罩见 [[reference/prefabs/umbrella#暗影伞雨罩机|暗影伞雨罩机]]，传送伞技能门禁与 pad 供电见 [[reference/prefabs/umbrella#传送伞传送与充能|传送伞传送与充能]]。
- 步行手杖的七种皮肤与跟随 FX 见 [[reference/prefabs/fx-skin-index|FX·皮肤与装备挂件]] 及 [[reference/prefabs/cane#皮肤差异与特效映射|手杖皮肤]]。
- 实用与投掷装备的投掷侧（种壳整叠投掷、瓶中信显影）见 [[reference/prefabs/utility-and-throwables|对应页]]；投掷武器族见 [[reference/prefabs/weapons-combat-index|武器与战斗]]。

返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
