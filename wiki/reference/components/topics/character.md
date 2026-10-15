---
title: 人物状态与外观组件索引
created: 2026-10-17
updated: 2026-10-29
type: index
tags: [overview]
---

# 人物状态与外观

用于人物三维、睡眠、图鉴资料、外观换装及人物侧能力宿主；环境和移动分组已拆出。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 人物三维与进食

### 生命与寿命

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/health]] | 生命上限／人物65535同步边界、惩罚、回血与来源再生 | 人物最大生命值、持续回血、装备回血 / active |
| [[reference/components/oldager]] | 旺达剩余寿命、自然老化、受伤转换、回春与必要同步 | 旺达生命值流逝、衰老、回春表 / active |
| [[reference/components/healthsyncer]] | 生命百分比／趋势双端同步、状态编码与消费，区别生命数据修改 | 生命百分比同步、船血条同步 / active |

### 饥饿与精神

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/hunger]] | 饥饿补充／上限、耗速倍率、暂停与归零伤害 | 降低饥饿消耗、饥饿耗速 / active |
| [[reference/components/sanity]] | 精神上限／即时恢复、持续贡献与疯狂／启蒙 | 每分钟回精神、精神贡献 / active |
| [[reference/components/spooked]] | 人物惊吓累计／衰减、年龄与幻象、spooked事件及SG／精神分开响应 | 砍树惊吓、树木幻象、自定义惊吓 / active |
| [[reference/components/staffsanity]] | 施法者精神变化倍率、实际调用覆盖与人物配置 | 法杖精神消耗、施法精神倍率 / active |

### 进食偏好与记忆

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/eater]] | 食用者饮食／吸收、成功进食与消耗顺序 | 挑食、角色饮食、进食回调 / active |
| [[reference/components/foodaffinity]] | 食用者偏好最大饥饿倍率、调味食品与负生命／精神基础值豁免 | 喜爱食物、偏好饥饿倍率 / active |
| [[reference/components/foodmemory]] | 重复食物计数／到期遗忘、食用收益倍率与存读 | 沃利食物记忆、重复进食惩罚 / active |


## 人物变身与特殊状态

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/strongman]] | Wolfgang 健身会话状态＋ingym 标记，联动 mightiness | 健身会话、ingym 标记 / active |
| [[reference/components/beaverness]] | 伍迪海狸值遗留组件，全库无消费者，已被 wereness 取代 | 海狸值（旧） / active |
| [[reference/components/wereeater]] | Woodie 吃怪肉计数变身：≥2 块或 wereitem 触发强制变身回调 | 怪物食物变身计数 / active |
| [[reference/components/wereness]] | Woodie 变身值计量：负速率消耗经 player_classified 同步并推 werenessdelta | 变身值 / active |
| [[reference/components/age]] | 实体存活时长：跨档秒龄换算天数，玩家周期上报成就/统计 | 实体年龄 / active |
| [[reference/components/avengingghost]] | Wendy 技能驱动的变鬼复仇 15 秒强化：伤害/特效/光环 | 复仇幽灵 / active |
| [[reference/components/grue]] | 黑暗惩罚计时循环，免疫来源表/抗性/darkness 攻击与扣理智 | 查理、黑暗惩罚 / active |
| [[reference/components/acidlevel]] | 酸雨侵蚀值：酸雨周期伤害装备/食物/本体，普通雨反向消退 | 酸雨侵蚀 / active |
| [[reference/components/mightiness]] | Wolfgang 三态体格值，切换联动伤害/保温/工作/划船 | 沃尔夫冈体格 / active |
| [[reference/components/bloomness]] | Wormwood 绽放等级状态机，施肥延长满级/加速 | 绽放度 / active |
| [[reference/components/ghostgestalter]] | 月晷 MUTATE 目标端，DoMutate 委托 domutatefn | 幽灵变形器 / active |

