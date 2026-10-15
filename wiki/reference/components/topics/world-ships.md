---
title: 船只与航海组件索引
created: 2026-10-21
updated: 2026-10-29
type: index
tags: [overview]
---

# 船只与航海

用于船体、船具、登离船、落水及垂钓工具；海鱼自然生成见[[reference/components/topics/world-creatures#鱼群与海上生物生成|生物生成]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 船体、平台与船具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/boatphysics]] | 船体推进／转向／阻力、锚与桅杆增删、碰撞沉没与存读 | 船的推进与转向、船速与舵向 / active |
| [[reference/components/walkableplatform]] | 平台成员与玩家集合、IsFull、UID 存读、移除沉没清理 | 可行走平台、船上平台、平台摄像机 / active |
| [[reference/components/embarker]] | 平台／固定落点运动、到点和取消通知、locomotor与SG完成收尾 | 登船运动、离船落点、done_embark_movement / active |
| [[reference/components/crewmember]] | 船员登退船引用、划桨条件与施力，船组和Brain／SG负责调度 | 海盗猴船员、船员划桨 / active |
| [[reference/components/mast]] | 帆力／最大速度、收放帆、舵向写回与移除清理 | 船帆、收帆、帆力 / active |
| [[reference/components/steeringwheel]] | 驾驶者绑定与转向输入、移除兜底与监听残留 | 船舵、驾驶、转向轮 / active |
| [[reference/components/boatpatch]] | 船体补丁的生效对象、修补与消耗边界 | 补船、船体补丁、修补船 / active |
| [[reference/components/boatdrag]] | drag／max_velocity_mod／forcedampening／sailforcemodifier 与注册 | 锚阻力、船体阻力、限速因子 / active |
| [[reference/components/hullhealth]] | 碰撞到漏水与直伤、固定漏点、修复与自降解 | 船体损伤、撞船掉血、漏水 / active |


## 船体连接、受损与浮力

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/boatring]] | 整船旋转、船缘 bumper 登记与命中查询 | 整船旋转、船体转向、船缘保险杠 / active |
| [[reference/components/boatringdata]] | 客户端可读的旋转布尔与半径／分段数据 | 船旋转状态、平台半径数据 / active |
| [[reference/components/hull]] | 船唇、跳板挂接与半径，不含生命／漏水 | 船唇、跳板、hull / active |
| [[reference/components/drownable]] | 海面／虚空落水判定、冲岸传送与溺水伤害，状态归 SG | 落水判定、溺水、冲岸 / active |
| [[reference/components/flotationdevice]] | 浮具标记与落水回调，不检测也不阻止下沉 | 浮具、救生衣、flotationdevice / active |
| [[reference/components/boatmagnet]] | 磁铁配对、磁力方向与 boatphysics 合力累加 | 船磁铁、磁力拖船、boat_magnet / active |
| [[reference/components/boatmagnetbeacon]] | 磁铁信标的配对、开关与半径 | 磁力信标、boat_magnet_beacon / active |
| [[reference/components/boatleak]] | 船体漏点实体的修复状态、动态漏点与存读 | 漏水点、补船、boat_leak / active |
| [[reference/components/waterphysics]] | 水面弹性数据：restitution 决定船对船反弹推力，唯一读取者 boatphysics | 船碰撞反弹系数 / active |

## 船只航行、驾驶与平台

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/boatrotator]] | 控件绑定船、方向归一化与 SG 联动，含解绑残留 | 船转向控件、boat_rotator / active |
| [[reference/components/boattrailmover]] | 船尾水花 FX 方向漂移驱动，含死字段 track_boat_time | 船尾水花移动器 / active |
| [[reference/components/boatdrifter]] | 船移动/漂移状态机，延迟停物理与唤醒测试 | 船漂移 / active |
| [[reference/components/walkableplatformmanager]] | 平台 UID 注册表＋每帧成员更新驱动 | 平台管理器 / active |
| [[reference/components/steeringwheeluser]] | 玩家操舵：登舵钉位、方向转舵向、左右转动画，断舵自动回落 | 操舵 / active |
| [[reference/components/anchor]] | 船锚状态机：多人合力起锚/自动落锚进度、boatdrag 联动 | 船锚 / active |
| [[reference/components/expertsailor]] | 四组航海加成 getter/setter 数值容器（沃尔夫冈） | 资深水手 / active |
| [[reference/components/walkingplank]] | 船踏板占用与伸缩/登板/弃船事件控制端 | 船踏板 / active |
| [[reference/components/boattrail]] | 按移动距离在船尾生成水花 FX 交给 boattrailmover | 船尾水花 / active |
| [[reference/components/boatai]] | 原型期船 AI：模仿附近升帆船航向；全库无消费者 | 船 AI、船只跟随原型 / active |
| [[reference/components/savedrotation]] | 朝向持久化：OnSave/LoadPostPass 复原，船载物件需后处理兜底 | 朝向存档 / active |

