---
title: summoningitem
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "识别阿比盖尔花朵的召唤物品标记；召唤、收回和行为切换由使用者的 ghostlybond 执行。"
aliases: ["summoningitem", "召唤物品", "阿比盖尔花朵", "CASTSUMMON", "CASTUNSUMMON", "COMMUNEWITHSUMMONED"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/summoningitem.lua", "componentactions.lua", "actions.lua", "components/ghostlybond.lua", "prefabs/abigail_flower.lua", "prefabs/wendy.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# summoningitem

## 快速判断

- 职责与适用：只有 `self.inst = inst` 的标记组件，服务阿比盖尔专用动作入口；自身不实现召唤方法、冷却、存档或网络变量。[完整定义:2](../../../raw/current/scripts/components/summoningitem.lua#L2)
- 优先使用：修改阿比盖尔交互时沿花朵 → 动作 → 使用者 `ghostlybond` 查实际执行；通用自定义施法按需求另选 [[reference/components/spellcaster]] 或 [[reference/components/spellbook]]，不把此标记当通用召唤系统。[actions.lua:3687](../../../raw/current/scripts/actions.lua#L3687)、[abigail_flower.lua:311](../../../raw/current/scripts/prefabs/abigail_flower.lua#L311
- 重要副作用：标记被动作收集器用于候选动作，最终效果改变的是执行者已关联的幽灵；新实体由 ghostlybond 的 `SpawnGhost` 创建，`Summon` 则将已有幽灵放回场景。[componentactions.lua:2989](../../../raw/current/scripts/componentactions.lua#L2989)、[ghostlybond.lua:155](../../../raw/current/scripts/components/ghostlybond.lua#L155)、[:167](../../../raw/current/scripts/components/ghostlybond.lua#L167)
- 修改边界：原版花朵在 SetPristine 后客户端返回，服务端才挂 summoningitem；使用者必须具备 ghostlybond，单给任意物品加组件不构成可用召唤方案。[abigail_flower.lua:333](../../../raw/current/scripts/prefabs/abigail_flower.lua#L333)、[:370](../../../raw/current/scripts/prefabs/abigail_flower.lua#L370)、[actions.lua:3688](../../../raw/current/scripts/actions.lua#L3688

## 关键接口

唯一字段 `inst` 是宿主物品实体；本组件没有 `Summon`、`Recall` 或设置回调的方法，下面均是**读取方**接口。[summoningitem.lua:2](../../../raw/current/scripts/components/summoningitem.lua#L2

| 动作／入口 | 收集条件 | 服务端执行与证据 |
|---|---|---|
| INVENTORY → `CASTSUMMON` | doer 有 `ghostfriend_notsummoned` | 检查 invobject 有组件、doer 有 ghostlybond，再调用 `ghostlybond:Summon(summoningitem.inst)`，返回其结果。[componentactions.lua:2989](../../../raw/current/scripts/componentactions.lua#L2989)、[actions.lua:3687](../../../raw/current/scripts/actions.lua#L3687 |
| INVENTORY → `COMMUNEWITHSUMMONED` | 前项不满足且有 `ghostfriend_summoned` | 同样核对两组件，再调用 `ghostlybond:ChangeBehaviour()`；不是组件内部的随从 AI。[componentactions.lua:2992](../../../raw/current/scripts/componentactions.lua#L2992)、[actions.lua:3703](../../../raw/current/scripts/actions.lua#L3703 |
| USEITEM → `CASTUNSUMMON` | 目标不在 limbo、有 follower replica 且 leader 是 doer；doer 有 summoned 标签、目标有 `abigail` | 最终转交 `ghostlybond:Recall(false)`；该动作函数不使用 act.target 指定另一个随从。[componentactions.lua:1818](../../../raw/current/scripts/componentactions.lua#L1818)、[actions.lua:3693](../../../raw/current/scripts/actions.lua#L3693 |

两种 summoned 标签由 ghostlybond 的属性回调维护，不是 summoningitem 构造时赋给物品的标签。[ghostlybond.lua:1](../../../raw/current/scripts/components/ghostlybond.lua#L1)、[属性绑定:40](../../../raw/current/scripts/components/ghostlybond.lua#L40)

## 典型接入

### 定位原版阿比盖尔花朵的标记与实际召唤分工

1. 花朵工厂服务端加 inventoryitem 与 summoningitem，最终注册名为 `abigail_flower`；物品归属行为复用 [[reference/components/inventoryitem]]。[abigail_flower.lua:366](../../../raw/current/scripts/prefabs/abigail_flower.lua#L366)、[注册:456](../../../raw/current/scripts/prefabs/abigail_flower.lua#L456)
2. 温蒂原版常规模式分支挂 ghostlybond，配置召唤／收回／行为回调，并 `Init("abigail", ...)`；Init 安排任务创建幽灵，不能由空标记替代这些宿主配置。[wendy.lua:380](../../../raw/current/scripts/prefabs/wendy.lua#L380)、[初始化:386](../../../raw/current/scripts/prefabs/wendy.lua#L386)、[ghostlybond.lua:147](../../../raw/current/scripts/components/ghostlybond.lua#L147
3. `Summon` 仅在已有幽灵且 notsummoned 时解除父级、定位、ReturnToScene，使用花朵实体的 `linked_skinname`／`skin_id` 换肤，并调用宿主 onsummonfn。皮肤字段属于物品，不由 summoningitem 初始化。[ghostlybond.lua:167](../../../raw/current/scripts/components/ghostlybond.lua#L167
4. 当前花朵还在公共初始化配置 spellbook（要求 `ghostlyfriend` 标签）和 aoetargeting，服务端另有 aoespell；不能仅凭此标记推导完整轮盘、技能或范围命令。扩展时按 [[reference/components/spellbook]]、[[reference/components/aoespell]] 的分工查询。[abigail_flower.lua:311](../../../raw/current/scripts/prefabs/abigail_flower.lua#L311)、[范围执行:358](../../../raw/current/scripts/prefabs/abigail_flower.lua#L358)

本页仅给已核验入口与分工，不提供复制花朵即可召唤自定义生物的 demo。

## 常见陷阱与限制

- **手动加 summoned 标签就算完成召唤**：标签只影响候选动作，ghostlybond 还检查实际 ghost 与状态；`Summon` 不满足条件返回 false，Recall／ChangeBehaviour 也各有状态条件。[ghostlybond.lua:167](../../../raw/current/scripts/components/ghostlybond.lua#L167)、[收回:202](../../../raw/current/scripts/components/ghostlybond.lua#L202)、[切换:228](../../../raw/current/scripts/components/ghostlybond.lua#L228)
- **收回立即删除召唤物**：动作调用 Recall(false)，温蒂回调让幽灵进入 `dissipate`；`RecallComplete` 另负责 RemoveFromScene、挂到玩家父级并维护状态。本页未审阅完整 SG 完成链，不把动作成功当成已完成收回。[wendy.lua:216](../../../raw/current/scripts/prefabs/wendy.lua#L216)、[ghostlybond.lua:214](../../../raw/current/scripts/components/ghostlybond.lua#L214
- **物品拥有通用召唤权限或保存宠物关系**：空定义没有权限配置或保存接口，执行者关系在 ghostlybond，物品上此组件仅供动作识别与取得宿主实体。[summoningitem.lua:2](../../../raw/current/scripts/components/summoningitem.lua#L2)、[actions.lua:3687](../../../raw/current/scripts/actions.lua#L3687

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua（更新报告 R-01/R-04）——CASTSUMMON／CASTUNSUMMON／COMMUNEWITHSUMMONED 收集与执行未变（本轮新增 spellcaster／corruption 处理器不触及）；锚点重定位 10 处；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整审阅 6 行定义，核对三项动作收集／执行、花朵实际挂载与注册、温蒂初始化及 ghostlybond 相关执行分支。当前 scripts 下精确字面 AddComponent 挂载检索仅命中 abigail_flower；不据此排除动态挂载或外部模组。
- 结构检查：本页元数据、大小、Wiki／Markdown 链接与源码锚点检查通过；无新增完整 Lua 示例，导航登记由本批整合检查。
- 游戏运行：未运行；未覆盖完整阿比盖尔技能轮盘、状态图、存档／迁移、多人与皮肤资源交互，也未交付通用召唤实现。
