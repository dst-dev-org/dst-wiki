---
title: "spellbook"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "管理物品技能轮盘与选择，分开客户端 UI、服务端动作及技能实际效果。"
aliases: ["技能轮盘", "物品技能菜单", "轮盘技能选择", "spellbook"]
tags: ["component", "item", "callback", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "components/inventory.lua", "components/spellbook.lua", "networkclientrpc.lua", "prefabs/abigail_flower.lua", "prefabs/ghostcommand_defs.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# spellbook

## 快速判断

- 职责与适用：轮盘配置、选中项与执行入口；不是普通 spellcaster 的别名，也不自动生成所有技能效果。[components/spellbook.lua:15](../../../raw/current/scripts/components/spellbook.lua#L15
- 优先使用：双方 SetItems，条目 onselect 选择实际 action／spellfn，execute 走正常 inventory／动作链。[prefabs/ghostcommand_defs.lua:26](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L26
- 重要副作用：此组件原版在客户端也添加；OpenSpellBook 需要 user.HUD，不能只写服务端配置。[prefabs/abigail_flower.lua:311](../../../raw/current/scripts/prefabs/abigail_flower.lua#L311 [components/spellbook.lua:88](../../../raw/current/scripts/components/spellbook.lua#L88
- 修改边界：轮盘选择并非服务端授权，最终效果函数仍核验归属、目标、冷却与角色条件。[actions.lua:5998](../../../raw/current/scripts/actions.lua#L5998

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetItems(items)` | 有序条目数组 | CanBeUsedBy 还检查 items 非空；双方条目序号／含义须一致。[components/spellbook.lua:54](../../../raw/current/scripts/components/spellbook.lua#L54 [components/spellbook.lua:80](../../../raw/current/scripts/components/spellbook.lua#L80 |
| `SetRequiredTag(tag)`／`SetCanUseFn(fn)` | 标签／`(inst, user)->boolean` | 决定 CanBeUsedBy；自定义函数必须兼容两端可用信息。[components/spellbook.lua:80](../../../raw/current/scripts/components/spellbook.lua#L80 |
| `SelectSpell(id)` | 条目编号；返回 boolean | 设置 spell_id，执行 items[id].onselect(inst)；失效编号也先写入 spell_id，不能据选中值推断有效。[components/spellbook.lua:97](../../../raw/current/scripts/components/spellbook.lua#L97 |
| `SetSpellAction(action)` | Action 或 nil | 客户端／服务端可用；nil 走默认 CAST_SPELLBOOK。[components/spellbook.lua:124](../../../raw/current/scripts/components/spellbook.lua#L124 [components/inventory.lua:2444](../../../raw/current/scripts/components/inventory.lua#L2444 |
| `SetSpellFn(fn)`／`CastSpell(user)` | `(inst, user)->结果` | 服务端效果路径；不自查 CanBeUsedBy，动作调用前检查。[components/spellbook.lua:136](../../../raw/current/scripts/components/spellbook.lua#L136 [actions.lua:5998](../../../raw/current/scripts/actions.lua#L5998 |
| `SetRadius(radius)`／`SetFocusRadius(radius)` | UI 数值 | 默认 175／178，轮盘尺寸不是攻击／施法距离。[components/spellbook.lua:15](../../../raw/current/scripts/components/spellbook.lua#L15 |

## 典型接入

### 阿比盖尔之花的轮盘分层

花的工厂在 SetPristine／客户端返回前添加 spellbook，配置 GHOSTCOMMAND_DEFS.GetBaseCommands 与 UI；不要把组件放进服务端分支。[prefabs/abigail_flower.lua:311](../../../raw/current/scripts/prefabs/abigail_flower.lua#L311 [prefabs/abigail_flower.lua:333](../../../raw/current/scripts/prefabs/abigail_flower.lua#L333

命令项 onselect 在两端设名称／选择状态，但仅服务端 SetSpellFn；execute 从本地 ThePlayer.replica.inventory 发起 CastSpellBookFromInv。服务端 inventory 选择同一编号并派发动作，实际动作再核验归属与 CanBeUsedBy；效果回调处理召回结果。[prefabs/ghostcommand_defs.lua:14](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L14 [prefabs/ghostcommand_defs.lua:29](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L29 [prefabs/ghostcommand_defs.lua:40](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L40 [components/inventory.lua:2435](../../../raw/current/scripts/components/inventory.lua#L2435 [actions.lua:5998](../../../raw/current/scripts/actions.lua#L5998

复用时按“条目 label／图标 → onselect 选效果 → execute 发动作 → 服务端效果”组织；图标 bank／build／anims 使用已加载资源。不同模式（指定 Action、CAST_SPELLBOOK、地点 AOE）各自核验动作与状态图，本页不提供未核验的完整新轮盘工程。

## 常见陷阱与限制

- **只在服务端 SetItems**：客户端无法打开／理解条目；原版组件双方存在且序号参与 RPC，公共 UI 配置与服务端效果分开。[prefabs/abigail_flower.lua:311](../../../raw/current/scripts/prefabs/abigail_flower.lua#L311 [networkclientrpc.lua:812](../../../raw/current/scripts/networkclientrpc.lua#L812
- **条目 execute 直接改世界状态**：原版 execute 可发生在客户端，走 replica／动作请求；最终效果放服务端并做资格检查。[prefabs/ghostcommand_defs.lua:40](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L40
- **SetRadius 增加施法范围**：它只传给 HUD 轮盘；范围由实际动作／目标机制决定。[components/spellbook.lua:93](../../../raw/current/scripts/components/spellbook.lua#L93

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）与 networkclientrpc.lua 逐 hunk 对照，锚点重定位 3 处（CAST_SPELLBOOK.fn actions.lua 5979→5988），结论修订 0 处——本页动作 CAST_SPELLBOOK 未加 `invalid_hold_action`、无 `nomagiccast` 门控（该门控拦的是 CASTSPELL／BLINK／CORRUPTNIGHTMARE，R-01/R-04），networkclientrpc 仅新增 Shard RPC `SyncCharlieDefeated`，不触及本页 RPC 结论；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：双方条目配置、onselect／execute 到服务端动作与归属检查、阿比盖尔之花路径，依据见上文对应锚点；未交付新轮盘工程或核验全部 AOE／自定义 Action／SG 分支。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页为机制与原版路径说明，未附新轮盘工程。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/spellcaster]]；[[reference/components/rechargeable]]；[[reference/components/inventory]]。
