---
title: 实体关联与世界管理组件索引
created: 2026-10-27
updated: 2026-10-29
type: index
tags: [overview]
---

# 实体关联与世界管理

用于关联保存、玩家绑定、虚拟房间和世界侧状态协议；生成策略、移动和战斗行为由对应主题承载。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 实体引用与玩家绑定

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/entitytracker]] | 名称关联、移除监听、同名重绑与 GUID 保存恢复 | 关联实体、记录建造者 / active |
| [[reference/components/objectspawner]] | 对象数组／生成接管回调与 GUID 恢复，部件定位补生清理由宿主处理 | 建筑部件保存、遗迹重置辅助 / active |
| [[reference/components/linkeditem]] | 物品按 userid 关联玩家、网络身份与回调，区别持有和装备限制 | 物品绑定玩家、离线主人、LinkToOwnerUserID / active |
| [[reference/components/linkeditemmanager]] | 世界侧关联登记、玩家重连与技能初始化分派，不持久化玩家引用 | 查询玩家绑定物品、关联物品管理 / active |

## 实体替换与恢复

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/transformer]] | 生成替代实体与恢复原对象、离屏排队和存读缺口；原版示例为注释 | 实体变形、替换Prefab、变形恢复边界 / active；接入边界说明 |

## 虚拟房间与静态布局

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/virtualroommanager]] | 世界侧虚拟房间集登记／碰撞检查／tick 调度与实体登记表，不定义房间内容 | 虚拟房间管理、房间集登记 / active |
| [[reference/components/virtualroomset]] | 单个房间集实例：房间/链接声明、切房卸载重建、投票传送序列、defs 布局数据组织 | 虚拟房间集、金库房间切换 / active |
| [[reference/components/virtualroomteleporter]] | 传送点协议字段（方向/洗牌/房间集名/临时目标）、投票者管理与回调对 | 虚拟房间传送器 / active |
| [[reference/components/worldstaticlayouts]] | 世界必放静态布局登记与读档后统一放置，失败断言、已放名单随档 | 必放布局、RegisterLayout / active |

## 暗影任务线与查理Boss状态

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/stalkermanager]] | 世界侧暗影任务线键值数据跨实例保存、唯一 stalker 登记与读档重建 | Fuelweaver 数据、暗影NPC数据 / active |
| [[reference/components/stalkerinspectable]] | 暗影NPC检视目标的名称键（GetName），供 NPC 脑去重检视；标签为注释现状 | NPC检视命名 / active |
| [[reference/components/charlie_tracker]] | 世界侧查理击败状态只读门面，主分片存档与读档重同步 | 查理是否被击败 / active |
| [[reference/components/shard_charlieinfo]] | shard_network 上净变量持有、跨分片同步与客户端成就 RPC | 查理状态跨分片 / active |
| [[reference/components/charliearenawatcher]] | 竞技场 FX 实体双端观察器：玩家进出广播 changearea、渐隐/粒子切换，判定走 Map | 查理竞技场进出检测 / active |
| [[reference/components/atriumritualorgantracker]] | 中庭仪式已掉落器官登记、激活且缺蠕虫器官时保底 worm_boss 波、shrouden 被击败清表 | 仪式器官追踪 / active |

## 活动期地表生成管理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/leafpilemanager]] | 万圣夜秋季按 DataGrid 分区限密生成叶堆、禁用期逐只衰减与冬季冻结、登记表随档；生成点限桦树林瓦 | 叶堆生成管理、万圣夜叶堆、leafpile 生成门控 / active |


## 实体标识、引用与存档

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/vanish_on_sleep]] | 休眠 10 秒后 vanishfn＋删除；消费者 piratespawner/boatcrew | 休眠消失、睡眠移除 / active |
| [[reference/components/uniqueid]] | prefab 分桶唯一编号，从世界 uniqueprefabids 计数器取号并存读；唯一宿主 beefalo | 实体唯一编号、牛编号 / active |
| [[reference/components/uniqueprefabids]] | 世界级每 prefab 自增计数器，存读稳定序号 | prefab 计数器 / active |
| [[reference/components/touchstonetracker]] | 按会话记重生石使用，classified 同步＋迁移 | 重生石记录 / active |
| [[reference/components/corpsepersistmanager]] | 世界级尸体留驻来源管理，200 具硬上限 | 尸体留驻 / active |
| [[reference/components/savedscale]] | Transform 缩放自动存读，随组件存档协议工作 | 缩放存档 / active |

## 世界设置、时间与事件

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/scenariorunner]] | 挂 scenarios 脚本按 OnCreate/OnLoad 约定执行 | 场景脚本执行器 / active |
| [[reference/components/debugger]] | 实体线段调试渲染（孤儿组件）：knownlocations 消费全为注释代码 | 调试画线 / active |
| [[reference/components/nightlightmanager]] | 世界夜灯注册表＋节点 tag 缓存，查理复活点按过滤找最近夜灯 | 夜灯管理器 / active |
| [[reference/components/chessunlocks]] | 世界棋子解锁登记，供饰品/风滚草掉落池过滤 | 棋子解锁 / active |
| [[reference/components/carnivalevent]] | 嘉年华主持人生成保活与广场注册表 | 嘉年华活动 / active |
| [[reference/components/worldsettingstimer]] | 实体命名计时器，按 maxtime 比例存档并随世界设置缩放 | 设置缩放计时 / active |
| [[reference/components/nightmareclock]] | 洞穴梦魇时钟：calm/warn/wild/dawn 四相位轮转与跨端同步 | 梦魇时钟 / active |
| [[reference/components/specialeventsetup]] | 节日事件装配器：读档对比启停事件，万圣节补饰品、猫猫年藏猫 | 节日事件装配器 / active |
| [[reference/components/worldoverseer]] | Klei 遥测统计上报，无玩法接口，仅 METRICS_ENABLED 安装 | 遥测统计 / active |
| [[reference/components/worldsettings]] | 世界设置内存表：ms_setworldsetting 写入，gamemodes 读取 | 世界设置、ms_setworldsetting / active |

