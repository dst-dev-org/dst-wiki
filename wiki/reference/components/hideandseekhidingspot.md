---
title: "hideandseekhidingspot"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "捉迷藏藏身点：运行时把 kitcoon 藏进任意实体（假身体跟随符号），被翻找/破坏等互动即放人并自移除组件。"
aliases: ["捉迷藏藏身点", "藏猫点", "hideandseekhidingspot"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/hideandseekhidingspot.lua", "components/hideandseekhider.lua", "components/hideandseekgame.lua", "components/specialeventsetup.lua", "prefabs/kitcoonden.lua", "prefabs/ticoon.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hideandseekhidingspot

## 快速判断

- 职责与适用：让任意实体临时成为 kitcoon 捉迷藏的藏身处；组件由藏匿者（hideandseekhider）运行时 `AddComponent`，翻出或中止即 `RemoveComponent`，prefab 不预挂。[运行时挂载](../../../raw/current/scripts/components/hideandseekhider.lua#L42-L43)
- 优先使用：藏入 `HideInSpot(hider)`，预登记 `SetHider(hider)`，翻出 `SearchHidingSpot(doer)`，无声中止 `Abort()`。[HideInSpot](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L58-L83)
- 重要副作用：藏入后 hider 脱离场景（parent 到藏身点、`RemoveFromScene`），生成 `_hiding_prop` 假身跟随原实体符号；假身上监听 `picked`/`worked`/`onignite`/`onopen`/`onactivated`/`onpickup` 六类互动即翻出。[假身与监听](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L65-L82)
- 修改边界：组件服务端运行；`TUNING.KITCOON_HIDING_OFFSET` 没配的 prefab 不做符号跟随（假身直接叠在原点）；`OnSave` 带 `add_component_if_missing` 让读档自恢复。[存档](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L96-L107)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetHider(hider)` | 无返回 | 只登记将到未到的藏匿者并挂其 `onremove`（移除时自动拆组件）；已有人则忽略。[L50-L56](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L50-L56) |
| `HideInSpot(hider)` | 无返回 | 登记后把 hider 挂进藏身点、生成 `hider._hiding_prop` 假身并按 `TUNING.KITCOON_HIDING_OFFSET[self.inst.prefab]` 跟随符号；翻出监听挂在假身上以便假身清理时自动解绑。[L58-L83](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L58-L83) |
| `SearchHidingSpot(doer)` | 无返回 | 放出 hider（记 `finder`）并 `RemoveComponent`；`HIDEANSEEK_FIND` 动作入口，doer 只接受玩家。[L85-L88](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L85-L88)、[动作](../../../raw/current/scripts/actions.lua#L4263-L4272) |
| `Abort()` | 无返回 | 不记 finder 直接放出并拆组件；游戏结束/超时的清理路径。[L90-L93](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L90-L93) |
| `_ReleaesHider(doer)`（内部） | 无返回 | hider 回到场景并定位到藏身点坐标，调 `hideandseekhider:Found(doer)`；假身未生成时跳过坐标复位。[L18-L32](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L18-L32) |
| `finder` | 玩家或 nil | 最近翻出者；组件拆除时随 `onhidingspotremoved` 事件带出。[L43](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L43) |

## 生命周期

- 登记期：`hideandseekhider:GoHide` 对目标 `AddComponent` + `SetHider`，kitcoon 可以先跑去藏身点再真正藏入（超时未到由 hider 侧处理）。[GoHide](../../../raw/current/scripts/components/hideandseekhider.lua#L38-L60)
- 藏匿期：hider 离场、假身替换；互动翻出（`evict_fn` 从 `data.picker/doer/worker/owner` 提取玩家 finder）。[evict_fn](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L4-L11)
- 拆除期：`OnRemoveFromEntity` 放人、退订世界事件 `ms_collecthiddenkitcoons`（ticoon 收猫时用于枚举藏身点）、推 `onhidingspotremoved` 并移除假身。[L38-L48](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L38-L48)
- 读档：`OnSave` 若正藏有 kitcoon 则连同其存档记录一起保存，`OnLoad` 重建后立即 `HideInSpot`；未藏人时也返回 `{add_component_if_missing = true}` 维持组件存在。[OnLoad](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L109-L116)

## 典型接入

### 原版宿主：kitcoon 捉迷藏活动

- 候选筛选：kitcoonden 与 YOT 猫活动都在"无本组件 + 地图可通行 + 距离达标"的实体里挑藏身点（树丛、构造物等），然后经 `hideandseekhider:GoHide` 动态挂组件。[den 挑选](../../../raw/current/scripts/prefabs/kitcoonden.lua#L164-L170)、[活动布置](../../../raw/current/scripts/components/specialeventsetup.lua#L121-L126)
- 翻找入口：右键动作 `HIDEANSEEK_FIND`（componentactions 注入）；`hideandseekgame` 在轮次推进/收尾时对藏身点 `Abort()`。[动作过滤](../../../raw/current/scripts/componentactions.lua#L425-L428)、[游戏清理](../../../raw/current/scripts/components/hideandseekgame.lua#L52-L63)
- 活动汇总：`ms_collecthiddenkitcoons` 世界事件把带本组件的实体收进清单，供 ticoon（猫型 NPC）巡场找猫。[登记](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L14-L15)、[收集](../../../raw/current/scripts/prefabs/ticoon.lua#L95)

### 模组：新增可藏身的建筑

- 前提与选择：服务端实体即可；组件按需自挂自拆，prefab 无需预挂。若想被 kitcoonden 选中，需命中其 `hiding_spot_*` tag 过滤（活动侧配置）。
- 生效与撤销：生命周期完全由 hideandseekhider 与翻找动作管理；手工 `SetHider` 后忘记 `HideInSpot` 只会卡在登记态，kitcoon 不离场。[SetHider](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L50-L56)

## 常见陷阱与限制

- `_ReleaesHider` 的坐标复位包在 `hiding_prop ~= nil` 判断内：只登记未真正藏入时（假身未生成），kitcoon 回场景后不校正位置，可能留在原地去途中的坐标。[L20-L25](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L20-L25)
- 翻出 finder 提取顺序是 `picker → doer → worker → owner`，全部缺失或非玩家则 finder 为 nil，成就/计数类逻辑拿不到人。[L5-L8](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L5-L8)
- 假身 build 直接抄 hider 当前 `AnimState:GetBuild()`：换肤/形态变化后假身不会跟随，重藏才刷新。[L67](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L67)
- 拆组件即"翻出"语义：`inst:RemoveComponent("hideandseekhidingspot")` 外部强拆不会走 `_ReleaesHider`，kitcoon 会留在离场状态，必须用 `SearchHidingSpot`/`Abort`。[L87](../../../raw/current/scripts/components/hideandseekhidingspot.lua#L87)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文（含假身生命周期与存读）；核对 hideandseekhider 的动态挂载、动作与过滤、hideandseekgame/specialeventsetup/kitcoonden/ticoon 的全部消费点；未覆盖假身动画资源细节与活动计分。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过（见批记录）。

相关：[[reference/components/hideandseeker]]。
