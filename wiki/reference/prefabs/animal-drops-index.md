---
title: "动物掉落与巢材（暂行索引）"
created: "2026-11-10"
updated: "2026-11-10"
type: index
tags: [overview]
---

# 动物掉落与巢材（暂行索引）

非装备物品栏 Prefab 扩库 B11 登记：18 个物品注册（14 页）＋宿主页 `beefalo`；B12 登记：10 个物品注册（6 页）。`malbatross_feather_fall` 与 `beefalo_carry` 都是载体、不计物品数；正式的叶级细分留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/beardhair|胡须（beardhair）]] | 兔子 beardling 池 20%、梦魇猴 100%、伯尼 1440 秒腐坏、兔王 minion 3/16、威尔逊剃须按 bit；40 叠／45 秒燃料／雪人贴饰；作祟只弹飞 | 胡须、兔毛、雪人装饰、燃料 / active |
| [[reference/prefabs/manrabbit-tail|兔绒（manrabbit_tail）]] | 兔人 3/3/2 只抽 1（25%）、吸血鬼兔固定含 1、兔王 minion 4/16；HORRIBLE 10/10/0、40 叠、金值 2；可喂眼面具修甲 | 兔绒、兔人尾巴、毛皮铺盖材料 / active |
| [[reference/prefabs/beefalowool|牛毛（beefalowool）]] | 刷毛跨天 1 份、剃毛 3 份、成牛非保尸死亡保底 3、幼牛 1/2/2；DropLoot 结算时仍燃烧则转灰；40 叠／45 秒燃料／金值 0；伯尼腐坏也给 1 | 牛毛、刷牛毛、剃牛毛 / active |
| [[reference/prefabs/beefalo|皮弗娄牛（beefalo）]] | 牛群中心漂移 6–8 秒（每实例固定周期）、剃毛要睡眠、3 天再生、非保尸死亡 4 肉＋3 牛毛＋33% 牛角、DropLoot 结算时仍燃烧才转灰、骑乘死亡待甩下才结算、暗影铃保尸先熄火不掉落；辅助注册 beefalo_carry 不计物品 | 皮弗娄牛、牛群、剃牛毛、beefalo_carry / active |
| [[reference/prefabs/coontail|猫尾（coontail）]] | 浣猫 meat 1.00＋猫尾 0.33 同表独立判定、大虎 0.05；默认 20 叠、六条配方；cattoy 给忠诚（48 秒是参数，实得受 leader 倍率与 480 秒上限约束）但被敌视时只吞不给；地上会被叼走 | 猫尾、Cat Tail、浣猫掉落 / active |
| [[reference/prefabs/furtuft|毛丛（furtuft）]] | 熊獾 45 秒 1 撮、震地 20%×3、到点 25–35 秒进入消失序列；40 叠／45 秒燃料、90 撮换 3 熊皮、尸体仅在 meat_level≥2 且多件表时按剩余肉量折算 | 毛丛、熊毛、Fur Tuft / active |
| [[reference/prefabs/pigskin|猪皮（pigskin）]] | 猪人／守卫 3:1 抽 1（25%）、疯猪与科学怪猪与月光猪固定出表、猪王与精英猪无掉落、猪头锤击 2、燃烧转灰；玩家不可食却能喂眼面具 | 猪皮、Pig Skin、猪人皮 / active |
| [[reference/prefabs/bird-feathers|四色鸟羽（feather_crow、feather_robin、feather_robin_winter、feather_canary）]] | 同工厂四注册、属性零差异（40 叠／燃料 7.5 秒／燃烧 7.5 秒／可装饰雪人）、金丝雀 1/11；tradable 无金值故猪王不收；16 条制作＋1 条拆解（按颜色展开 18 行） | 黑色羽毛、红色羽毛、蓝色羽毛、黄色羽毛、鸟羽、金丝雀羽毛、吹箭材料 / active |
| [[reference/prefabs/goose-feather|麋鹿鹅羽毛（goose_feather）]] | 20 叠／燃料 7.5 秒／燃烧 3 秒、无 tradable；麋鹿鹅 3＋2×0.33、小鹅 2＋0.33、克劳斯袋 2/9、弃船宝箱整堆同色 2–5；6 条配方 | 麋鹿鹅羽毛、鹅毛、羽毛扇材料、龙卷风法杖材料 / active |
| [[reference/prefabs/malbatross-feather|邪天翁羽毛与羽毛帆布（malbatross_feather、malbatross_feathered_weave）]] | 羽毛不在死亡表，靠受击 4×0.05 与撞桅 3＋2×0.3 的 spawnfeather（撞桅限可做工的桅杆、同目标 3 秒去重；另有弃船宝箱）；帆布 6 羽毛＋1 丝、只进 4 块一组的邪天翁帆；20 叠但燃料与标签不同 | 邪天翁羽毛、羽毛帆布、邪天翁帆材料 / active |
| [[reference/prefabs/honeycomb|蜜脾（honeycomb）]] | 默认 20 叠、honeyed 但 edible 被注释：不能吃／不被熊獾吃／却能进炖锅；蜂窝 3 蜜＋1 脾、杀人蜂蜂窝同表、蜂王 1.00 必中＋0.50 先过 GetChance（期望 1.5 为无幸运基准）、蜂王冠拆解按耐久独立向上取整（80% 仍 4、75% 起 3）、长成巢锤击概率＝0.01×本击扣减后工作量/16（仅工作量从 16 起、本击扣 1 时首锤为 0.9375%；归零不给） | 蜜脾、honeycomb、蜂箱材料、蜂王冠拆解 / active |
| [[reference/prefabs/houndstooth|犬牙（houndstooth）]] | 40 叠＋selfstacker 速度门槛、blowpipeammo 只给嚎弹炮弹舱；七种猎犬与座狼／犬骨／猎犬丘（300 血、不可锤）／宝箱拟态掉率；风滚草约 1%／抽 | 犬牙、houndstooth、嚎弹炮弹药、猎犬掉落 / active |
| [[reference/prefabs/stinger|针刺（stinger）]] | 40 叠、忽略速度并堆、燃烧掉同叠灰烬；蜜蜂与杀人蜂 1:5 权重、蜂王 1.00 必中、嗡嗡蜜蜂 1% 为无幸运基准；无 edible 但无锅料登记（锅按注册名收料）；睡眠吹箭与碎裂弹等配方 | 针刺、stinger、睡眠吹箭、弹弓弹药 / active |
| [[reference/prefabs/tentaclespots|触手皮（tentaclespots）]] | 40 叠／15 秒燃料／7.5 秒燃烧；触手 0.2、大触手 0.40、变异鱼人 0.25、巨石枝地形权重 2；鞭子等九条配方 | 触手皮、tentaclespots、鞭子、燃料 / active |
| [[reference/prefabs/walrus-tusk|海象牙（walrus_tusk）]] | 10 叠大件、无标签、不可燃不当燃料；宿主来源仅成年海象 0.50（无幸运基准；小海象与营地不掉）；另有拆解回收：步行手杖／刷子直接可拆各返 1 牙，回溯怀表按 no_deconstruction 求值判定 | 海象牙、walrus_tusk、步行手杖、海象掉落 / active |

