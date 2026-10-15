---
title: 生物生成与巢穴组件索引
created: 2026-10-22
updated: 2026-10-29
type: index
tags: [overview]
---

# 生物生成与巢穴

用于生成配额、巢穴／住户、孵化藏身，以及鸟、蝴蝶、食人花和鱼群自然排程。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 巢穴与子体

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/childspawner]] | 巢内／在外计数、补员回巢与外部计时；含蝙蝠昼夜出巢接线 | 洞穴黄昏刷新蝙蝠、蜂箱容量 / active |
| [[reference/components/minionspawner]] | 食人花眼球草编号落点／延时补生／丢失回收与恢复，宿主控制冬眠火伤 | 眼球草补生、位置池 / active |
| [[reference/components/rampingspawner]] | 波次配额／在外追踪／波间任务，实际逐只生成由龙蝇 Brain 与动画驱动 | 递增波次、龙蝇吐熔岩虫 / active |
| [[reference/components/hatchable]] | 蛋破壳／舒适孵化／失败、热源偏好与背包暂停，宿主负责出雏 | 孵化冷热偏好、高脚鸟蛋与岩浆虫卵 / active |
| [[reference/components/homeseeker]] | 生物家园引用／移除监听、回家请求与 Brain／巢穴登记分工 | 召回工蜂、生物回家、设置家园 / active |
| [[reference/components/occupiable]] | 容纳活体的资格／占用／收取、回调与嵌套保存 | 鸟笼占用、放入与取回鸟 / active |
| [[reference/components/occupier]] | 被容纳者的 owner 引用、占用通知与鸟笼宿主协作 | 笼中鸟关联、占用者 owner / active |
| [[reference/components/spawner]] | 单个住户进屋隐藏／原体释放、补员等待、排队与内外保存 | 猪人重生等待减半、保持同一只住户 / active |
| [[reference/components/hideout]] | 暂存并释放原实体、特殊回调与 GUID 恢复；释放返回值和批量循环限制 | 洞穴光源暂存蝙蝠、生物藏身点 / active |
| [[reference/components/periodicspawner]] | 独立周期产物、随机增量／当前计时、密度／重试与离屏存读 | 格罗姆后续产黏液加快、周期生成物品 / active |
| [[reference/components/rockybossspawner]] | 洞穴世界计时把满体型睡眠 rocky 换体升级 rocky_boss、群登记与无群补生 | 岩石Boss升级、岩石龙虾Boss / active |

宠物、召唤与血条见[[reference/components/topics/companions#宠物与召唤|驯养协作]]。

## 飞鸟生成与吸引

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/birdattractor]] | 玩家诱鸟修正按鸟数／最短与最长间隔三键读取，装备来源与清理 | 羽毛帽诱鸟、鸟数修正、spawnmodifier / active |
| [[reference/components/birdspawner]] | 世界按玩家调度飞鸟、数量／时段门禁和直接生成差异，旧配置接口空实现 | 自然刷鸟、SpawnBird、飞鸟生成间隔 / active |
| [[reference/components/batspawner]] | 未导出的追踪／选点代码；黄昏出巢查childspawner宿主接线 | 蝙蝠生成器边界、batspawner / active |


## 季节生物、袭击与 Boss 生成

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/moosespawner]] | 春季驼鹿巢抽样调度，软/硬两路生成 moose 与蛋 | 驼鹿生成调度 / active |
| [[reference/components/wanderingtraderspawner]] | 流浪商人生成调度：出生点登记、世界设置开关与跨档引用 | 流浪商人生成器 / active |
| [[reference/components/forestdaywalkerspawner]] | 地表 daywalker 倒计时埋入垃圾堆与强度成长 | 地表挖掘者生成 / active |
| [[reference/components/squidspawner]] | 森林主模拟乌贼群调度：黄昏/夜晚按月相概率修正 | 乌贼生成器 / active |
| [[reference/components/toadstoolspawner]] | 洞穴蛤蟆菌协调器：worldsettingstimer 全场单只上限 | 蛤蟆菌生成器 / active |
| [[reference/components/yoth_knightmanager]] | 马年骑士神殿与公主帽登记，按冷却生成 knight_yoth | 马年骑士管理器 / active |
| [[reference/components/klaussackspawner]] | Klaus 袋冬季生成：鹿群点择优、worldsettingstimer 计时 | 克劳斯袋生成器 / active |
| [[reference/components/daywalkerspawner]] | 洞穴挖掘者生成：cavejail 门控、天数倒计时、清场造柱竞技场 | 洞穴挖掘者生成 / active |
| [[reference/components/malbatrossspawner]] | 海鸥 Boss 生成：鱼群登记＋worldsettingstimer 计时＋玩家靠近触发 | 海鸥 Boss 生成器 / active |
| [[reference/components/sinkholespawner]] | 蚁狮地陷袭击：按年龄加权选目标，警告后按波次生成陷坑 | 地陷生成器 / active |
| [[reference/components/decoratedgrave_ghostmanager]] | 装饰墓碑守墓幽灵调度，温蒂技能门控限距限量生成 | 守墓幽灵 / active |
| [[reference/components/frograin]] | 春季蛙雨窗口调度：每玩家计时、双蛙量帽、月裂换月蛙 | 蛙雨 / active |
| [[reference/components/beargerspawner]] | 秋季破坏王掷骰、预警生成与击杀冷却 | 破坏王 / active |
| [[reference/components/deerclopsspawner]] | 冬季独眼鹿袭击：结构偏好选目标、离场存档还原、信标召唤 | 独眼鹿 / active |
| [[reference/components/penguinspawner]] | 冬季近水建企鹅族群（冰面＋冰岩），非冬季清场 | 企鹅生成 / active |
| [[reference/components/yoth_hecklermanager]] | 吐槽鸟单名额落殿调度，世界定时器与舞台演出联动 | 吐槽鸟管理器 / active |

