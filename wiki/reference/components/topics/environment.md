---
title: 环境影响与防护组件索引
created: 2026-10-27
updated: 2026-10-29
type: index
tags: [overview]
---

# 环境影响与防护

用于温湿度、天气防护、冻结滑面与附近精神影响；物品自身湿度／温度见[[reference/components/topics/items#物品湿度与温度|收纳索引]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 温度与热源

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/temperature]] | 自身温度、锁定／读档、自然换热与伤害；区分人物／物品／热源 | 物品恒温、暖石温度、人物体温 / active |
| [[reference/components/heater]] | 附近／携带／装备热源输出、方向与实际收纳范围 | 随身热源、携带供热、热源输出 / active |
| [[reference/components/temperatureoverrider]] | 范围内局部环境温度覆盖、最近来源与双端启停 | 局部环境温度、冰眼塔温度覆盖 / active |
| [[reference/components/frostybreather]] | 寒冷吐息本地粒子与网络事件、温度判定／强制模式和动画触发，仅负责表现 | 寒冷呼气、白色吐息、ForceBreathOn / active |

## 潮湿与免湿

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/moisture]] | 角色湿度改值／持续速率、防雨与强制干燥；来源撤销、同步与保存 | 角色湿度、持续干燥、防雨 / active |
| [[reference/components/moistureimmunity]] | 目标多来源免湿、原版食物 Buff、生效／撤销与湿润查询边界 | 免湿、防水Buff、保持干燥 / active |
| [[reference/components/moistureabsorberuser]] | 持有者侧吸湿集合：取最高速率物品并应用干燥 | 持有者吸湿、干燥来源集合 / active |

## 遮雨庇护与避雷

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/sheltered]] | 逐帧庇护判定、0.2 秒副本与事件时序、骑乘门槛、雨天防水贡献 | 庇护、树荫庇护、避雨判定 / active |
| [[reference/components/rainimmunity]] | 目标来源集合式免雨、自动移除、只挡雨水路径与客户端标签边界 | 免雨、雨水免疫、雨淋免疫 / active |
| [[reference/components/lightningblocker]] | 平方半径避雷登记、weather 改 thunder、回调与分支顺序 | 避雷、落雷阻挡、避雷柱 / active |
| [[reference/components/raindome]] | 雨棚的周期授予免雨、半径登记与关停清理 | 雨棚、免雨范围、voidcloth_umbrella / active |
| [[reference/components/raindomewatcher]] | 玩家雨棚传感器：enter／under／exit 事件，不授予免疫 | 雨棚监听、是否在雨棚下 / active |

## 冻结、滑面与附墨

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/freezable]] | 寒冷积累／冻结抗性、两段解冻、受击打碎与 SG 协作 | 冰杖冻结、冻结抗性、解冻 / active |
| [[reference/components/nonslipgrituser]] | 持有者侧防滑来源集合，供电滑状态消费 | 防滑来源、nonslipgrituser / active |
| [[reference/components/nonslipgritpool]] | 地面防滑区域的位置判定、滑脚消费者与宿主过期清理，区别随身砂来源 | 地面防滑砂池、防滑区域、IsGritAtPosition / active |
| [[reference/components/slipperyfeet]] | 角色滑度累计／衰减、冰面与具名来源、事件及 SG 跌倒协作 | 冰面打滑、滑脚状态、滑度倍率 / active |
| [[reference/components/slipperyfeettarget]] | 滑面位置检测与倍率、物理半径和自定义形状，供角色附近搜索消费 | 企鹅冰面、池塘滑面、滑面范围 / active |
| [[reference/components/inkable]] | 人物附墨倒计时、网络脉冲与效果，区分转移和屏幕遮罩 | 鱿鱼墨水、附墨状态、deinked / active |


## 天气、风暴与自然灾害

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/oasis]] | 绿洲范围标记：半径内判定＋0~1 接近度供沙暴消减 | 绿洲范围、绿洲接近度 / active |
| [[reference/components/playerlightningtarget]] | 玩家雷击目标：命中率掷骰与受击回调 | 玩家雷击目标 / active |
| [[reference/components/sandstormwatcher]] | 玩家沙暴监听：等级＋护目/骑乘定移速惩罚 | 沙尘暴监听 / active |
| [[reference/components/moonstormwatcher]] | 玩家月暴响应：按等级与护目/骑乘控制月暴减速修正 | 月暴减速 / active |
| [[reference/components/stormwatcher]] | 聚合沙尘/月风暴等级 7 档量化同步客户端并联动减速 | 风暴观测 / active |
| [[reference/components/sandstorms]] | 沙尘暴开关与拓扑/绿洲强度量化 | 沙尘暴管理 / active |
| [[reference/components/moonstorms]] | 月暴节点登记与同步 | 月暴管理 / active |
| [[reference/components/caveins]] | 洞穴落石执行器：主分片蚁狮目标经 shard_sinkholes 中继 | 洞穴落石 / active |
| [[reference/components/lunarhailbuildup]] | 月雹堆积：雹天累积、堆满可敲除掉月亮玻璃 | 月雹堆积 / active |
| [[reference/components/meteorshower]] | 单体 meteorspawner 三级流星雨状态机：冷却轮询、离屏降频 | 流星雨 / active |
| [[reference/components/lunarhailmanager]] | 月雹碎块按玩家调度坠落与落地伤害；已弃用无安装点 | 月雹（弃用） / active |
| [[reference/components/caveweather]] | 洞穴双端湿度-降水循环（雨/酸雨），netvar 周期同步推 weathertick | 洞穴天气 / active |
| [[reference/components/quaker]] | 洞穴地震三态机，逐玩家投碎片并管落点砸伤/碎裂 | 地震、落石 / active |
| [[reference/components/moonstormmanager]] | 锁月相开风暴区，瓦斯塔夫猎寻/无 NPC 实验双变体 | 月风暴管理 / active |
| [[reference/components/weather]] | 地表双端天气：雨/雪/月雹循环、闪电分流、花粉与雪地 overlay | 天气 / active |

