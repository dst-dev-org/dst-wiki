---
title: "commander：士兵登记、通知与距离脱队"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "在指挥者上维护士兵集合，分发战斗建议与事件，并按实体休眠或距离剔除成员。"
aliases: [commander, 士兵指挥, 蜂后蜂卫登记, 范围内指挥士兵, gotcommander]
tags: [component, lifecycle, event, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/commander.lua, prefabs/beequeen.lua, prefabs/beeguard.lua, stategraphs/SGbeequeen.lua, components/combat.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
---

# commander：士兵登记、通知与距离脱队

## 快速判断

- 职责与适用：指挥者维护 `soldiers[实体]=true` 与数量，批量建议目标、清目标、唤醒／解冻或广播事件；不生成士兵，也不直接建立 leader／follower 关系。[commander.lua:1-12](../../../raw/current/scripts/components/commander.lua#L1-L12)、[commander.lua:59-125](../../../raw/current/scripts/components/commander.lua#L59-L125)
- 优先使用：参考蜂后显式添加蜂卫、成员消费 `gotcommander`／`lostcommander` 的接线；普通招募、忠诚与跟随归 [[reference/components/leader]]／[[reference/components/follower]]，不能只登记士兵就期望这些行为自动出现。[beeguard.lua:271-287](../../../raw/current/scripts/prefabs/beeguard.lua#L271-L287)、[beeguard.lua:384-437](../../../raw/current/scripts/prefabs/beeguard.lua#L384-L437)
- 重要副作用：增删成员会在指挥者上发 `soldierschanged`，在成员上发带 `{commander=指挥者}` 的得到／失去事件；启用距离跟踪后会剔除超距或实体休眠的士兵。[commander.lua:101-148](../../../raw/current/scripts/components/commander.lua#L101-L148)
- 修改边界：原版蜂后在服务端添加；组件自身未定义网络字段、存读或自动搜兵机制，宿主负责生成、重连及成员对命令的反应。[beequeen.lua:404-406](../../../raw/current/scripts/prefabs/beequeen.lua#L404-L406)、[beequeen.lua:452-453](../../../raw/current/scripts/prefabs/beequeen.lua#L452-L453)、[commander.lua:1-176](../../../raw/current/scripts/components/commander.lua#L1-L176)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、副作用与证据 |
|---|---|---|
| `AddSoldier(ent)`／`RemoveSoldier(ent)` | 成员实体；无显式返回 | add 拒绝已登记者及有 health 且已死者；监听成员 `death`／`onremove`。remove 撤监听、扣数、发失去事件，最后一名离开会停距离任务。[commander.lua:101-125](../../../raw/current/scripts/components/commander.lua#L101-L125) |
| `IsSoldier(ent)`／`GetNumSoldiers(prefab?)` | 布尔／数量 | 前者查表；不传 prefab 返回维护计数，传则按成员 `prefab` 精确匹配重数，不是 tag 过滤。[commander.lua:22-33](../../../raw/current/scripts/components/commander.lua#L22-L33)、[commander.lua:49-51](../../../raw/current/scripts/components/commander.lua#L49-L51) |
| `GetAllSoldiers(prefab?)`／`CollectSoldiers(tbl, prefab?)` | 新实体数组／向给定表追加 | 可按 prefab 过滤；GetAll 返回新数组，改该数组不更改登记表；Collect 不清空输入表。[commander.lua:35-47](../../../raw/current/scripts/components/commander.lua#L35-L47) |
| `ShareTargetToAllSoldiers(target)`／`DropAllSoldierTargets()` | 目标实体／无参数 | 仅处理有 combat 的成员，分别调 `SuggestTarget(target)`／`SetTarget(nil)`；建议不会替换成员现有目标，也不保证目标最终被接纳。[commander.lua:59-73](../../../raw/current/scripts/components/commander.lua#L59-L73)、[combat.lua:234-240](../../../raw/current/scripts/components/combat.lua#L234-L240) |
| `IsAnySoldierNotAlert()`／`AlertAllSoldiers()` | 前者命中返回 true，未命中无显式返回 | 检查 sleeper 睡眠或 freezable 冻结；Alert 解冻并唤醒这些成员，不会自动分配攻击对象。[commander.lua:75-93](../../../raw/current/scripts/components/commander.lua#L75-L93) |
| `PushEventToAllSoldiers(ev, data)` | 事件名与数据 | 向当前每个士兵发同一事件／数据；具体行为由成员消费者实现。[commander.lua:95-99](../../../raw/current/scripts/components/commander.lua#L95-L99) |
| `ForEachSoldier(fn, ...)` | 实际为 `fn(指挥者inst, ...)` | 每个士兵执行一次，但传入的是 `self.inst`，没有传当前士兵；不能当作常规逐成员回调，见限制。[commander.lua:53-57](../../../raw/current/scripts/components/commander.lua#L53-L57) |
| `SetTrackingDistance(dist)`、`trackingperiod` | 初始距离0、周期3秒 | 正距离尝试启动；非正距离停止任务；周期只在新建任务时读，不因赋值自动重排。[commander.lua:5-6](../../../raw/current/scripts/components/commander.lua#L5-L6)、[commander.lua:127-161](../../../raw/current/scripts/components/commander.lua#L127-L161) |

## 关键流程

### 登记事件与距离脱队

`AddSoldier` 先入表、注册死亡／移除监听、增加计数并尝试启动跟踪，再发指挥者的 `soldierschanged` 和成员的 `gotcommander`。`RemoveSoldier` 则先清表／监听及扣数，必要时停任务，随后发两端事件。组件不要求士兵带专用成员组件，也不保存独占指挥者反向引用；成员如何切换归属由事件接收方负责。[commander.lua:101-125](../../../raw/current/scripts/components/commander.lua#L101-L125)

跟踪只有在距离>0、有士兵、无现有任务且指挥者醒着时启动。每轮先收集“不在范围内或 `ent:IsAsleep()`”的成员，再逐个 Remove；这里是实体休眠检测，和 Alert 接口检查的 `sleeper:IsAsleep()` 睡眠不是同一调用。指挥者休眠停跟踪但保留集合，唤醒后再按条件启动；默认距离0不进行范围剔除。[commander.lua:75-93](../../../raw/current/scripts/components/commander.lua#L75-L93)、[commander.lua:127-164](../../../raw/current/scripts/components/commander.lua#L127-L164)

### 保存与清理边界

组件没有 `OnSave`／`OnLoad`，不保存士兵 GUID 或配置。原版蜂卫的 `OnLoadPostPass` 取回自己的 queen 后再次 `AddToArmy`；后者按玩家／非玩家分支处理 follower，最后才在 commander 中登记。这是宿主重连，不是本组件内建持久化。[commander.lua:1-176](../../../raw/current/scripts/components/commander.lua#L1-L176)、[beeguard.lua:271-287](../../../raw/current/scripts/prefabs/beeguard.lua#L271-L287)、[beeguard.lua:330-337](../../../raw/current/scripts/prefabs/beeguard.lua#L330-L337)

`OnRemoveFromEntity` 逐个 RemoveSoldier，因此沿移除组件路径会撤监听、通知失去指挥者，正常扣至零时停止距离任务。不要把它等同于所有宿主销毁路径：整实体 `Remove()` 检查的是 `OnRemoveEntity`，本组件没有实现该钩子；成员侧对指挥者消失的处理仍依赖各自宿主机制。[commander.lua:14-20](../../../raw/current/scripts/components/commander.lua#L14-L20)、[commander.lua:113-125](../../../raw/current/scripts/components/commander.lua#L113-L125)、[entityscript.lua:1730-1751](../../../raw/current/scripts/entityscript.lua#L1730-L1751)

## 典型接入

### 蜂后与蜂卫：生成、登记、归属反应分工

蜂后服务端添加 commander 并设置跟踪距离。`SGbeequeen` 的 `spawnguards` 时间线生成 `beeguard`，完成定位后调用其 `OnSpawnedGuard(queen)`；该方法进入出生状态并调用 queen 的 `AddSoldier`。数量与生成节奏在宿主 SG 中控制，组件本身没有最大士兵数。[beequeen.lua:452-453](../../../raw/current/scripts/prefabs/beequeen.lua#L452-L453)、[SGbeequeen.lua:508-533](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L508-L533)、[beeguard.lua:339-344](../../../raw/current/scripts/prefabs/beeguard.lua#L339-L344)、[beeguard.lua:579-587](../../../raw/current/scripts/prefabs/beeguard.lua#L579-L587)

蜂卫监听得到／失去事件：得到时按玩家或蜂后处理友好关系／queen 追踪，并重算蜂卫分布；失去时清 queen 与偏移记录、取消聚焦并启动寻找玩家 queen 的任务。蜂后受击还会把攻击者建议给全体士兵。这些归属、布局与战斗反应是 prefab 消费者实现，不能推广为所有 AddSoldier 对象的自动效果。[beeguard.lua:384-437](../../../raw/current/scripts/prefabs/beeguard.lua#L384-L437)、[beeguard.lua:559-560](../../../raw/current/scripts/prefabs/beeguard.lua#L559-L560)、[beequeen.lua:199-209](../../../raw/current/scripts/prefabs/beequeen.lua#L199-L209)

## 常见陷阱与限制

- **逐成员回调参数错误**：当前 `ForEachSoldier` 循环变量没有传给 fn，实际重复传指挥者。需要对成员执行自定义操作时，遍历 `GetAllSoldiers()` 的返回数组，并仍用 Add／Remove 维护真实集合。[commander.lua:43-57](../../../raw/current/scripts/components/commander.lua#L43-L57)
- **登记自动迁移旧主人**：组件未检查其他 commander，也不写 follower；直接把同一实体加入多个指挥者不会由本组件自动清旧表。原版蜂卫有专用事件消费者，不代表任意实体具备这一协议。[commander.lua:101-111](../../../raw/current/scripts/components/commander.lua#L101-L111)、[beeguard.lua:384-437](../../../raw/current/scripts/prefabs/beeguard.lua#L384-L437)
- **动态改周期即刻重排**：已存在任务时 StartTrackingDistance 不重建；改 trackingperiod 需停启任务才读新周期。只改正的 trackingdist 则由下一轮检查读取新距离，不立即逐个脱队。[commander.lua:127-161](../../../raw/current/scripts/components/commander.lua#L127-L161)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、combat 目标建议门禁、蜂后服务端配置／受击、蜂后 SG 一段完整生成时间线回调、蜂卫登记及得到／失去／读档消费者，并区分 EntityScript 的实体移除钩子。
- 游戏运行：未运行；未实测回调重入、跨指挥者登记、距离休眠剔除和读档重连；未审阅其他指挥者宿主或所有战斗／跟随行为。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/combat]]、[[reference/components/leader]]、[[reference/components/follower]]。
