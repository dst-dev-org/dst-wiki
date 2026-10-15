---
title: "worldreset：世界重置倒计时与执行"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在 world_network 上的世界重置组件：全员变鬼或空服时按难度开启倒计时并同步各端，到点执行存档删除重开。"
aliases: [世界重置, 重置倒计时, worldreset, 世界重生倒计时]
tags: [component, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/worldreset.lua", "prefabs/world_network.lua", "worldsettings_overrides.lua", "networking.lua", "widgets/worldresettimer.lua", "widgets/playerdeathnotification.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldreset（世界重置倒计时与执行）

## 快速判断

- 职责与适用：世界级组件，由 `world_network` prefab 挂载（[world_network.lua:60](../../../raw/current/scripts/prefabs/world_network.lua#L60），主分片权威、从分片与客户端经 net_byte 同步倒计时。世界设置 `reset_time` 开启后，"全员皆鬼"触发倒计时（默认 120s，读档期 180s），全端弹出重置提示，到点删档重开。
- 优先使用：要开关/改重置难度 → 用世界设置覆盖 `reset_time`（none/slow/default/fast/always），其实现推送 `ms_setworldresettime`（([worldsettings_overrides.lua:3845-3860](../../../raw/current/scripts/worldsettings_overrides.lua#L3845-L3860)）；模组不要直接改组件内部状态。
- 重要副作用：倒计时开启即 `TheNet:SetIsWorldResetting(true)`，只剩活人时取消；HUD 经 `showworldreset`/`hideworldreset`/`worldresettick` 三个世界事件驱动（[UI 监听](../../../raw/current/scripts/widgets/worldresettimer.lua#L53-L54)）。
- 修改边界：`instant` 档（always）无倒计时，全员在场即直接重置；倒计时不写档，读档后按玩家计数与设置重新起算。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnPostInit()` | 载入后调用 | 重放一次 net 值处理，非主模拟端倒计时>0 时补 4 秒 dt 对齐（[OnPostInit](../../../raw/current/scripts/components/worldreset.lua#L249-L256)）。 |
| `OnUpdate(dt)` | 组件更新 | 本地推进 `_countdownf`；主分片每 `_syncperiod`（>10s 每 5s，之后每 2s）set 网络值，到 0 时主分片执行 `WorldReset()`（[更新](../../../raw/current/scripts/components/worldreset.lua#L262-L296)）。 |
| net_byte `_countdown` | `worldreset._countdown`，dirty 事件 `countdowndirty` | 唯一同步通道；非主端只在整秒降档时本地推进（[网络变量](../../../raw/current/scripts/components/worldreset.lua#L41)、[监听](../../../raw/current/scripts/components/worldreset.lua#L216)）。 |

流程与事件（均核验自组件内部注册）：

1. 设置接入：主分片监听 `ms_setworldresettime`，记录 `_instant/_countdownmax/_countdownloadingmax` 并挂 `ms_playercounts`（[OnSetWorldResetTime](../../../raw/current/scripts/components/worldreset.lua#L185-L205)）。
2. 触发：玩家计数显示全员皆鬼且倒计时未启用 → `SetIsWorldResetting(true)`，倒计时取 `loadingmax`（空服刚载入）或 `max`，上限 255（[OnPlayerCounts](../../../raw/current/scripts/components/worldreset.lua#L160-L183)）；`instant` 档直接 `WorldReset()`。出现活人即 `CancelCountdown`（[取消](../../../raw/current/scripts/components/worldreset.lua#L107-L124)）。
3. 执行：主分片专用服 `TheNet:SendWorldResetRequestToServer()`，客户端主机走 `TheFrontEnd:Fade` 后重置；`ms_worldreset`（由 `networking.lua` 的 `WorldResetFromSim` 推送，([networking.lua:722-725](../../../raw/current/scripts/networking.lua#L722-L725)）置 `_resetting` 停止后续倒计时（[WorldReset](../../../raw/current/scripts/components/worldreset.lua#L91-L101)）。
4. 分片同步：从分片监听 `secondary_worldresetupdate` 直接镜像倒计时（[镜像](../../../raw/current/scripts/components/worldreset.lua#L207-L209)）。

## 典型接入

### 服务端调整重置难度

- 前提与选择：世界设置层（modinfo 的 configuration_options 或服务端设置映射到 `worldsettings_overrides`）；组件自身无公开 setter。
- 生效与撤销：难度函数按 `{ time, loadingtime }` 或 `{ instant = true }` 推送 `ms_setworldresettime`，组件自动挂/摘 `ms_playercounts`；传 `nil`（"none"）即关闭。

```lua
-- 服务端，世界可用后（对照 worldsettings_overrides.lua 的 resettime 实现）
TheWorld:PushEvent("ms_setworldresettime", { time = 120, loadingtime = 180 })
-- 立即重置（always 档语义）
TheWorld:PushEvent("ms_setworldresettime", { instant = true })
-- 关闭
TheWorld:PushEvent("ms_setworldresettime", nil)
```

- 调整与验证：time/loadingtime 单位为秒；上限 255（net_byte 上限），超出被钳制（[钳制](../../../raw/current/scripts/components/worldreset.lua#L178)）。

## 常见陷阱与限制

- **只有主分片（`ismastershard`）有权威逻辑**：`OnPlayerCounts`、`WorldReset` 等处理函数以 `_ismastershard` 条件构造，从分片只是倒计时镜像（[分支构造](../../../raw/current/scripts/components/worldreset.lua#L160-L183)）。
- **挂载点在 world_network 而非 world**：`TheWorld.components.worldreset` 在 world_network prefab 上注册，森林/洞穴世界实体本身不挂（[挂载](../../../raw/current/scripts/prefabs/world_network.lua#L60)）。
- **专用服无淡出**：`TheNet:IsDedicated()` 为真时跳过 `TheFrontEnd:Fade`，直接发重置请求（[执行分支](../../../raw/current/scripts/components/worldreset.lua#L96-L100)）。
- **无存读钩子**：组件不实现 OnSave/OnLoad，倒计时中断后由玩家计数重新触发；HUD 计时条（worldresettimer）与死亡界面（playerdeathnotification）各自监听同一组事件（[死亡界面监听](../../../raw/current/scripts/widgets/playerdeathnotification.lua#L95-L96)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/worldreset.lua` 全部 303 行、world_network 挂载行、worldsettings_overrides 的 reset_time 实现、networking.lua 重置请求链、两个 HUD widget 的事件监听。未覆盖：ShardGameIndex 删档细节、前端 Fade 表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