## 暗影、月亮与裂隙生物

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/riftthralltype]] | 裂隙门登记爪牙类型，管理器生成时读取分支 | 裂隙爪牙类型 / active |
| [[reference/components/ruinsshadelingspawner]] | 遗迹梦魇区空椅生成暗影小妖，搜刮后一天冷却 | 遗迹暗影小妖生成器 / active |
| [[reference/components/lunaralterguardianspawner]] | 世界组件：找点延时生成月裂守卫并跨存档跟踪 | 月裂守卫生成器 / active |
| [[reference/components/moonbeastspawner]] | 月台充能期波次刷月亮兽，变身替换实体 | 月亮怪兽波次 / active |
| [[reference/components/brightmarespawner]] | 月狂 gestalt 人口生成器，理智档位掷运 | 月狂影灵生成器 / active |
| [[reference/components/shadowthrall_mimics]] | 世界组件：黑暗中物品克隆成拟态，宝箱变拟态箱 | 影子拟态生成 / active |
| [[reference/components/gelblobspawner]] | 裂隙软泥怪生成：暗影门户激活后按生成点管理与回收 | 软泥怪生成器 / active |
| [[reference/components/shadowhandspawner]] | 夜间按玩家刷暗影之手与船上 waveyjones，含年龄/理智/虚拟房间门禁 | 暗影之手生成器 / active |
| [[reference/components/shadowcreaturespawner]] | 按玩家理智的人口制影怪生成：海陆分支、离屏即删 | 暗影生物生成器 / active |
| [[reference/components/grottowarmanager]] | 地底战争：archives 破关布防＋推 wild 相位，lunacyarea 刷 mare | 地底战争 / active |
| [[reference/components/shadowparasitemanager]] | 暗影裂隙寄生体波次：宿主池、blob 拦截、玩家替身 | 寄生体 / active |
| [[reference/components/lunarthrall_plantspawner]] | 月裂隙达峰后波次投放 gestalt 寄生植物 | 月裂隙植物潮 / active |

## 生物群体、迁徙与活动

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fishschool]] | 鱼群等级维护，被网时按级生成鱼并周期补充 | 鱼群等级 / active |
| [[reference/components/lavaarenamobtracker]] | 熔炉怪物登记表；GetNumMobs 恒 0 已记陷阱 | 熔炉怪物追踪 / active |
| [[reference/components/deerherding]] | 世界级鹿群放牧：吃草/游荡节奏、受惊逃离、城市化与盐舔判 inactive | 鹿群放牧 / active |
| [[reference/components/deerherdspawner]] | 秋季召鹿群/冬季迁徙排程、活跃鹿表，与 deerherding 分工 | 鹿群生成器 / active |
| [[reference/components/mutatedbuzzardcircler]] | 月雹影鸟盘旋：尸体降落投真身、信号弹击落 | 变异秃鹫盘旋 / active |
| [[reference/components/mutatedbuzzardmanager]] | 变异秃鹫 gestalt 迁徙附身玩家盘旋与弃置管理 | 秃鹫迁徙 / active |
| [[reference/components/migrationmanager]] | 拓扑迁移图上种群组沿节点周期移动，变异秃鹫为唯一原版使用者 | 生物迁徙 / active |
| [[reference/components/hunter]] | 每玩家一场土堆-脚印狩猎，按分数/季节/活动结算猎物 | 脚印狩猎 / active |
| [[reference/components/houndedtarget]] | 猎犬目标权重＋偷猎犬者身份，hounded 消费 | 猎犬目标权重、猎犬小偷 / active |
| [[reference/components/circler]] | 普通秃鹫影子的绕目标位移／朝向／缩放及唤醒重随机，不是寻路 AI | 秃鹫盘旋影子、圆周移动 / active |

