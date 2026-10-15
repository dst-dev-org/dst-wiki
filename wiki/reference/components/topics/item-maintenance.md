---
title: 物品消耗与维护组件索引
created: 2026-10-27
updated: 2026-10-28
type: index
tags: [overview]
---

# 物品消耗与维护

用于使用资源、供料、修补及专用维护工具；容器与持有者见[[reference/components/topics/items|物品收纳与显示]]，食物新鲜度见[[reference/components/topics/processing#食物与保鲜|加工索引]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 燃料、耐久与充能

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fueled]] | 启用时间消耗、补充与耗尽，区别按次数／冷却 | 燃料消耗、装备时间耐久 / active |
| [[reference/components/decay]] | 旧式量值增减与燃料事件、线程计时及未定义变量疑点，区别当前燃料组件 | 旧Decay计时、SetTimeDelta、spentfuel / active；源码边界说明 |
| [[reference/components/fuel]] | 作为燃料来源的物品类型／量与取用回调 | 燃料物品、木头燃料 / active |
| [[reference/components/fueler]] | 供料实体的类型／数值、无物品添料与消费边界 | 生物添火、供料者、乌鸦小孩添火 / active |
| [[reference/components/fuelmaster]] | 添料者的补充量倍率、按目标回调、零倍率消费与恢复 | 薇洛添火倍率、人物燃料效率 / active |
| [[reference/components/rechargeable]] | 容量与恢复秒数、成功用后充能及技能门禁 | 技能冷却、自动充能 / active |

## 补水与浇水工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/wateryprotection]] | 洒水／灭火／降温、实体半径与单格土壤、收尾消费 | 洒水壶浇水量翻倍、范围灭火 / active |
| [[reference/components/fillable]] | 接收方补水／换物、海水门禁、先消费与回调返回 | 洒水壶装海水、容器补水 / active |
| [[reference/components/watersource]] | 水源可用标签、物品消费、份量与接收方分工 | 一块冰补两次水、水源消耗 / active |

## 扇风与降温

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fan]] | 单次／持续扇风回调、原版降温灭火与动作耗次，额外干燥需另配 | 羽毛扇、扇风降温、持续扇风 / active |

## 治疗与修复

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/healer]] | 治疗物品的基础回血、执行者倍率、门禁／回调及消费 | 治疗物品、药膏回血、治疗失败 / active |
| [[reference/components/maxhealer]] | 减少生命上限惩罚、比例符号与强心针消费边界 | 强心针、上限惩罚恢复 / active |
| [[reference/components/repairable]] | 目标材料匹配、四类恢复顺序、标签初始化与回调 | 材料修复装备、石头修斧头、修复目标 / active |
| [[reference/components/repairer]] | 材料类型／数值／比例、动作标签与影响范围 | 修复材料、噩梦燃料修护符 / active |
| [[reference/components/forgerepair]] | 修理包材料标签、按优先级补满一类耐久、消耗与材料回调 | lunarplant_kit、voidcloth_kit、wagpunkbits_kit / active |
| [[reference/components/forgerepairable]] | 装备侧材料匹配／可修标签与REPAIR分派，耐久刷新和修复回调 | 锻造装备可修状态、MakeForgeRepairable / active |
| [[reference/components/sewing]] | 衣物时间耐久修补、燃料单位与工具次数／堆叠消费 | 针线包、缝补装备、时间耐久修复 / active |

## 怀表拆解

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/pocketwatch-dismantler]] | 拆表资格／执行分离、配方全返与排除噩梦燃料、拟态分支 | 禁止拆解回溯表、精密工具拆表 / active |

## 刷剃与装饰工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/brush]] | 工具空标记、动作候选与目标产出／耐久分工 | 刷子耐久减半、刷毛工具 / active |
| [[reference/components/shaver]] | 剃毛工具标记、目标委派与 SG 前检，区别产物和耗材逻辑 | 燧石当剃刀、剃毛工具 / active |
| [[reference/components/bedazzler]] | 目标状态检查／直接耐久消费，区分角色候选与目标效果 | 蜘蛛巢装饰工具耗用减半、装饰工具消耗 / active |

## 装瓶、诱饵与诅咒物品

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/bottler]] | 装瓶资格与宿主回调委派，区分拆瓶／捕获／生成和成功返回 | 空瓶装取、BOTTLE、canbebottled / active |
| [[reference/components/bait]] | 与陷阱双向关联、吃／偷／拾取通知和安全解除 | 服务端解除熟胡萝卜诱饵关联、陷阱诱饵 / active |
| [[reference/components/curseditem]] | 寻人／追逐、绑定主人、拾取施咒与堆叠字段 | 猴子代币再次拾取静默、诅咒代币追人 / active |

## 升级材料与电路模块

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/upgrader]] | 材料类型／单位贡献、角色标签与目标最终检查 | 蜘蛛丝每件提供两点升级值、升级材料类型 / active |
| [[reference/components/upgrademodule]] | 模块槽位／类型、绑定／激活与成对撤销、移出耐久 | 健康电路激活时额外加速、WX78模块激活与撤销 / active |
| [[reference/components/upgrademoduleremover]] | 空工具标记、检视／手持准备、人物 SG 拔出与停止链 | 服务端结束拆卸工具持有者检视、WX78模块拆卸工具 / active |
| [[reference/components/moonsparkchargeable]] | 月暴火花充电：DoSpark 按百分比回充 fueled，护目镜挂载 | 月暴充电、火花充电 / active |
| [[reference/components/disappears]] | 倒计时播消失动画并自删，拾取可停表；disappearFn/disappearsFn 死字段陷阱 | 自动消失、消失倒计时 / active |
| [[reference/components/fumaroletool]] | 温度档位热工具：冷却自毁、复热修复、使用耗热 | 气孔工具 / active |

返回 [[reference/components/index|组件主题导航]]。
