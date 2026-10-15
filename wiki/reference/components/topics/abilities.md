---
title: 交互技能与持续效果组件索引
created: 2026-10-17
updated: 2026-10-28
type: index
tags: [overview]
---

# 交互技能与持续效果

只读与当前需求相关的分组；组件页的版本、条件与验证记录决定适用范围，本文不承载机制正文。

## 更新调度

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/updatelooper]] | 普通／长／墙钟／后置回调，首次调用与延迟删除、组件移除边界 | 逐帧更新、AddOnUpdateFn、回调清理 / active |

## 计时与持续效果

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/timer]] | 具名秒数倒计时／刷新／暂停、完成事件与保存推进；区分取消 | 定时关闭箱子、刷新计时器、具名倒计时 / active |
| [[reference/components/debuff]] | 效果实体回调、限时来源施加／刷新／撤销与保存重施 | 自定义Buff、三十秒加速、Buff撤销 / active |
| [[reference/components/debuffable]] | 目标具名效果集合、施加门禁／返回、同键及保存／清理边界 | 吃食物获得Buff、吃胡萝卜加速、Buff管理 / active |
| [[reference/components/coach]] | 沃尔夫冈鼓舞排程、附近玩家回精神与随从增伤，哨子门禁及停止边界 | 教练鼓舞、coaching、随从鼓舞Buff / active |

## 物品交互与技能

交易、物品使用、持续施法、阅读、传送与魔术工具见 [[reference/components/topics/item-actions|物品交互与施法]]；此处保留原章节入口，具体页只在该主题登记。

## 纸张擦除

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/papereraser]] | 设备侧擦纸标记与委派，区分纸张产物和操作成功 | 制图桌擦纸、ERASE_PAPER / active |
| [[reference/components/erasablepaper]] | 纸张侧消耗一件与返还产物，区分生成失败和批量交付 | 图纸回收、擦除后纸张数量 / active |

## 涂蜡与保形

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/wax]] | 普通蜡／喷雾来源标记及动作匹配，区分材料消费和目标效果 | 蜂蜡、封蜡喷雾、waxspray / active |
| [[reference/components/waxable]] | 目标涂蜡委派、喷雾门禁与成功后的消耗，效果由宿主定义 | 巨型作物封蜡、雪人保形、Wax / active |

## 绘图与图像

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/drawingtool]] | 寻找附近物品图像、图集／背景选择、绘制委派与工具消费 | 羽毛笔、绘制物品图标、DRAW / active |
| [[reference/components/drawable]] | 图像状态／变化回调及存读、绘制开关与宿主显示分工 | 小木牌重绘、相框图像、OnDrawn / active |

## 地图记录与投递

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/maprecorder]] | 记录／学习玩家地图、世界会话核验、回调与数据转存 | 地图卷轴、共享探索地图、TeachMap / active |
| [[reference/components/mapdeliverable]] | 地图选点会话、投递计时与宿主进度回调，取消及恢复分工 | 无人机投递、地图选点、SendToPoint / active |
| [[reference/components/treasuremarked]] | 沉藏宝物的标记子实体、地图登记与开关存读，重复开启不去重 | 漂流瓶宝藏标记、海底宝物图标 / active |

## 钥匙与目标锁

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/key]] | 旧式钥匙类型标签、使用／取回通知与参数缺口 | 探测杖插入底座后通知、旧式钥匙类型 / active |
| [[reference/components/lock]] | 类型／卡住、锁状态、钥匙暂存归还及 GUID 恢复 | 服务端安全解锁探测杖底座、锁内保留钥匙 / active |
| [[reference/components/klaussackkey]] | 鹿角真假、世界恢复通知、门的宿主扩展键 | 普通鹿角解锁暴食公园门、克劳斯真钥匙恢复 / active |
| [[reference/components/klaussacklock]] | 目标回调、成功／原因／单件消费与原宿主分工 | 赃物袋服务端玩家检查、错误钥匙也消耗 / active |

## 机器与节点连接

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/machine]] | 开关状态／回调、候选门禁、交互冷却与加载重放 | 灭火器开关冷却改为一秒、机器开关 / active |
| [[reference/components/circuitnode]] | 节点启用／标签发现、双向集合、宿主供电与重连 | 服务端断开聚光灯电路、薇诺娜电路节点 / active |
| [[reference/components/powerload]] | 设备负载和待机标志、供电宿主消费与显式刷新，区别电量或持续供电 | 设备耗电负载、薇诺娜待机、SetLoad / active |
| [[reference/components/battery]] | 电源资格／部分充电／消费回调，区别电路节点持续供电 | 可交互电源、电池消费协议 / active |
| [[reference/components/batteryuser]] | 充电请求倍率、结果／消费顺序与物品／人物动作分派 | 晨星补能、WX78充电、用电者 / active |
| [[reference/components/electricconnector]] | 电围栏配对／共享场、搜索与直接连接差异、GUID 重建 | 电围栏找连接范围限制为四、电围栏连接 / active |

