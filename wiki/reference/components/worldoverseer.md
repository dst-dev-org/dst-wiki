---
title: "worldoverseer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Klei 遥测统计组件：周期性汇总玩家游玩时长、穿戴皮肤、合成物与房间装备情况，经 SendProfileStats 上报；仅 METRICS_ENABLED 构建安装。"
aliases: ["worldoverseer", "遥测统计", "world overseer", "统计上报"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/worldoverseer.lua", "prefabs/cave.lua", "prefabs/forest.lua", "main.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldoverseer

## 快速判断

- 职责与适用：世界组件，只做数据统计与上报（TheSim:SendProfileStats），无任何游戏玩法接口；仅当 `METRICS_ENABLED` 为真时在森林与洞穴 master_postinit 安装（正式客户端 main.lua 恒为 true）。[安装](../../../raw/current/scripts/prefabs/cave.lua#L547-L549) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L720-L722) [开关](../../../raw/current/scripts/main.lua#L70
- 优先使用：无对外用例——模组不需要也不会受益于它；只想挂自己的统计可参考其事件监听集合（death/changeclothes/builditem/equipskinneditem 等）。[玩家监听](../../../raw/current/scripts/components/worldoverseer.lua#L328-L336)
- 重要副作用：构造起两个周期任务——每 5 分钟真实时间 Heartbeat（上报玩家与房间统计）、每 5 秒 HeartbeatPoll（仅主分片，追踪玩家会话 v2）；玩家加入即挂 6 类监听。[心跳](../../../raw/current/scripts/components/worldoverseer.lua#L355-L358) [会话轮询](../../../raw/current/scripts/components/worldoverseer.lua#L362-L417)
- 修改边界：无 OnSave/OnLoad/OnRemoveFromEntity，纯内存统计随会话丢弃；时间源在控制台平台用 GetTime、其余用 GetTimeReal（避免主机挂起计时失真）。[时间源](../../../raw/current/scripts/components/worldoverseer.lua#L6-L12)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Heartbeat()`（每 300s） | 内部 | DumpPlayerStats + DumpSessionStats：按玩家分段结算游玩秒数、累计穿戴皮肤时长（worn_items）、本段合成（crafted_items）与当前库存，再附房间元信息（人数/模式/密码/PVP/mod 状态等）上报。[玩家统计](../../../raw/current/scripts/components/worldoverseer.lua#L152-L193) [会话统计](../../../raw/current/scripts/components/worldoverseer.lua#L287-L326) |
| `HeartbeatPoll()`（每 5s，仅 mastershard） | 内部 | 以 os.time 维护 `_v2_seenplayers` 会话表：新玩家发 SESSION_BEGIN、退出发 SESSION_END、挂起超 15 分钟重开会话、累计 5 分钟发 HEARTBEAT。[会话轮询](../../../raw/current/scripts/components/worldoverseer.lua#L362-L417) |
| `OnPlayerDeath(player, data)`（death 监听） | 内部 | 上报 player.death：玩家年龄天数、世界天数、死因（cause，玩家击杀时为角色 prefab）。[死亡上报](../../../raw/current/scripts/components/worldoverseer.lua#L195-L213) |
| 皮肤/合成监听 | changeclothes / buildstructure / builditem / equipskinneditem / unequipskinneditem | 维护 worn_items 时间段与 crafted_items 计数；加入时手动补录初始穿戴（初始化早于监听）。[监听注册](../../../raw/current/scripts/components/worldoverseer.lua#L330-L347) |
| `GetWorldRecipeItems()` | 内部 | 遍历 Ents 统计 AllRecipes 命中实体与 preparedfood，供会话心跳附带房间物品概况。[房间物品](../../../raw/current/scripts/components/worldoverseer.lua#L273-L285) |
| `QuitAll()` | 内部 | 给所有 v2 会话补发 SESSION_END（关服路径）。[收尾](../../../raw/current/scripts/components/worldoverseer.lua#L420-L424) |

## 典型接入

### 参考其事件模式做模组侧统计

- 前提与选择：本组件无公开服务价值；模组做自己的统计时，可仿照它在 `ms_playerjoined` 时对玩家挂 death／changeclothes／builditem 监听、在离开时结算的结构，但应使用自己的存储与上报。
- 生效与撤销：组件常驻世界、不可卸载（无 OnRemoveFromEntity）；强行移除组件会留下已注册的世界事件监听（ms_playerjoined 等），这是它的实现边界而非接入指引。[注册](../../../raw/current/scripts/components/worldoverseer.lua#L23-L26)
- 原版依据：仅 METRICS_ENABLED 分支安装的既成事实——统计失败或剔除该组件不影响玩法。[安装分支](../../../raw/current/scripts/prefabs/cave.lua#L547-L549)

```lua
-- 仅供参考：读取世界是否有遥测组件（模组侧一般无需与其交互）
local function HasTelemetry()
    return TheWorld.components.worldoverseer ~= nil
end
```

- 调整与验证：无参数可调；上报行为依赖 Klei 后端，模组环境不可复现。

## 常见陷阱与限制

- **统计时间轴不进存档：** 会话中途重载世界后 `_seenplayers`/`_v2_seenplayers` 全空，重载前未结算的游玩时长丢失；这不是 bug 而是实现现状。[无存读](../../../raw/current/scripts/components/worldoverseer.lua#L1-L35)
- **HeartbeatPoll 只在主分片跑：** 从分片的 v2 会话完全依赖主分片视角（TheNet 客户端表），跨分片玩家的加入/退出只按主分片可见性记录。[分片门禁](../../../raw/current/scripts/components/worldoverseer.lua#L363-L365)
- **构造即重放在线玩家：** 组件创建时把 AllPlayers 逐个 OnPlayerJoined；晚于组件创建的动态玩家靠 ms_playerjoined。在这两个窗口之间挂自己的逻辑会漏人。[重放](../../../raw/current/scripts/components/worldoverseer.lua#L28-L30)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 worldoverseer.lua，核验心跳/轮询任务、统计字段、事件监听集与无存读事实；核对 forest/cave 的 METRICS_ENABLED 安装与 main.lua 开关。Stats 模块（stats.lua）的上下文表构建未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；上报内容与频率未实测（依赖 Klei 后端）。

## 相关链接

- [[reference/components/autosaver]]：同属世界后台组件的存档侧对照。