## 船员、武器与海盗

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/boatcannon]] | 船炮弹药/瞄准/发射：complexprojectile 弹道＋船体后坐力 | 船炮 / active |
| [[reference/components/boatcannonuser]] | 玩家船炮瞄准：服务端 SetCannon 权威，客户端经 classified 渲染 | 船炮瞄准 / active |
| [[reference/components/boatcrew]] | 海盗船员编队：成员登记退队、胜负判定与 hunting/assault/retreat 状态 | 海盗船员编队 / active |
| [[reference/components/piratespawner]] | 猴女王在场才分区加权掷骰刷海盗船，管藏匿点与战利品回仓 | 海盗刷怪 / active |

## 船赛与位置检测

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/boatrace_proximitychecker]] | 船赛接近检测：开赛后 5 帧扫信标，驻留 1.5s 命中，20 天防重触发 | 船赛检查点 / active |
| [[reference/components/boatracecrew]] | 赛影船船员名册 | 船赛船员 / active |
| [[reference/components/boatrace_proximitybeacon]] | 船赛检查点信标：tag 供搜索，转发开始／结束事件 | 船赛接近信标 / active |

## 海上资源、设施与漂流物

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/flotsamgenerator]] | 世界侧残骸生成：周期、密度与存档，非船组件 | 海上残骸、漂浮物生成 / active |
| [[reference/components/winch]] | 绞盘收放线状态机，瓦片深度换算、到位回调推事件 | 绞盘 / active |
| [[reference/components/oceanwhirlportalphysics]] | 漩涡涡流：快慢节拍卷船、力衰减反解、中心回调传送/沉没 | 漩涡卷船 / active |
| [[reference/components/messagebottlemanager]] | 漂流瓶寻宝：地图边缘环布点、远海距离门槛、隐士蟹指引 | 漂流瓶寻宝 / active |
| [[reference/components/oceantrawler]] | 拖网船捕鱼：醒/睡两路捕获、饵料、溢出逃逸 | 海洋拖网 / active |
| [[reference/components/dockmanager]] | 码头瓦片网格：建造/损伤修复/BFS 断裂连锁与沉没清算 | 码头管理器 / active |
| [[reference/components/oceanicemanager]] | OCEAN_ICE 地块创建伤害破坏，浮冰弹出与落水处置 | 海冰管理 / active |
| [[reference/components/winchtarget]] | 沉没物登记打捞回调与深度，供绞车等取回物品 | 沉没物打捞目标 / active |
| [[reference/components/cookiecutterdrill]] | 饼干切割者钻船计时，完成触发船体漏水 | 钻船计时 / active |

## 垂钓与船只工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fishingrod]] | 池塘钓状态机、等待／脱杆／捕获事件、replica 字段与存档边界 | 钓竿、池塘钓鱼、鱼竿 / active |
| [[reference/components/oceanfishingrod]] | 海钓抛竿／收线／断线、tackle 与 lure、客户端预测与中断损失 | 海钓竿、抛竿、鱼线张力 / active |
| [[reference/components/oar]] | 划船入口与方向语义、RowFail 加湿与台词键、耐久由宿主扣 | 船桨组件、划桨、划船失败 / active |
| [[reference/components/fishable]] | 鱼塘鱼量、上钩实体化、放生与定时刷新 | 鱼塘鱼量、pond 钓鱼、鱼群刷新 / active |
| [[reference/components/oceanfishable]] | 海钓被钓对象的钓竿互链、体力挣扎与可捕获判定 | 海钓鱼体力、收线张力、挣扎状态 / active |
| [[reference/components/oceanfishinghook]] | 浮标诱鱼 charm／interest 与墙钟更新，不执行捕获 | 海钓浮标、诱鱼 charm、bobber hook / active |
| [[reference/components/tacklesketch]] | 旧渔具站点教学／消费路径；当前原版图纸使用teacher，区别冲撞组件 | 渔具图纸旧接口、GIVE_TACKLESKETCH / active；旧路径说明 |
| [[reference/components/fishingnet]] | 投网逻辑入口：生成表现实体，耐久与动作条件在宿主 | 渔网、投网、fishingnet / active |
| [[reference/components/fishingnetvisualizer]] | 网的表现与捕获执行，客户端不可见的表现实体 | 渔网表现、撒网动画 / active |

返回 [[reference/components/index|组件主题导航]]。
