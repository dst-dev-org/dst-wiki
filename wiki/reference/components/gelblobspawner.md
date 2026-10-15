---
title: "gelblobspawner：暗影裂隙软泥怪生成管理"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界组件：暗影裂隙激活后在 gelblobspawningground 周期为玩家生成 gelblob 软泥怪，管理总上限、点冷却与裂隙关闭回收。"
aliases: [软泥怪生成器, gelblobspawner, 软泥怪管理, 裂隙软泥怪]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/gelblobspawner.lua", "prefabs/cave.lua", "prefabs/gelblob.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# gelblobspawner（暗影裂隙软泥怪生成管理）

## 快速判断

- 职责与适用：`cave` 世界挂载的服务端组件（[cave.lua:557](../../../raw/current/scripts/prefabs/cave.lua#L557），载入一帧后 `AddCaveGelSpawns` 撒点生成 `gelblobspawningground` 标记实体（([cave.lua:555-558](../../../raw/current/scripts/prefabs/cave.lua#L555-L558)、[撒点](../../../raw/current/scripts/prefabs/cave.lua#L479)）。世界事件 `ms_registergelblobspawningground`（由 [gelblob.lua:959](../../../raw/current/scripts/prefabs/gelblob.lua#L959 推送）登记生成点；`riftspawner` 暗影门户激活时 `StartGelBlobs`，关闭时 `StopGelBlobs` 并逐只回收（[裂隙联动](../../../raw/current/scripts/components/gelblobspawner.lua#L32-L41)）。
- 优先使用：调整数量与节奏 → 改调谐值（每点 5~10 只、全图上限 100、冷却 5 天，([tuning.lua:8362-8370](../../../raw/current/scripts/tuning.lua#L8362-L8370)）；模组要注入生成点 → `TryToRegisterSpawningPoint(spawnpoint)`（源内注明"为模组保留"）。
- 重要副作用：生成受 `shadowparasitemanager:OverrideBlobSpawn` 劫持——被劫持时不生成但消耗该点冷却（[劫持](../../../raw/current/scripts/components/gelblobspawner.lua#L121-L126)）。
- 修改边界：整个组件在 `TUNING.SPAWN_GELBLOBS`（世界设置）为假时不注册任何事件（[总开关](../../../raw/current/scripts/components/gelblobspawner.lua#L27)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TryToRegisterSpawningPoint(spawnpoint)` | 实体 → 布尔 | 去重后 `RegisterGelBlobSpawningPoint` 入表并监听其移除（[实现](../../../raw/current/scripts/components/gelblobspawner.lua#L55-L62)）。 |
| `StartGelBlobs()`／`StopGelBlobs()` | 无参 | 置 `enabled` 并启停更新循环（[开始](../../../raw/current/scripts/components/gelblobspawner.lua#L92-L103)）。 |
| `SpawnGelBlobFromSpawner(spawner, player)` | 实体、实体或 nil → 布尔 | 该点未占用且未冷却才生成；数量 `random(MIN, MAX_GELBLOBS_PER_SPAWNER)`，超全图上限直接失败；位置在点周围 `MAX_GELBLOB_DIST_FROM_SPAWNER`(12) 内 `FindWalkableOffset` 搜索且彼此间距 ≥3（[生成](../../../raw/current/scripts/components/gelblobspawner.lua#L116-L192)）。 |
| `IsGelBlobbed(spawner)`／`IsCooldowned(spawner)` | 实体 → 布尔 | 查占用/冷却表（[查询](../../../raw/current/scripts/components/gelblobspawner.lua#L105-L111)）。 |
| `TryRemovingGelBlobs()` | 无参 | 取一只存活 gelblob 调 `DoDespawn`（[回收](../../../raw/current/scripts/components/gelblobspawner.lua#L209-L212)）。 |
| `OnUpdate(dt)` | 组件更新 | 递减点冷却；每 1s 逻辑跳调 `CheckGelBlobs`（未满生成/超编回收）；`LongUpdate` 同实现（[更新](../../../raw/current/scripts/components/gelblobspawner.lua#L232-L243)）。 |
| `OnSave/LoadPostPass` | 存读钩子 | 存 gelblob↔spawner GUID 对与点冷却；读回重建监听与冷却（[存读](../../../raw/current/scripts/components/gelblobspawner.lua#L247-L298)）。 |

生成逻辑 `CheckGelBlobs`（[实现](../../../raw/current/scripts/components/gelblobspawner.lua#L214-L230)）：enabled 且未达上限 → 对每个存活玩家在 12 单位内找 `gelblobspawningground` 标签实体尝试生成；disabled → 每次逻辑跳回收一只，清零后停更新并清冷却表。

## 典型接入

### 模组注入自定义生成点

- 前提与选择：服务端、洞穴世界（组件在 cave 挂载）；生成点须是任意有 Transform 的实体，组件只在其位置搜点。
- 生效与撤销：注册即生效；实体被移除时经监听自动出表（[出表](../../../raw/current/scripts/components/gelblobspawner.lua#L45-L53)）。

```lua
-- 服务端，洞穴世界可用后
local ground = SpawnPrefab("gelblobspawningground")  -- 自带注册事件
ground.Transform:SetPosition(x, 0, z)
-- 或对已有实体直接登记（不走事件，可自定义标记实体）：
TheWorld.components.gelblobspawner:TryToRegisterSpawningPoint(ground)
```

- 调整与验证：gelblob 落地后进 `spawndelay` 状态延迟 0.25~1.25s（[调谐](../../../raw/current/scripts/tuning.lua#L8368-L8369)）；裂隙关闭时每秒回收一只而非瞬间清场。

## 常见陷阱与限制

- **只在洞穴世界工作**：挂载点在 cave prefab；地表世界没有该组件（[挂载](../../../raw/current/scripts/prefabs/cave.lua#L557)）。
- **玩家存活判定**：`TrySpawningGelBlobs` 跳过死亡与鬼魂玩家（[筛选](../../../raw/current/scripts/components/gelblobspawner.lua#L196-L206)），无活人时不生成。
- **冷却按"该点已无软泥怪"触发**：某点的最后一只被移除才进入 `COOLDOWN_GELBLOB_SPAWNER_TIME`(5 天) 冷却，占用中不受冷却限制（[冷却入表](../../../raw/current/scripts/components/gelblobspawner.lua#L85-L88)）。
- **总上限先于单点数量**：`GetGelBlobCount() + spawncount > MAX_GELBLOBS_TOTAL_IN_WORLD` 直接返回 false，点内不产出（[上限](../../../raw/current/scripts/components/gelblobspawner.lua#L129-L131)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/gelblobspawner.lua` 全部 320 行、cave 挂载与 AddCaveGelSpawns 撒点段、gelblob.lua 的 gelblobspawningground 注册、tuning GELBLOB 块。未覆盖：gelblob prefab 行为、shadowparasitemanager.OverrideBlobSpawn 内部。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
