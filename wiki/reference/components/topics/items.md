---
title: 物品收纳与显示组件索引
created: 2026-10-17
updated: 2026-10-29
type: index
tags: [overview]
---

# 物品收纳与显示

用于持有者、堆叠、容器、打包和物品显示；食物、消耗维护与垂钓已有独立主题。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 基础收纳与显示

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/inventoryitem]] | 可拾取物品的收纳限制／图标、持有者与入栏／移出顺序；区分容器内部能力 | 可拾取物品、持有者、入栏回调 / active；源码审阅 |
| [[reference/components/heavyobstaclephysics]] | 可搬重物的物品／障碍／下落碰撞状态、避人调径，推行和落地检测另有配套 | 重物碰撞、重物落地卡人 / active |
| [[reference/components/grabbable]] | 废弃抓取组件与仍存 NET 读取；现行 inventoryitem 标签拾取覆盖的迁移边界 | 旧抓取回调、grabbableoverridetag / active；旧接口说明 |
| [[reference/components/luckitem]] | 携带／装备幸运二选一与堆叠倍率、持有者来源登记和显式刷新 | 幸运物品、马蹄铁幸运、updateownerluck / active |
| [[reference/components/inventory]] | 主栏／装备／手持／背包范围，发放、查找与材料消耗 | 人物物品栏、主栏检测、携带范围 / active |
| [[reference/components/inventoryitemholder]] | 单个活体物品／堆叠的持有、交入取回与宿主回调 | 单物品展示槽、物品持有架 / active |
| [[reference/components/itemstore]] | 物品存档记录队列、重建取出与跨分片事务分工 | 物品记录队列、漩涡出口物资 / active |
| [[reference/components/stackable]] | 件数、拆分／合并、状态继承与固定堆叠编码 | 物品堆叠、拆分一件、堆叠上限 / active |
| [[reference/components/inspectable]] | 固定／动态检查描述、角色检查语与状态分支 | 检查物品、动态描述 / active |
| [[reference/components/named]] | 实例显示名同步／存档，与固定名称和检查语区分 | 动态名称、物品改名 / active |
| [[reference/components/symbolswapdata]] | 物品build／symbol／皮肤标记数据，供搬运或展示宿主读取，不自行换符号 | 搬重物外观、展示符号数据 / active |
| [[reference/components/autojiggle]] | 装备跟随 FX 的自动摆动表现：盯 owner 奔跑／受击／冰冻／电击动画边际触发回调 | 自动摆动、跑动摆动回调 / active |


## 物品显示、外观与交互声音

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/yotb_skinunlocker]] | 蓝图记录皮肤名，使用即解锁并消耗 | YOTB 皮肤蓝图 / active |
| [[reference/components/clientpickupsoundsuppressor]] | net_bool 同步两帧置 NONE，抑制客户端拾取音效 | 拾取音效抑制 / active |
| [[reference/components/furnituredecor]] | 可摆上承物家具的装饰标记与摆放/取下回调 | 家具装饰 / active |
| [[reference/components/driedsalticon]] | 干货盐覆盖图开关，net_bool 双端同步物品栏图像 | 盐粒图标 / active |

## 特殊物品与活动道具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/soul]] | 灵魂物品标记：soul 标签＋组件双要件，souleater 按组件识别食魂 | 沃拓克斯的灵魂、食魂目标 / active |
| [[reference/components/oceanfishingtackle]] | 海钓渔具数据载体：鱼漂/鱼饵参数，single_use 收线即删 | 海钓渔具、鱼漂鱼饵数据 / active |
| [[reference/components/shadowdominance]] | 装备级暗影压制：装备时给持有者维护压制标签 | 暗影压制、骸骨帽压制 / active |
| [[reference/components/playingcard]] | 扑克 id 编码 suit*barrier＋pips，读写与花色点数分解 | 扑克牌 / active |
| [[reference/components/playerfloater]] | 游泳圈落水自动装备/回岸重置 | 游泳圈、漂浮装备 / active |
| [[reference/components/gestaltcapturable]] | 织影牢笼可捕获标记：等级/位面/瞄准者集合 | gestalt 捕获目标 / active |
| [[reference/components/mightydumbbell]] | 哑铃锻炼负载：按 mightiness 阶层选效率、扣耐久、投掷涨力 | 哑铃锻炼 / active |
| [[reference/components/klaussackloot]] | 克劳斯袋战利品抽取 | 克劳斯袋掉落 / active |
| [[reference/components/pumpkinhatcarvable]] | 南瓜帽雕刻协议：三部位×3 档刀×全脸或全空校验 | 南瓜帽雕刻 / active |
| [[reference/components/submersible]] | 物品沉水：onsink 转 underwater_salvageable 沉底，浅水自动重定位 | 沉水 / active |
| [[reference/components/slingshotmods]] | 弹弓改装隐藏容器，开关/零件迁移/存档，技能与归属校验 | 弹弓改装 / active |
| [[reference/components/possessedaxe]] | Lucy 归属链接与超时还原，woodcutter 校验与耐久回填 | 露西斧组件 / active |
| [[reference/components/balloonmaker]] | MakeBalloon 生成气球；已弃用且动作无登记入口 | 气球制造器、MAKEBALLOON / active |

