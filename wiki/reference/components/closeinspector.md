---
title: closeinspector：装备驱动的近距离检查分派
created: 2026-10-23
updated: "2026-10-28"
type: component
summary: 将玩家的实体／地点近距离检查交给装备回调；以玫瑰眼镜为例核对技能启用、卸装清理和动作返回。
aliases: [closeinspector, 近距离检查, 玫瑰眼镜检查, 装备检查回调]
tags: [component, equipment, actions, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/closeinspector.lua, componentutil.lua, actions.lua, prefabs/hats.lua, components/roseinspectableuser.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua]
reliability: verified
validation: source-reviewed
status: active
---

# closeinspector：装备驱动的近距离检查分派

## 快速判断

- 职责与适用：挂在装备上，为近距离检查分别保存实体目标、地点回调；不是普通物体检查文本组件。[完整组件](../../../raw/current/scripts/components/closeinspector.lua#L1-L36) 普通检查文字看 [[reference/components/inspectable]]。
- 优先使用：需要复用原版近距离检查时配置两类回调；只改玫瑰眼镜的检查结果，应保留其动态挂载与 `roseinspectableuser` 委托，不另造常驻启用状态。[原版回调与启用](../../../raw/current/scripts/prefabs/hats.lua#L5085-L5110)
- 重要副作用：构造时添加 `closeinspector` 标签，移除组件时清标签；两类检查原样返回回调结果，未设置时没有默认成功。[组件生命周期与分派](../../../raw/current/scripts/components/closeinspector.lua#L1-L34)
- 修改边界：原版在服务端按装备者技能动态添加组件；客户端菜单依赖装备标签，不能只改客户端回调或给玩家自身挂组件。[宿主](../../../raw/current/scripts/prefabs/hats.lua#L5157-L5177) [双端门禁](../../../raw/current/scripts/componentutil.lua#L740-L763)

## 关键接口

| 接口 | 参数、返回与边界 |
|---|---|
| `SetInspectTargetFn(fn)` | 保存 `fn(item, doer, target)`；约定返回动作 `success, reason`，调用端会按真假处理成功状态。[设置与调用](../../../raw/current/scripts/components/closeinspector.lua#L14-L27) |
| `SetInspectPointFn(fn)` | 保存 `fn(item, doer, pt)`；同样原样传回全部返回值，`pt` 为动作地点，不是目标实体。[设置与调用](../../../raw/current/scripts/components/closeinspector.lua#L19-L34) |
| `CloseInspectTarget(doer, target)`／`CloseInspectPoint(doer, pt)` | 有回调才执行；不自行验证目标、距离、装备、技能或冷却。无回调返回 `nil`，不是 `true`。[分派](../../../raw/current/scripts/components/closeinspector.lua#L24-L34) |
| `OnRemoveFromEntity()` | 移除 `closeinspector` 标签；组件本身没有保存／加载或事件监听清理逻辑。[构造与移除](../../../raw/current/scripts/components/closeinspector.lua#L1-L12) |

`CLOSEINSPECTORUTIL.CanCloseInspect` 要求玩家装备中有该标签且未骑乘；实体目标还排除有非零质量物理、移动组件／标签、物品栏组件／副本或 `character` 标签的对象。地点走 `ROSEPOINT_CONFIGURATIONS` 的检查，不是任意地面都可触发。[实体资格](../../../raw/current/scripts/componentutil.lua#L699-L715) [地点资格](../../../raw/current/scripts/componentutil.lua#L717-L738) [公共门禁](../../../raw/current/scripts/componentutil.lua#L740-L763)

`LOOKAT` 在 `invobject == nil` 时尝试近距离检查，按 `pairs(EQUIPSLOTS)` 找到首个有该组件的已装备物品，调用后立即返回；失败会按 `reason` 说失败台词，**不会继续试另一装备或退回普通检查**。多个检查装备没有稳定的槽位优先约定。[分派全分支](../../../raw/current/scripts/actions.lua#L1226-L1268) [普通检查后备](../../../raw/current/scripts/actions.lua#L1271-L1283)

## 典型接入

### 修改玫瑰眼镜效果时保留技能与卸装边界

原版注册链：`MakeHat("roseglasses")` 以 `name.."hat"` 命名，选择 `fns.roseglasses`，最终注册 **`roseglasseshat`**；不要把工厂局部名误当 Prefab 名。[命名](../../../raw/current/scripts/prefabs/hats.lua#L16-L19) [工厂选择](../../../raw/current/scripts/prefabs/hats.lua#L6946-L6947) [返回 Prefab](../../../raw/current/scripts/prefabs/hats.lua#L6983) [调用](../../../raw/current/scripts/prefabs/hats.lua#L7938)

服务端工厂安装装备／卸装回调与技能刷新闭包；装备时先接收装备者技能变化，再按 `winona_charlie_1` 决定是否添加组件并设置两类检查函数。卸装会取消装备者的技能事件监听并移除组件。不要在 PrefabPostInit 里无条件补回组件，否则会破坏技能及卸装边界。[工厂](../../../raw/current/scripts/prefabs/hats.lua#L5157-L5177) [技能条件](../../../raw/current/scripts/prefabs/hats.lua#L5099-L5110) [监听及装备生命周期](../../../raw/current/scripts/prefabs/hats.lua#L5114-L5137)

原版目标回调是 `fn(item, owner, target)`，地点回调是 `fn(item, owner, pt)`；二者先确认玩家有 `roseinspectableuser`，再分别委托其 `TryToDoRoseInspectionOnTarget`／`TryToDoRoseInspectionOnPoint`，并传回动作结果。组件自身不生成残影、不管理冷却。[装备委托](../../../raw/current/scripts/prefabs/hats.lua#L5085-L5096)

其中目标分支还检查目标支持、禁止标签、残影生成资格和冷却，并返回对应失败原因；地点分支设置检查地点、说话后返回 `true`。修改“能否检查哪种对象”应区别公共动作门禁与玩家实际效果，不能只替换一段菜单显示。[实际目标执行](../../../raw/current/scripts/components/roseinspectableuser.lua#L163-L195) [地点执行](../../../raw/current/scripts/components/roseinspectableuser.lua#L197-L202)

本节是核验后的原版接入路径，未提供独立新装备 demo；新增装备还需 [[reference/components/equippable]] 的工厂、同步与卸装约定。改写两类回调时保留原回调返回，额外效果仅在其成功后处理；若要替换原有检查规则，须单独核验上述门禁及玩家效果，而不是全局改所有装备。

## 常见陷阱与限制

- **把组件加在玩家上**：动作扫描的是已装备物品，不会从玩家的 `components.closeinspector` 取回调。[动作查找](../../../raw/current/scripts/actions.lua#L1226-L1244)
- **只配置实体回调却假定地点成功**：无地点回调时返回 `nil`；动作仍命中该装备并结束，见[关键接口](#关键接口)。
- **用标签代替服务端组件**：标签用于资格判断，实际动作仍要求装备有组件；反过来组件回调也不替代目标门禁。[门禁](../../../raw/current/scripts/componentutil.lua#L740-L763) [实际查找](../../../raw/current/scripts/actions.lua#L1231-L1243)
- **有检查动画就认定执行成功**：客户端 `closeinspect` 会先执行预览动作；服务端还可能由带 `closeinspect` 参数的失败台词进入相同动画，骑乘／重物／引导施法另有状态限制。[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L7342-L7368) [服务端台词处理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2164-L2180)
- **无条件常驻添加组件**：会越过原版技能切换与卸装移除；技能监听属于宿主维护，移除组件本身只清标签。[宿主清理](../../../raw/current/scripts/prefabs/hats.lua#L5099-L5138) [组件清理](../../../raw/current/scripts/components/closeinspector.lua#L10-L12)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentutil.lua（R-05）、stategraphs/SGwilson.lua（SB-17）与 SGwilson_client.lua 对照 build-26148036 更新报告逐条核验，锚点重定位 11 处、结论修订 0 处：LOOKAT 装备分派与后备、CLOSEINSPECTORUTIL 三段门禁（R-05 的 +287 行新增函数均在别处，IsValidTarget／IsValidPos／CanCloseInspect 逐行未变）及双端 closeinspect 动画入口均仅行号位移；hats.lua 三处证据锚点（工厂选择／返回 Prefab／调用）初核错位至 mask_charlie_boss 区块与结构行，经人工复核纠正到 roseglasses 实际选择 6946-6947／返回 Prefab 6983／注册调用 7938，另 4 处空行起止锚点收紧；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、公共实体／地点门禁、LOOKAT 装备分派及后备分支、玫瑰眼镜命名／工厂／动态技能挂载／装备与卸装、玩家检查执行分支、双端近距离检查动画入口；不代表残影／桥梁全部效果已审校。
- 结构检查：本页格式、链接、源码锚点检查通过；无新增完整 Lua 示例。
- 游戏运行：未运行；技能切换、卸装、不同检查目标／地点、多人同步和多个检查装备冲突待实测。
