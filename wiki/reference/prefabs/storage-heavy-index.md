---
title: "装备·背包收纳与重物搬运"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# 装备·背包收纳与重物搬运

背包与袋类收纳、重物搬运的叶级索引；收纳类多为 BODY 槽，重物类占 BODY 槽并带 heavy 标签，伴随减速或持握限制。按主要用途唯一登记，皮肤 FX、角色门禁与采矿工具从文末“跨用途入口”进入。返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。

## 背包收纳

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/backpack|背包（backpack）]] | BODY 槽；八格背包、穿脱开关、收纳限制、燃烧与皮肤跟随 FX | 普通背包、Backpack、背包格子、背包放冰箱、穿戴开包 / active |
| [[reference/prefabs/piggyback|猪皮包（piggyback）]] | BODY 槽；12 格减速 0.9（强壮态豁免、vigorbuff 回补）、自身不吃湿但贡献 0 防水、无耐久不可燃；三皮肤 | piggyback、猪皮包、大象背包、血色背包 / active |
| [[reference/prefabs/krampus_sack|坎普斯背包（krampus_sack）]] | BODY 槽；14 格、无配方（坎普斯共享表 1% 基础概率、经 `GetChance` 吃幸运；克劳斯袋 10% 走 `math.random()`）、无减速、防水 0 语义辨析、酸雨排除 | krampus_sack、坎普斯包、小偷背包 / active |
| [[reference/prefabs/icepack|保鲜背包（icepack）]] | BODY 槽；8 格（共享 backpack 布局）、fridge 给 0.5 **基础倍率**（环境 <0 且 `frozen` 又未开 `frozenfiremult` 时才直接停腐；湿润/季节再加成）＋nocool 跳过冰箱主动降温、无 insulator 不影响体温 | icepack、保鲜背包、Insulated Pack、冷藏箱 / active |
| [[reference/prefabs/candybag|糖果袋（candybag）]] | BODY 槽；14 格糖果/挂饰/小玩偶专用过滤、HALLOWED_NIGHTS 活动科技解锁链、可燃烧燃尽掉物成灰；穿戴不触发糖果收集 | candybag、糖果袋、Candy Bag / active |
| [[reference/prefabs/pouch-bags|小袋族（seedpouch 等 6 注册）]] | BODY 槽；种子袋 14 格过滤+保鲜 0.5、厨师袋 6 格 Warly 限定（foodpreserver 标签非组件）、土豆袋实为健身配重、胡须袋 BEARD 槽威尔逊技能三档 **1/2/3 格**（胡须位数门槛 1/3/9） | seedpouch、spicepack、potatosack、beard_sack、种子袋、厨师袋、土豆袋、胡须袋 / active |

## 重物搬运

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/heavy-items|重型障碍持握族（glassspike 五注册＋oceantreenut）]] | BODY 槽，带 heavy 标签；玻璃尖刺/玻璃城堡的 item/obstacle 双态与锤毁、疙瘩树果撞船掉落与沉水生长（船/树遮挡 60 秒重试）、减速 0.15 | glassspike、glassspike_short、glassspike_med、glassspike_tall、glassblock、oceantreenut、玻璃尖刺、玻璃城堡、疙瘩树果 / active |
| [[reference/prefabs/giant-crops|巨型作物（oversized／waxed／rotten 三变体）]] | BODY 槽，带 heavy 标签；14 种有种子蔬菜×3 变体：腐烂三分支、蜂蜡换手、gymweight 4/3、HAMMER 1 掉落 | 巨型作物、oversized、waxed、rotten、carrot_oversized、巨型腐烂胡萝卜 / active |
| [[reference/prefabs/moon-altar-pieces|天体祭坛部件与吸引人的结构（moon_altar_pieces）]] | BODY 槽，带 heavy 标签；6 件部件，另有 3 种 MINE 20 岩石（吸引人的结构/Inviting Formation，旧检索词「岩石包裹」）；socket 三件（glass/crown/icon）决定裂隙生成哪个祭坛，seed/idol/ward 是祭坛按 stage 认的补件；落海可能换水点或上岸，沉水件由 `calling_moon_relics` 检索 | moon_altar_idol、moon_altar_glass、moon_altar_seed、moon_altar_crown、moon_altar_ward、moon_altar_icon、祭坛部件、月坛部件、吸引人的结构 / active |

## 跨用途入口

- 普通背包已从 [[reference/prefabs/index|Prefab 索引]] 的表格迁入本叶；其皮肤换装代理与跟随实体见 [[reference/prefabs/backpack#皮肤差异与跟随实体|背包的皮肤与跟随实体]]，全部装备皮肤 FX 见 [[reference/prefabs/fx-skin-index|FX·皮肤与装备挂件]]。
- 胡须袋是 BEARD 槽的威尔逊角色装备，登记在小袋族；角色专属装备总入口见 [[reference/prefabs/staffs-character-index|法杖与角色道具]]。
- 重型障碍的减速、item／obstacle 双态、撞船掉落与沉水生长见本页；锤毁与采集工具入口见 [[reference/prefabs/tools-field-index|工具与野外实用]]。
- 天体祭坛部件的 MINE 岩石与祭坛 socket 链见本页；采矿工具与耐久见 [[reference/prefabs/tools-field-index|工具与野外实用]]。
- 巨型作物与天体祭坛部件的生成／获取链见各页；普通食物、建筑与容器机制不登记到本叶。

返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
