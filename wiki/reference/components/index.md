---
title: 组件索引
created: 2026-10-10
updated: 2026-10-29
type: index
tags: [overview]
---

# 组件

已知组件名称或章节时直接读 `reference/components/<组件名>.md`，无需经过主题索引。未知名称先搜索页面摘要／别名，或按需求只展开一个相关主题，再选少量页面；跨主题机制用正文互链。

## 原版接入量 Top 30

统计快照 `steam-buildid:26148036`，日期 2026-10-27。按原版 `scripts/` 中直接调用 `:AddComponent("组件名")` 的不同 Lua 文件数降序，同数按组件名排序；同文件重复调用只计一次，跳过注释与字符串中的伪调用。

这是静态直接接入量，不是查询频率、实体数量或完整使用率；不展开公共 helper、工厂变体或动态参数，`burnable` 等经 helper 挂载的组件会被低估。下列仅为快捷入口，已有页面的唯一叶级登记仍在主题索引；缺页不补位，也不表示原版能力缺失。

1. [[reference/components/inspectable|inspectable]] — 1016 文件；检查交互与描述。
2. [[reference/components/inventoryitem|inventoryitem]] — 522 文件；拾取、入栏与持有者。
3. [[reference/components/lootdropper|lootdropper]] — 455 文件；掉落名单与散落。
4. [[reference/components/workable|workable]] — 294 文件；砍、挖、锤与工作量。
5. [[reference/components/stackable|stackable]] — 226 文件；堆叠、拆分与消耗。
6. [[reference/components/combat|combat]] — 211 文件；攻击、目标与受击。
7. [[reference/components/timer|timer]] — 206 文件；具名计时、暂停与保存。
8. [[reference/components/locomotor|locomotor]] — 191 文件；移动、目的地与预测。
9. [[reference/components/health|health]] — 188 文件；生命上限、回血与再生。
10. [[reference/components/tradable|tradable]] — 162 文件；可交付物品端数据。
11. [[reference/components/equippable|equippable]] — 151 文件；装备槽、回调与穿戴效果。
12. [[reference/components/hauntable|hauntable]] — 145 文件；幽灵作祟与冷却。
13. [[reference/components/knownlocations|knownlocations]] — 123 文件；命名位置快照与存读。
14. [[reference/components/fuel|fuel]] — 122 文件；燃料物品类型与燃料量。
15. [[reference/components/updatelooper|updatelooper]] — 120 文件；更新回调调度与清理。
16. [[reference/components/sanityaura|sanityaura]] — 100 文件；附近精神光环。
17. [[reference/components/sleeper|sleeper]] — 96 文件；生物睡眠与催眠。
18. [[reference/components/edible|edible]] — 94 文件；食物类型与三维收益。
19. [[reference/components/weapon|weapon]] — 89 文件；物品伤害与攻击回调。
20. [[reference/components/entitytracker|entitytracker]] — 78 文件；命名实体关联与恢复。
21. [[reference/components/finiteuses|finiteuses]] — 76 文件；动作次数与耗尽回调。
22. [[reference/components/trader|trader]] — 71 文件；接收交易、投喂与拒收。
23. [[reference/components/eater|eater]] — 69 文件；食用者饮食与吸收规则。
24. [[reference/components/inventory|inventory]] — 68 文件；持有者物品栏与装备。
25. [[reference/components/perishable|perishable]] — 67 文件；腐败、暂停与变质。
26. [[reference/components/drownable|drownable]] — 61 文件；落水判定与事件。
27. [[reference/components/deployable|deployable]] — 59 文件；物品部署与位置检查。
28. [[reference/components/planardamage|planardamage]] — 59 文件；位面伤害与来源修饰。
29. [[reference/components/container|container]] — 56 文件；容器格子、过滤与开关。
30. [[reference/components/fueled|fueled]] — 52 文件；持续消耗、补充与耗尽。

## 按需求浏览

按需求选下列一个主题；这些分组直接链接叶级索引，不增加必读的中间页。

### 人物与环境

| 主题入口 | 覆盖需求 |
|---|---|
| [[reference/components/topics/character|人物状态与外观]] | 三维、饮食、睡眠、人物资料、外观换装、调谐与模块宿主 |
| [[reference/components/topics/environment|环境影响与防护]] | 温湿度、天气与风暴、冻结滑面、环境状态与表现 |
| [[reference/components/topics/movement|移动、距离与传送]] | 移动预测、地面受力、水陆切换、位置检测、通用传送 |

### 物品与交互

| 主题入口 | 覆盖需求 |
|---|---|
| [[reference/components/topics/items|物品收纳与显示]] | 拾取、持有者、堆叠、容器、打包、物品温湿度与显示 |
| [[reference/components/topics/item-maintenance|物品消耗与维护]] | 燃料与充能、补水、治疗修理、刷剃装饰、维护材料 |
| [[reference/components/topics/processing|物品加工与产物]] | 食物与保鲜、烹饪晾晒、消化焚烧、条件产物 |
| [[reference/components/topics/item-actions|物品交互与施法]] | 交易部署、使用切档、重物交互、施法、读写演奏与传送工具 |
| [[reference/components/topics/abilities|交互技能与持续效果]] | 更新调度、计时与 Buff、地图书写、锁钥、电路、鉴定标记、瞄准与陷阱 |

### 战斗与生物

| 主题入口 | 覆盖需求 |
|---|---|
| [[reference/components/topics/combat|装备战斗与投射物]] | 武器护甲、伤害与抗性、控制、弹道、爆炸与范围命中 |
| [[reference/components/topics/world-creatures|生物生成与巢穴]] | 巢穴住户、季节与裂隙生物生成、群体迁徙、生物状态 |
| [[reference/components/topics/companions|驯养、随从与队伍]] | 宠物召唤、群体发情、驯化骑乘、随从主人与战斗编队 |

### 制作与世界

| 主题入口 | 覆盖需求 |
|---|---|
| [[reference/components/topics/crafting|制作科技与建造]] | 制作学习、科技与图纸、放置预览、结构拆装、施工交接 |
| [[reference/components/topics/farming|植物成长与农作]] | 采收成长、授粉枯萎、土壤与压力、研究、旧农场边界 |
| [[reference/components/topics/world|工作、掉落与火情]] | 工具工作量、地皮与迁墓、掉落、燃烧传播与灭火 |
| [[reference/components/topics/world-management|实体关联与世界管理]] | 实体引用与存档、虚拟房间、世界设置、地图迁移与场景管理 |
| [[reference/components/topics/world-ships|船只与航海]] | 船体浮力、驾驶与平台、船员武器、船赛及海上设施 |

### 专项与活动

| 主题入口 | 覆盖需求 |
|---|---|
| [[reference/components/topics/activity|活动与节日对象]] | 节日装饰、赛跑、小游戏、舞台演出与活动设施 |
| [[reference/components/topics/display|显示与表现]] | HUD 与地图、颜色光照、相机、特效、音频与台词 |
| [[reference/components/topics/network-shard|网络与分片]] | 跨分片同步、世界会话、迁移与服务器管理 |

各组件仅在所属主题登记一次。当前页面均为源码审阅、未游戏运行，具体覆盖与未知以页面验证记录为准。

返回 [[reference/index|机制与接口参考]]。
