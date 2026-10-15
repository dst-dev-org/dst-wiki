---
title: "charlie_tracker"
created: "2026-10-24"
updated: "2026-10-28"
type: "component"
summary: "世界实体上的服务端读取门面：IsCharlieDefeated 统一转发 shard_charlieinfo 的净变量，并在主分片负责击败状态的存档与读档重同步。"
aliases: ["查理是否被击败", "查理击败状态查询", "charlie_tracker"]
tags: ["component", "network-sync"]
dst_build: "steam-buildid:26230496"
sources: ["components/charlie_tracker.lua", "components/shard_charlieinfo.lua", "prefabs/world.lua", "prefabs/shard_network.lua", "shardnetworking.lua", "prefabs/charlie_boss_trial.lua", "components/combat.lua", "components/wisecracker.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# charlie_tracker

## 快速判断

- 职责与适用：挂在世界实体（world.lua 主端分支）上的查询门面：`IsCharlieDefeated()` 转发 `TheWorld.shard.components.shard_charlieinfo` 的净变量；主分片上额外用 `OnSave`/`LoadPostPass` 持久化击败标志。[components/charlie_tracker.lua:18](../../../raw/current/scripts/components/charlie_tracker.lua#L18 [prefabs/world.lua:597](../../../raw/current/scripts/prefabs/world.lua#L597
- 优先使用：玩法代码统一读 `TheWorld.components.charlie_tracker:IsCharlieDefeated()`；不要直接碰 shard_charlieinfo（挂在 shard_network 实体上的内部数据持有者，见 [[reference/components/shard_charlieinfo]]）。[components/charlie_tracker.lua:18](../../../raw/current/scripts/components/charlie_tracker.lua#L18
- 重要副作用：主分片读档后 `LoadPostPass` 调 `Shard_SyncCharlieDefeated(true)` 重新走一遍同步链（重建净变量并补发客户端成就 RPC）。[components/charlie_tracker.lua:36](../../../raw/current/scripts/components/charlie_tracker.lua#L36
- 修改边界：本组件无写入接口——击败状态的唯一原版写入点是 charlie_boss_trial 的 Boss 死亡回调；`OnSave`/`LoadPostPass` 仅在 `ismastershard` 分支定义（条件式函数定义）。[prefabs/charlie_boss_trial.lua:1080](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L1080 [components/charlie_tracker.lua:26](../../../raw/current/scripts/components/charlie_tracker.lua#L26

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `IsCharlieDefeated()` | boolean | 转发 `TheWorld.shard.components.shard_charlieinfo:IsCharlieDefeated()`；`TheWorld.shard` 指向 shard_network 实体（服务端存在）。[components/charlie_tracker.lua:18](../../../raw/current/scripts/components/charlie_tracker.lua#L18 [prefabs/shard_network.lua:23](../../../raw/current/scripts/prefabs/shard_network.lua#L23 |
| `OnSave()` | 仅主分片定义 | 已击败时返回 `{ isdefeated = true }`，否则 nil；从分片无此函数、世界存档不含该数据。[components/charlie_tracker.lua:26](../../../raw/current/scripts/components/charlie_tracker.lua#L26 |
| `LoadPostPass(newents, data)` | 仅主分片定义 | `data.isdefeated` 为真时调 `Shard_SyncCharlieDefeated(true)`：主分片上直接推送世界事件 `master_charlieinfoupdate` 完成重建。[components/charlie_tracker.lua:36](../../../raw/current/scripts/components/charlie_tracker.lua#L36 [shardnetworking.lua:309](../../../raw/current/scripts/shardnetworking.lua#L309 |

与 shard_charlieinfo 的分工：本组件（world 实体）＝只读门面＋主分片存档入口；shard_charlieinfo（shard_network 实体）＝净变量持有与跨分片同步（主分片消费 `master_charlieinfoupdate`、从分片收 dirty 后推 `secondary_charlieinfoupdate`）。[components/shard_charlieinfo.lua:53](../../../raw/current/scripts/components/shard_charlieinfo.lua#L53

已核验消费者（均服务端读 `IsCharlieDefeated`）：grue 击杀死亡原因改写 DARKNESS（[components/combat.lua:671](../../../raw/current/scripts/components/combat.lua#L671）；击败后台词切 `ANNOUNCE_GRUE_NOCHARLIE*`（[components/wisecracker.lua:105](../../../raw/current/scripts/components/wisecracker.lua#L105）；Boss 死亡动画二选一（[stategraphs/SGcharlie_boss.lua:1048](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L1048）；atrium_gate 冷却与重置分支（[prefabs/atrium_gate.lua:415](../../../raw/current/scripts/prefabs/atrium_gate.lua#L415）；charlie_hand 直接插钥匙或播过场（[prefabs/charlie_hand.lua:283](../../../raw/current/scripts/prefabs/charlie_hand.lua#L283）；死亡公告展示 CHARLIE 或 DARKNESS（[widgets/eventannouncer.lua:156](../../../raw/current/scripts/widgets/eventannouncer.lua#L156）。

## 典型接入

### 主分片监听击败状态同步，读档也能补触发

- 前提与选择：目标模组 `modmain.lua`；主分片监听世界事件 `master_charlieinfoupdate`（击杀与读档两条路径都会经过，见快速判断）。
- 生效与撤销：事件由 `Shard_SyncCharlieDefeated` 推送（从分片经 SHARD_RPC 转回主分片后同样触发）；仅服务端有效。

```lua
AddPrefabPostInit("world", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        inst:ListenForEvent("master_charlieinfoupdate", function(world, data)
            if data.isdefeated then
                print("[mymod] this save has defeated charlie")
            end
        end)
    end
end)
```

- 调整与验证：从分片侧的对应事件是 `secondary_charlieinfoupdate`，见 [[reference/components/shard_charlieinfo]]。

## 常见陷阱与限制

- **客户端无此组件**：world.lua 在 `ismastersim` 检查后才添加，客户端模组直接读会得到 nil。[prefabs/world.lua:568](../../../raw/current/scripts/prefabs/world.lua#L568 [prefabs/world.lua:597](../../../raw/current/scripts/prefabs/world.lua#L597
- **存档只记录已击败**：未击败时 `OnSave` 返回 nil，读档后数据为空；消费逻辑应把缺键当"未击败"处理。[components/charlie_tracker.lua:29](../../../raw/current/scripts/components/charlie_tracker.lua#L29
- **没有 setter**：直接改 shard_charlieinfo 的净变量会跳过事件与成就 RPC 流程；需要设定状态时沿用 `Shard_SyncCharlieDefeated` 链路。[components/shard_charlieinfo.lua:34](../../../raw/current/scripts/components/shard_charlieinfo.lua#L34

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖组件 44 行全部逻辑（含条件式函数定义）、shard_network 挂载、Shard_SyncCharlieDefeated 链路与上列 6 类消费者锚点；依据见上文。未覆盖各消费者完整行为与多分片实际同步时序。
- 游戏运行：未运行。
- 结构检查：2026-10-24 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/shard_charlieinfo]]、[[reference/components/atriumritualorgantracker]]（同源 ms_defeated_shrouden 链消费者）。
