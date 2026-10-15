---
title: "followermemory"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "按玩家 userid 与角色 prefab 记住原主人，在失去归属后等待本分片重逢，并由回调决定是否重新招募。"
aliases: ["记住随从主人", "随从重逢", "发条生物重新招募", "followermemory"]
tags: [component, lifecycle, callback, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/followermemory.lua", "components/follower.lua", "prefabs/clockwork_common.lua", "prefabs/knight.lua", "prefabs/chessjunk.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# followermemory

## 快速判断

- 职责与适用：给随从保存玩家身份 `userid + prefab`，当前没有主人时寻找匹配玩家；实际归属由 [[reference/components/follower]] 维护。[followermemory.lua:31](../../../raw/current/scripts/components/followermemory.lua#L31 [followermemory.lua:134](../../../raw/current/scripts/components/followermemory.lua#L134
- 优先使用：有玩家实体用 `RememberAndSetLeader`；只有身份数据用 `RememberLeaderDetails`；通过重逢回调实现资格判断与重新设置归属。[followermemory.lua:21](../../../raw/current/scripts/components/followermemory.lua#L21 [followermemory.lua:42](../../../raw/current/scripts/components/followermemory.lua#L42
- 重要副作用：匹配身份的玩家靠近时，回调可能每 3 秒再执行；`ForgetLeader()` 清记忆、监听和追踪任务，**不解除当前 follower 关系**。[followermemory.lua:56](../../../raw/current/scripts/components/followermemory.lua#L56 [followermemory.lua:106](../../../raw/current/scripts/components/followermemory.lua#L106
- 修改边界：原版在服务端 Prefab 组装；查找用当前 `TheWorld` 的玩家加入事件和 `AllPlayers`，不负责跨分片搬运玩家或随从，也不提供移动行为。[knight.lua:116](../../../raw/current/scripts/prefabs/knight.lua#L116 [followermemory.lua:162](../../../raw/current/scripts/components/followermemory.lua#L162

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `RememberAndSetLeader(player)` | 有 `userid`、`prefab` 的玩家实体 | 先记身份；有 follower 时调 `SetLeader(player)`，否则只开始等待；不自行校验存活、容量或招募资格。[followermemory.lua:31](../../../raw/current/scripts/components/followermemory.lua#L31 |
| `RememberLeaderDetails(id, prefab)` | 玩家 userid、角色 prefab | 无当前主人时记忆并等待；已有主人时仅在其两项身份均匹配时更新，否则不改动。不是强制换主人接口。[followermemory.lua:42](../../../raw/current/scripts/components/followermemory.lua#L42 |
| `HasRememberedLeader()`／`IsRememberedLeader(target)` | 布尔值 | 前者仅查 id 非 nil；后者比较两项身份，target 应为非 nil 实体；都不代表当前仍有归属或玩家已在本分片。[followermemory.lua:61](../../../raw/current/scripts/components/followermemory.lua#L61 |
| `GetTrackingPlayer()` | 当前任务的 player 或 nil | 返回追踪任务引用；休眠或停止追踪后可为 nil，即使身份仍被记住。[followermemory.lua:69](../../../raw/current/scripts/components/followermemory.lua#L69 [followermemory.lua:127](../../../raw/current/scripts/components/followermemory.lua#L127 |
| `SetReuniteRange(dist)` | 距离；默认 40 | 供 `IsNear(player, range)` 判定是否调用重逢回调；不是自动传送／跟随半径。[followermemory.lua:3](../../../raw/current/scripts/components/followermemory.lua#L3 [followermemory.lua:106](../../../raw/current/scripts/components/followermemory.lua#L106 |
| `SetOnReuniteLeaderFn(fn)` | `fn(inst, player)` | 靠近时调用；返回值不消费，不保证招募成功，回调自己决定是否设置主人。[followermemory.lua:19](../../../raw/current/scripts/components/followermemory.lua#L19 [followermemory.lua:106](../../../raw/current/scripts/components/followermemory.lua#L106 |
| `SetOnLeaderLostFn(fn)` | `fn(inst)` | `OnChangedLeader(nil)` 且仍有记忆时调用；装载期间另受保存标记控制，见下节。[followermemory.lua:178](../../../raw/current/scripts/components/followermemory.lua#L178 |
| `ForgetLeader()` | 无参数 | 清身份、受击监听、世界玩家加入监听与追踪任务；组件移除复用此清理，不调用 follower:SetLeader(nil)。[followermemory.lua:56](../../../raw/current/scripts/components/followermemory.lua#L56 [followermemory.lua:80](../../../raw/current/scripts/components/followermemory.lua#L80 |

## 关键流程

### 失主、等待与重新归属

1. follower 的 `SetLeader` 只有在原始主人实体实际改变时才通知 `followermemory:OnChangedLeader(new_leader)`。新主人两项身份与记忆不符会清记忆；任何非 nil 新主人都会停止等待。[follower.lua:293](../../../raw/current/scripts/components/follower.lua#L293 [followermemory.lua:178](../../../raw/current/scripts/components/followermemory.lua#L178
2. 失主且有记忆时，按条件执行失主回调，然后监听 **TheWorld** 的 `ms_playerjoined`；实体醒着还会扫描 `AllPlayers`。匹配 userid 但 prefab 不符则遗忘，不把换角色视为同一记忆对象。[followermemory.lua:134](../../../raw/current/scripts/components/followermemory.lua#L134 [followermemory.lua:162](../../../raw/current/scripts/components/followermemory.lua#L162
3. 追踪任务初次延迟为 0、周期 3 秒；角色不符则遗忘，玩家无效则只停追踪，范围内才执行重逢回调。实体休眠只停追踪，保留身份与等待监听；醒来再查当前玩家。[followermemory.lua:106](../../../raw/current/scripts/components/followermemory.lua#L106 [followermemory.lua:160](../../../raw/current/scripts/components/followermemory.lua#L160
4. 有记忆期间监听随从自身 `attacked`：攻击者 userid 等于记忆 id 就遗忘，这里不再比较角色 prefab。原版是否同时解除当前关系由 follower 等消费者决定。[followermemory.lua:82](../../../raw/current/scripts/components/followermemory.lua#L82

### 保存与恢复

`OnSave` 只在有记忆 id 时保存 `id`、`char`，以及 `lost`：该标记表示**保存当时 follower 仍有主人，重载将失去这一关系**，并非“保存时已经失主”。不保存玩家实体、任务或回调。[followermemory.lua:194](../../../raw/current/scripts/components/followermemory.lua#L194

`OnLoad` 恢复身份，将 `data.lost` 临时交给 `OnChangedLeader`；在 `POPULATING` 期间仅该标记为真才允许失主回调，所以已失主后保存的记录不会在正常装载时再触发一次。随后 `LoadPostPass` 清 follower 的普通玩家缓存及其加入／新玩家监听，恢复改走本组件的身份与重逢流程。[followermemory.lua:178](../../../raw/current/scripts/components/followermemory.lua#L178 [followermemory.lua:204](../../../raw/current/scripts/components/followermemory.lua#L204 [follower.lua:262](../../../raw/current/scripts/components/follower.lua#L262

## 典型接入

### 原版发条骑士：容量限制下等待旧主人

`knight` 与 `knight_nightmare` 经 `MakeKnight` 的服务端工厂先添加 follower、knownlocations 等，再由各自 master_postinit 调 `clockwork_common.MakeBefriendable`。该组装把重逢回调设为 `TryBefriendChess`，失主回调设为将当前位置记作新 home；本页沿用原版参照，不将其抽成通用宠物 demo。[knight.lua:86](../../../raw/current/scripts/prefabs/knight.lua#L86 [knight.lua:131](../../../raw/current/scripts/prefabs/knight.lua#L131 [knight.lua:181](../../../raw/current/scripts/prefabs/knight.lua#L181 [knight.lua:198](../../../raw/current/scripts/prefabs/knight.lua#L198 [knight.lua:632](../../../raw/current/scripts/prefabs/knight.lua#L632 [clockwork_common.lua:271](../../../raw/current/scripts/prefabs/clockwork_common.lua#L271 [clockwork_common.lua:3](../../../raw/current/scripts/prefabs/clockwork_common.lua#L3

`TryBefriendChess` 要求生物存活且无主人、玩家有效且未死亡／成鬼、有 leader、未参加小游戏。已记住的主人若 clockworktracker 容量不足，直接拒绝；首次尝试则先记忆并设主人，再检查／登记追踪器，超限时解除 follower 而保留身份，供之后重试。容量算法由追踪器另行负责，本页未展开。[clockwork_common.lua:231](../../../raw/current/scripts/prefabs/clockwork_common.lua#L231

修复棋堆生成友好后代时，`SpawnScionAtXZ` 先尝试修理者及附近玩家；仍无主人且有修理者身份数据时调用 `RememberLeaderDetails`，无需此时持有修理者实体。已有其他主人不会被该补记覆盖。[chessjunk.lua:46](../../../raw/current/scripts/prefabs/chessjunk.lua#L46

记忆还有发条专用消费者：`Retarget` 在仍记住主人时停止主动选新目标；`HasSameLeaderOrMem` 用当前主人或追踪玩家比较关系。它们不意味着本组件统一实现了免伤、和平或禁止反击。[clockwork_common.lua:72](../../../raw/current/scripts/prefabs/clockwork_common.lua#L72 [clockwork_common.lua:94](../../../raw/current/scripts/prefabs/clockwork_common.lua#L94

## 常见陷阱与限制

- **只加记忆就会自动重新跟随**：无重逢回调不会设置归属；回调拒绝或不更改关系时，任务可继续尝试。需要移动行为另见 [[reference/components/follower#生命周期与实际跟随]]。[followermemory.lua:106](../../../raw/current/scripts/components/followermemory.lua#L106
- **把遗忘当解约**：`ForgetLeader` 仅清本组件状态；强制解除应另走 follower 的归属接口。反之普通失主保留记忆，正是之后重逢的前提。[followermemory.lua:56](../../../raw/current/scripts/components/followermemory.lua#L56 [followermemory.lua:178](../../../raw/current/scripts/components/followermemory.lua#L178
- **把身份记忆当分片定位服务**：这里没有跨分片查询、迁移或对不存在玩家的实体返回保证。只依据保存数据与本分片玩家事件重建等待；角色 prefab 改变还会主动遗忘。[followermemory.lua:73](../../../raw/current/scripts/components/followermemory.lua#L73 [followermemory.lua:134](../../../raw/current/scripts/components/followermemory.lua#L134

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部方法、follower 归属变更通知／缓存清除，发条公共回调、骑士工厂与棋堆身份补记。未展开 clockworktracker 容量内部算法、全部发条变体、跨分片迁移协议。
- 结构检查：本页元数据、章节、大小及站内链接检查通过；源码引用 strict 检查通过。未含完整 Lua 示例，未检查索引登记或记录哈希基线。
- 游戏运行：未运行；读档／离线重返、休眠唤醒、换角色、受击遗忘与容量释放后的重逢待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/follower]]；[[reference/components/leader]]。
