---
title: "leaderrollcall"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "以持有者或自身为主人定期招募合格猪人／鱼人、补充已有随从忠诚，并可共用扫描照料农作物。"
aliases: ["点名招募随从", "独奏乐器招募", "范围招募猪人鱼人", "leaderrollcall"]
tags: [component, lifecycle, callback, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/leaderrollcall.lua", "components/leader.lua", "components/follower.lua", "componentutil.lua", "entityscript.lua", "prefabs/onemanband.lua", "prefabs/merm.lua", "wx78_moduledefs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# leaderrollcall

## 快速判断

- 职责与适用：服务端点名循环，围绕选定主人扫描 `pig`／`merm`／`farm_plant`，招募资格与归属分别交给 follower／leader；不是任意生物的通用招募器。[leaderrollcall.lua:103](../../../raw/current/scripts/components/leaderrollcall.lua#L103 [leaderrollcall.lua:116](../../../raw/current/scripts/components/leaderrollcall.lua#L116
- 优先使用：以 `Enable`／`Disable` 管理更新；物品先加 inventoryitem 再加本组件，才会建立持有者追踪。原版两种组装见[典型接入](#典型接入)。[leaderrollcall.lua:14](../../../raw/current/scripts/components/leaderrollcall.lua#L14 [leaderrollcall.lua:47](../../../raw/current/scripts/components/leaderrollcall.lua#L47
- 重要副作用：每次点名可添加普通随从，并对全部符合标签与资格的已有普通随从调用 `AddLoyaltyTime(5)`；后者不重新检查距离。[leaderrollcall.lua:119](../../../raw/current/scripts/components/leaderrollcall.lua#L119
- 修改边界：`maxfollowers` 只限制新增，不删除超额旧随从；禁用停止周期更新，不解除已经建立的归属；内部 `DoRollCall` 自身没有 enable 门禁。[leaderrollcall.lua:57](../../../raw/current/scripts/components/leaderrollcall.lua#L57 [leaderrollcall.lua:119](../../../raw/current/scripts/components/leaderrollcall.lua#L119

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `Enable(inittime?)`／`Disable()`／`IsEnabled()` | 首次剩余时间（秒）／无参数／布尔 | 默认未启用；Enable 开始组件更新，累积量设 `update_time - inittime`，省略则为 0。重复调用会重置累积；Disable 清累积并停止更新，更新物品持有者的点名来源标记。[leaderrollcall.lua:47](../../../raw/current/scripts/components/leaderrollcall.lua#L47 |
| `SetUpdateTime(time)` | 周期间隔秒数；默认 1 | OnUpdate 累加 dt，达到阈值才执行一次并清零，超出部分不保留；不是补齐遗漏次数的循环。若用 inittime，应先设置周期再 Enable。[leaderrollcall.lua:8](../../../raw/current/scripts/components/leaderrollcall.lua#L8 [leaderrollcall.lua:95](../../../raw/current/scripts/components/leaderrollcall.lua#L95 [leaderrollcall.lua:157](../../../raw/current/scripts/components/leaderrollcall.lua#L157 |
| `SetRadius(radius)`／`GetRadius()` | 世界距离 | 默认 `TUNING.ONEMANBAND_RANGE`；只控制主人周围候选扫描，不限制既有随从补忠诚的距离。[leaderrollcall.lua:5](../../../raw/current/scripts/components/leaderrollcall.lua#L5 [leaderrollcall.lua:71](../../../raw/current/scripts/components/leaderrollcall.lua#L71 [leaderrollcall.lua:127](../../../raw/current/scripts/components/leaderrollcall.lua#L127 |
| `SetMaxFollowers(max)`／`GetMaxFollowers()` | 数量 | 默认 `TUNING.ONEMANBAND_MAXFOLLOWERS`；比较主人普通随从总数，不只猪人／鱼人数，也不计 itemfollowers。[leaderrollcall.lua:6](../../../raw/current/scripts/components/leaderrollcall.lua#L6 [leaderrollcall.lua:79](../../../raw/current/scripts/components/leaderrollcall.lua#L79 [leader.lua:65](../../../raw/current/scripts/components/leader.lua#L65 |
| `SetCanTendFarmPlant(boolval)`／`GetCanTendFarmPlant()` | 真值或 nil；默认 nil | 开启后扫描中无 follower、但有 farmplanttendable 的候选调用 `TendTo(leader)`；不带本页扫描标签的实体不会进入该分支。[leaderrollcall.lua:87](../../../raw/current/scripts/components/leaderrollcall.lua#L87 [leaderrollcall.lua:128](../../../raw/current/scripts/components/leaderrollcall.lua#L128 |
| `SetOnRollCallFn(fn)` | `fn(inst)` | 有有效主人且完成点名流程后调用，即使没招到新随从；无主人直接返回，不调用。[leaderrollcall.lua:99](../../../raw/current/scripts/components/leaderrollcall.lua#L99 [leaderrollcall.lua:119](../../../raw/current/scripts/components/leaderrollcall.lua#L119 |
| `GetLeader()`／`IsValidLeader(leader)` | 主人实体或 nil／布尔 | 优先有 leader 组件的 itemsource_owner，再选有 leader 的自身；有效条件只查该组件，不含存活、玩家或装备判定。[leaderrollcall.lua:103](../../../raw/current/scripts/components/leaderrollcall.lua#L103 |

## 关键流程

### 点名扫描与忠诚补充

1. 以 `GetLeader()` 的坐标为圆心扫描，至少有 `pig`、`merm`、`farm_plant` 之一，排除 `werepig`、`player`；使用返回的遍历顺序，本组件不另排序。[leaderrollcall.lua:116](../../../raw/current/scripts/components/leaderrollcall.lua#L116
2. 候选有 follower 时，只有普通随从总数小于上限、候选无主人、尚非该 leader 的随从且 `CanBeRollCalled(leader)` 通过才 `AddFollower`。已有主人不会被抢走；达到上限且未启用照料时，会在遇到有 follower 的候选后提前结束扫描。[leaderrollcall.lua:128](../../../raw/current/scripts/components/leaderrollcall.lua#L128
3. 照料走上述分支的 `elseif`：候选必须没有 follower，且启用照料、有 farmplanttendable。启用照料后，即便随从已满也继续扫描，调用 `TendTo(leader)`。[leaderrollcall.lua:137](../../../raw/current/scripts/components/leaderrollcall.lua#L137
4. 随后另遍历 `leader.followers`，具有上述任一标签且 `CanBeRollCalled` 通过就输入 5 秒忠诚。这里不再检查半径、招募上限或扫描排除标签。实际增加仍受 follower 的 neverexpire、主人 loyaltyeffectiveness 与 maxfollowtime 限制，不能将输入 5 秒当成必然净增 5 秒。[leaderrollcall.lua:146](../../../raw/current/scripts/components/leaderrollcall.lua#L146 [follower.lua:365](../../../raw/current/scripts/components/follower.lua#L365

`CanBeRollCalled` 会先拒绝“主人自身具有本组件但未启用”的情形，再调用随从的 `canberollcalledfn`，未设置回调则放行。鱼人 Prefab 就设置了该回调，按月亮／暗影追随技能、鱼人守卫身份或鱼人王存在等条件判断；不能只按扫描标签认定所有鱼人都可招募。[follower.lua:201](../../../raw/current/scripts/components/follower.lua#L201 [merm.lua:935](../../../raw/current/scripts/prefabs/merm.lua#L935 [merm.lua:1095](../../../raw/current/scripts/prefabs/merm.lua#L1095

### 物品持有者、移除与保存边界

构造时已有 inventoryitem 才调用 `MakeComponentAnInventoryItemSource`。该工具根据入包／掉落及容器相关事件更新 `itemsource_owner`，容器物品使用 `GetGrandOwner()`；换主人时先移除旧来源，再在启用状态下给新主人的 leader 标记点名来源。掉地清持有者，但不自动 Disable，`GetLeader()` 可回退到物品自身。[leaderrollcall.lua:14](../../../raw/current/scripts/components/leaderrollcall.lua#L14 [leaderrollcall.lua:30](../../../raw/current/scripts/components/leaderrollcall.lua#L30 [componentutil.lua:1777](../../../raw/current/scripts/componentutil.lua#L1777

leader 的 `IsRollCalling()` 将自身启用状态与外部来源合并；该标记不是本组件的点名调度器。移除本组件时清物品来源监听及原主人的来源，宿主 `RemoveComponent` 先停止组件更新。移除也不遍历解除已招募的随从。[leader.lua:108](../../../raw/current/scripts/components/leader.lua#L108 [leaderrollcall.lua:20](../../../raw/current/scripts/components/leaderrollcall.lua#L20 [componentutil.lua:1842](../../../raw/current/scripts/componentutil.lua#L1842 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

本组件没有 `OnSave`／`OnLoad`，启用状态、参数与回调由宿主初始化或技能／装备流程重施；已建立关系的保存是 leader／follower 的职责，不能把点名器当作随从名单存档。[leaderrollcall.lua:1](../../../raw/current/scripts/components/leaderrollcall.lua#L1 [[reference/components/leader#关系解除与恢复]]

## 典型接入

### 原版独奏乐器：装备物品与地面物品共用点名

`Prefab("onemanband", fn, assets)` 的服务端分支先加 inventoryitem、自身 leader，再加 leaderrollcall 并开启农作物照料。正常装备 Enable 并消耗燃料，卸下／装备到模型 Disable；闹鬼也可启用地面物品自身的点名，结束闹鬼时禁用，收入物品栏会停止闹鬼。可据此判断“用持有者 leader”与“用自身 leader”的组装差别，无需另造最小 demo。[onemanband.lua:73](../../../raw/current/scripts/prefabs/onemanband.lua#L73 [onemanband.lua:98](../../../raw/current/scripts/prefabs/onemanband.lua#L98 [onemanband.lua:12](../../../raw/current/scripts/prefabs/onemanband.lua#L12 [onemanband.lua:32](../../../raw/current/scripts/prefabs/onemanband.lua#L32 [onemanband.lua:113](../../../raw/current/scripts/prefabs/onemanband.lua#L113

### 原版 WX 音乐技能：组件挂在人物自身

音乐模块以 `wx78_circuitry_betabuffs_1` 注册技能回调。`music_skill_activate` 取消原来的照料任务，必要时给 WX 添加本组件，先设置更新周期，再 `Enable(1)`，配置照料、点名特效、按模块数变化的范围与上限。技能不再激活或模块数为零时移除组件；仍有音乐模块时可恢复原照料任务。撤销条件由这一宿主流程负责，不由组件感知模块／技能。[wx78_moduledefs.lua:979](../../../raw/current/scripts/wx78_moduledefs.lua#L979 [wx78_moduledefs.lua:998](../../../raw/current/scripts/wx78_moduledefs.lua#L998 [wx78_moduledefs.lua:1039](../../../raw/current/scripts/wx78_moduledefs.lua#L1039

## 常见陷阱与限制

- **Disable 或缩小上限就解散多余随从**：这两个入口均无解除循环；禁用只停止后续周期补忠诚，普通随从仍按自身忠诚／归属规则处理。[leaderrollcall.lua:57](../../../raw/current/scripts/components/leaderrollcall.lua#L57 [leaderrollcall.lua:79](../../../raw/current/scripts/components/leaderrollcall.lua#L79
- **把 enable 当所有调用的硬门禁**：正常入口通过 Enable 开更新、Disable 停更新；内部 `DoRollCall()` 本身不检查 enable，直接调用不能保证禁用即无行为，且另受 follower 资格检查影响。[leaderrollcall.lua:119](../../../raw/current/scripts/components/leaderrollcall.lua#L119 [follower.lua:205](../../../raw/current/scripts/components/follower.lua#L205
- **有照料标签又有 follower 就必然被照料**：follower 分支先匹配，照料仅在 `elseif` 中；该组件只负责调用 TendTo，农作物的照料效果由目标组件决定。[leaderrollcall.lua:129](../../../raw/current/scripts/components/leaderrollcall.lua#L129

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部接口与分派、物品来源工具、leader 来源／数量及 follower 资格／忠诚消费，独奏乐器和 WX 音乐技能两种组装、鱼人资格回调与组件移除顺序。未展开所有生物行为树、TendTo 内部效果或 WX 全部技能回调基础设施。
- 结构检查：本页元数据、章节、大小及站内链接检查通过；源码引用 strict 检查通过。未含完整 Lua 示例，未检查索引登记或记录哈希基线。
- 游戏运行：未运行；装备／转交／掉落／闹鬼、招募先后与上限、远距补忠诚、鱼人不同形态、技能切换与读档重施待实测。

相关：[[reference/components/leader]]；[[reference/components/follower]]。