## 世界路线、传送与地图迁移

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/vaultorbteleportdestination]] | 宝库球传送底座地图目的地标记，随宝石插拔挂卸 | 宝库球传送目的地 / active |
| [[reference/components/worldroutes]] | 世界实体按名缓存巡游路线点列并随存档持久化 | 世界路线缓存 / active |
| [[reference/components/undertile]] | 临时地皮底层 tile 记忆网格，供码头造地读写还原 | 地皮底层 / active |
| [[reference/components/townportalregistry]] | 传送门登记器：激活单槽配对广播 linktownportals | 传送门登记器 / active |
| [[reference/components/worldmigrator]] | 跨分片门户：绑定/三态/ms_playerdespawnandmigrate 与物品转移事务 | 迁移门户 / active |
| [[reference/components/retrofitcavemap_anr]] | 按存档一次性旗标给旧洞穴档补各世代内容，需重置时自动回滚 | 洞穴地图迁移 / active |
| [[reference/components/retrofitforestmap_anr]] | 地表老档迁移组件：消费版本迁移旗标按拓扑节点补内容 | 地表地图迁移 / active |
| [[reference/components/ropebridgemanager]] | 绳桥瓦片管理：改地皮四网格、血量/dock_damage、地震整线伤害 | 绳桥管理器 / active |
| [[reference/components/migrationpetsoverrider]] | 迁移宠物落点偏移回调，playerspawner 消费 | 宠物迁移落点 / active |

## 场景、裂隙与 Boss 管理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/wagboss_tracker]] | Wagboss 击败状态世界组件：存读持久＋分片同步 | wagboss 击败记录 / active |
| [[reference/components/area_trigger]] | 区域切入调参覆盖器：全库零消费者＋changearea 数据已无 story 字段 | 区域触发器、死代码 / active |
| [[reference/components/crabkingspawner]] | 读档兜底重建丢失的蟹王生成器（旧存档兼容） | 蟹王生成器恢复 / active |
| [[reference/components/archivemanager]] | 档案馆供电世界开关：ARCHIVES_ENERGIZED 标签＋arhivepoweron/off 事件（源码拼写如此） | 档案馆供电 / active |
| [[reference/components/shard_daywalkerspawner]] | 跨分片同步挖掘者囚禁位置（cavejail/forestjunkpile） | 挖掘者跨分片位置 / active |
| [[reference/components/shard_sinkholes]] | 跨分片同步蚁后塌陷 2 目标 warn/attack 状态 | 塌陷跨分片同步 / active |
| [[reference/components/lunarriftmutationsmanager]] | 世界级月裂突变任务进度与 Wagstaff 出场管理 | 月裂突变管理 / active |
| [[reference/components/miasmamanager]] | 洞穴雾气网格：生成/扩散/消退排程与显式创建查询 | 世界雾气 / active |
| [[reference/components/riftspawner]] | 世界裂隙生成器：难度模式、裂隙池、亲和查询与启用事件 | 裂隙 / active |
| [[reference/components/hermitcrab_relocation_manager]] | 珍珠全家当按猴岛布局旋转传送的世界级管理器 | 隐士蟹搬家 / active |
| [[reference/components/rabbitkingmanager]] | 喂胡萝卜/淘气值累积生成三种兔子王，拴绳追逐清扫 | 兔子王管理器 / active |
| [[reference/components/wagpunk_manager]] | 月裂开启后循环刷瓦斯塔夫机械与提示 NPC，布景围栏一次性生成 | 瓦冈朋克管理 / active |
| [[reference/components/wagpunk_arena_manager]] | 竞技场任务线组件：8 态状态机驱动铺地/组装/拉杆/首领战 | 瓦冈朋克竞技场 / active |

## 世界资源与环境管理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/mutatedbirdmanager]] | 空壳弃用组件：无接口无消费者，森林挂载已注释 | 弃用变异鸟管理器 / active |
| [[reference/components/worlddeciduoustreeupdater]] | 弃用空转组件：update 为空函数仅留兼容 | 弃用桦树更新器 / active |
| [[reference/components/skeletonsweeper]] | 玩家骷髅上限管理：超 100 具按生成时间从旧到新清除/腐坏 | 玩家骷髅上限 / active |
| [[reference/components/forestresourcespawner]] | 出生点锚点附近补充燧石/树枝/草/浆果的再生器 | 基础资源再生 / active |
| [[reference/components/wildfires]] | 夏季白天高温干燥按玩家掷运点燃可燃物，纯事件驱动 | 野火、夏季自燃 / active |
| [[reference/components/snowballmanager]] | 冬宴雪球分区限密生成与衰减，与 leafpilemanager 同构 | 雪球管理器 / active |

返回 [[reference/components/index|组件主题导航]]。
