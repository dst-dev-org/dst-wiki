---
title: 装备战斗与投射物组件索引
created: 2026-10-17
updated: 2026-10-29
type: index
tags: [overview]
---

# 装备战斗与投射物

只读与当前需求相关的分组；组件页的版本、条件与验证记录决定适用范围，本文不承载机制正文。

## 装备与战斗

装备耐久按损耗场景选页：承伤护甲见 [[reference/components/armor]]，动作次数见 [[reference/components/finiteuses]]，启用／穿戴时间见 [[reference/components/fueled#燃料与使用耐久的选择]]；穿脱与效果回调见 [[reference/components/equippable]]。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/combat]] | 普通增伤、目标／受击入口与武器／护甲／生命协作 | 角色攻击倍率、受击事件、造成普通伤害 / active |
| [[reference/components/finiteuses]] | 按动作／显式调用消耗次数、耗尽回调与存档边界 | 工具耐久、有限使用次数 / active；源码审阅 |
| [[reference/components/equippable]] | 装备／入袋回调、装备加速和精神贡献、网络与清理边界 | 穿戴与卸下、装备加速 / active；源码审阅 |
| [[reference/components/tool]] | 工具动作与效率，和次数消耗独立 | 砍树效率、挖矿工具、多功能工具 / active |
| [[reference/components/weapon]] | 普通／特殊伤害、攻击回调和自动耐久消耗 | 武器伤害、攻击扣耐久、远程入口 / active |
| [[reference/components/armor]] | 普通伤害吸收、耐久修复／破损与多护甲计算 | 护甲耐久、装备减伤 / active |
| [[reference/components/planardamage]] | 独立位面伤害，武器／攻击者来源与倍率 | 位面攻击、伤害加成来源 / active |
| [[reference/components/planardefense]] | 位面数值防御与装备护甲耐久协作 | 位面防御、装备位面抗性 / active |
| [[reference/components/waterproofer]] | 物品防潮与装备防雨贡献的区别 | 物品防潮、装备防雨 / active |
| [[reference/components/insulator]] | 装备冬季保温／夏季隔热，区别于热源／恒温 | 冬季保温、夏季隔热 / active |
| [[reference/components/corruption]] | 王杖腐化值账本与 CORRUPTNIGHTMARE 目标分发（stalker／噩梦／裂缝／棋子），满值腐化态与月底清除 | 腐化值、王杖腐化 / active |

## 套装与伤害类别

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/setbonus]] | 原头身同名套装匹配、逐件启停与成对撤销 | 同套头盔护甲、套装效果 / active |
| [[reference/components/damagetypebonus]] | 按目标标签／动态回调增伤，来源键与武器／人物叠加 | 对暗影额外伤害、类别增伤 / active |
| [[reference/components/damagetyperesist]] | 按攻击者／武器标签乘剩余伤害，装备与自身消费边界 | 标签抗性、装备类别减伤 / active |
| [[reference/components/resistance]] | 装备标签门禁与整次抵挡，原骨甲冷却／燃料协作 | 骨甲白天不抵挡、挡伤回调 / active |
| [[reference/components/electricattacks]] | 带电来源、刺激选择／湿度倍率和显式撤销 | 攻击带电、电击来源 / active |
| [[reference/components/damagereflect]] | 装备／宿主反伤聚合、预计算和攻击后返还事件 | 装备反伤、onreflectdamage / active |
| [[reference/components/planarentity]] | 位面实体普通伤害压缩、特殊伤害透传与表现 | 位面实体防护、非位面攻击 / active |

## 格挡与控制

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/parryweapon]] | 武器招架角度、SG 监听窗口与重定向清理 | 盾牌格挡角度、combat_parry / active |
| [[reference/components/epicscare]] | 范围恐吓事件、目标筛选与实际接收反应 | 巨兽恐吓、范围惊吓事件 / active |
| [[reference/components/attackdodger]] | 条件或冷却闪避、伍迪形态与宿主特效／清理 | 受击闪避、鹅形态冷却 / active |
| [[reference/components/pinnable]] | 黏液受困、受击磨损、玩家 SG 与他人解救 | 唾液定身、UNPIN / active |
| [[reference/components/rooted]] | 多来源零移动倍率、临时零质量及成对撤销 | 根须束缚、多来源定身 / active |
| [[reference/components/stunnable]] | 时间窗生命变化触发事件、龙蝇倒地与清理边界 | 伤害阈值眩晕、龙蝇倒地 / active |

## 生命阈值与伤害记录

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/healthtrigger]] | 生命比例双向跨越阈值、回调顺序和监听清理 | 低血量阶段、healthtrigger / active |
| [[reference/components/damagetracker]] | 启停生命变化绝对值累计、阈值回调与清理边界 | 伤害阈值、治疗也计入 / active |
| [[reference/components/dpstracker]] | 生命净变化采样、窗口统计和更新／空窗边界 | 净生命变化速率、DPS记录 / active |

