---
title: "lavaarenaeventstate"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔炉模式挂在世界 net 实体上的比赛状态网络组件，客户端只读查询当前轮次与社区进度 JSON。"
aliases: ["熔炉比赛状态", "熔炉轮次查询", "lavaarenaeventstate"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
sources: ["components/lavaarenaeventstate.lua", "prefabs/lavaarena_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lavaarenaeventstate

## 快速判断

- 职责与适用：熔炉（Forge）模式下随 `lavaarena_network` 挂到 `TheWorld.net`，把比赛轮次、胜负状态与社区进度打包成网络变量供客户端查询（[安装](../../../raw/current/scripts/prefabs/lavaarena_network.lua#L54)）。
- 优先使用：需要显示熔炉轮次或读取服务器进度/任务 JSON 时直接经 `TheWorld.net.components.lavaarenaeventstate` 查询。
- 重要副作用：客户端收到 `victorystatedirty` 且值非 0 时向世界推送 `endofmatch` 事件（[处理](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L62-L66)）。
- 修改边界：服务器写入逻辑由 `event_server_data("lavaarena", ...)` 注入（[注入点](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L75-L77)），对应的事件服文件不在本 scripts 快照内，无法核验。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_netvars.round` | `net_smallbyte`，脏事件 `rounddirty` | 当前轮次；`GetCurrentRound()` 返回 `math.max(1, value)`。[声明](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L30-L40) [读取](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L50-L52) |
| `_netvars.victorystate` | `net_tinybyte`，脏事件 `victorystatedirty` | 0=进行中、1=胜利、2=失败（[枚举](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L11-L16)）。 |
| `_netvars.progression_json` | `net_string`，脏事件 `progressionjsondirty` | 服务器写入的社区进度 JSON；客户端经 `GetServerProgressionJson()` 读。[读取](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L46-L48) |
| `_netvars.player_quest_json[i]` | `net_string` 数组，按 `TheNet:GetServerMaxPlayers()` 逐玩家建槽（[建槽](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L38-L40)） | 经 `GetServerPlayerQuestJson(quest_slot)` 读（[实现](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L54-L56)）。 |

## 典型接入

### 客户端读取熔炉当前轮次

- 前提与选择：仅熔炉模式存在该组件；消费方为客户端界面，直接访问 `TheWorld.net.components.lavaarenaeventstate`。
- 生效与撤销：net 实体随世界存在，组件始终可查询，无需安装或清理。
- 原版依据：`playerstatusscreen` 用 `GetCurrentRound()` 拼服务器状态文案（[消费](../../../raw/current/scripts/screens/playerstatusscreen.lua#L170)）；社区进度面板解码 progression/quest JSON（[消费](../../../raw/current/scripts/lavaarena_communityprogression.lua#L465)、[消费](../../../raw/current/scripts/lavaarena_communityprogression.lua#L490)）。

```lua
-- 客户端 UI 内查询熔炉轮次（仅 lavaarena 模式下组件存在）
local state = TheWorld.net ~= nil and TheWorld.net.components.lavaarenaeventstate or nil
if state ~= nil then
    local round = state:GetCurrentRound()
end
```

- 调整与验证：非熔炉模式不要访问该组件；服务器侧不应读取这些 netvar（构造注释明确"server should not use this data"，[注释](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L29)）。

## 常见陷阱与限制

- 服务器写入逻辑（`master_postinit`）在 `lavaarena_event_server` 事件服文件内，本快照 `raw/current/scripts` 无该目录，写入时机与取值范围未核验；模组改写进度 JSON 需自行找到对应事件服文件。
- `endofmatch` 推送带 `{victory = bool}`，由组件在客户端侧触发；dedicated 服务器与无效玩家不推送（[守卫](../../../raw/current/scripts/components/lavaarenaeventstate.lua#L63)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、lavaarena_network 安装点与 playerstatusscreen / lavaarena_communityprogression 消费链；事件服服务器逻辑不在快照内未核验，未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