## 物品湿度与温度

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/inventoryitemmoisture]] | 物品湿度／湿润标志、环境目标／外部控制、睡醒存读与堆叠 | 物品自身湿度、暂停自然吸湿 / active |
| [[reference/components/inventoryitemtemperature]] | 物品可选温度、目标偏移／绝热／容器与网络编码；区别热源 | 物品温度偏移、吸湿器温度 / active |
| [[reference/components/moistureabsorbersource]] | 吸湿物品的速率回调与持有者 user 注册，雨天效率折算 | 吸湿物品、干燥剂、desiccant / active |
| [[reference/components/nonslipgritsource]] | 防滑砂来源：持有者 user 注册与燃料扣减回调 | 防滑砂、nonslipgrit / active |

## 容器与保鲜

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/container]] | 内部格子／双端布局、过滤、转移、开关与删除内容 | 可拾取容器、容器格子、背包过滤 / active |
| [[reference/components/container-proxy]] | 双端容器入口代理、共享 master、开关会话与解绑清理 | 共享容器入口、container_proxy / active |
| [[reference/components/containerinstallableitem]] | 入栏事件驱动安装／卸载、有效容器筛选与延后卸载 | 容器安装零件、弹弓零件安装 / active |
| [[reference/components/container_transform]] | RUMMAGE 前按回调把被翻找实体换成另一个容器 | 翻找前替换容器、打开时变身容器 / active |
| [[reference/components/amorphous]] | 按容器标签与有序形态表变形，开箱延后检查、回调和名称存读 | 哈奇形态、容器物品触发变形 / active |
| [[reference/components/shelf]] | 展示槽的取物判定与回调，含命名不一致的陷阱 | 货架、展示台、shelf / active |
| [[reference/components/counter]] | 被动计数的读写接口，无事件无 replica | 计数器、计数提供者 / active |
| [[reference/components/selfstacker]] | 唤醒时自动堆叠的触发条件与标签配对 | 自动堆叠、自堆叠 / active |

保鲜倍率与恢复新鲜度材料见[[reference/components/topics/processing#食物与保鲜|食物与保鲜]]。

## 打包与拆包

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/bundlemaker]] | 包装材料的临时容器／成品配置、开始回调与消费 | 包装材料、打包袋改礼物 / active |
| [[reference/components/bundler]] | 人物两阶段打包、完成条件、取消返还与中途恢复 | 人物打包、每包至少两格 / active |
| [[reference/components/unwrappable]] | 内容记录与重建、拆包／延迟／门禁、只读预览 | 包裹内容保存、关闭包裹预览 / active |

## 食物与消耗

食物、宰杀与保鲜见[[reference/components/topics/processing#食物与保鲜|物品加工与产物]]；燃料、时间耐久与充能见[[reference/components/topics/item-maintenance#燃料耐久与充能|物品消耗与维护]]。

## 补水与浇水工具

入口已移至[[reference/components/topics/item-maintenance#补水与浇水工具|补水与浇水工具]]；组件正文地址不变。

## 扇风与降温

入口已移至[[reference/components/topics/item-maintenance#扇风与降温|扇风与降温]]；组件正文地址不变。

## 烹饪与干燥

入口见[[reference/components/topics/processing#烹饪与干燥|物品加工与产物]]，此处保留原章节锚点。

## 治疗与修复

入口已移至[[reference/components/topics/item-maintenance#治疗与修复|治疗与修复]]；组件正文地址不变。

## 怀表拆解

入口已移至[[reference/components/topics/item-maintenance#怀表拆解|怀表拆解]]；组件正文地址不变。

## 刷剃工具

入口已移至[[reference/components/topics/item-maintenance#刷剃与装饰工具|刷剃工具]]；组件正文地址不变。

## 装瓶工具

入口已移至[[reference/components/topics/item-maintenance#装瓶诱饵与诅咒物品|装瓶工具]]；组件正文地址不变。

## 蜘蛛巢装饰工具

入口已移至[[reference/components/topics/item-maintenance#刷剃与装饰工具|蜘蛛巢装饰工具]]；组件正文地址不变。

## 诱饵与诅咒物品

入口已移至[[reference/components/topics/item-maintenance#装瓶诱饵与诅咒物品|诱饵与诅咒物品]]；组件正文地址不变。

## 垂钓与船只工具

入口已移至[[reference/components/topics/world-ships#垂钓与船只工具|垂钓与船只工具]]；组件正文地址不变。

## 升级材料与电路模块

入口已移至[[reference/components/topics/item-maintenance#升级材料与电路模块|升级材料与电路模块]]；组件正文地址不变。

## 物品收纳与维护

收纳与显示留在本页；燃料、修理和维护工具见[[reference/components/topics/item-maintenance|物品消耗与维护]]，保留原总标题锚点。

返回 [[reference/components/index|组件主题导航]]。
