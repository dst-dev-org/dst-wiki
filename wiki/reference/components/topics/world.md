---
title: 工作、掉落与火情组件索引
created: 2026-10-17
updated: 2026-10-29
type: index
tags: [overview]
---

# 工作、掉落与火情

用于工具工作、地皮处理、掉落、燃烧与灭火；制作和农作已按需求拆开。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 制作、采集与掉落

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/workable]] | 工作目标的动作／剩余量、反弹与完成顺序，区分工具效率 | 采集工作量、锤拆建筑、禁用工作动作 / active |
| [[reference/components/worker]] | 执行者自身基础工作量、工具优先级与海狸动态宿主 | 自身工作能力、海狸人砍树效率 / active |
| [[reference/components/workmultiplier]] | 人物工作量来源倍率、反弹后特殊量与目标结算分工 | 人物砍树效率、工作量加倍 / active |
| [[reference/components/efficientuser]] | 执行者动作使用倍率、工具／攻击耐久与治疗消费者 | 砍树耐久减半、攻击耐久倍率 / active |
| [[reference/components/terraformer]] | 改地皮资格、下层地皮恢复／掉落委派、草叉与犁的分工 | 草叉铲地皮、铲地耐久 / active |
| [[reference/components/autoterraformer]] | 容器地皮自动铺挖、当前格重复与耗料耐久、装备启停和缓存边界 | 自动改地皮、步行铺地皮、StartTerraforming / active |
| [[reference/components/terraformerremoveable]] | 铲除目标标记、移除回调与高尔夫场地生命周期 | 草叉移除场地、terraformerremoveable / active |
| [[reference/components/gravedigger]] | 迁墓工具回调、骨灰坛保存与部署转换，先工具后目标 | 骨灰坛、整墓搬迁工具 / active |
| [[reference/components/gravediggable]] | 迁墓目标标签／回调／开关存档，拒绝需早于工具副作用 | 可迁移墓碑、阻止整墓搬迁 / active |
| [[reference/components/lootdropper]] | 固定／概率／加权／配方掉落、触发点与实际实体散落 | 额外掉落、掉落表、拆除返材料 / active |
| [[reference/components/shedder]] | 周期／手动脱落物、绝对出生高度与物理散射，熊獾宿主控制冬眠死亡停机 | 熊獾掉毛、定时毛簇、批量脱落 / active |

制作与科技学习已移至[[reference/components/topics/crafting|制作科技与建造]]；此组保留工作、地皮工具与掉落。


## 世界状态、季节与环境

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/ocean]] | 洋流方向原型（角度＋速度向量）；全库无消费者 | 洋流原型、水流方向 / active |
| [[reference/components/worldwind]] | 世界风向服务，周期随机转向并推 windchange | 世界风向、windchange 事件 / active |
| [[reference/components/worldmeteorshower]] | 流星矿石升级月岩壳的累积概率与外部修正表 | 月岩壳概率 / active |
| [[reference/components/worldstate]] | TheWorld.state 数据本体：15 类世界事件汇聚、watcher 通知 | 世界状态 / active |
| [[reference/components/clock]] | 16 段昼夜相位＋月相循环时钟，net 同步与事件推送 | 世界时钟 / active |
| [[reference/components/seasons]] | 四季三模式推进并按进度插值下发时钟昼夜段 | 季节、四季循环 / active |

## 地形、桥梁与资源再生

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/area_unlock]] | 区域剧情解锁原型：changearea 记 story 到访；全库无消费者 | 区域解锁、剧情区域追踪 / active |
| [[reference/components/groundcreep]] | TheWorld 蠕变存读垫片，数据本体在引擎 inst.GroundCreep | 地面蠕变 / active |
| [[reference/components/vinebridgemanager]] | 藤桥地皮管理：铺 CHARLIE_VINE 格保留底层、耐久计时破坏 | 藤桥管理器 / active |
| [[reference/components/regrowthmanager]] | 世界资源再生：beginregrowth 入队、季节倍率时钟、原位抖动重生 | 世界资源再生 / active |

## 月坛、演出与特殊生成

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/winter_treeseed]] | 冬日盛宴树种：winter_tree 字段＋SetTree 决定种出的树，仅可种 winter_treestand | 冬之树种、冬日盛宴树种 / active |
| [[reference/components/moonaltarlink]] | 月坛连线实体：质心定位＋三坛 moonaltarlinktarget 双向引用 | 月坛连线 / active |
| [[reference/components/charliecutscene]] | 中庭门修门/嵌钥匙两段过场编排与镜头锁 | 中庭过场 / active |

## 科技与建造接入

制作科技、放置和施工见[[reference/components/topics/crafting|制作科技与建造]]。

## 燃烧与传播

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/burnable]] | 冒烟／点燃／计时／熄灭和物品／建筑清理，区分存档恢复 | 燃烧时间、建筑烧毁、熄灭 / active |
| [[reference/components/lighter]] | 点火工具 LIGHT／Ignite、尝试回调与额外燃料成本 | 火把点火、onlighterlight / active |
| [[reference/components/firebug]] | 定时脚下生成／理智门禁与睡眠启停；未定位原版构造宿主 | 定时生火、脚下生成火源 / active |
| [[reference/components/propagator]] | 闪点／累计热、范围／火伤资格，区分供热与防火门禁 | 火势传播、燃烧火伤、阻止受热 / active |
| [[reference/components/firedetector]] | 灭火器目标扫描、主动／警戒／应急切换与回调，区别实际灭火效果 | 自动灭火检测、防枯萎扫描 / active |
| [[reference/components/smotherer]] | 空动作标记：让可燃烧物被“闷熄”而非点燃，消耗在 burnable | 闷熄、熄火、smotherer / active |

## 采收与成长

本组条目已移至[[reference/components/topics/farming#采收与成长|采收与成长]]，原组件正文不动。

## 枯萎保护、授粉与树木促进

本组条目已移至[[reference/components/topics/farming#枯萎保护授粉与树木促进|枯萎保护、授粉与树木促进]]，原组件正文不动。

## 农作与施肥基础

本组条目已移至[[reference/components/topics/farming#农作与施肥基础|农作与施肥基础]]，原组件正文不动。

## 播种、土壤饮水与植物压力

本组条目已移至[[reference/components/topics/farming#播种土壤饮水与植物压力|播种、土壤饮水与植物压力]]，原组件正文不动。

## 植物与肥料研究

本组条目已移至[[reference/components/topics/farming#植物与肥料研究|植物与肥料研究]]，原组件正文不动。

## 旧农场种植路径

本组条目已移至[[reference/components/topics/farming#旧农场种植路径|旧农场种植路径]]，原组件正文不动。

## 制作采集与农作

工作与火情留在本页；制作见[[reference/components/topics/crafting|制作科技与建造]]，植物见[[reference/components/topics/farming|植物成长与农作]]，保留原总标题锚点。

返回 [[reference/components/index|组件主题导航]]。
