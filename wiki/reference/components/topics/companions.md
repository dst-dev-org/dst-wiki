---
title: 驯养、随从与队伍组件索引
created: 2026-10-27
updated: 2026-10-29
type: index
tags: [overview]
---

# 驯养、随从与队伍

用于宠物所有者、群体、驯化骑乘、随从和战斗编队；自然生成和巢穴见[[reference/components/topics/world-creatures|生物生成与巢穴]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 宠物与召唤

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/petleash]] | 主人宠物集合、普通／专用限额、生成／接管与嵌套存读 | 威尔逊普通宠物容量改为二、宠物生成与接管 / active |
| [[reference/components/pethealthbar]] | 玩家侧单宠物血条、双端字段／事件、显示上限与 HUD 分工 | 刷新阿比盖尔血条上限、宠物血条同步 / active |
| [[reference/components/summoningitem]] | 空召唤物品标记、CASTSUMMON到ghostlybond的分派及阿比盖尔花轮盘边界 | 召唤标记、阿比盖尔花、CASTSUMMON / active |
| [[reference/components/healthbar]] | 实体头顶世界空间百分比血条、双端显示实体与 net_float，只做显示 | 头顶血条、世界空间血条 / active |
| [[reference/components/crittertraits]] | 小宠物四类积分／主导锁、两端事件／SG 计分与存读 | 小猫正向玩耍性格收益翻倍、宠物主导性格 / active |

## 群体与发情

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/herd]] | 群体集合／阈值、聚集／合并、中心坐标、GUID 重连与节点清理 | 牛群目标上限改为二十、牛群中心 / active |
| [[reference/components/herdmember]] | 成员引用／标签、延迟建群、退出／禁用及读档登记分工 | 草蜥蜴不自动入群、生物退出群体 / active |
| [[reference/components/mood]] | 群体发情天数／季节、进入／退出回调与存读倍率，区分实际繁殖 | 延长牛群非发情等待、牛群发情周期 / active |

## 驯化骑乘与照料

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/domesticatable]] | 驯化／服从／倾向、锁存完成、衰减与骑具保存，区分公开和内部增量 | 刷毛驯化收益翻倍、牛驯化度 / active |
| [[reference/components/beefalometrics]] | 牛事件统计与最近参与者实体／userid，区分实际驯化和所有权 | 读取牛最近驯养参与者userid、牛行为统计 / active |
| [[reference/components/rideable]] | 坐骑侧鞍交接、骑手引用／事件、服从测试与嵌套保存 | 上牛需要七成服从、上鞍资格 / active |
| [[reference/components/rider]] | 骑手上下骑转换、事件／鞍跟踪、同步转发与保存 | 骑乘时每分钟加三点精神、玩家下骑 / active |
| [[reference/components/saddler]] | 鞍侧速度／攻击／防御、同步通道与丢弃耐久分工 | 普通牛鞍速度倍率、鞍减伤 / active |
| [[reference/components/unsaddler]] | 卸鞍工具标记、候选／执行与动作消费分工 | 鞍角卸鞍耐久减半、卸鞍工具标记 / active |
| [[reference/components/hitchable]] | 被拴生物引用／标签、双方解拴与 Brain／宿主清理 | 服务端释放拴牛、牛解拴 / active |
| [[reference/components/hitcher]] | 拴柱占用／回调、候选锁与空存读／销毁清理 | 装扮台锁定拴牛交互、拴柱占用 / active |
| [[reference/components/groomer]] | 装扮台用户／占用、距离关闭、皮肤回调与禁用清理 | 装扮台换装距离、关闭牛换装 / active |
| [[reference/components/skinner-beefalo]] | 牛五部位服装、事件／显示、原脱装与牛铃恢复分工 | 清除牛装扮、牛皮肤部位、skinner_beefalo / active |
| [[reference/components/saltlicker]] | 生物舔盐搜索／状态、延后耗用、暂停与盐砖通知分工 | 牛舔盐消耗减半、盐砖防驯化衰减 / active |
| [[reference/components/brushable]] | 可刷目标的天数／奖品、回调、门禁表达式与存读 | 刷毛积攒两份牛毛、刷毛产出间隔 / active |
| [[reference/components/shaveable]] | 剃取资格／奖品与宿主消耗，区分毛发生长、巢降级和植物共享产量 | 一级蜘蛛巢不允许剃取、剃蜘蛛巢 / active |
| [[reference/components/bedazzlement]] | 蜘蛛巢装饰／成长暂停、周期 Buff、宿主受击／剃取与恢复 | 服务端解除蜘蛛巢装饰、装饰蜘蛛巢停止成长 / active |