## 投射物与爆炸

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/projectile]] | 实体追踪／命中、发射武器归属与落空清理 | 吹箭速度、追踪弹、远程弹体 / active |
| [[reference/components/catcher]] | 角色接取提示／准备状态、投射物监视及回旋镖交接 | 接回旋镖、接取投射物、cancatch / active |
| [[reference/components/complexprojectile]] | 落点抛物线、高低弧线、重力与落地回调 | 抛掷落点、投掷重力 / active |
| [[reference/components/explosive]] | 整堆爆伤、点燃／工作量、过滤和燃尽销毁 | 火药爆炸、爆炸不点火 / active |
| [[reference/components/explosiveresist]] | 普通爆伤累积临时抗性、延迟衰减和百分数存读 | 熊獾连续爆炸、爆炸抗性衰减 / active |
| [[reference/components/aoediminishingreturns]] | 旋转副效果来源倍率、战斧吸血／工具热量；不改战斗伤害 | WX78旋转吸血递减、范围副效果 / active |
| [[reference/components/aura]] | 周期范围战斗判定、启停与应用事件、目标筛选及任务清理，区别精神光环 | 伤害光环、aura周期攻击、startaura / active |
| [[reference/components/deflectable]] | 非弹道目标的偏转端点（伪投射物／跟随 FX），与 projectile／complexprojectile 偏转 API 同构协议 | 可偏转目标、偏转协议目标侧 / active |

## 跳劈与突刺范围武器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/aoeweapon-base]] | 范围武器共用筛选、破坏／采收／攻击分支与抛物，实际攻击读取手持武器 | 范围武器基类、AOE命中回调 / active |
| [[reference/components/aoeweapon-leap]] | 落点圆形命中与散落物抛起、临时伤害／磨损，位移由 SG 配合 | 跳劈、落地范围攻击、DoLeap / active |
| [[reference/components/aoeweapon-lunge]] | 突刺线段及终点前方命中、抛物／轨迹，区别实际冲刺位移 | 突刺范围攻击、DoLunge / active |
| [[reference/components/tackler]] | 冲撞碰撞／工作与践踏、忽略表及边缘检测，SG负责移动和停止 | 驼鹿冲撞、CheckCollision、CheckEdge / active |
| [[reference/components/joustsource]] | 骑枪参数、线段命中与异向冲刺相撞，时间戳由调用方维护 | 骑枪冲刺来源、冲刺命中 / active |
| [[reference/components/joustuser]] | 人物冲刺资格／起止回调、三方向探边，动作与SG驱动运动 | 骑枪冲刺使用者、冲刺边缘检测 / active |

## 震地攻击

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/groundpounder]] | 逐环震地伤害／破坏与物品弹射、点模式和环模式、SG触发及任务边界 | 范围震地、GroundPound、UseRingMode / active |

## 目标锁定计时

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/targettracker]] | 装备目标锁定累计、暂停／克隆与重置回调，区别战斗目标和时间复制 | 锁定目标时间、armor_wagpunk锁定、TrackTarget / active |


## 角色技能与武器效果

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/shadowlevel]] | 装备影子等级（静态/动态 fn），驱动麦斯威尔影随从与描述 | 影子等级、暗影装备等级 / active |
| [[reference/components/singable]] | 战斗歌谣演唱件：SING 动作经灵感系统 AddSong 注册歌效果 | 战斗歌谣、可演唱 / active |
| [[reference/components/lunarplant_tentacle_weapon]] | 月晶武器触手附伤：命中 20%＋幸运掷点刷触手，默认仅虫木技能解锁 | 月晶触手武器 / active |
| [[reference/components/battleborn]] | 攻击积攒点数超阈值回血/修甲/回 san（Wathgrithr 战吼） | 战吼积攒 / active |
| [[reference/components/wx78_shield]] | WX-78 电荷护盾：health.deltamodifierfn 拦伤，过阈值免疫 | WX78 护盾 / active |
| [[reference/components/wx78_taserbuildup]] | WX-78 电击积攒：受击充能/闲置漏电，满值电爆 | WX78 电击积攒 / active |

## 战斗场景与袭击管理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/lunarsupernovablocker]] | 天体英雄超新星遮挡物标记：来源特效管理与出界清理 | 超新星遮挡物标记 / active |
| [[reference/components/wagpunk_floor_helper]] | TheWorld.net 竞技场几何 net 同步与三矩形范围/厚度查询 | wagpunk 竞技场范围 / active |
| [[reference/components/lunarsupernovaburning]] | 超新星多源 DOT：12 tick 脉冲扣血＋眩晕 | 超新星灼烧 / active |
| [[reference/components/acidbatwavemanager]] | 酸雨期按背包硝石几率预警生成蝙蝠潮与 boss | 蝙蝠潮 / active |
| [[reference/components/shadowthrallmanager]] | 接管梦魇裂缝布瘴气环刷三随从/巨口，脱战或全灭释放进冷却 | 梦魇裂缝控制 / active |
| [[reference/components/sharkboimanager]] | 深海自动铺冰竞技场，冰洞钓满 3 鱼刷 boss | 鲨鱼仔竞技场 / active |
| [[reference/components/hounded]] | 猎犬袭击等待-警告-放狗状态机，年龄/聚团定波，支持蠕虫 boss | 猎犬袭击 / active |

## 死亡战利品记录

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/deathloothandler]] | 存死亡战利品等级与尸体战利品表，读档保留 | 死亡战利品记录 / active |

返回 [[reference/components/index|组件主题导航]]。