## 人物技能、控制与角色资料

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/socket_shadow_heart]] | WX-78 影心脏插座视觉件：备用机身挂心静脉特效实体 | 影子心脏插座、WX78 心静脉特效 / active |
| [[reference/components/playermetrics]] | 监听 unlockrecipe 上报角色原型化遥测事件 | 玩家指标、角色原型化上报 / active |
| [[reference/components/playerinspectable]] | 装备皮肤/技能选择写引擎网络变量，供头像弹窗远端读取 | 头像弹窗数据 / active |
| [[reference/components/socket_shadow_mimicry]] | WX78 影子插槽拟真开关，刷新 itemmimic 暂停源 | 影子插槽拟真 / active |
| [[reference/components/dumbbelllifter]] | 沃尔夫冈举哑铃会话：驱动 mightydumbbell 训练 | 哑铃举起 / active |
| [[reference/components/experiencecollector]] | 按日为有技能树角色累积 1 点技能经验 | 技能经验累积 / active |
| [[reference/components/playeravatardata]] | 玩家头像数据 net 化，尸体/骨架/稻草人复用 GetData | 玩家头像数据 / active |
| [[reference/components/skilltreeupdater]] | 技能树端组件：查询/激活/XP 抽象层、主机-客户端 RPC 分派 | 技能点接口 / active |
| [[reference/components/playercontroller]] | 玩家输入枢纽：缓存执行左右键与动作按钮动作，RPC 主机重推导校验（精选覆盖） | 玩家控制器 / active |
| [[reference/components/inspectaclesparticipant]] | 温妮观测镜谜题箱布置与 netvar 同步、发奖冷却 | 观测镜小游戏 / active |
| [[reference/components/seamlessplayerswapper]] | 不退场换角色：TheNet 生成替换玩家并迁移数据隐藏旧体 | 无缝换角色 / active |
| [[reference/components/itemaffinity]] | 携带指定物品按优先级取一条理智修正 | 物品亲和 / active |

## 歌唱、发言与复活

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/singingshelltrigger]] | 玩家逐帧扫海螺，三态表防重复触发 _activatefn | 海螺触发器 / active |
| [[reference/components/revivablecorpse]] | 可复活 corpse 标记＋资格判定＋复活事件 | 可复活尸体 / active |
| [[reference/components/singingshellmanager]] | 世界动态海螺管理器，开关全体玩家扫描并自移除 | 海螺管理器 / active |
| [[reference/components/singinginspiration]] | Wathgrithr 灵感值：伤害/骑乘积累、战歌槽位与施加 | 战歌灵感 / active |
| [[reference/components/wisecracker]] | 玩家状态事件自动播报 ANNOUNCE 台词（含节流） | 自动播报 / active |
| [[reference/components/storyteller]] | 对故事道具播剧本：talker 台词＋tick 回调，字符串返回即拒绝 | 讲故事、营火故事 / active |

## 温度与环境

温湿度、防雨、冻结与滑面见[[reference/components/topics/environment|环境影响与防护]]；地面减速、跑速和侧移见[[reference/components/topics/movement|移动、距离与传送]]。原组的淡出／拉伸／符号覆盖已归[[reference/components/topics/character#外观与换装|外观与换装]]。

## 睡眠与困倦

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/grogginess]] | 困倦累积／减速、抗性与免疫来源、昏睡 SG 及人物重建 | 困倦、昏睡、催眠抗性 / active |
| [[reference/components/sleeper]] | 生物睡／醒测试、催眠门槛、Brain／SG、离屏与保存清理 | 生物睡眠、夜行生物、睡眠抗性 / active |
| [[reference/components/sleepingbag]] | 床具占用与回调、每 tick 参数、睡袋／帐篷动作及唤醒 | 睡袋、帐篷回血、午睡凉棚 / active |
| [[reference/components/sleepingbaguser]] | 人物睡眠收益倍率、可睡回调消费差异、tick 与退出 | 睡眠回血倍率、睡眠饥饿消耗、睡眠门禁 / active |

