---
title: 制作科技与建造组件索引
created: 2026-10-27
updated: 2026-10-28
type: index
tags: [overview]
---

# 制作科技与建造

用于制作、科技学习、放置预览、结构拆装和施工材料交接；工具工作量与掉落见[[reference/components/topics/world#制作采集与掉落|工作索引]]。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 制作科技与学习

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/builder]] | 配方学习／材料折扣、预制作及制作事件，区分配方注册 | 学习配方、预制作建筑、建造回调 / active |
| [[reference/components/prototyper]] | 科技来源树、多人使用状态与激活回调，区分配方注册／学习 | 科技站、科学机器升级 / active |
| [[reference/components/researchpointconverter]] | 源码明确弃用的研究转换器字段／回调，现行科技入口用prototyper | 旧研究点转换器 / active；旧接口说明 |
| [[reference/components/teacher]] | 图纸指定配方、学习资格与成功消费，区分宿主存档 | 蓝图学习、Teach、learnrecipe / active |
| [[reference/components/recipescanner]] | 从目标映射配方、学习门禁与扫描事件，消费由宿主负责 | 配方扫描器、SCANNABLE_RECIPENAME / active |
| [[reference/components/craftingstation]] | 站点条目与配方对应、nounlock 消费、限额与保存恢复 | 工作站配方、雕塑台图纸 / active |
| [[reference/components/recipestockpile]] | 制作配额记账、定时逐份补货与存读，制作门禁及清空缺口需另行核对 | 配方库存、配额补货 / active；未见原版宿主 |

## 放置与结构拆装

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/placer]] | 本地建筑／部署虚影、手柄距离、吸附与判断／执行分工 | 建筑预览、放置虚影、网格吸附 / active |
| [[reference/components/deployhelper]] | 附近实体本地辅助、配方／key 过滤、回调与自动关闭 | 部署辅助圈、放置范围提示 / active |
| [[reference/components/fencerotator]] | 工具旋转目标朝向、目标资格与动作、事件消费和特效 | 栅栏旋转、定向物体、ROTATE_FENCE / active |
| [[reference/components/portablestructure]] | 可拆卸实体回调、动作准入与宿主打包，区分完整状态转存 | 便携锅收起、拆卸建筑、Dismantle / active |

## 施工与材料交接

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/constructionsite]] | 分阶段站点材料进度、启停／完工回调、超量返还与存读边界 | 建造站点、材料进度、施工完成回调 / active |
| [[reference/components/constructionbuilder]] | 玩家侧施工执行态、临时容器开合返还、SG 三段与读档不重开 | 玩家建造状态、施工中断、施工容器 / active |
| [[reference/components/constructionplans]] | 计划物品把已有实体原位替换成站点，目标映射与 tag 边界 | 建造计划、计划物品改造站点 / active |
| [[reference/components/constructionbuilderuidata]] | 客户端施工容器／站点的 net_entity 桥接与构造顺序 | 施工 UI 数据、施工容器引用 / active |
| [[reference/components/madsciencelab]] | 分阶段制作进度机：stages 计时推进＋三回调，实验室/织机/烤箱共用 | 疯狂科学实验室进度 / active |
| [[reference/components/yotb_sewer]] | 猫年缝纫机：按容器服装配方缝制产物，拒绝退料/断线续缝 | 缝纫机 / active |

物品部署动作见[[reference/components/topics/item-actions#物品交互与技能|物品交互]]；配方／Hook 注册等未覆盖 API 仍需定向查源码。

返回 [[reference/components/index|组件主题导航]]。
