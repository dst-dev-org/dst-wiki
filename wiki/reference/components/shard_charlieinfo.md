---
title: "shard_charlieinfo"
created: "2026-10-24"
updated: "2026-10-28"
type: "component"
summary: "shard_network 实体上的分片同步组件：持有查理击败状态的净变量，主分片消费更新事件、从分片收同步后转发世界事件并补发客户端成就 RPC。"
aliases: ["查理状态跨分片同步", "shard_charlieinfo"]
tags: ["component", "network-sync"]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_charlieinfo.lua", "prefabs/shard_network.lua", "shardnetworking.lua", "networkclientrpc.lua", "prefabs/charlie_boss_trial.lua", "components/charlie_tracker.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# shard_charlieinfo

## 快速判断

- 职责与适用：挂在 shard_network 实体（`TheWorld.shard`，服务端）上，持有净变量 `"shard_charlieinfo._isdefeated"`（dirty 事件 `ischarliedefeateddirty`）；查理击败状态的跨分片同步都在这里完成。[prefabs/shard_network.lua:42](../../../raw/current/scripts/prefabs/shard_network.lua#L42 [components/shard_charlieinfo.lua:21](../../../raw/current/scripts/components/shard_charlieinfo.lua#L21
- 优先使用：world 侧读状态走 [[reference/components/charlie_tracker]] 门面；从分片玩法代码监听世界事件 `secondary_charlieinfoupdate`（净变量 dirty 时推送）。[components/shard_charlieinfo.lua:41](../../../raw/current/scripts/components/shard_charlieinfo.lua#L41
- 重要副作用：置为已击败时分片各自向本分片全体客户端发 RPC `UpdateAccomplishment("charlie_defeated")`（客户端写 TheGenericKV 持久键）。[components/shard_charlieinfo.lua:30](../../../raw/current/scripts/components/shard_charlieinfo.lua#L30 [networkclientrpc.lua:1469](../../../raw/current/scripts/networkclientrpc.lua#L1469
- 修改边界：仅 master sim（构造 assert；shard_network 本身只在服务端创建）；设定状态不要直接 `set()` 净变量，走 `Shard_SyncCharlieDefeated` 链路。[components/shard_charlieinfo.lua:7](../../../raw/current/scripts/components/shard_charlieinfo.lua#L7 [shardnetworking.lua:309](../../../raw/current/scripts/shardnetworking.lua#L309

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `_isdefeated` | net_bool，dirty 事件 `ischarliedefeateddirty` | 主分片 set、从分片经网络同步；本组件的唯一状态。[components/shard_charlieinfo.lua:21](../../../raw/current/scripts/components/shard_charlieinfo.lua#L21 |
| `IsCharlieDefeated()` | boolean | 读 `_isdefeated:value()`；charlie_tracker 的转发目标。[components/shard_charlieinfo.lua:23](../../../raw/current/scripts/components/shard_charlieinfo.lua#L23 |
| `master_charlieinfoupdate`（世界事件，主分片监听） | data.isdefeated | 主分片上的更新入口：置净变量，已击败时补发成就 RPC。`Shard_SyncCharlieDefeated` 在主分片直接推此事件，从分片则经 SHARD_RPC 转到主分片再推。[components/shard_charlieinfo.lua:34](../../../raw/current/scripts/components/shard_charlieinfo.lua#L34 [shardnetworking.lua:309](../../../raw/current/scripts/shardnetworking.lua#L309 [networkclientrpc.lua:1620](../../../raw/current/scripts/networkclientrpc.lua#L1620 |
| `secondary_charlieinfoupdate`（世界事件，从分片推送） | data.isdefeated | 从分片净变量 dirty 时推送；当前快照无原版监听者，属预留/模组接入点。[components/shard_charlieinfo.lua:41](../../../raw/current/scripts/components/shard_charlieinfo.lua#L41 |

同步时机（关键流程）：

1. Boss 死亡（主分片）：charlie_boss_trial 回调 `Shard_SyncCharlieDefeated(true)`。[prefabs/charlie_boss_trial.lua:1080](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L1080
2. 主分片：推 `master_charlieinfoupdate` → 本组件 set 净变量、发本分片客户端成就 RPC。[components/shard_charlieinfo.lua:34](../../../raw/current/scripts/components/shard_charlieinfo.lua#L34
3. 从分片：净变量 dirty → 推 `secondary_charlieinfoupdate`、发该分片客户端成就 RPC。[components/shard_charlieinfo.lua:41](../../../raw/current/scripts/components/shard_charlieinfo.lua#L41
4. 主分片读档：charlie_tracker 的 `LoadPostPass` 重新触发第 2 步（本组件自身无存档，从分片状态完全靠净变量同步）。[components/charlie_tracker.lua:36](../../../raw/current/scripts/components/charlie_tracker.lua#L36

## 典型接入

### 从分片服务端响应主分片的查理击败同步

- 前提与选择：目标模组 `modmain.lua`；从分片只收同步不存档，监听 `secondary_charlieinfoupdate` 是唯一原版预留入口。
- 生效与撤销：事件带 `data.isdefeated`；主分片读档重同步时从分片也会再次收到 dirty。

```lua
AddPrefabPostInit("world", function(inst)
    if GLOBAL.TheWorld.ismastersim and not GLOBAL.TheWorld.ismastershard then
        inst:ListenForEvent("secondary_charlieinfoupdate", function(world, data)
            print("[mymod] secondary shard charlie defeated =", data.isdefeated)
        end)
    end
end)
```

- 调整与验证：`ismastershard` 由 world.lua 定义（`ismastersim and not TheShard:IsSecondary()`）；主分片侧对应接入见 [[reference/components/charlie_tracker]]。[prefabs/world.lua:431](../../../raw/current/scripts/prefabs/world.lua#L431

## 常见陷阱与限制

- **主分片直接 set 净变量会漏副作用**：主分片不注册 dirty 监听（只监听 `master_charlieinfoupdate`），直接 `set()` 只影响从分片的转发链，主分片自己的成就 RPC 不会发出。[components/shard_charlieinfo.lua:53](../../../raw/current/scripts/components/shard_charlieinfo.lua#L53
- **本组件无存档**：持久化在 charlie_tracker（仅主分片）的 OnSave/LoadPostPass；从分片重启后依赖主分片读档或再次同步恢复。[components/charlie_tracker.lua:26](../../../raw/current/scripts/components/charlie_tracker.lua#L26
- **客户端不可用**：shard_network 构造 assert ismastersim，客户端没有 `TheWorld.shard`；客户端感知击败靠成就 RPC 与各系统服务端表现。[prefabs/shard_network.lua:22](../../../raw/current/scripts/prefabs/shard_network.lua#L22

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖组件 65 行全部逻辑（两端监听注册拆分、成就 RPC）、shard_network 挂载、SHARD_RPC 与 CLIENT_RPC 处理器、charlie_boss_trial 写入点；依据见上文锚点。未覆盖多分片实际网络时序与 TheGenericKV 客户端落键行为。
- 游戏运行：未运行。
- 结构检查：2026-10-24 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/charlie_tracker]]。