| [[reference/prefabs/boss-materials|大型生物掉落材料：熊皮与鳞片（bearger_fur、dragon_scales）]] | 熊獾与龙蝇必掉；龙蝇倒地累计血量变化 2500 补鳞一次（正负都计、同实例至多一次、组件不存档）；克劳斯袋 0–1；尸体被啃到 2 级且原列表多项含熊皮时按 30 基数折算 1–20 撮毛丛（啃空按旧级别结算）；熊皮未换算时普通燃烧转灰、受控燃烧保留（不闷烧），鳞片不进入该路径；冰包／熊皮背心／极地熊獾桶（LUNARFORGING=2＋nounlock）／龙鳞火炉（TECH.LOST 蓝图）／鳞甲／龙鳞宝箱／龙鳞地板／坍塌宝箱重建的配方与拆解 | 熊皮、鳞片、龙蝇掉鳞、龙鳞火炉、极地熊獾桶、熊皮背心 / active |
| [[reference/prefabs/boss-parts|大型生物掉落器官：独眼巨鹿眼球与守护者之角（deerclops_eyeball、minotaurhorn）]] | 可食 60／75／−15 MEAT、badfood、不可烤不可进锅；眼球 tradable 值 0 被猪王与蚁狮拒收（不吞物品）、雪人装饰 canflip、可交存哨戒设施（GiveItem 入、TakeItem 出、锤拆归还）；眼球伞与眼睛炮塔配方、eyeturret 显式拆解与眼球伞普通配方拆解；变异独眼巨鹿表不含眼球 | 独眼巨鹿眼球、守护者之角、眼球伞、眼睛炮塔、雪人装饰 / active |
| [[reference/prefabs/shell-materials|海陆生物外壳（饼干切割机壳、外壳碎片）]] | cookiecutter 0.50＋0.25 两条独立掷骰 0–2 个；slurtle／snurtle 的 AddIfNotChanceLoot 兜底（1.0 条目不置 flag，头盔／护甲未中签才补 1）；slurtlehole 死亡回调后 0.66 秒掉落（烧尽爆炸会取消掉落回调）；tradable 空壳（goldvalue 0）；repairer 20 点为请求值、按生命上限截断（95/100 回 5）且只修贝壳保险杠；普通燃烧转灰、受控燃烧保留（碎片自身无 burnable、不闷烧）；隐士蟹线 TECH.LOST／nounlock／装饰分门禁；绿杖与锤拆两套拆解数量 | 饼干切割机壳、外壳碎片、贝壳保险杠、隐士蟹、贝壳地皮 / active |
| [[reference/prefabs/steelwool|钢丝绵（steelwool）]] | 45 秒燃料兼 15 秒可燃（烧尽整堆变等量灰烬；海上不产灰、灰烬受堆叠上限）、作祟只弹起；spat 2–3 块（常规燃烧死亡转灰、受控燃烧保留原名）、克劳斯袋 0–1；战争鞍具／刷子／小钢羊宠物蛋／兔人神龛两件枕头配方与门禁 | 钢丝绵、钢丝绒、钢羊掉落、兔人神龛枕头、燃料 / active |
| [[reference/prefabs/lightninggoathorn|伏特羊角（lightninggoathorn）]] | 伏特羊两张表各 0.25（基础概率，正向幸运提高、≤0 不修正）；普通燃烧／烧毁尸体转灰，受控燃烧保留原名落地；堆叠 10、作祟只弹飞；暗夜灯／龙卷风法杖／奔雷矛（技能门禁）与 Warly 便携锅食材；锅料 inedible=1 不可烤 | 伏特羊角、闪电羊角、暗夜灯材料、奔雷矛、龙卷风法杖 / active |
| [[reference/prefabs/cave-drops|洞穴掉落：蘑菇皮（shroom_skin）与啜食者皮（slurper_pelt）]] | 蘑菇皮：毒菌蟾蜍阶段抛出（血量阈值 70%／40%，悲惨版另 20%；healthtrigger 跨阈值、秒杀反例）＋击杀表 1／2 张（普通燃烧转灰、受控保留）；蘑菇灯／菌伞灯／睡袋蓝图（TECH.LOST）。啜食者皮：0.5 基础概率（正向幸运提高、≤0 不修正）、HORRIBLE 默认 10/10/0 玩家不可食、金值 1、可燃（受控燃烧保留并冒烟）；饥饿腰带与 WX-78 饥饿模块 | 蘑菇皮、啜食者皮、毒菌蟾蜍掉落、睡袋蓝图、饥饿腰带 / active |

返回 [[reference/prefabs/index|原版 Prefab 索引]]。