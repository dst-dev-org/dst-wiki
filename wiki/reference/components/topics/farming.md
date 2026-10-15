---
title: 植物成长与农作组件索引
created: 2026-10-27
updated: 2026-10-28
type: index
tags: [overview]
---

# 植物成长与农作

用于采收、成长、授粉、农田与研究，旧农场路径单独标明；不与制作科技混列。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 采收与成长

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/pickable]] | 采收产物／数量、再生与暂停、回调顺序；保留移植／枯萎边界 | 采收产量、草再生时间、植物采摘 / active |
| [[reference/components/searchable]] | 简化搜寻交互、PICK 分流与宿主产出，区分开关存读和事件／删除 | 搜寻水獭巢、searchable、searched / active |
| [[reference/components/growable]] | 阶段／回调、成长计时／暂停／离屏与保存；复制共享阶段表 | 树成长速度、阶段成长、成长暂停 / active |
| [[reference/components/simplemagicgrower]] | 逐步调用growable推进到指定阶段，魔法生长标签、重入和失败重试边界 | 林业书催长、连续魔法生长 / active |
| [[reference/components/harvestable]] | 产物逐次累积／整批收获、计时与宿主回调，区分蜂箱事件增长 | 蘑菇农场生长间隔减半、蜂箱产蜜 / active |
| [[reference/components/plantregrowth]] | 母株周边后代、种类时钟／桶调度与原始密度，区分本株成长 | 常青树周边再生加快、树木自然繁殖 / active |

## 枯萎保护、授粉与树木促进

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/witherable]] | 自然枯萎／复壮、保护与强制调用、贫瘠和睡醒存读 | 采草后防枯萎一天、植物复壮 / active |
| [[reference/components/pollinator]] | 访问实体／收集门槛、寻花回巢与无家造花分工 | 蜜蜂采三朵花回巢、授粉收集数量 / active |
| [[reference/components/treegrowthsolution]] | 催长入口、目标回调／普通成长、返回与材料消费 | 树木催长、催长剂无成长入口不消耗 / active |

## 农作与施肥基础

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fertilizer]] | 肥料养分、部署／目标消费、耐久与堆叠结算 | 肥料养分、便便施肥加倍 / active |
| [[reference/components/compostingbin]] | 绿料棕料周期、投料／刷新公式、产物回调与保存清理 | 堆肥桶、堆肥速度 / active |
| [[reference/components/forcecompostable]] | 物品绿料／棕料标记与桶分类优先级，双false阻断食物回退，区别可见候选 | 强制堆肥分类、green与brown、堆肥材料标记 / active |
| [[reference/components/fertilizable]] | 人物自肥委托、回调拼写、成功返回与统一消耗 | 人物自肥、沃姆伍德施肥效果 / active |
| [[reference/components/farmtiller]] | 农田锄地、土堆事件与动作耐久消费分工 | 锄地、锄头耐久减半 / active |
| [[reference/components/farmplanttendable]] | 照料开关、阶段重置、快乐压力与无照料者边界 | 与植物对话、植物交互奖励 / active |

## 播种、土壤饮水与植物压力

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/farmplantable]] | 土堆播种、单颗拆分、产物与事件，区分地面部署 | 种子播种、普通种子固定种胡萝卜 / active |
| [[reference/components/farmsoildrinker]] | 土壤耗水率、干湿时长比例、阶段重置与世界登记 | 作物耗水减半、土壤饮水 / active |
| [[reference/components/farmplantstress]] | 阶段压力、累计评级、查看与成长检测分工 | 作物压力、免季节压力、巨型作物条件 / active |
| [[reference/components/farming-manager]] | 世界地块三养分／水分、坐标与资格、阶段消费和存读 | 土壤养分、地块浇水、锄地补水施肥、farming_manager / active |
| [[reference/components/nutrients-visual-manager]] | 本地养分标记样式／登记、等级同步与玩家视野分工，区分实际土壤数据 | 调整养分标记绘制排序、农田养分显示、nutrients_visual_manager / active |

## 植物与肥料研究

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/plantresearchable]] | 植物键／研究阶段、客户端候选与人物学习事件 | 研究胡萝卜同时学会种子、植物图鉴研究 / active |
| [[reference/components/fertilizerresearchable]] | 肥料研究键／事件，区分实际施肥及消费 | 施肥自动研究便便、肥料图鉴 / active |

## 旧农场种植路径

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/plantable]] | 旧种子产物／基准秒数，区分现代土堆播种与地面部署 | 旧农场种子产物、胡萝卜种子旧农场种南瓜 / active |
| [[reference/components/grower]] | 旧设施种植点／时长乘数、肥力轮次与子作物存读 | 旧高级农场时长减半、旧农场肥力轮次 / active |
| [[reference/components/crop]] | 旧作物进度／自然门禁、成熟枯萎、收获清理与保存 | 旧作物生长时间减半、旧农场收获 / active |
| [[reference/components/lordfruitflytrigger]] | 果蝇王范围触发器（孤儿组件）：全库无挂载者 | 果蝇王触发器 / active |
| [[reference/components/desolationspawner]] | 荒芜再播种：按世界生成密度补种树苗，季节月相调制 | 荒芜再播种 / active |
| [[reference/components/forestpetrification]] | 森林石化周期：冷却数年后 BFS 搜林区，低于阈值整片石化 | 森林石化 / active |
| [[reference/components/deciduoustreeupdater]] | 桦树精怪物化：索敌砸根须放桦果鸭，休眠/烧毁即解除 | 桦树精更新器 / active |

浇水／补水工具见[[reference/components/topics/item-maintenance#补水与浇水工具|物品维护]]；人物图鉴更新见[[reference/components/topics/character#图鉴与研究记录|人物记录]]。

返回 [[reference/components/index|组件主题导航]]。
