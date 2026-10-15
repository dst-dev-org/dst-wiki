---
title: "constructionbuilder"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "玩家侧的建造执行态：绑定站点、打开材料容器并驱动建造状态机。"
aliases: ["玩家建造状态", "施工动作", "建造中断", "施工容器", "constructionbuilder"]
tags: [component, lifecycle, state-machine, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/constructionbuilder.lua", "components/constructionsite.lua", "components/constructionbuilderuidata.lua", "components/builder.lua", "prefabs/player_common.lua", "stategraphs/SGwilson.lua", "actions.lua", "containers.lua", "prefabs/mermthrone.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# constructionbuilder

## 快速判断

- 职责与适用：挂在玩家身上的服务端执行态，只保存当前站点与临时材料容器，并把玩家送进 `constructing` 状态；材料进度与产物归站点，不归本组件。[构造](../../../raw/current/scripts/components/constructionbuilder.lua#L12-L26) [开始](../../../raw/current/scripts/components/constructionbuilder.lua#L42-L78)
- 优先使用：通过 CONSTRUCT／STOPCONSTRUCTION／APPLYCONSTRUCTION 动作驱动；代码里判断、打断用 `CanStartConstruction`／`IsConstructing`／`StopConstruction`，正常完成必须经状态机 `FinishConstruction` 再 `OnFinishConstruction`。[动作入口](../../../raw/current/scripts/actions.lua#L4426-L4463)
- 重要副作用：开始时生成站点指定的容器、对玩家 `Open`、挂到玩家实体且 `persists=false`；停止时把容器内物品逐格还回背包（无背包则全部倒出）并向玩家推 `stopconstruction`。[L58-L108](../../../raw/current/scripts/components/constructionbuilder.lua#L58-L108)
- 修改边界：仅服务端；客户端没有本组件，依靠 [constructionbuilderuidata](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L5-L9) 的两个 net_entity 与站点 replica 显示进度。宿主只有 `player_common`，需要 `sg` 与 `inventory`；uidata 在 pristine 前创建，builder 在其后的 master 段创建，属性同步时依赖前者已存在。[顺序](../../../raw/current/scripts/prefabs/player_common.lua#L2512-L2721)

### 与 constructionsite 的分工

- 站点持有材料／完成判定与产物回调；本组件只持有“当前站点 + 容器 + 状态机位置”。开始与停止由本组件调用站点的 `OnStartConstruction`／`OnStopConstruction`，完成由本组件调用站点 `OnConstruct`。[站点侧](../../../raw/current/scripts/components/constructionsite.lua#L73-L121)
- 站点不保存 builder，本组件也不保存站点；两者是运行时绑定，读档后由下一次开始动作重新建立。[存档](../../../raw/current/scripts/components/constructionbuilder.lua#L149-L176)

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `CanStartConstruction()` | boolean | 当前 sg 状态必须是 `construct`，且没有已绑定的容器和站点；不检查目标。[L28-L32](../../../raw/current/scripts/components/constructionbuilder.lua#L28-L32) |
| `StartConstruction(target)` | `target, reason?` | 目标需有站点组件、`constructionprefab` 且启用；通过后先 `StopConstruction`，再拒绝 `INUSE`／`BURNT`。[L42-L57](../../../raw/current/scripts/components/constructionbuilder.lua#L42-L57) |
| 开始成功条件 | 容器存在、可被玩家打开 | `SpawnPrefab` 后 `container:Open(inst)`；`IsOpenedBy` 成立才绑定站点、`SetParent`、`persists=false`、`GoToState("constructing")`；否则移除容器并返回 false。[L58-L78](../../../raw/current/scripts/components/constructionbuilder.lua#L58-L78) |
| `StopConstruction()` | 无参数 | 逐格把容器物品还回背包并清 `prevcontainer`／`prevslot`；移除容器；解绑站点并调用其 `OnStopConstruction`；最后无条件推 `stopconstruction`。[L80-L108](../../../raw/current/scripts/components/constructionbuilder.lua#L80-L108) |
| `FinishConstruction()` | boolean? | 需容器非空、站点启用、状态为 `constructing`；关闭容器、标记 `statemem.constructing` 后切到 `construct_pst`，让退出 `constructing` 时不触发停止。[L110-L125](../../../raw/current/scripts/components/constructionbuilder.lua#L110-L125) |
| `OnFinishConstruction()` | 无返回 | 从容器收集物品（不先移除）、解绑 onremove、调用站点 `OnConstruct(doer, items)`，再删容器并清站点；由 `construct_pst` 超时调用。[L127-L147](../../../raw/current/scripts/components/constructionbuilder.lua#L127-L147) |
| `IsConstructing(inst)`／`IsConstructingAny()` | boolean | 前者比较容器实体，后者只判断“有容器且状态名为 `constructing`”；APPLYCONSTRUCTION 用前者确认目标。[L34-L40](../../../raw/current/scripts/components/constructionbuilder.lua#L34-L40) |
| `OnSave()`／`OnLoad(data)` | 存档表 | 仅当容器非空时保存容器的 save record；载入后生成、挂到玩家并 `persists=false`，不重新打开、不重绑站点。[L149-L176](../../../raw/current/scripts/components/constructionbuilder.lua#L149-L176) |
| `OnRemoveFromEntity`／`OnRemoveEntity` | 钩子 | 均指向 `StopConstruction`，移除组件或实体时返还容器物品并停止站点。[L178-L179](../../../raw/current/scripts/components/constructionbuilder.lua#L178-L179) |
| `constructioninst`／`constructionsite` | 实体／实体 | 属性赋值会同步 `constructionbuilderuidata` 的 net_entity，供客户端定位容器与目标。[L4-L10](../../../raw/current/scripts/components/constructionbuilder.lua#L4-L10) |

### 状态与中断顺序

- 动作：CONSTRUCT 的 `fn` 在 `CanStartConstruction` 通过后调用 `StartConstruction`；`construct` 状态第 9 帧或 0.7 秒超时才执行缓冲动作，成功后进入 `constructing`。[动作](../../../raw/current/scripts/actions.lua#L4435-L4463) [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19941-L19997)
- 完成：容器 UI 的 APPLYCONSTRUCTION 调 `FinishConstruction`，进入 `construct_pst`，1 秒（`fastbuilder` 0.5 秒）后 `OnFinishConstruction`；`construct_pst` 退出时未标记 `finished` 会补一次 `StopConstruction`。[APPLY](../../../raw/current/scripts/actions.lua#L4514-L4523) [收尾](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20052-L20085)
- 中断：`constructing` 的 `onexit` 在未由完成流程标记时停止；`stopconstruction` 事件（由 `StopConstruction` 同步推给玩家）会置 `statemem.stoppedconstruction` 标记并切到 `construct_pst` 动画回 idle，`onexit` 据该标记跳过重复的 `StopConstruction`（26148036 核验：事件处理器新增置标记、`onexit` 加 guard，旧版"退出时再次调用 StopConstruction 只重复推事件"的行为已移除）；玩家 despawn 也会主动停止。[中断](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20034-L20048) [despawn](../../../raw/current/scripts/prefabs/player_common.lua#L1437)

## 典型接入

### 原版参照：玩家在 mermthrone_construction 上施工

- 目标与入口：没有独立模组文件；原版配方 `mermthrone_construction`（boards 5／rope 5，`TECH.SCIENCE_ONE`，`merm_builder`）由 [[reference/components/builder]] 放置出站点，随后玩家对其按 CONSTRUCT（距离 2.5，非瞬时）。[配方](../../../raw/current/scripts/recipes.lua#L108) [动作定义](../../../raw/current/scripts/actions.lua#L481-L483)
- 端与资源：`constructionbuilder` 只在 `player_common` 的 master 段添加，因此宿主是所有经 `MakePlayerCharacter` 注册的玩家角色（如 `wilson`、`wx78`），不在某个角色 prefab 内单独添加；站点容器为 `construction_container`（`ui_construction_4x1` 四槽），站点动画与音效由 `mermthrone_construction` prefab 提供。[宿主](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2721) [容器](../../../raw/current/scripts/containers.lua#L491-L541)
- 流程：开始后容器对玩家打开并挂到玩家实体；玩家把材料放进对应槽（STORE 在站点无 container 时会回退到 `constructionbuilder.constructioninst`），再点容器 UI 的 APPLYCONSTRUCTION；完成后站点回调替换为 `mermthrone`。[STORE 回退](../../../raw/current/scripts/actions.lua#L2473-L2481) [站点完成](../../../raw/current/scripts/prefabs/mermthrone.lua#L17-L28)
- 生效与撤销：停止或拆除时容器内物品还回玩家，站点进度保留；施工中被锤击时站点反向调用 `ForceStopConstruction` 结束施工。整体替换发生在站点回调，本组件不生成产物。[反向停止](../../../raw/current/scripts/components/constructionsite.lua#L33-L40)

### 与 builder、deployable 的边界

- `builder` 负责配方学习、材料折扣与初次放置的配方材料；本组件只负责放置之后的第二次、分次材料投入，不读取 `builder` 的任何字段，完成也不通知它。[builder 扣料](../../../raw/current/scripts/components/builder.lua#L1131-L1180)
- `deployable` 是物品的一次性部署（DEPLOY 动作先取出一件再调 `ondeploy`），没有站点、容器与建造者绑定；需要“放置后继续投料”时用本站点链路，而不是给物品加 deployable。[deployable](../../../raw/current/scripts/components/deployable.lua#L124-L146) [DEPLOY 动作](../../../raw/current/scripts/actions.lua#L1622-L1631)

## 常见陷阱与限制

- **开始前会先拆掉当前施工**：`StartConstruction` 在检查 `INUSE`／`BURNT` 之前就调用 `StopConstruction`；对另一个站点按 CONSTRUCT 会先把现有容器物品收回，再因目标被占用而返回 false。[L47-L55](../../../raw/current/scripts/components/constructionbuilder.lua#L47-L55)
- **完成窗口内禁用站点会中止而不是完工**：`FinishConstruction` 先关容器并进入 1 秒收尾；若期间站点被 `Disable`，`Disable` 的 `ForceStopConstruction` 会立即停止施工并退回物品，随后 `OnFinishConstruction` 因绑定已清空而直接返回，不生成产物。[L110-L147](../../../raw/current/scripts/components/constructionbuilder.lua#L110-L147) [Disable](../../../raw/current/scripts/components/constructionsite.lua#L48-L51)
- **读档不会恢复界面与绑定**：保存的只是容器内容，载入后不重新 `Open`、也不设 `constructionsite`，客户端 target 为空；要恢复施工需重新开始，而开始会先停止并把旧容器物品退回。[L149-L176](../../../raw/current/scripts/components/constructionbuilder.lua#L149-L176)
- **容器打不开就按失败处理**：`SpawnPrefab` 成功但 `Open`／`IsOpenedBy` 失败时会移除容器、清空绑定并返回 false，且没有 reason；目标站点本身缺少 container 也会走这条路径。[L58-L75](../../../raw/current/scripts/components/constructionbuilder.lua#L58-L75)
- **容器为空时不存档**：只开始、尚未投料的施工不会写入玩家存档；容器实体 `persists=false`，不能指望它作为独立实体留存。[L64](../../../raw/current/scripts/components/constructionbuilder.lua#L64) [L149-L156](../../../raw/current/scripts/components/constructionbuilder.lua#L149-L156)
- **停止事件是无条件推送**：即使当前没有施工，`StopConstruction` 也会推 `stopconstruction`（26148036 核验：改用 `PushEventImmediate` 同步派发，监听者在 `StopConstruction` 返回前执行完毕）；监听该事件的模组逻辑不能假定必然存在进行中的站点。[L107](../../../raw/current/scripts/components/constructionbuilder.lua#L107)
- **不要手写完成流程**：直接调 `OnFinishConstruction` 会跳过关容器与状态切换，玩家仍停在 `constructing` 且容器被直接移除；只应由 `construct_pst` 超时调用。`IsConstructingAny` 也依赖状态名，替换 SGwilson 状态的模组需自行保持该名称。[L38-L40](../../../raw/current/scripts/components/constructionbuilder.lua#L38-L40) [L127-L147](../../../raw/current/scripts/components/constructionbuilder.lua#L127-L147)
- **只核验到玩家宿主**：组件本身没有玩家 tag 断言，但客户端 UI 数据由 `player_common` 的 `constructionbuilderuidata` 提供，且动作与状态机假定 `sg`／`inventory` 存在；给非玩家实体添加不在本次核验范围。[uidata](../../../raw/current/scripts/prefabs/player_common.lua#L2512) [builder](../../../raw/current/scripts/prefabs/player_common.lua#L2721)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 components/constructionbuilder.lua、prefabs/player_common.lua、stategraphs/SGwilson.lua、actions.lua、containers.lua、recipes.lua 对照 build-26148036 更新报告（constructionbuilder 内部重构条目 detail-components.md:19、SB-17、R-01、R-04、R-07）逐条核验，锚点重定位 12 处、结论修订 2 处（stopconstruction 改 PushEventImmediate 同步派发；SGwilson `constructing` 的 stopconstruction 处理器新增 `statemem.stoppedconstruction` 标记、onexit 加 guard 不再重复调用 StopConstruction，见正文"26148036 核验"标注）；containers/recipes 漂移（socket_keystone 按钮简化、charlie_hand_keystone 材料）不触及本页结论；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与属性同步、开始／停止／完成条件、容器生命周期、onremove 监听、OnSave／OnLoad、SGwilson 三段状态与 actions 三动作、player_common 宿主与 despawn 清理。`AddComponent("constructionbuilder")` 全库仅 `prefabs/player_common.lua:2733` 一处。
- 游戏运行：未运行；背包满时 `GiveItem` 的实际落点、客户端 UI 在停止／读档后的刷新、多人同时争用同一站点仍需游戏验证（背包满行为未核验）。
- 结构检查：`validate_page.py` 通过（82 行，低于 200 行／16 KiB 上限；0 issue）；`validate_source_links.py --strict` 共 46 条源码引用、0 error／0 warning；本页无新增 Lua 示例，未生成临时 `.lua`。

相关：[[reference/components/constructionsite]]；[[reference/components/builder]]；[[reference/components/deployable]]；[[reference/components/container]]；[[reference/components/workable]]。
