---
title: "autosaver"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在世界网络实体上的自动存档调度器：主机分片按事件与每日周期排 1 秒延迟真实存档，次分片经快照号与主机同步（必要时回滚重启），客户端顺带序列化用户会话并驱动 HUD 保存提示。"
aliases: ["autosaver", "自动存档", "世界存档调度", "AutoSaver"]
tags: [component, network, server-client, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/autosaver.lua", "prefabs/world_network.lua", "components/shard_autosaver.lua", "shardnetworking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# autosaver

## 快速判断

- 职责与适用：挂在 `world_network`（以及 lavaarena／quagmire 的网络实体）上的存档调度组件，三种语境各管一段：主机分片（mastershard）响应 `ms_save` 与开关并落地存档；次分片（mastersim）向主机请求存档并按快照号对齐；纯客户端序列化自己的用户会话。[挂载](../../../raw/current/scripts/prefabs/world_network.lua#L56 [语境分支](../../../raw/current/scripts/components/autosaver.lua#L190-L210)
- 优先使用：想让世界"现在存一次" → 世界事件 `ms_save`（可带 mintime）；读上次存档时间 → `GetLastSaveTime()`。[保存事件](../../../raw/current/scripts/components/autosaver.lua#L103-L117)
- 重要副作用：真实存档延迟 1 秒执行（`DoTaskInTime(1)`），期间 `TheNet:SetIsWorldSaving(true)`；net 变量 `autosaver._issaving` 脏事件驱动 HUD 保存图标并让客户端同步存会话。[调度](../../../raw/current/scripts/components/autosaver.lua#L89-L97) [HUD](../../../raw/current/scripts/components/autosaver.lua#L134-L149)
- 修改边界：快照错位超过 1 时主机分片不回滚自身、次分片用 `TruncateSnapshots`＋`StartNextInstance` 重启回滚；`_restarting` 后忽略一切存档请求。[回滚](../../../raw/current/scripts/components/autosaver.lua#L45-L57)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | 建 net_bool `autosaver._issaving`（脏事件 `issavingdirty`）；因网络实体 fn 在两端同样执行（([world_network.lua:50-56](../../../raw/current/scripts/prefabs/world_network.lua#L50-L56)），客户端也持有组件实例与该 netvar。 |
| `ms_save`（监听，世界事件） | `src, mintime, snapshot` | 距开场不足 `mintime` 秒或正在重启时忽略；主机分片走 `ScheduleActualSave` 并推 `master_autosaverupdate`，次分片改发 `SendWorldSaveRequestToMaster`，客户端直接 `SerializeUserSession(ThePlayer)`。[OnSave](../../../raw/current/scripts/components/autosaver.lua#L103-L117) |
| `ms_setautosaveenabled`（监听） | `src, enable` | 仅主机分片；切换是否按周期自动存档（引擎设置界面推送，脚本内无推送点）；开启时 WatchWorldState("cycles")。[OnSetAutoSaveEnabled](../../../raw/current/scripts/components/autosaver.lua#L123-L132) |
| 周期自动存档 | `OnCyclesChanged` | 每天变更时调 `OnSave(nil, 60)`：距上次实际存档不足 60 秒则跳过。([autosaver.lua:119-121](../../../raw/current/scripts/components/autosaver.lua#L119-L121) |
| `ScheduleActualSave(snapshot)` | 私有闭包 | 置世界保存标记，1 秒后 `DoActualSave`：写 `ShardGameIndex:SaveCurrent()` 并刷新 `_lastsavetime`；按传入快照号做前向同步或回滚判定。([autosaver.lua:59-97](../../../raw/current/scripts/components/autosaver.lua#L59-L97) |
| 快照同步链 | 跨分片 | 主机 autosaver 推 `master_autosaverupdate` → 主机 [[reference/components/shard_autosaver]] 的 net_uint → 次分片 shard_autosaver 推 `secondary_autosaverupdate` → 次分片 autosaver 前向对齐或回滚重启。([shard_autosaver.lua:23-43](../../../raw/current/scripts/components/shard_autosaver.lua#L23-L43) [OnAutoSaverUpdate](../../../raw/current/scripts/components/autosaver.lua#L162-L181) |
| `GetLastSaveTime()` | 返回 number | 本机最近一次实际写档（或客户端会话序列化）的 `GetTime()`。([autosaver.lua:216-218](../../../raw/current/scripts/components/autosaver.lua#L216-L218) |

## 典型接入

### 触发一次受控的世界存档

- 前提与选择：脚本侧向世界推 `ms_save`；需要"距开场至少 N 秒"的节流用第二个参数，只有主机分片的调用会真正写档，次分片与客户端自动走各自分支。
- 生效与撤销：延迟 1 秒生效，重复推送各自排队（`_savetasks` 按序号）；无法撤销，`_restarting` 状态下被忽略。
- 原版依据：控制台 `c_save()`、调试键与分片管理器都推此事件（[consolecommands.lua:174](../../../raw/current/scripts/consolecommands.lua#L174)、[debugkeys.lua:1327](../../../raw/current/scripts/debugkeys.lua#L1327)、[shardnetworking.lua:159](../../../raw/current/scripts/shardnetworking.lua#L159）。

```lua
-- 任意服务端脚本（建议仅主机分片执行时才有写档效果）：
TheWorld:PushEvent("ms_save")
-- 带节流：开场 120 秒内的推送被忽略
TheWorld:PushEvent("ms_save", 120) -- 位置参数，见 OnSave(src, mintime, snapshot)
```

- 调整与验证：推送后 1 秒在主机分片写档；`GetLastSaveTime()` 可用于确认；客户端只表现为会话序列化，不产生世界存档文件。

## 常见陷阱与限制

- 存档是"延迟 1 秒"的：推 `ms_save` 后立刻读档或断言文件已更新会失败（([autosaver.lua:95](../../../raw/current/scripts/components/autosaver.lua#L95）。
- 分片快照错位会触发回滚重启：次分片发现自己落后主机超过 1 个快照时 `TruncateSnapshots`＋重启（([autosaver.lua:75-82](../../../raw/current/scripts/components/autosaver.lua#L75-L82)），模组在次分片上做未持久化的状态改动会在该过程中丢失。
- `ms_setautosaveenabled` 的周期自动存档只在"主机分片"生效（[L190-199](../../../raw/current/scripts/components/autosaver.lua#L190-L199)），单机非分片（非 dedicated 的 host 即 mastershard）同属该分支；客户端永远只存自己的会话。
- 组件挂在 `SetPristine` 之后但仍于两端构造：其行为依赖"网络实体 fn 在所有机器一致执行"这一前提，把它照搬到普通实体的 prefab 上不会获得 netvar 同步。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 world_network／lavaarena_network／quagmire_network 三处挂载、shard_autosaver 快照中继与 ms_save 三个脚本推送点。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/shard_autosaver]]
