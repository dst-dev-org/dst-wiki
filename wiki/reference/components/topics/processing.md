---
title: 物品加工与产物组件索引
created: 2026-10-24
updated: 2026-10-28
type: index
tags: [overview]
---

# 物品加工与产物

用于食物属性与新鲜度、烹饪晾晒，以及消化／焚烧和条件产物；食用者规则见[[reference/components/topics/character#人物三维与进食|人物状态]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 食物与保鲜

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/edible]] | 食物类型、三维基础收益和吃下回调／消耗 | 新增食物、食物三维 / active |
| [[reference/components/murderable]] | 可宰杀物品标记／音效、动作移出与掉落，区别生命死亡流程 | 宰杀活物、库存生物处置 / active |
| [[reference/components/perishable]] | 基础保质期、启动／暂停、比例与变质替换 | 腐败时间、新鲜度 / active |
| [[reference/components/preserver]] | 直接持有者的腐败／温度倍率；普通背包禁入容器与模组嵌套边界 | 背包放冰箱、保鲜背包、反鲜 / active |
| [[reference/components/preservative]] | 消耗材料一次性恢复新鲜度、目标堆叠分摊与先恢复后扣料 | 盐补新鲜度、一次性防腐剂 / active |

## 烹饪与干燥

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/cookable]] | 食材直接烤制产物、烤前回调与新鲜度，区分原料消费 | 烤熟产物、烤肉变肉干 / active |
| [[reference/components/cooker]] | 直接烤制设备门禁、结果回调、动作拆堆与发放分工 | 火堆烤食物、烤熟恢复新鲜 / active |
| [[reference/components/dryable]] | 原料晾晒产物／秒数／显示 build，区分架子计时 | 肉干时间、晾晒原料 / active |
| [[reference/components/dryingrack]] | 多格晾晒、天气／免雨、保鲜与转移／保存恢复 | 晾晒架、雨天继续晾晒 / active |
| [[reference/components/stewer]] | 锅中配方／耗料、时长倍率、锅内腐败与收获 | 厨锅时间减半、锅内料理 / active |
| [[reference/components/dryer]] | 旧单物晾晒记录、输入删除与暂停／腐败／保存迁移，当前晾架使用dryingrack | 旧晾晒组件、StartDrying、旧档迁移 / active；旧路径说明 |
| [[reference/components/wobyrack]] | Woby 随身晾晒架：独立容器＋技能开关＋按组件名存读 | 沃比晾晒架 / active |
| [[reference/components/dryingracksaltcollector]] | 晾架盐槽布尔与数量、回调／存档；实际生成和返还由宿主负责 | 晾架集盐、盐槽记录、AddSalt / active |

## 背包消化与容器焚烧

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/digester]] | 周期挑选背包主栏物品、按prefab消费与任务启停，不自动生成产物 | 背包自动消化、TryDigest、inventory消耗 / active |
| [[reference/components/incinerator]] | 条件删除容器整堆、事件和操作者归属，动作权限与效果由宿主分工 | 焚烧容器内容、Incinerate、onincinerated / active |

## 条件产物累积

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/incrementalproducer]] | 按条件装填产物预算与逐次生产，数量／间隔与回调由宿主驱动 | 产物批次预算、CanProduce、TryProduce / active |

## 节庆大餐

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/feasts]] | 世界大餐桌组与进食者调度、食物消费及增益，实际进出由宿主／SG协作 | 冬季盛宴、大餐桌组、feasts / active |

返回 [[reference/components/index|组件主题导航]]。