## 环境温度与灼烧

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/lunarfissureburning]] | 月裂灼烧动态组件：持续扣血(坐骑同扣)、眩晕、离裂 1 秒自毁 | 月裂灼烧 / active |
| [[reference/components/snowballmelting]] | 雪球 solid/melting 状态机，按温度与雪覆盖周期复查 | 雪球融化 / active |
| [[reference/components/trapfumaroleburning]] | 熔岩陷阱灼烧 DOT：火伤、点燃累积、烤熟/引燃 | 地热灼烧 / active |
| [[reference/components/fumarolelocaltemperature]] | 洞穴 net 实体喷气孔局部温度层：9x9 瓦片密度插值 | 地热温度 / active |
| [[reference/components/worldtemperature]] | 世界气温源：季节＋相位＋Perlin 公式、temperaturetick、洞穴倍率 | 世界温度 / active |

## 区域感知、瘴气与环境状态

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/miasmawatcher]] | 瘴气进出检测＋classified/debuff＋视觉骑乘豁免减速 | 瘴气检测 / active |
| [[reference/components/areaaware]] | 拓扑区域感知：current_area_data＋changearea 广播＋地块进出事件 | 区域感知、changearea / active |
| [[reference/components/petrifiable]] | 可石化注册＋链式波及 8 距离，支持唤醒倒计时 | 可石化 / active |
| [[reference/components/acidinfusible]] | 酸雨自动灌注生物：统一乘区加成/移速＋烟雾特效 | 酸液灌注 / active |
| [[reference/components/hallucinations]] | 低理智幻觉生成器（窥视眼/暗影窥伺者/惊走者） | 幻觉 / active |
| [[reference/components/diseaseable]] | 疾病传播组件（延迟→预警→发病→传染）；无原版 AddComponent 消费者 | 疾病、病害传播 / active |

## 环境声音与视觉表现

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/canopylightrays]] | 4 格网格随机铺树冠光柱，全局引用计数共享 | 树冠光柱 / active |
| [[reference/components/maxlightspawner]] | 圆周生成 maxwelllight 并登记所有权；全库无 AddComponent 消费者 | 麦克斯韦光球生成器 / active |
| [[reference/components/lunarhailbirdsoundmanager]] | 月雹鸟声效 0-3 档，net_tinybyte 世界网络同步 | 月雹鸟音效管理 / active |
| [[reference/components/canopyshadows]] | 树冠叶子遮阴贴图，全局引用计数共享 | 树冠遮阴 / active |
| [[reference/components/sharklistener]] | 世界组件按鲨鱼距离写 _sharksoundparam 驱动音效；无组件读取者 | 鲨鱼监听 / active |
| [[reference/components/wavemanager]] | 客户端水面波纹装饰：按地皮撒浪花/波光，专用服务器不安装 | 水面波纹 / active |
| [[reference/components/ambientsound]] | 11x11 地块统计混环境音，季节雨夜调参＋理智双轨 | 环境音 / active |
| [[reference/components/shatterfx]] | 冻结破碎特效分级动画，shatter 经 net 变量同步 | 碎冰特效层级 / active |

## 环境对象与空间设施

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/scaler]] | 等比缩放统一入口：SetScale 即时生效＋OnApplyScale 钩子＋存读 | 缩放组件 / active |
| [[reference/components/moonaltarlinktarget]] | 三座月坛集齐组三角，校验后生成月坛连线实体 | 月坛连线端点 / active |
| [[reference/components/lightpostpartner]] | 灯杆锁扣槽位经 net_entity 同步，传送/燃烧断链 | 灯杆挂点 / active |
| [[reference/components/abysspillargroup]] | 深渊柱阵生成/坍塌/复活编排与存读 | 深渊石柱组 / active |

## 精神光环

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/sanityaura]] | 附近精神速率、范围／衰减及命名去重，区别持有者效果 | 精神光环、附近回精神、光环不叠加 / active |
| [[reference/components/sanityauraadjuster]] | 周期委托调整附近玩家的光环倍率，集合返回与效果撤销 | 温蒂光环增益、精神光环倍率调整 / active |

不平地面减速与移动预测见[[reference/components/topics/movement#地面受力与水陆切换|移动索引]]；人物自身三维见[[reference/components/topics/character#人物三维与进食|人物状态]]。

返回 [[reference/components/index|组件主题导航]]。