## 称重与重量

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/itemweigher]] | 称重工具类型／回调、动作方向和玩家播报，消费由宿主负责 | 便携秤、不耗耐久称重、WEIGH_ITEM / active |
| [[reference/components/weighable]] | 重量／范围比例、捕获者记录与存读，海鱼形态转存 | 鱼重量、重量比例、CopyWeighable / active |

## 检查、评选与鉴定

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/appraisable]] | 玩偶鉴定资格与委托回调、APPRAISE 和原版评委消费链 | 牛玩偶鉴定、评选物品 / active |
| [[reference/components/closeinspector]] | 装备近距离检查目标／地点回调、LOOKAT 门禁与动态技能挂载 | 玫瑰色眼镜、近距离检查 / active |
| [[reference/components/roseinspectable]] | 玫瑰检查目标资格、残影创建／激活回调与冷却决定，效果由宿主实现 | 玫瑰检查目标、残影目标协议 / active |
| [[reference/components/roseinspectableuser]] | 使用者残影生命周期、实体／地点分派与冷却，接眼镜和查理残影 | 薇诺娜玫瑰眼镜、残影激活、玫瑰检查冷却 / active |

## 标记与代理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/markable]] | 按执行者新增／取消、编号池、资格与原彩带／赛程清理 | 牛年评选柱最多四个标记、牛年投票标记 / active |
| [[reference/components/markable-proxy]] | 牛侧转交目标、候选同步、绑定／关闭顺序与执行复查 | 牛侧标记执行复查代理状态、点击牛给评选柱投票 / active |
| [[reference/components/replayproxy]] | 代理实体的原实体GUID／Prefab名称字段与格式化显示，无内置回放执行 | entityproxy、原实体标识 / active |

## 定向技能与瞄准

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/aoetargeting]] | 地点技能选点配置、双端挂载与启用、地图及骑乘边界 | 地点技能瞄准、选点范围 / active |
| [[reference/components/reticule]] | 本地鼠标／手柄指示器、颜色、位置函数与监听清理 | 瞄准圈、手柄选点、瞄准颜色 / active |
| [[reference/components/chargingreticule]] | 本地蓄力方向跟随、手柄旋转插值、输入／相机监听与清理 | 手柄蓄力方向转向更快、蓄力方向指示器 / active |
| [[reference/components/aoespell]] | 地点技能资格与施放分离、回调返回、服务端距离校验 | 地点施法、定向技能、施法失败 / active |
| [[reference/components/aoecharging]] | 蓄力帧计数、网络状态、释放回调与状态机协作 | 蓄力攻击、蓄力瞄准、释放攻击 / active |
| [[reference/components/cooldown]] | 单实体服务端充能状态、回调、存档恢复与首充区别 | 骨甲冷却、单实体冷却 / active |
| [[reference/components/spellbookcooldowns]] | 人物按技能键同步多冷却、剩余比例与主动消费检查 | 多技能冷却、共享技能CD / active |

## 触发与捕捉陷阱

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/mine]] | 搜索／阵营、触发回调、重置／停用与睡醒调度 | 犬牙陷阱触发范围改为二、地雷式触发 / active |
| [[reference/components/trap]] | SG 捕捉、产物重建、收获／饥饿、诱饵与保存 | 普通捕捉陷阱搜索范围改为二、陷阱收获 / active |

## 座椅与落座

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/sittable]] | 座椅占位／标签与双方事件，动作及 SG 负责落座和退出 | 可坐家具、驱离坐者、SITON / active |

## 阶段升级

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/upgradeable]] | 目标门禁／累计、单件消费、阶段回调与宿主成长恢复 | 蜘蛛巢每阶段三点升级值、升级累计与阶段 / active |

## 作祟与实体发言

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/hauntable]] | 作祟helper选型、动作资格／回调组合、冷却与恐慌清理 | 幽灵作祟、作祟冷却 / active |
| [[reference/components/talker]] | 玩家／NPC台词、Say／Chatter选型、双端消息与取消 | 头顶文字、角色说话、NPC台词 / active |
| [[reference/components/souleater]] | 食魂者：EatSoul 拆堆/发事件/回调/删灵魂，原版唯一宿主 wortox | 食魂者 / active |
| [[reference/components/pocketwatch]] | 旺达怀表施法基座：inactive 联动标签，DoCastSpell/CanCastFn 由 prefab 注入 | 怀表施法组件 / active |
| [[reference/components/winonateleportpadmanager]] | TheWorld 传送垫注册表，供温妮技能挑选落点 | 传送垫注册表 / active |
| [[reference/components/wx78_abilitycooldowns]] | WX78 技能冷却实体表，哈希键控双端查询 | WX78 技能冷却 / active |

返回 [[reference/components/index|组件主题导航]]。
