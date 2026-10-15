---
title: 移动、距离与传送组件索引
created: 2026-10-27
updated: 2026-10-28
type: index
tags: [overview]
---

# 移动、距离与传送

用于移动预测、地面受力、位置和距离检测，以及通用传送协议；不按人物／生物重复登记。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 移动与本地预测

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/locomotor]] | 来源移动倍率／撤销、目的地与到达／动作；区分停步和取消 | 生物移动加速、走到目标、取消移动动作 / active |
| [[reference/components/playerspeedmult]] | 玩家自身跑速与本地预测、四组来源和可封顶加成，不叠坐骑基速 | 角色跑速、预测倍率、开花加速 / active |
| [[reference/components/strafer]] | 本地玩家侧移时的鼠标／手柄朝向采样、预测与服务端RPC接线 | 侧移瞄准、保持移动方向转身 / active |

## 地面、受力与水陆切换

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/carefulwalker]] | 不平地面目标的时效／范围减速、移动能力门槛与玩家表现，移除钩子有限制 | 谨慎行走、carefulwalking、地面减速 / active |
| [[reference/components/unevenground]] | 环境实体按检测圈通知玩家，影响半径和启停／调度由生产端与消费者分工 | 不平地面通知、蚁狮陷坑 / active |
| [[reference/components/physicsmodifiedexternally]] | 来源实体合成水平外部速度、船锚修正及玩家同步，撤销记录与重算分开 | 多来源外部速度、漩涡牵引 / active |
| [[reference/components/amphibiouscreature]] | 生物水陆状态／bank／回调、转换预判与跳跃分工，不自动提供游泳寻路 | 水陆切换、两栖生物、swimming / active |

## 位置、距离与停滞检测

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/knownlocations]] | 命名坐标快照、引用／覆盖、延迟记录与实体存读，区分家园引用 | 记录初始位置、保存巡逻点 / active |
| [[reference/components/distancetracker]] | 玩家相邻位置采样占位：只保留 previous_pos，无累计里程或统计输出 | 玩家距离采样、里程没有累计 / active |
| [[reference/components/playerprox]] | 整体／逐人／锁定的近远回调、生死过滤与离屏边界 | 玩家靠近触发、玩家离开范围 / active |
| [[reference/components/stuckdetection]] | 按查询采样的水平小位移计时、重置与剩余时间，脱困由Brain决定 | 卡住检测、移动停滞计时 / active |
| [[reference/components/platformhopdelay]] | 船平台起跳延迟 ticks，locomotor 取双侧最大值 | 平台跳跃延迟 / active |
| [[reference/components/walkingplankuser]] | 玩家踏板配对组件，onremove 监听防悬挂，Dismount 联动 plank | 踏板使用者、登板状态 / active |
| [[reference/components/blowinwind]] | 风向积分驱动 locomotor 的无脑飘行（风滚草） | 随风移动 / active |
| [[reference/components/wanderingroutefollower]] | 未完成路线巡游组件，原版无任何挂载/读取者 | 漫游路线跟随 / active |
| [[reference/components/walkableplatformplayer]] | 玩家侧平台跟随：登离船双端登记、船音乐/镜头/预测碰撞 | 玩家平台跟随 / active |
| [[reference/components/worldroutefollower]] | 实体沿 worldroutes 环线巡游：到点/卡住传送/休眠虚拟步行 | 世界路线跟随 / active |

## 传送与位置回溯

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/positionalwarp]] | 移动历史采样、回退点数、旺达倒走表与消费分工 | 旺达回溯距离、倒走表历史位置 / active |
| [[reference/components/teleportedoverride]] | 紫法杖目的实体／位置回调、优先级与取消边界 | 传送落点覆盖、紫法杖目的地 / active |
| [[reference/components/teleporter]] | 虫洞配对、接收端偏移、人物 SG／物品延迟与迁移边界 | 虫洞传送、传送门配对 / active |

打滑状态与防滑来源见[[reference/components/topics/environment#冻结滑面与附墨|环境影响]]；物品传送工具见[[reference/components/topics/item-actions#传送与魔术工具|物品交互]]，虚拟房间见[[reference/components/topics/world-management#虚拟房间与静态布局|世界管理]]。

返回 [[reference/components/index|组件主题导航]]。