## 图鉴与研究记录

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/plantregistryupdater]] | 人物研究登记、服务端去重／客户端档案、RPC 与奖励边界 | 植物图鉴登记、新研究奖励精神、研究去重与重登 / active |
| [[reference/components/cookbookupdater]] | 玩家烹饪指南配方／食物属性记录、本地持久化与服务端增量RPC，区别注册食谱 | 料理图鉴记录、TheCookbook、LearnFoodStats / active |
| [[reference/components/dataanalyzer]] | WX扫描数据余额、惰性初始化／扣取／再生与存档，区别扫描执行和配方注册 | WX78扫描数据、scan_id余额、SpendData / active |
| [[reference/components/scrapbookable]] | TEACH到图鉴教学回调的薄协议，具体学习对象和物品消耗由宿主实现 | 图鉴纸页、图鉴笔记、Teach / active |
| [[reference/components/pointofinterest]] | 按图鉴解锁等级控制本地地标提示、世界表现和HUD指示器 | 未调查地标、图鉴POI提示 / active |

## 毛发生长与剃毛

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/beard]] | 天数阶段回调、分级剃毛、保温／库存与宿主长毛协作 | 威尔逊胡须保温翻倍、胡须分阶段生长 / active |

## 外观与换装

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/skinner]] | 角色皮肤模式／服装数据与重绘、复制及存读，区分显示与所有权校验 | 角色换肤、服装槽、SetSkinMode / active |
| [[reference/components/wardrobe]] | 衣柜换装会话、弹窗提交、距离结束与皮肤应用，支持目标装扮分支 | 衣柜换装、多人衣柜、换装会话 / active |
| [[reference/components/giftreceiver]] | 玩家礼物计数、开礼物机器引用与弹窗／状态图协作，不负责授权发礼物 | 私人皮肤礼物、开礼物、giftmachine / active |
| [[reference/components/distancefade]] | 按摄像机方向投影计算本地透明度、附加倍率与墙钟更新，非玩家距离检测 | 树冠淡出、光束透明度 / active |
| [[reference/components/stretcher]] | 朝目标定向并按三维距离拉伸动画，停更不自动恢复缩放 | 影手手臂拉伸、连接目标的表现 / active |
| [[reference/components/custombuildmanager]] | 按符号组切换build覆盖并保存选择，清组不撤旧覆盖；未见原版接线 | 分组替换符号、build组选择 / active；接入边界说明 |

## 诅咒与强制收纳

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/cursable]] | 猴子诅咒计数、强制入栏／挤物、解除与死亡转移 | 服务端净化一枚猴子代币、玩家猴子诅咒 / active |

## 调谐与远程关联

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/attunable]] | 目标调谐成本／链接／替换、离线保存与重连，复活效果归宿主 | 复活雕像调谐、关联目标、LinkToPlayer / active |
| [[reference/components/attuner]] | 玩家调谐代理集合、双端查询与角色迁移，区别远处实体引用 | 查询远程复活关联、gotnewattunement / active |

## 镶嵌与电路模块宿主

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/upgrademoduleowner]] | 多栏模块、实际／有效电量、按序激活、返还／成本与嵌套存读 | 服务端给WX78补一格电、WX78模块宿主 / active |
| [[reference/components/socketable]] | 镶嵌物名称／品质与双端资格，实际插入及效果归宿主 | 镶嵌材料、socketname、SOCKETQUALITY / active |
| [[reference/components/socketholder]] | 双端插槽创建、准入／序列化收纳／还原、回调及玩家读档协作 | 插槽宿主、镶嵌与卸载、TryToSocket / active |

## 幸运与概率来源

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/luckuser]] | 多来源幸运值相加／覆盖与撤销、负值猎犬修正，实际概率由消费者计算 | 人物幸运值、SetLuckSource、负幸运 / active |

## 人物状态与环境

人物状态留在本页；环境类入口已分至[[reference/components/topics/environment|环境影响与防护]]，保留原总标题锚点。

返回 [[reference/components/index|组件主题导航]]。
