---
title: "playervoter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家投票客户端组件：桥接 classified 的票选/禁言网络变量，提交选票并维护 UI 用的 selection、canvote、squelched 状态。"
aliases: ["玩家投票", "投票组件", "playervoter"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/playervoter.lua", "components/worldvoter.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "usercommands.lua", "widgets/votedialog.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playervoter

## 快速判断

- 职责与适用：挂在每个玩家实体上的投票端组件，双端运行——master 端由 worldvoter 写入状态，客户端提交选票并读同步值给投票弹窗。[构造](../../../raw/current/scripts/components/playervoter.lua#L7-L17)
- 优先使用：客户端提交用 `SubmitVote(sel)`（内部走 `TheNet:Vote`），读取用 `GetSelection()`／`CanVote()`／`HasVoted()`／`IsSquelched()`；master 端写入用 `SetSelection`／`SetSquelched`。[SubmitVote](../../../raw/current/scripts/components/playervoter.lua#L74-L84)
- 重要副作用：状态变化对本地推 `playervotechanged {selection, canvote}` 事件（投票弹窗监听它刷新）；提交后 3 秒内用本地乐观值显示，超时回读 classified。[事件](../../../raw/current/scripts/components/playervoter.lua#L27-L30)
- 修改边界：与 `worldvoter`／`shard_worldvoter` 的常量约定人工同步（`CANNOT_VOTE=0`、`VOTE_PENDING=MAX_VOTE_OPTIONS+1`）；组件随玩家控制权增删，不是开局即有。[同步注释](../../../raw/current/scripts/components/playervoter.lua#L3-L5)、[增删](../../../raw/current/scripts/prefabs/player_common.lua#L990-L1025)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `classified.voteselection` | net_tinybyte `playervoter.selection` | 0=不可投，1..6=已投选项，7（`MAX_VOTE_OPTIONS+1`）=可投票中；定义在 player_classified。[定义](../../../raw/current/scripts/prefabs/player_classified.lua#L1481-L1482)、[常量](../../../raw/current/scripts/constants.lua#L2596) |
| `SubmitVote(sel)` | 无返回 | 仅当当前值为 `VOTE_PENDING` 时有效：本地先 `OnVoteChanged` 再 3 秒刷新任务兜底，最后 `TheNet:Vote(sel)` 发包。[L74-L84](../../../raw/current/scripts/components/playervoter.lua#L74-L84) |
| `SetSelection(sel)`／`SetSquelched(val)` | 无返回 | 仅 mastersim；worldvoter 在投票开启/玩家进出时批量写每个玩家的 selection 与 squelched。[L86-L91](../../../raw/current/scripts/components/playervoter.lua#L86-L91)、[worldvoter 写入](../../../raw/current/scripts/components/worldvoter.lua#L170-L178) |
| `GetSelection()`／`HasVoted()`／`CanVote()` | 选项或 nil／布尔 | 全部读 classified 当前值并按常量区间解释；classified 缺失时 `HasVoted`/`CanVote` 为 false。[L93-L110](../../../raw/current/scripts/components/playervoter.lua#L93-L110) |
| `IsSquelched()` | 布尔 | 读 `votesquelched`（net_bool `playervoter.issquelched`）；classified 为 nil 时按 true 处理（未授权玩家视为禁言）。[L118-L120](../../../raw/current/scripts/components/playervoter.lua#L118-L120) |
| `AttachClassified`／`DetachClassified` | 无返回 | 客户端绑定/解绑 classified 的脏事件与 `onremove`；构造时自动挂。[L57-L70](../../../raw/current/scripts/components/playervoter.lua#L57-L70) |

## 生命周期

- 增：`OnSetOwner` 在"是本地玩家或 master"时挂上（与 playeractionpicker/playercontroller 同批），移除控制权（幽灵/重连切换）时 `RemovePlayerComponents` 摘除。[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L1019-L1025)、[移除](../../../raw/current/scripts/prefabs/player_common.lua#L990-L995)
- 客户端 classified 目标变化时通过 `ondetachclassified` 自动重挂；`OnRemoveFromEntity` 与 `OnRemoveEntity` 同实现。[L40-L55](../../../raw/current/scripts/components/playervoter.lua#L40-L55)

## 典型接入

### 原版消费者

- 投票弹窗：`votedialog` 监听 `playervotechanged` 刷新选中态，点击选项调 `SubmitVote(closure_index)`。[监听](../../../raw/current/scripts/widgets/votedialog.lua#L114)、[提交](../../../raw/current/scripts/widgets/votedialog.lua#L87-L88)
- 投票发起校验：`usercommands` 用 `IsSquelched()` 判定发起者投票等级（被禁言者降为 0）与是否允许。[校验](../../../raw/current/scripts/usercommands.lua#L206-L209)、[L275-L277](../../../raw/current/scripts/usercommands.lua#L275-L277)
- master 状态源：worldvoter 持有 `_voterdata`/`_squelched` 表，变更时逐玩家 `SetSelection`/`SetSquelched`（shard 投票另有 shard_worldvoter，常量三方同步）。[worldvoter](../../../raw/current/scripts/components/worldvoter.lua#L169-L178)

### 模组：监听/校验投票

- 前提与选择：读状态在双端都判组件与 `CanVote`；写状态只有 master 有效（`SetSelection` 有 ismastersim 守卫，客户端调用静默无效）。
- 生效与撤销：监听 `playervotechanged` 即可响应投票 UI 状态；无需清理。

```lua
-- modmain.lua：玩家投票发生变化时记录（本地端）
AddPlayerPostInit(function(inst)
    inst:DoTaskInTime(0, function()
        if inst.components.playervoter ~= nil then
            inst:ListenForEvent("playervotechanged", function(_, data)
                print("vote sel:", tostring(data.selection), "canvote:", tostring(data.canvote))
            end)
        end
    end)
end)
```

- 调整与验证：示例仅语法检查未运行；`data.selection` 为 1..6 或 nil，`data.canvote` 表示当前处于可投票窗口。

## 常见陷阱与限制

- `SubmitVote` 是客户端接口但服务端也有组件实例：master 端调用会走同一逻辑却发 `TheNet:Vote`，属无意义调用；写入请用 `SetSelection`。[L74-L84](../../../raw/current/scripts/components/playervoter.lua#L74-L84)
- 乐观显示有 3 秒窗口：提交后立即再提交会被 `OnVoteChanged` 清刷新任务重置窗口；服务端真实结果以 classified 回读为准。[L79-L82](../../../raw/current/scripts/components/playervoter.lua#L79-L82)
- 常量三方手工同步：改 `MAX_VOTE_OPTIONS` 或区间含义要同时改 worldvoter/shard_worldvoter/playervoter 三处注释约定的值。[L3-L5](../../../raw/current/scripts/components/playervoter.lua#L3-L5)
- 组件生命周期跟随控制权而非实体：本地玩家组件可能在实体存在期间被移除重建，监听 `playervotechanged` 前先确认当帧组件存在。[移除](../../../raw/current/scripts/prefabs/player_common.lua#L993-L995)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 playervoter.lua 全文（含 classified 挂解与全部读写方法）；核对 player_common 挂载/移除、player_classified 变量定义、worldvoter 写入端、votedialog 与 usercommands 消费端、constants 常量；未覆盖 shard_worldvoter 的分片同步细节。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。
