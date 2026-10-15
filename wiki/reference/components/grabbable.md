---
title: grabbable
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "识别已废弃的徒手抓取回调，迁移到 inventoryitem 的标签拾取覆盖，并保留 NET 兼容边界。"
aliases: ["grabbable", "徒手抓取", "抓兔子", "抓蚊子", "grabbableoverridetag"]
tags: [component, item, actions, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/grabbable.lua", "componentactions.lua", "actions.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "prefabs/mosquito.lua", "prefabs/rabbit.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# grabbable

## 快速判断

- 职责与适用：兼容旧 `CanGrab(doer)` 回调；源码已明确废弃，本页用于识别旧代码，新物品拾取优先用 [[reference/components/inventoryitem]]。[grabbable.lua:1](../../../raw/current/scripts/components/grabbable.lua#L1
- 优先使用：特定标签角色可拾取平时不可拾取的物品，配置 `inventoryitem.grabbableoverridetag`；它走 PICKUP，不是旧 NET 回调的等价替换。[actions.lua:894](../../../raw/current/scripts/actions.lua#L894)、[NET:1859](../../../raw/current/scripts/actions.lua#L1859)
- 重要副作用：构造加 `grabbable` 标签，移组件删标签；当前 SCENE 收集器为空，单加组件或标签不会由该收集器生成抓取动作。[grabbable.lua:5](../../../raw/current/scripts/components/grabbable.lua#L5)、[:14](../../../raw/current/scripts/components/grabbable.lua#L14)、[componentactions.lua:393](../../../raw/current/scripts/componentactions.lua#L393
- 修改边界：旧 NET 执行仍检查本组件，不能据 deprecated 当成完全无效；无网工具的执行者在回调不通过时被拒绝。[actions.lua:1859](../../../raw/current/scripts/actions.lua#L1859

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `SetCanGrabFn(fn)` | 回调 `(inst, doer)`，或 nil 清除 | 单回调赋值替换；没有默认可抓取回调。[grabbable.lua:18](../../../raw/current/scripts/components/grabbable.lua#L18 |
| `CanGrab(doer)` | 返回回调原值；无回调返回 false | 不处理入栏、捕捉、距离或动作生成；调用方解释返回值。[grabbable.lua:22](../../../raw/current/scripts/components/grabbable.lua#L22 |
| `OnRemoveFromEntity()` | 移组件钩子 | 仅删除 `grabbable` 标签；本组件没有自定义存档、更新循环或网络变量。[完整定义:5](../../../raw/current/scripts/components/grabbable.lua#L5) |

`ACTIONS.NET` 先要求目标有可工作的 `workable`、工作动作是 NET、目标未死亡。仅在工具不存在或没有 `NET_tool` 标签、且目标有 grabbable 时检查 `CanGrab`；通过后调用 `WorkedBy(doer)`。有网工具可绕过该回调，故不能用它实现通用捕捉权限。[actions.lua:1859](../../../raw/current/scripts/actions.lua#L1859)、[回调门禁:1866](../../../raw/current/scripts/actions.lua#L1866)；工作消耗见 [[reference/components/workable]]。

## 典型接入

### 让指定标签的角色拾取活体物品

不新挂 grabbable。沿原版服务端工厂配置已有 inventoryitem 的 `canbepickedup = false` 与 `grabbableoverridetag`，允许具有该标签的 doer 通过拾取权限分支；正常拾取流程仍见 [[reference/components/inventoryitem#快速判断]]。

| 原版模式 | 配置与证据 | 选择边界 |
|---|---|---|
| 蚊子 | `mosquitograbber`；[mosquito.lua:203](../../../raw/current/scripts/prefabs/mosquito.lua#L203)、[服务端分界:181](../../../raw/current/scripts/prefabs/mosquito.lua#L181) | 另有 `canbepickedupalive = true`，不把非玩家活体拾取权限误算成标签效果。 |
| 兔子 | `rabbitdisguise`；[rabbit.lua:386](../../../raw/current/scripts/prefabs/rabbit.lua#L386)、[服务端分界:370](../../../raw/current/scripts/prefabs/rabbit.lua#L370) | 标签赋在目标物品的策略字段，匹配的是执行者的标签。 |

字段赋值通过属性回调同步至 inventoryitem replica；客户端 `CanBePickedUp(doer)` 检查同一标签，SCENE 再决定是否提供 PICKUP，服务端动作复核。[inventoryitem.lua:37](../../../raw/current/scripts/components/inventoryitem.lua#L37)、[inventoryitem_replica.lua:88](../../../raw/current/scripts/components/inventoryitem_replica.lua#L88)、[componentactions.lua:470](../../../raw/current/scripts/componentactions.lua#L470)、[actions.lua:901](../../../raw/current/scripts/actions.lua#L901

## 常见陷阱与限制

- **将 override 当独占白名单**：PICKUP 用 `canbepickedup`、非玩家活体权限与标签分支作 OR；若普通权限仍允许，缺标签也可能可拾取。按实际目标核对其他分支。[actions.lua:895](../../../raw/current/scripts/actions.lua#L895
- **将标签当全部动作限制的豁免**：override 通过后仍有 SCENE 的着火／catchable／物品栏等条件，以及服务端 INLIMBO、燃烧、抛射、占用与角色限制（26148036 核验：燃烧排除对带 `torch` 标签物放宽，服务端与客户端 SCENE 两侧同步）；它不保证动作出现或入栏成功。[componentactions.lua:474](../../../raw/current/scripts/componentactions.lua#L474)、[actions.lua:904](../../../raw/current/scripts/actions.lua#L904
- **直接把任意 CanGrabFn 换成一个标签**：旧回调能按 `(inst, doer)` 动态判断，替代字段只比较一个标签且走拾取路径。若旧逻辑依赖 NET 工作与回调，须另核验动作流程，不能批量机械替换。[grabbable.lua:22](../../../raw/current/scripts/components/grabbable.lua#L22)、[actions.lua:1866](../../../raw/current/scripts/actions.lua#L1866

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、components/inventoryitem.lua、prefabs/rabbit.lua）对照 build-26148036 更新报告条目 R-01／R-04／C-19／C-27 逐条核验，锚点重定位 10 处、结论修订 1 处（燃烧 torch 标签物可拾取的放宽例外，见正文"26148036 核验"标注）；rabbit.lua 仅新增 `nightmarecorruptable` 标签（+1 行移）；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整审阅 30 行定义，核对废弃收集器、NET 消费者、替代拾取分支／replica 及兔子、蚊子工厂。当前 scripts 下精确字面 `AddComponent("grabbable")`／单引号形式未命中；不据此排除动态挂载或外部模组使用。
- 结构检查：本页元数据、大小、Wiki／Markdown 链接与源码锚点检查通过；未提供完整 Lua 示例，导航登记由本批整合检查。
- 游戏运行：未运行；旧模组兼容、实际抓取交互与多人表现未验证。