## 生物状态、变形与任务

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/sharkboimanagerhelper]] | 竞技场 net_float 同步助手：双端 IsPointInArena 瓦片对齐判定 | 竞技场判定、鲨鱼竞技场网络同步 / active |
| [[reference/components/mermcandidate]] | 鱼人吃食计卡路里，达标供 mermkingmanager 变身 | 鱼人王候选 / active |
| [[reference/components/spidermutator]] | 变异器物品把蜘蛛换成目标 prefab，执行即消耗自身 | 蜘蛛变异器 / active |
| [[reference/components/shadowsubmissive]] | 暗影生物臣服：持压制标签者不被敌对，受击反制 | 暗影臣服 / active |
| [[reference/components/despawnfader]] | NPC 淡出移除：1 秒渐隐并网络同步，终至移除 | 淡出移除 / active |
| [[reference/components/halloweenmoonmutable]] | 月突变：Mutate 原地换实体、迁血量与容器 | 月突变、Mutate / active |
| [[reference/components/moonstormstaticcapturable]] | 静电可捕捉目标：瞄准集合/开关/被抓结算 | 静电捕捉目标 / active |
| [[reference/components/questowner]] | 接/弃任务宿主：questing 状态与 tag 支撑 BEGIN/ABANDON_QUEST 动作 | 任务宿主 / active |
| [[reference/components/knowndynamiclocations]] | 命名记忆点（DynamicPosition）随船漂移，平台移除回退世界坐标，可存档 | 动态位置记忆 / active |
| [[reference/components/prophider]] | 藏身替身道具组件 | 藏身替身 / active |
| [[reference/components/werebeast]] | 满月/触发计数驱动的兽形态切换与限时变回 | 月圆变身 / active |
| [[reference/components/guardian]] | 计数召唤守护者：满额自动召、衰减/死亡即解除 | 召唤守护兽 / active |
| [[reference/components/kramped]] | 淘气值计数衰减与坎普斯生成，NAUGHTY_VALUE 驱动 | 淘气值管理器 / active |
| [[reference/components/centipedebody]] | 蜈蚣型 Boss 体节编排：P2P 链、双头换控、吃瘴气增节 | 蜈蚣体节 / active |
| [[reference/components/mermkingmanager]] | 鱼人王候选登记/变身/王位更替与跨分片查询 | 鱼人王 / active |

## 蝴蝶与食人花自然生成

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/butterflyspawner]] | 白天非冬按玩家／花点／局部数量排程，野外追踪与库存切换 | 花旁不刷蝴蝶、蝴蝶休眠消失 / active |
| [[reference/components/lureplantspawner]] | 春季按玩家轨迹与远点尝试生成，幸运概率／地皮／间隔及旧模式空实现 | 春季不刷食人花、自然食人花频率 / active |

## 鱼群与海上生物生成

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/schoolspawner]] | 自然海鱼密度／阻挡幸运、季节选鱼与直接生鱼，含伴随生物及延后计数 | 海鱼群生成、鱼书、诱鱼剂 / active |

## 随从与移动

驯化、骑乘、随从和编队见[[reference/components/topics/companions|驯养、随从与队伍]]；单体移动与检测见[[reference/components/topics/movement|移动、距离与传送]]。

## 组队包围与轮流攻击

入口已移至[[reference/components/topics/companions#编队与战斗协作|编队与战斗协作]]。

## 实体替换与变形

入口已移至[[reference/components/topics/world-management#实体替换与恢复|实体关联与世界管理]]。

## 实体关联、距离与传送

实体引用、绑定和房间见[[reference/components/topics/world-management|实体关联与世界管理]]；位置、距离与通用传送见[[reference/components/topics/movement|移动、距离与传送]]。

## 暗影任务线与查理Boss状态

入口已移至[[reference/components/topics/world-management#暗影任务线与查理boss状态|暗影任务线与查理Boss状态]]。

## 生物协作与实体关联

生成与巢穴留在本页；协作见[[reference/components/topics/companions|驯养、随从与队伍]]，实体引用见[[reference/components/topics/world-management|实体关联与世界管理]]，保留原总标题锚点。

返回 [[reference/components/index|组件主题导航]]。