## 随从与主人

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/leader]] | 主人端登记／数量、目标建议与忠诚增量倍率，区分物品关联集合 | 招募随从、喂食忠诚收益、随从数量 / active |
| [[reference/components/follower]] | 随从归属／期限、实际跟随协作、玩家缓存与解除清理 | 忠诚时间、跟随主人、不共享战斗目标 / active |
| [[reference/components/followerherder]] | 物品开关写现有随从实体 hostile 字段、使用资格与蜘蛛哨额外回调 | 蜘蛛哨、驱赶随从 / active |
| [[reference/components/followermemory]] | 按 userid 与角色记住原主人、失主等待／重逢与恢复，遗忘不解除归属 | 随从记住主人、发条生物重逢 / active |
| [[reference/components/leaderrollcall]] | 物品或实体定期招募猪人／鱼人、已有随从忠诚与农作物照料分派 | 独奏乐器招募、点名上限与距离 / active |

## 编队与战斗协作

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/formationleader]] | 环形编队位置／多圈分层、成员容量与退出副作用，中心移动由宿主处理 | 编队队长、围绕玩家飞行 / active |
| [[reference/components/formationfollower]] | 编队成员搜队／坐标回调、收纳与休眠；active 不直接停止更新 | 编队成员、发光飞虫搜索半径 / active |
| [[reference/components/teamattacker]] | 成员队伍引用／命令与站位、战斗和回家协作，区别普通跟随关系 | 团队攻击成员、组队包围、ORDERS / active |
| [[reference/components/teamleader]] | 战斗队伍招募／编队／轮攻、威胁与解散，容量和成员退出边界 | 团队攻击调度、蝙蝠包围、轮流攻击 / active |
| [[reference/components/commander]] | 士兵集合、目标共享、距离追踪与离队事件，区别普通随从所有权 | 蜂后护卫、士兵指挥、ForEachSoldier / active |
| [[reference/components/squadmember]] | 同名成员事件互相登记、精英猪人选敌引用；退队和换组监听限制 | 小队成员集合、精英猪人 / active |
| [[reference/components/grouptargeter]] | 目标池权重与换目标尝试概率、增删重分配，筛选与战斗归宿主 | 加权选敌、蜂后换目标 / active |

单体移动、地面影响和位置检测见[[reference/components/topics/movement|移动、距离与传送]]；刷剃工具见[[reference/components/topics/item-maintenance#刷剃与装饰工具|物品维护]]。


## 宠物照料与伙伴关系

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/kitcoon]] | 小猫可空标记，PET 命中推 on_petted | 小猫可宠物标记、on_petted / active |
| [[reference/components/ghostlyelixirable]] | 幽灵灵药目标标记，可覆写实际受益人 | 阿比盖尔灵药目标 / active |
| [[reference/components/cattoy]] | 猫玩具交互：Play 回调＋BypassLastAirTime，配 cattoy 标签被猫脑检索 | 猫玩具、逗猫交互 / active |
| [[reference/components/migrationpetowner]] | 物品声明随身宠物，迁移与房间重置据此保留放行 | 迁移宠物持有者 / active |
| [[reference/components/kitcoonden]] | 小猫狸窝收容账本：增减员、onremove 自动清点 | 小猫狸窝 / active |
| [[reference/components/ghostbabysitter]] | 幽灵照看名单（孤儿组件）：存 GUID 存档，与温蒂 babysitter 机制无关 | 幽灵照看者 / active |
| [[reference/components/wobycourier]] | Woby 快递箱坐标分片存取＋netvar 地图标记 | 沃比快递 / active |
| [[reference/components/wx78_dronescouttracker]] | 侦察无人机登记/事件通知/分片存取 | 侦察无人机追踪 / active |
| [[reference/components/sisturnregistry]] | 世界级祭坛注册表：汇总有花/月树花状态广播 onsisturnstatechanged | 姐弟祭坛注册表 / active |
| [[reference/components/friendlevels]] | 任务式好感等级：升级入队奖励＋事件通知，存档顺序敏感 | 好感等级、友谊任务 / active |
| [[reference/components/hideandseekhider]] | kitcoon 捉迷藏：GoHide 入点/找出/中止，每日一次限制 | 捉迷藏藏匿者 / active |
| [[reference/components/ghostlybond]] | 玩家-幽灵纽带：生成/召回/行为切换与 1~3 级成长 | 幽灵纽带 / active |
| [[reference/components/pearldecorationscore]] | 蟹后小屋 19 类装饰按占地网格评分 0-100，事件驱动解锁 | 珍珠装饰评分 / active |

## 召唤物与随从追踪

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/clockworktracker]] | 车/马/象三类发条随从上限与加成槽管理 | 发条随从计数 / active |
| [[reference/components/sentientaxe]] | 露西人格：owner 生命周期监听＋场景台词，经 lucy_classified 播放 | 会说话的斧子 / active |
| [[reference/components/socket_shadow_harvester]] | WX-78 影子收割无人机调度：宠物自动拾取与虚拟收割 | 影子收割插座 / active |

返回 [[reference/components/index|组件主题导航]]。
