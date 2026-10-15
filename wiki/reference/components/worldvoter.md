---
title: "worldvoter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界投票器：承载用户指令发起的投票会话，倒计时经 net 变量全网同步，主 shard 收票计票并驱动 votedialog UI 与禁言冷却。"
aliases: ["世界投票", "投票踢人", "worldvoter 组件"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/worldvoter.lua", "prefabs/world_network.lua", "widgets/votedialog.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldvoter

## 快速判断

- 职责与适用：挂在 world_network 实体（prefabs/world_network.lua:61），三端一体：主 shard 组织投票、副 shard 接收 `secondary_worldvoterupdate`、所有端驱动 UI（:457-485）。
- 优先使用：发起投票走 usercommands（推 `ms_startvote`，:472）；监听 UI 状态 → 世界事件 `showvotedialog`/`hidevotedialog`/`worldvotertick`/`votecountschanged`（widgets/votedialog.lua:110-113）。
- 重要副作用：投票结束调 `UserCommands.FinishVote` 并按结果禁言发起者（`VOTE_PASSED_SQUELCH_TIME`=0/`VOTE_FAILED_SQUELCH_TIME`=30/取消 30，tuning.lua:3578-3581）。
- 修改边界：选项 ≤ `MAX_VOTE_OPTIONS`=6 且 `VOTE_PENDING` ≤7（net_tinybyte 限制，:24-28 断言）；同时仅一票（:323-324）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsVoteActive`/`IsEnabled` | `() -> boolean` | `_shown` 为 UI 展示态；`_enabled` 由 OnPostInit 按服务器投票开关或 dev 分支置真（:491-495）。:509-515 |
| `OnUpdate` | `(dt) -> nil` | 本地倒计时每帧推进：主 shard 按同步周期（>10s 每 5s，否则每 2s，:19-20、:342）set 网络值并 `master_worldvoterupdate`，其余端 `set_local`；到点结算（:521-563）。 |
| `OnStartVote`（局部） | `(data) -> nil` | 校验发起人未被禁言、`UserCommands.CanUserStartVote`（:331）；按 `command.votetimeout`（默认 30s）起票，目标自己 `CANNOT_VOTE` 其余 `VOTE_PENDING`（:336-353）。 |
| `OnReceiveVote`（局部） | `(data) -> nil` | 仅 PENDING 且选项存在才记票（:363-379）；`CheckVoteResults` 全员投完延迟 `END_VOTE_DELAY`=2s 收尾或超时按未投票结算（:381-424）。 |
| `SquelchPlayer`（局部） | `(userid, duration) -> nil` | 禁言任务表上限 `MAX_SLOTS`=64，满了替换最快到期的（:231-263）。 |
| `GetVoteDialogData`（局部） | `(hash, target, starter) -> table\|nil` | 按 hash 缓存对话框数据（名称/选项/目标与发起人 client），`votecountvisible` 才刷新计数（:78-133）。 |
| net 变量组 | `_countdown` net_byte、`_commandid` net_uint、`_targetuserid`/`_starteruserid` net_string、`_votecounts[6]` net_byte、`_enabled` net_bool | :60-68。 |

## 关键流程：一票从发起到结算

1. 发起：usercommands 层推 `ms_startvote` → OnStartVote 填 net 变量 → `countdowndirty` 触发 OnCountdownDirty 开启更新并 `showvotedialog`（:291-307、:142-159）。
2. 投票：客户端 playervoter 提交（`ms_receivevote`），主 shard 记票并推送 `master_worldvoterupdate`；副 shard 由 `secondary_worldvoterupdate` 镜像 net 值（:426-433）。
3. 结算：全员投完 → 2 秒延迟 → `CancelCountdown` → `UserCommands.FinishVote(commandname, {user, username}, results)` → 禁言发起者 → `SendVoteMetricsEvent`（:407-420）。
4. 提前终止：`ms_stopvote` 对发起者禁言 30s 并清场（:358-361）。

## 典型接入

### 监听投票 UI 事件（mod 侧安全入口）

- 前提与选择：投票组件由服务器开关控制（`IsEnabled`），世界事件在所有端都推，适合 HUD/提示类 mod。
- 生效与撤销：`showvotedialog` 数据含 `options[].vote_count`（仅 votecountvisible）；`worldvotertick` 每秒带剩余秒数。
- 原版依据：widgets/votedialog.lua:110-113。

```lua
AddPrefabPostInit("world", function(inst)
    inst:ListenForEvent("showvotedialog", function(world, data)
        -- data.name / data.options / data.targetuserid / data.starteruserid
    end)
    inst:ListenForEvent("worldvotertick", function(world, data)
        -- data.time: 剩余秒数
    end)
end)
```

- 调整与验证：切勿在客户端推 `ms_*` 事件（仅主 shard 监听，:463-474）；`_countdown` 是"秒+1"编码（:536），读 net 值需减 1。

## 常见陷阱与限制

- 选项数受 net_tinybyte 硬限：`assert(VOTE_PENDING <= 7)`（:28），加自定义投票选项超 6 项会断言失败。
- 禁言只对发起者：跟随投票的参与者无惩罚；禁言表按 userid 存任务，掉线不计（重进可再投）。
- 计数 UI 依赖 `votecountvisible`：command 未开可见计数时 `_votecounts` 仍同步但 UI 不显示（:126-130）。
- 载入不恢复投票：OnSave 未定义，投票中断即消失（组件无存档逻辑，:491-503 仅 OnPostInit）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 569 行；挂载 world_network.lua:61、UI 消费 widgets/votedialog.lua:110-113、常量 constants.lua:2596 与 tuning.lua:3578-3581、交互组件 playervoter（SetSelection/SetSquelched，:169-179）逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/shard_worldvoter]]
- [[reference/components/playervoter]]
