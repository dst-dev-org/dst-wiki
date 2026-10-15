---
title: linkeditemmanager
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "世界侧按 userid 管理绑定物品，恢复在线主人与技能初始化，并批量处理同一玩家的关联物品。"
aliases: [linkeditemmanager, 绑定物品管理器, 批量解除物品绑定, 玩家重连物品恢复, 玩家关联物品遍历]
tags: [component, item, lifecycle, event, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/linkeditemmanager.lua, components/linkeditem.lua, prefabs/forest.lua, prefabs/cave.lua, prefabs/world.lua, prefabs/player_common_extensions.lua, prefabs/skilltree_wortox.lua, prefabs/wortox_reviver.lua, util.lua]
reliability: verified
validation: source-reviewed
status: active
---

# linkeditemmanager

## 快速判断

- 职责与适用：世界服务端维护 `userid → 物品集合` 与在线玩家映射；玩家离开／重连时更新关联实体。物品本身用 [[reference/components/linkeditem]]，不要把管理器装到物品上。[构造与监听](../../../raw/current/scripts/components/linkeditemmanager.lua#L5-L33)
- 优先使用：通过物品 `LinkToOwnerUserID` 登记／解绑；确需批处理时用 `TheWorld.components.linkeditemmanager:ForEachLinkedItemForPlayer…`。[登记协议](../../../raw/current/scripts/components/linkeditem.lua#L61-L76)、[遍历](../../../raw/current/scripts/components/linkeditemmanager.lua#L35-L53)
- 重要副作用：登记到在线玩家会立即设置主人实体，已完成技能初始化时立即通知物品；玩家离开会清空实体，但不清除 userid 关系。[登记](../../../raw/current/scripts/components/linkeditemmanager.lua#L99-L113)、[离开](../../../raw/current/scripts/components/linkeditemmanager.lua#L80-L97)
- 修改边界：原版 forest／cave 的 master_postinit 已创建管理器，客户端不能创建；它管理当前世界记录，不是跨分片远程查询服务。[forest](../../../raw/current/scripts/prefabs/forest.lua#L639)、[添加位置](../../../raw/current/scripts/prefabs/forest.lua#L671-L674)、[cave](../../../raw/current/scripts/prefabs/cave.lua#L518-L534)、[世界分端](../../../raw/current/scripts/prefabs/world.lua#L568-L606)

## 关键接口

| 接口／状态 | 条件、作用与证据 |
|---|---|
| `ForEachLinkedItemForPlayer(player, callback, ...)` | 对该 `player.userid` 的已登记集合浅拷贝后遍历，回调 `(item, player, ...)`；没有集合就不回调，无聚合返回值。不是扫描玩家物品栏。[实现](../../../raw/current/scripts/components/linkeditemmanager.lua#L46-L53)、[shallowcopy](../../../raw/current/scripts/util.lua#L946-L957) |
| `ForEachLinkedItemForPlayerOfPrefab(player, prefab, callback, ...)` | 同上，再以 `item.prefab == prefab` 筛选；适合某种专属物品的技能状态批量刷新。[实现](../../../raw/current/scripts/components/linkeditemmanager.lua#L35-L44) |
| `linkeditems[userid]`／`players[userid]` | 前者为 `{[item]=true}`，后者为当前玩家实体；由正常事件维护，不直接增删表项。[初始化](../../../raw/current/scripts/components/linkeditemmanager.lua#L9-L18)、[登记](../../../raw/current/scripts/components/linkeditemmanager.lua#L99-L127) |
| `waitingforinitialization[player]` | 按实体记录等待技能握手；注册物品若仍在等待，只设主人，不提前发技能初始化。[等待](../../../raw/current/scripts/components/linkeditemmanager.lua#L68-L77)、[登记分支](../../../raw/current/scripts/components/linkeditemmanager.lua#L104-L111) |

`OnPlayerJoined`／`OnPlayerLeft`／`OnRegisterLinkedItem`／`OnUnregisterLinkedItem` 是世界事件处理入口；正常接入不手动调用，也不自行 PushEvent 伪造条目。登记数据依赖合法的 item、userid 与 linkeditem 生命周期。[事件接线](../../../raw/current/scripts/components/linkeditemmanager.lua#L15-L18)、[物品绑定入口](../../../raw/current/scripts/components/linkeditem.lua#L61-L76)

### 玩家、技能与存档的恢复顺序

1. 玩家加入且不是 `is_snapshot_user_session`：记录 userid 对应实体，为其现有物品 `SetOwnerInst(player)`。握手已 READY 则通知技能初始化，否则监听该玩家的 `ms_skilltreeinitialized`。[OnPlayerJoined](../../../raw/current/scripts/components/linkeditemmanager.lua#L55-L78)
2. 服务端玩家握手进入 READY，**玩家实体**发送 `ms_skilltreeinitialized`；管理器在世界上注册的跨实体监听先解除等待，再通知该 userid 的关联物品。新登记物品也会按等待状态决定是否立即通知。[事件生产](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1085-L1103)、[监听处理](../../../raw/current/scripts/components/linkeditemmanager.lua#L20-L33)
3. 玩家离开：各物品 `SetOwnerInst(nil)`，清理玩家映射及尚未完成的技能监听；绑定集合仍保留。重连后恢复新实体，不复用旧玩家引用。[OnPlayerLeft](../../../raw/current/scripts/components/linkeditemmanager.lua#L80-L97)
4. 管理器无自己的 OnSave／OnLoad；物品 linkeditem 保存 userid 并在读档时重新登记，管理器依玩家事件重建在线关系。注销最后一个物品时删除该 userid 的空集合。[物品读档](../../../raw/current/scripts/components/linkeditem.lua#L123-L144)、[注销](../../../raw/current/scripts/components/linkeditemmanager.lua#L115-L127)

## 典型接入

### 技能取消时处理所有绑定物品，包括已经离开物品栏的

采用原版 Wortox 技能协作链，不另造通用轮询器。位置为 `prefabs/skilltree_wortox.lua` 的技能 onactivate／ondeactivate；自定义技能需接入目标角色实际技能表，不能把这些回调原样当作独立 `modmain.lua` 工程。

- 开启 `wortox_lifebringer_1` 时遍历当前 inventory，找 `wortox_reviver` 并尝试绑定；这是“发现尚未登记的物品”，不能交给只认识登记集合的管理器。[开启入口](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L458-L471)、[辅助函数](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L74-L85)
- 取消该技能时用 `ForEachLinkedItemForPlayerOfPrefab(player, "wortox_reviver", Unlink_wortox_reviver)`，回调对每件物品调用 `LinkToOwnerUserID(nil)`；覆盖当前世界已登记、但不一定还在玩家身上的复活心。[取消入口](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L472-L476)
- `wortox_lifebringer_3` 启用／取消时，同样遍历登记集合调用物品的 `SetAllowConsumption(true/false)`；物品把它转为 spellcaster 的法术类型限制。此入口改变可用状态，不额外覆盖 linkeditem 的归属。[技能入口](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L486-L505)、[回调](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L66-L72)、[物品消费者](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L64-L78)

选择此模式时，先确保目标 Prefab 已按 linkeditem 工厂流程创建并保存绑定。回调可在遍历中解绑自己，因为遍历浅拷贝集合；这不等于回调可以安全删除其他待处理实体，执行动作前仍应核对自己的目标与组件状态。[遍历与注销](../../../raw/current/scripts/components/linkeditemmanager.lua#L35-L53)、[注销条目](../../../raw/current/scripts/components/linkeditemmanager.lua#L115-L127)

离线限制还需物品自己的 removed／技能初始化回调；原版复活心在失去主人实体时关闭消耗，在主人技能初始化后重查技能。管理器仅通知，不会替任意物品自动加技能权限。[物品回调](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L70-L78)、[注册位置](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L263-L264)

## 常见陷阱与限制

- **把遍历当全服查物品：** 集合只含当前世界收到登记事件的实体，没有跨分片 RPC、按 userid 加载离线玩家或扫描未生成物品的逻辑；不承诺一个分片能修改另一分片物品。[集合与事件](../../../raw/current/scripts/components/linkeditemmanager.lua#L5-L33)
- **跳过 linkeditem 直接改表：** 会绕过原绑定注销、netvar 更新和 owner 回调，导致归属不同步。使用物品正常绑定接口；不要用修改管理器内部表实现私有箱权限。[绑定协议](../../../raw/current/scripts/components/linkeditem.lua#L61-L76)、[登记消费者](../../../raw/current/scripts/components/linkeditemmanager.lua#L99-L127)
- **技能刚加入就读完整状态：** 加入事件未必代表技能握手完成；依 linkeditem 的技能初始化回调，不在 created 回调无条件假定技能可用。该初始化通知也不替代技能激活／取消时的刷新。[等待条件](../../../raw/current/scripts/components/linkeditemmanager.lua#L55-L78)、[技能变更消费者](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L496-L505)
- **在运行中重装世界管理器：** 构造先枚举 AllPlayers，后初始化等待表与技能回调，已有未就绪玩家时存在访问未初始化成员的风险；沿用世界工厂已有组件，不临时 RemoveComponent／AddComponent。实现也没有 OnRemoveFromEntity 清理协议。[构造顺序](../../../raw/current/scripts/components/linkeditemmanager.lua#L5-L33)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；仅锚点重定位（forest −3、cave +44、world +4~+11、player_common_extensions +4 段）；cave 新增 Vault 布局与世界新 4 全局组件（报告 detail-prefabs.md:44/:216）不影响本页"forest/cave master_postinit 挂载管理器"结论，util shallowcopy 未变，结论维持。
- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整审阅管理器／linkeditem，核验地表与洞穴服务端挂载、玩家握手事件、Wortox 技能发现／批处理／解绑链和浅拷贝实现。物品网络创建与装备限制详见 linkeditem；未据此扩展为跨分片全局管理。
- 游戏运行：未运行；没有新增独立 Lua demo。以上原版接入说明不代表自定义技能完整项目已验证。
- 结构检查：页面格式、现有 Wiki 互链与源码锚点严格检查通过；不以结构结果代替玩家重连／存档运行测试。

相关：[[reference/components/linkeditem]]、[[reference/components/inventoryitem]]、[[reference/components/entitytracker]]。
