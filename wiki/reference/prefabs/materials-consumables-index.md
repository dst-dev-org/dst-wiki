---
title: "材料、照明与特殊生物（暂行索引）"
created: "2026-11-09"
updated: "2026-11-10"
type: index
tags: [overview]
---

# 材料、照明与特殊生物（暂行索引）

非装备物品栏 Prefab 扩库 B08 登记：冰与迷你冰川、荧光果与南瓜灯、格罗姆花与格罗姆、蘑菇树孢子、晶洞果与夜莓、果蝇族、月熠、小痒痒、双尾心。`wortox_reviver_body` 的主注册归 [[reference/prefabs/wortox_soulecho_buff_fx]]，本索引只登记 `wortox_reviver` 主体；正式的叶级细分留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/ice|冰（ice）]] | 迷你冰川采掘产物，变异巨鹿掉落亦给冰；可食冷食、一次性水源、灭火与冰帽鲜度修补（右键＋材料标签匹配，满鲜有候选但不消耗）；地面腐坏按 25/件**请求**加土壤湿度（夹取前值，单格上限 100），持有者分支加潮湿 | 冰、冰块、冰灭火、冰补水 / active |
| [[reference/prefabs/rock-ice|迷你冰川（rock_ice）]] | 五阶段随季节升降的可再生冰源；采掘逐级随机掉冰、火烤约 4 秒融一级、沉没按当前级掉冰；名称随阶段在“迷你冰川／融化的迷你冰川”间切换 | 迷你冰川、冰矿、企鹅冰、冰川融化 / active |
| [[reference/prefabs/lightbulb|荧光果（lightbulb）]] | 洞穴荧光花与发光蟹／球状光虫来源；落地才亮的小光源、提灯与矿工帽的 CAVE 燃料、可食蔬菜；月亮精华液可变异为球状光虫 | 荧光果、提灯燃料、矿工帽燃料、荧光花 / active |
| [[reference/prefabs/pumpkin-lantern|南瓜灯（pumpkin_lantern）]] | 南瓜＋萤火虫合成；只在地面且非白天（黄昏与夜晚，`not isday`）触发淡入、入昼延迟 2+random() 秒才淡出、进包暂停腐坏；腐坏先播 rotten（受击死亡才播 broken），两路都掉萤火虫；三条世界获得都只在万圣节活动期间 | 南瓜灯、南瓜灯笼、万圣节南瓜灯 / active |
| [[reference/prefabs/glommer-flower|格罗姆花与格罗姆（glommerflower、glommer）]] | 主注册 `glommerflower`／`glommer`（雕像只作获得宿主）；雕像满月放花、非满月收花；花作 leader 绑定格罗姆、黏液 2–4 天；原版死花只挂 6 天定义值、从未启动计时，不会腐烂 | 格罗姆、格罗姆花、死花、格罗姆雕像 / active |
| [[reference/prefabs/mushtree-spores|蘑菇树孢子（spore_small、spore_medium、spore_tall、spore_moon）]] | 三色孢子：蘑菇树开花期喷发、NET 拾取、地面约 60 秒／持有者 15 天、可作农场种子（限彩色孢子）；月亮孢子三条入口——砍过的月亮树、月亮蘑菇帽（帽产跳过 `pre_pop` 直接爆）、蘑菇地精（周期喷发＋攻击 25–36 帧逐帧生成＋死亡表掉落）；生成门看同 prefab、拥挤门看全部 `spore` | 孢子、绿色孢子、蓝色孢子、月亮孢子 / active |
| [[reference/prefabs/ancient-fruit|晶洞果与夜莓（ancientfruit_gem、ancientfruit_nightvision、ancientfruit_nightvision_cooked、nightvision_buff）]] | 晶洞果靠火源升温到 35 才跑 30 秒开壳计时、按权重掉宝石；夜莓生熟两态，仅生莓可烤且食用后给 240 秒夜视 buff（需 playervision，重吃只重置）；负生命／负理智是否计入取决于 eater 的例外分支 | 晶洞果、夜莓、夜视、宝石果（俗称）/ active |
| [[reference/prefabs/fruit-fly|果蝇王与果蝇（lordfruitfly、fruitfly、friendlyfruitfly、fruitflyfruit）]] | 果蝇王首刷看 35 天计时（作物腐烂每次把计时前拨 240 秒），放行还要 `SPAWN_LORDFRUITFLY`、首轮玩家 15 距离（仅 `check_others=true`）与候选点 4 距离内 ≥15 个 spawner；死亡额外掉果或种子（`plantmeat` 照掉）；王死后已闹过（`hascausedhavoc`）且无 leader 的才转敌对；干壳不腐烂 | 果蝇王、果蝇、友好果蝇、授粉 / active |
| [[reference/prefabs/moonstorm-spark|月熠（moonstorm_spark）]] | 月风暴中生成、需手持 NET 工具捕捉；定义 360 秒、地面约 240 秒、持有者约 60 天；生成按逐玩家重数（月风暴内玩家半径 30 内少于 3 只才刷，不是全场上限）；5–15 秒是结算到下次前置 FX 的间隔（伤害另有 5 帧前摇）；充能每次 +10%，10 次≈一条满值回充量（不等于护目镜的实际流程），不是 10 只火花 | 月熠、月风暴火花、火花捕捉 / active |
| [[reference/prefabs/mosquitomusk|小痒痒（mosquitomusk）]] | 沃特一阶技能配方；携带时蚊子常规重选与炸弹刷出时的初始选敌跳过携带者；受击后仍可 SetTarget；炸弹的独立范围伤害不读该标签，携带者只要符合其目标筛选也会被命中（例外）；血囊刷鲜度、8 天腐坏成腐烂物、不可堆叠 | 小痒痒、驱蚊、沃特配方 / active |
| [[reference/prefabs/wortox-reviver|双尾心（wortox_reviver）]] | 沃拓克斯生命使者线：10 灵魂配方、linkeditem 按 userid 绑主人、幽灵复活、施法传送到主人身边；10 天是正常计时定义值；施法提前中断会停表且不重启；动作失败会重启并保留物品，成功则重启并消耗（`OnConsume`），腐坏返还 1 灵魂 | 双尾心、沃拓克斯复活、生命使者 / active |

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
