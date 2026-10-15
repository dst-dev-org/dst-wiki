---
title: "inspectaclesparticipant"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "温妮观测镜小游戏参与端：为玩家随机布置观测镜谜题箱，经 player_classified 网络变量同步到客户端，答题后发奖并进入冷却。"
aliases: ["观测镜小游戏", "inspectacles", "inspectaclesparticipant 组件"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/inspectaclesparticipant.lua", "prefabs/winona.lua", "prefabs/hats.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inspectaclesparticipant

## 快速判断

- 职责与适用：仅挂在温妮玩家上（prefabs/winona.lua:180），客户端与服务端都在；服务端负责选局、放箱、发奖与冷却，客户端只缓存谜题数据并响应信号。
- 优先使用：触发新局/显示 → `CreateNewAndOrShowCurrentGame()`（:322-344，帽子信号消费，hats.lua:4819 起）；客户端取局 → `GetCLIENTDetails`／`GetSERVERDetails`（:183-189）。
- 重要副作用：游戏 id 与坐标经 `player_classified.inspectacles_game/_posx/_posz` 网络变量同步（:199-219），dirty 事件在 player_classified.lua:496 桥接回本组件。
- 修改边界：洞穴世界不出局（`CanCreateGameInWorld`，:318-320）；`hide` 或跨 shard 时网络侧清零（`ShouldStopGameInteractions`，:221-223）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateNewAndOrShowCurrentGame` | `() -> boolean` | 冷却中或洞穴直接 false；无局时随机 gameid（1..#INSPECTACLES_GAMES_LOOKUP）、`FindGameLocation` 选点并 clamp 到 ±32767（:328-335）；温妮技能 `winona_wagstaff_2` 激活时按 0.25 几率升级箱（:337-338，TUNING.SKILLS.WINONA.INSPECTACLES_UPGRADE_CHANCE，tuning.lua:7233）。 |
| `SetCurrentGame`／`NetworkCurrentGame` | `(gameid, posx, posz) -> nil` | 算谜题种子并同步；`set_local(0)` 强制重复局也推事件（:209）。:225-248、:199-219 |
| `FinishCurrentGame` | `() -> nil` | `GrantRewards`（box:DoLootPinata，:367-374）→ 清局 → `ApplyCooldown`（默认 `INSPECTACLES_COOLDOWNTIME` = 半天，tuning.lua:7231）。:346-355 |
| `CalculateGamePuzzle` | `(gameid, posx, posz) -> number` | tile 坐标拼 24 位种子，主客端可独立算出同一谜题（:66-77）；`GetPuzzleData` 展开 12 个半字节并均匀注入 0..3（:98-120），环形缓冲见 :89-96。 |
| `CheckGameSolution` | `(solution) -> boolean` | 仅主机；当前实现只认 `solution == 0`（:122-131，注释 [IPGVR] 声明未做完整校验）。 |
| `IsParticipantClose` | `(range?) -> boolean` | 默认 4 距离内（:164-181），hats.lua:4822 用作显示条件。 |
| `OnSignalPulse` | `() -> nil` | 客户端推 `inspectaclesping`（:156）；服务端确保箱存在（hats.lua:4752-4757 触发）。:151-162 |
| `OnSave`/`OnLoad` | 常规存取 | 局数据或冷却剩余时间；载入时 `shardid` 缺省当前 shard（:456-491）。 |

## 典型接入

### 自定义玩家参与观测镜玩法（读状态为主）

- 前提与选择：只对温妮存在（winona.lua:180），其他角色读 `components.inspectaclesparticipant` 须判 nil；局名映射见 constants.lua:2910-2918（`NONE=0`、`WIRES=1`、`FREE1=2`）。
- 生效与撤销：谜题箱 prefab 为 `inspectaclesbox`/`inspectaclesbox2`（:288），生成前做 2 半径避障随机走位（:290-304）；免费局（`"NONE"` 或 `"FREE"` 前缀）直接 `SetRepaired`（:306-308、:191-194）。
- 原版依据：prefabs/hats.lua:4752-4757、:4811-4835（帽子按局态切图标与等级）。

```lua
local function GetInspectaclesState(player)
    local part = player.components.inspectaclesparticipant
    if part == nil then
        return nil
    end
    local game, puzzle, puzzledata = part:GetSERVERDetails()
    return game ~= nil and game or nil
end
```

- 调整与验证：客户端分支仅当 `self.inst == ThePlayer` 缓存（:134-148）；答案提交经 `POPUPS.INSPECTACLES` 弹窗关闭回调（:57-64）。

## 常见陷阱与限制

- `LongUpdate` 双定义：:445-454 与 :493-504 重复定义，后者覆盖前者；后者在剩余时间 >0 时引用未定义局部 `cooldowntime`（:498），该路径会报错——睡眠跳时触发冷却恢复时需留意。
- 组件移除清理有误：`OnRemoveFromEntity` 对 `"inspectaclesgamechanged"` 移除的是 `OnClosePopup_Bridge`（:39），真正的 `OnInspectaclesGameChanged_Bridge` 监听不会被移除。
- 解题校验是空壳：任何非 0 的 solution 都判失败（:130），mod 侧勿指望服务端验证谜题逻辑。
- 谜题种子仅 24 位且绑定 tile 坐标（:73-76），同格同局 id 必然同谜题；注释明确低 24 位内使用（:71）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 524 行；挂载 winona.lua:180、帽子消费 hats.lua:4752-4835、局表 constants.lua:2910-2919、netvar player_classified.lua:496/:1434、TUNING tuning.lua:7231/:7233 逐行核对。
- 游戏运行：未运行。
