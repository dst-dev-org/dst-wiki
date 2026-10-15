---
title: "worldcharacterselectlobby"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "竞技场／活动场 gamemode 的角色选择网络大厅：同步出生倒计时、锁定服务器并汇集玩家就绪状态，全员就绪后倒计时放行出生。"
aliases: ["worldcharacterselectlobby", "角色选择大厅", "大厅倒计时", "playerreadytostart", "出生倒计时"]
tags: [component, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/worldcharacterselectlobby.lua", "prefabs/lavaarena_network.lua", "prefabs/quagmire_network.lua", "networking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldcharacterselectlobby

## 快速判断

- 职责与适用：挂在 `TheWorld.net` 上的大厅组件，仅熔炉（lavaarena）与饥荒联机活动场（quagmire）gamemode 安装；同步"选择角色→全员就绪→倒计时→放行出生"流程，普通生存世界不存在此组件。[安装](../../../raw/current/scripts/prefabs/lavaarena_network.lua#L50-L54) [quagmire 安装](../../../raw/current/scripts/prefabs/quagmire_network.lua#L56-L57)
- 优先使用：客户端读出生延迟用 `GetSpawnDelay()`（无倒计时返回 -1）；判断可否出生用 `CanPlayersSpawn()`（仅倒计时归零为真）；就绪列表查询用 `IsPlayerReadyToStart(userid)`。[延迟](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L300-L303) [放行](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L337-L339)
- 重要副作用：倒计时启动即 `TheNet:SetAllowNewPlayersToConnect(false)` 并 `SetIsMatchStarting(true)`，新玩家此后无法连入；master 侧倒计时归零后组件改用普通 Update 轮询，直到有人出生或全员断开才清掉 match starting 标志。[锁连入](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L155-L162) [收尾轮询](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L258-L269) [OnUpdate](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L353-L369)
- 修改边界：组件常驻 `TheWorld.net`（该实体 persists=false），无 OnRemoveFromEntity；存档只记 `match_started`，倒计时进行中不会保留——重载后回到未开始状态。[存读](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L375-L392)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_countdowni` | net_byte（dirty 事件 `spawncharacterdelaydirty`） | 255 表示未开始；倒计时秒数（向上取整）。OnCountdownDirty 向世界推 `lobbyplayerspawndelay {time, active}`，客户端 UI 都消费它。[网络变量](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L34-L40) [dirty 处理](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L258-L269) |
| `_lockedforshutdown` | net_bool | OnPostInit 时若请求了延迟关服（`GetDeferredServerShutdownRequested`）置真，之后 `IsAllowingCharacterSelect` 恒假、就绪命令被拒。[锁定](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L322-L326) |
| `_players_ready_to_start` | net_string × 服务器最大人数 | 就绪玩家 userid 槽位（空串为空位）；任一槽位 set 触发 `player_ready_to_start_dirty`，master 以"就绪数==客户端数"判定全员就绪。[槽位](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L38-L40) [判定](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L174-L185) |
| `IsAllowingCharacterSelect()` | → bool（仅 mastersim 定义） | 无倒计时进行中且未锁定关服；就绪切换、角色请求与开始判定都受它门禁。[门禁](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L305-L307) |
| `GetSpawnDelay()` | → number | `_countdowni` 值，未开始时 -1；客户端出生延迟 UI 读取。[读取](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L300-L303) |
| `CanPlayersSpawn()` | → bool | 仅 master 的倒计时浮点归零（`_countdownf == 0`）为真；networking 的出生 gate 用它拦人。[判定](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L337-L339) [出生 gate](../../../raw/current/scripts/networking.lua#L186 |
| 用户命令 `playerreadytostart` | UserCommand（无斜杠、无菜单） | 参数 "ready"，master 侧 Toggle 就绪状态；canstartfn 要求未开始且未锁定。[命令](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L110-L125) |
| `OnRequestLobbyCharacter`（内部） | 监听 `ms_requestedlobbycharacter` | 转发到 `TheNet:SetLobbyCharacter(...)` 并先清该玩家就绪位，再尝试开始倒计时。[角色请求](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L187-L201) |
| `StarTimer`（内部） | 秒数 | 发 analytics 事件、锁新连入、清就绪表、启动 WallUpdate 倒计时；开发分支倒计时 1 秒否则 6 秒。[启动](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L141-L162) [时长](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L13-L14) |

### 连接／断开与 analytics

`ms_clientauthenticationcomplete` 进入：清空就绪表并记录该玩家等待起始时间；倒计时已开始时新到玩家无处可去（只能断开）。`ms_clientdisconnected` 退出：清就绪表、结算 lobby.leave／lobby.cancelmatch analytics（有 lavaarenaanalytics 或 quagmireanalytics 组件时）。组件构造即把当前已连接客户端视作进入。[进入](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L203-L222) [退出](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L224-L250) [构造重放](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L281-L294)

## 典型接入

### 在熔炉／活动场世界读取或拦截大厅状态

- 前置与选择：组件只在 lavaarena／quagmire 的 net 实体上存在；客户端侧 UI（等待玩家面板、大厅屏）监听世界 net 实体的 `player_ready_to_start_dirty` 与世界的 `lobbyplayerspawndelay` 事件刷新，不直接轮询组件方法。[客户端监听](../../../raw/current/scripts/widgets/waitingforplayers.lua#L75-L103) [大厅屏](../../../raw/current/scripts/screens/redux/lobbyscreen.lua#L595
- 生效与撤销：服务器想强制开始可借助既有 master 侧流程（全员就绪即自动触发）；没有公开的"立即开始"接口，控制倒计时的入口都在组件内部函数。锁定关服路径由系统 OnPostInit 设置，模组可直接 set `_lockedforshutdown` 网络变量但不推荐。
- 原版依据：networking 在玩家出生前检查 `CanPlayersSpawn()`，配合 `SetIsMatchStarting` 维持连接门禁直到有人出生。[gate](../../../raw/current/scripts/networking.lua#L186

```lua
-- 客户端/通用：监听出生延迟变化（世界 net 实体上）
AddPrefabPostInit("world_net", function(inst)
    if inst.components.worldcharacterselectlobby == nil then
        return
    end
    inst:ListenForEvent("lobbyplayerspawndelay", function(world, data)
        if data.active then
            print("spawn countdown:", data.time)
        end
    end)
end)
```

- 调整与验证：`data.time` 是 `_countdowni` 原值（255 表示未开始），`data.active` 已排除该值；普通生存世界不会触发此事件。[推送](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L267-L268)

## 常见陷阱与限制

- **在生存世界找这个组件：** 只有 lavaarena_network／quagmire_network 安装它；生存世界的出生等待不经过此组件。[安装点](../../../raw/current/scripts/prefabs/lavaarena_network.lua#L53
- **就绪判定按"客户端数"而非玩家数：** `TryStartCountdown` 比较就绪槽数与 `GetClientTable` 数（专用服务器会剔除 host 对象）；旁观者或未选角色客户端都会让倒计时不开。[客户端表](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L45-L56) [判定](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L179-L184)
- **倒计时中断开／断线重连的影响不对称：** 连入或断开都先 `ClearAllPlayersReadyToStart()`，所以任何连接波动都会重置全员就绪进度；倒计时一旦开始则不可回退。[清空](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L131-L135)
- **match_started 存档不保存倒计时中状态：** 只在倒计时已经归零时，重载世界会把 countdown 置 0 并再次锁新连入；倒计时进行中重载等于回滚到选择阶段。[存读](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L384-L392)
- **`IsAllowingCharacterSelect` 仅 master 有定义：** 客户端调用会得到 nil 方法错误；客户端应改用事件或 `GetSpawnDelay`。[条件定义](../../../raw/current/scripts/components/worldcharacterselectlobby.lua#L305-L307)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 worldcharacterselectlobby.lua，核验网络变量与 dirty 事件、就绪命令、倒计时启停、连接断开处理、analytics 分支、存读与两个安装点；核对 networking 出生 gate 与 waitingforplayers／lobbyscreen 客户端消费。未运行游戏，未审校 TheNet 底层语义。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；熔炉实际开局长延迟、锁定关服与断线重连表现未实测。

## 相关链接

- [[reference/components/lavaarenaeventstate]]：同挂在 lavaarena 网络实体上的活动状态。
- [[reference/components/quagmire_hangriness]]：quagmire 网络实体上必须先于此组件添加的组件。
