---
title: "constructionsite"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在世界实体上记录分阶段建造的材料进度、当前建造者与完工回调。"
aliases: ["建造站点", "分阶段建造", "材料进度", "施工完成回调", "constructionsite"]
tags: [component, lifecycle, callback, building, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/constructionsite.lua", "components/constructionsite_replica.lua", "prefabs/constructionsite_classified.lua", "components/constructionbuilder.lua", "components/constructionplans.lua", "components/builder.lua", "components/deployable.lua", "prefabs/mermthrone.lua", "prefabs/collapsedchest.lua", "prefabs/charlie_hand.lua", "prefabs/multiplayer_portal.lua", "containers.lua", "recipes.lua", "standardcomponents.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# constructionsite

## 快速判断

- 职责与适用：挂在世界实体上的站点组件，按 `CONSTRUCTION_PLANS[inst.prefab]` 记录各材料数量、当前建造者与启用状态；完成后由宿主回调生成产物。[构造](../../../raw/current/scripts/components/constructionsite.lua#L13-L31) [材料](../../../raw/current/scripts/components/constructionsite.lua#L131-L150) [完成判定](../../../raw/current/scripts/components/constructionsite.lua#L203-L210)
- 优先使用：先注册 plan 与容器预制体，再用 `SetConstructionPrefab`／`SetOnConstructedFn` 接线；判定完成用 `IsComplete`，脚本补全用 `ForceCompletion`，清场用 `DropAllMaterials`。[接线](../../../raw/current/scripts/components/constructionsite.lua#L57-L71) [补全](../../../raw/current/scripts/components/constructionsite.lua#L212-L220)
- 重要副作用：`OnConstruct` 消费传入物品并清空 builder，`Disable` 先停掉建造者，builder／enabled 会同步到 replica。[消费](../../../raw/current/scripts/components/constructionsite.lua#L87-L121) [停用](../../../raw/current/scripts/components/constructionsite.lua#L48-L51)
- 修改边界：服务端组件；replica 只读且只同步 4 个材料槽。站点不负责动画、产物生成或放置动作，产物替换由宿主回调与 [[reference/components/constructionbuilder]] 分工。

### 与 constructionbuilder 的分工

- 站点拥有进度、完成判定和产物回调；constructionbuilder 是玩家侧执行态（打开容器、驱动 `constructing` 状态），两者通过站点的 `builder` 与建造者的 `constructionsite` 字段互绑。[站点绑定](../../../raw/current/scripts/components/constructionsite.lua#L73-L85) [执行态](../../../raw/current/scripts/components/constructionbuilder.lua#L42-L78)
- 停止建造由建造者发起，站点只在 `Disable`、组件移除或宿主显式调用时反向请求。[反向停止](../../../raw/current/scripts/components/constructionsite.lua#L33-L42)

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `ForceStopConstruction()` | 无参数 | 仅当建造者的 `constructionbuilder.constructionsite` 正是本实体时调用其 `StopConstruction`；`OnRemoveFromEntity` 指向它，但该钩子只在 `RemoveComponent` 时触发。[L33-L42](../../../raw/current/scripts/components/constructionsite.lua#L33-L42) |
| `Enable()`／`Disable()`／`IsEnabled()` | 无参数／boolean | `Disable` 先强制停止再置 false（经 net_bool 复制）；constructionbuilder 的开始与完成都检查它。[L44-L55](../../../raw/current/scripts/components/constructionsite.lua#L44-L55) [replica](../../../raw/current/scripts/components/constructionsite_replica.lua#L4) |
| `SetConstructionPrefab(prefab)` | 预制体名 | 建造者据此 `SpawnPrefab` 并打开；该预制体必须带 container 组件，否则开始会失败并清掉生成物。[L57-L59](../../../raw/current/scripts/components/constructionsite.lua#L57-L59) [使用处](../../../raw/current/scripts/components/constructionbuilder.lua#L58-L74) |
| `SetOnStartConstructionFn(fn)`／`SetOnStopConstructionFn(fn)`／`SetOnConstructedFn(fn)` | `fn(inst, doer)` | 分别为开始、停止、完工回调；当前 prefab 只用第三个，前两个已核验存在但无原版接入。[L61-L71](../../../raw/current/scripts/components/constructionsite.lua#L61-L71) |
| `OnStartConstruction(doer)`／`OnStopConstruction(doer)` | 建造者实体 | 设置／清空 `builder` 后调用对应回调；由 constructionbuilder 调用，不是玩家输入入口。[L73-L85](../../../raw/current/scripts/components/constructionsite.lua#L73-L85) |
| `OnConstruct(doer, items)` | 物品实体列表 | 逐件 `AddMaterial`；超量部分退回建造者玩家或落地，其余删除；随后清 builder、发 metrics、调完工回调。[L87-L121](../../../raw/current/scripts/components/constructionsite.lua#L87-L121) |
| `AddMaterial(prefab, num)`／`RemoveMaterial(prefab, num)` | 返回剩余／移除量 | 只接受本 prefab 的 plan 类型；超出该槽上限的数量原样返回；载入后无槽的旧材料也不会再累加。[L131-L167](../../../raw/current/scripts/components/constructionsite.lua#L131-L167) |
| `HasBuilder()`／`IsBuilder(guy)` | boolean | 前者只判断 `builder ~= nil`，后者再比较实体相等；builder 在开始后设置、停止或完工后清空，因此完成回调执行时两者均为 false。[L123-L129](../../../raw/current/scripts/components/constructionsite.lua#L123-L129) |
| `DropAllMaterials(drop_pos?)` | 可选 `Vector3` | 按现存材料逐类 `SpawnPrefab`、按堆叠上限分包并掉落，不检查材料来源；未掉落前先经 `RemoveMaterial` 清零。[L169-L193](../../../raw/current/scripts/components/constructionsite.lua#L169-L193) |
| `IsComplete()`／`ForceCompletion(doer)` | boolean／无返回 | plan 缺失时 `IsComplete` 因空循环返回 true；`ForceCompletion` 补齐材料后以空物品列表调用 `OnConstruct`，不走开始／停止回调。[L203-L220](../../../raw/current/scripts/components/constructionsite.lua#L203-L220) |
| `OnSave()`／`OnLoad(data)` | 存档表 | 仅保存 `materials` 数量；不保存 builder 与 enabled。载入按 plan 重建槽位，未知材料保留为无槽条目。[L222-L247](../../../raw/current/scripts/components/constructionsite.lua#L222-L247) |
| `materials`／`builder`／`enabled`／`constructionprefab` | 表／实体／boolean／字符串 | 初始为空、`enabled=true`、其余 nil；builder 在开始后非空，停止或完工后清空。[L13-L31](../../../raw/current/scripts/components/constructionsite.lua#L13-L31) |

### 状态与回调顺序

- 开始：constructionbuilder 绑定站点并打开容器后调用 `OnStartConstruction`，站点记录 builder 并复制给客户端。[开始](../../../raw/current/scripts/components/constructionsite.lua#L73-L78)
- 停止：建造者清理容器后调用 `OnStopConstruction`，站点清 builder 并回调；`Disable` 会先走这一步。[停止](../../../raw/current/scripts/components/constructionsite.lua#L80-L85)
- 完成：`OnConstruct` 在消费完物品后清 builder，再调 `onconstructedfn`，因此回调内不能依赖 builder 仍是完工玩家。[完工](../../../raw/current/scripts/components/constructionsite.lua#L113-L120)

## 典型接入

### 原版参照：mermthrone_construction 从放置到替换

- 目标与入口：配方 `mermthrone_construction`（boards 5／rope 5，带 placer、`TECH.SCIENCE_ONE`、`merm_builder`）由 [[reference/components/builder]] 放置，产物注册名 `mermthrone_construction`；随后由玩家在站点上分次投入 plan 材料。[recipes.lua:108](../../../raw/current/scripts/recipes.lua#L108 [注册名](../../../raw/current/scripts/prefabs/mermthrone.lua#L252-L253)
- 站点配置：宿主在 pristine 阶段预加 `constructionsite` 标签，服务端 `AddComponent` 后设 `construction_container` 与 `OnConstructed`；plan 为 kelp 20／pigskin 10／beefalowool 15。[宿主](../../../raw/current/scripts/prefabs/mermthrone.lua#L111-L126) [plan](../../../raw/current/scripts/recipes.lua#L1520-L1523)
- 资源、端与单位：容器 UI 使用 `ui_construction_4x1` 四槽，单槽材料数按 plan 件数计；动画与音效由站点 prefab 提供，全部结算在服务端，客户端只看 replica。[容器](../../../raw/current/scripts/containers.lua#L491-L541)
- 生效与撤销：材料存满后回调 `ReplacePrefab(inst, "mermthrone")` 并发 `onthronebuilt`；施工中锤击会 `ForceStopConstruction`，拆除前宿主显式 `DropAllMaterials`。[完工回调](../../../raw/current/scripts/prefabs/mermthrone.lua#L17-L28) [中断与拆除](../../../raw/current/scripts/prefabs/mermthrone.lua#L51-L73)
- 变体：`collapsed_treasurechest` 换用 `construction_rebuild_container` 与 `rebuildconstructionsite` 标签，在拾取／下沉前 `DropAllMaterials`；charlie_hand 用 `offerconstructionsite` 加自定义容器。[collapsedchest](../../../raw/current/scripts/prefabs/collapsedchest.lua#L7-L47) [charlie_hand](../../../raw/current/scripts/prefabs/charlie_hand.lua#L269-L270)

### 计划物品把已有实体改造成站点

- `multiplayer_portal_moonrock_constr_plans` 对已有 `multiplayer_portal` 使用 CONSTRUCT：`constructionplans:StartConstruction` 生成 `multiplayer_portal_moonrock_constr`、移除旧实体，并向新站点推 `onstartconstruction`。[改造流程](../../../raw/current/scripts/components/constructionplans.lua#L20-L36) [注册名](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L279-L283) [配方](../../../raw/current/scripts/recipes.lua#L833)
- 这条路径不经过 placer／builder 的 `onbuilt`；站点之后仍由 constructionbuilder 走同一套开始与完成流程，动画由站点自己监听事件处理。

### 与 builder、deployable 的边界

- `builder` 在放置时结算配方材料（如 mermthrone 的 boards 5／rope 5），站点 plan 是第二笔、可分批投入的材料；两者各自记账，站点不读 `builder`。[配方](../../../raw/current/scripts/recipes.lua#L108) [plan](../../../raw/current/scripts/recipes.lua#L1523) [builder 扣料](../../../raw/current/scripts/components/builder.lua#L1131-L1180)
- `deployable` 是物品的一次性 DEPLOY：动作取出一件、由 `ondeploy` 生成实体，没有站点、容器或后续进度；需要“已放置实体继续投料”才用站点。[deployable](../../../raw/current/scripts/components/deployable.lua#L124-L146) [DEPLOY 动作](../../../raw/current/scripts/actions.lua#L1622-L1631)

## 常见陷阱与限制

- **plan 缺失不是报错而是“永远无法正常完工”**：`IsComplete` 遍历空表返回 true，`AddMaterial` 拒绝一切材料，容器槽位也匹配不到计划类型；结果是投不进料，而任何直接读完成状态或调 `ForceCompletion` 的路径都会立即通过。[L131-L150](../../../raw/current/scripts/components/constructionsite.lua#L131-L150) [L203-L210](../../../raw/current/scripts/components/constructionsite.lua#L203-L210)
- **直接删站点/组件不返还材料**：`OnRemoveFromEntity` 只停建造者，不调用 `DropAllMaterials`，而且它只在 `RemoveComponent` 时触发；删除实体时靠建造者监听 `onremove` 停止，材料同样不返还。原版宿主在锤击或拾取路径自行返还，只有默认烧毁结构流程会代为返还。[组件](../../../raw/current/scripts/components/constructionsite.lua#L42) [宿主示例](../../../raw/current/scripts/prefabs/collapsedchest.lua#L7-L47) [烧毁流程](../../../raw/current/scripts/standardcomponents.lua#L116-L119)
- **读档不回绑建造者**：存档只有材料数量，`builder` 与 `enabled` 都不保存；载入后站点处于启用且无 builder 的状态，需要建造者重新 `StartConstruction`，禁用状态也会丢失。[L222-L247](../../../raw/current/scripts/components/constructionsite.lua#L222-L247)
- **未知材料会“卡”在无槽条目**：载入时不属于当前 plan 的材料保留但没有 `slot`，之后 `AddMaterial` 只返回新数量、不再累加，只能通过 `RemoveMaterial`／`DropAllMaterials` 取出。[L143-L149](../../../raw/current/scripts/components/constructionsite.lua#L143-L149) [L241-L245](../../../raw/current/scripts/components/constructionsite.lua#L241-L245)
- **客户端 replica 不代表全部状态**：只复制 4 个槽（计划最多 4 种材料），槽计数是 `net_byte`；客户端 `IsBuilder` 在没有服务端组件时退化为 `guy == ThePlayer`，不能当作权威身份判断。[classified](../../../raw/current/scripts/prefabs/constructionsite_classified.lua#L5-L60) [replica](../../../raw/current/scripts/components/constructionsite_replica.lua#L82-L96)
- **强制完成不等于正常建造**：`ForceCompletion` 不接受物品、不触发开始／停止回调，也不检查 `enabled`；用于脚本推进（如 wagpunk 竞技场补完 hermithouse）时不会扣除玩家材料。[L212-L220](../../../raw/current/scripts/components/constructionsite.lua#L212-L220)
- **完工回调里 builder 已清空**：`OnConstruct` 先把 `builder` 置 nil 再回调，需要完工玩家时应使用传入的 `doer`，不要再读 `self.builder`。[L113-L120](../../../raw/current/scripts/components/constructionsite.lua#L113-L120)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（recipes mermthrone plan 段 +9、actions DEPLOY 段 +3、charlie_hand 接线段 +24）；无结论级修订——constructionbuilder:107 的 `stopconstruction` 事件 26148036 起改用 `PushEventImmediate`（同步派发，配合 SB-17 construction 停止修复），本页"停止由建造者发起／反向停止"结论不变；constructionsite.lua 及 replica／classified／constructionplans／mermthrone／collapsedchest／multiplayer_portal／containers 引用行均未变。依据：actions-ca／player-hunks 地图＋变更清单命中 recipes.lua、actions.lua、prefabs/charlie_hand.lua、components/constructionbuilder.lua（单行事件改派发）。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与 tag、启停与回调、材料增减／超量返还、完成与强制完成、OnSave／OnLoad、replica／classified、mermthrone／collapsedchest／charlie_hand／portal 接入。源码中 12 个 prefab 文件共 13 处 `AddComponent("constructionsite")` 调用（工厂展开后覆盖 22 个注册 prefab）已逐一确认注册名与接线。
- 游戏运行：未运行；客户端槽位刷新、读档后重新绑定、多人同时施工和锤击／烧毁的实际时序仍需游戏验证。
- 结构检查：`validate_page.py` 通过（89 行，低于 200 行／16 KiB 上限；0 issue）；`validate_source_links.py --strict` 共 56 条源码引用、0 error／0 warning；本页无新增 Lua 示例，未生成临时 `.lua`。

相关：[[reference/components/constructionbuilder]]；[[reference/components/builder]]；[[reference/components/deployable]]；[[reference/components/container]]。
