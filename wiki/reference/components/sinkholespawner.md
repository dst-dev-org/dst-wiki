---
title: "sinkholespawner：蚁狮地陷袭击调度"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "蚁狮震怒后在地面玩家附近选目标发起地陷：先警告震动与碎石，再按波次生成 antlion_sinkhole 陷坑，支持跨分片远程同步。"
aliases: [地陷生成器, 蚁狮地陷, sinkholespawner, 蚁狮袭击]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sinkholespawner.lua", "prefabs/antlion.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sinkholespawner（蚁狮地陷袭击调度）

## 快速判断

- 职责与适用：挂在 antlion 实体上的服务端组件（[antlion.lua:472](../../../raw/current/scripts/prefabs/antlion.lua#L472）。`StartSinkholes` 按玩家年龄加权选 1~2 名地面玩家为目标，先做 `NUM_WARNINGS`(12) 次镜头震动＋`sinkhole_warn_fx` 预警，再按剩余波次在目标附近生成 `antlion_sinkhole` 并推 `startcollapse`（[攻击](../../../raw/current/scripts/components/sinkholespawner.lua#L122-L140)）。
- 优先使用：由蚁狮状态机驱动——收贡/满足时 `StopSinkholes`，震怒时 `StartSinkholes`（[开袭](../../../raw/current/scripts/prefabs/antlion.lua#L168)、[停袭](../../../raw/current/scripts/prefabs/antlion.lua#L109)）；不要绕过蚁狮直接调。
- 重要副作用：开袭/结束推世界事件 `onsinkholesstarted`/`onsinkholesfinished`；目标玩家位置每帧刷新，跨分片玩家按 userid→userhash 推 `master_sinkholesupdate` 给对端表现（[远程推送](../../../raw/current/scripts/components/sinkholespawner.lua#L181-L195)）。
- 修改边界：同批最多 `MAX_TARGETS`(2) 个目标；洞穴迁移中的玩家只保留上次位置（`TheShard:IsMigrating`，[位置更新](../../../raw/current/scripts/components/sinkholespawner.lua#L75-L88)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartSinkholes()` | 无参 | 用 `TheNet:GetClientTable()` 以 `sqrt(playerage)` 加权随机选目标（>3 人选 2），波次数 = `min(WAVE_MIN_ATTACKS + floor((age-1)/年长)×2 + 季末1, WAVE_MAX_ATTACKS)`，开更新循环（[实现](../../../raw/current/scripts/components/sinkholespawner.lua#L16-L63)）。 |
| `StopSinkholes()` | 无参 | 清空目标、停更新、推 `onsinkholesfinished`（[实现](../../../raw/current/scripts/components/sinkholespawner.lua#L65-L73)）。 |
| `OnUpdate(dt)` | 组件更新 | 刷新目标位置→到点的警告/攻击入队→`PushRemoteTargets`→逐个执行；清零后自动停更新（[实现](../../../raw/current/scripts/components/sinkholespawner.lua#L197-L246)）。 |
| `SpawnSinkhole(spawnpt)` | 世界坐标 | 原点±`RADIUS`(2.5) 随机点，验证失败按 1.8/2.9/3.9 倍半径扇形重试：禁临时地皮（农田除外）、避让 `antlion_sinkhole_blocker` 实体、3×3 倍半径须可通行；成功生成并 `PushEvent("startcollapse")`（[选点](../../../raw/current/scripts/components/sinkholespawner.lua#L143-L179)）。 |
| `OnSave/OnLoad` | 存读钩子 | 只存已有位置目标的 `{x, z, attacks, next_attack, next_warning}`；读回即恢复更新循环（[存读](../../../raw/current/scripts/components/sinkholespawner.lua#L248-L292)）。 |

调谐值（([tuning.lua:3288-3300](../../../raw/current/scripts/tuning.lua#L3288-L3300)）：`WAVE_MAX_ATTACKS=5`、`WAVE_MIN_ATTACKS=2`、`WAVE_ATTACK_DELAY=0.75±1`、`NUM_WARNINGS=12`、`WARNING_DELAY=1±0.3`、`RADIUS=2.5`。

## 典型接入

### 跟随蚁狮仇恨开袭（原版路径）

- 前提与选择：组件在 antlion prefab 构造期挂载；`StartSinkholes`/`StopSinkholes` 由其 tribute/ rage 逻辑调用，模组应通过给蚁狮送礼或修改其状态间接驱动。
- 生效与撤销：开袭后每个目标独立走"警告→攻击"计时；警告期内目标离线则该目标作废，波次清零（[目标失效](../../../raw/current/scripts/components/sinkholespawner.lua#L136-L139)）。

```lua
-- 服务端，调试用途：手动开/停一轮地陷袭击（对照 antlion.lua 的调用）
local antlion = TheSim:FindFirstEntityWithTag("antlion")
if antlion and antlion.components.sinkholespawner then
    antlion.components.sinkholespawner:StartSinkholes()
    -- antlion.components.sinkholespawner:StopSinkholes()
end
```

- 调整与验证：目标必须是"当前在地面"的玩家（`IsVisualGroundAtPoint`），纯海洋/洞穴玩家不入选（[筛选](../../../raw/current/scripts/components/sinkholespawner.lua#L19-L24)）。

## 常见陷阱与限制

- **警告与攻击的时序交错**：同一目标先消耗 `next_warning` 计时做完全部警告，才转入 `next_attack` 攻击计时；两计时互斥（[计时分支](../../../raw/current/scripts/components/sinkholespawner.lua#L216-L227)）。
- **远程目标只播一次效果**：目标位置为 nil（跨分片）时警告/攻击只经 `PushRemoteTargets` 通知一次并把 `attacks` 清零，不实际生成陷坑（[远程分支](../../../raw/current/scripts/components/sinkholespawner.lua#L136-L138)）。
- **警告台词与震动只在本地有位置时播**：`DoTargetWarning` 对 `targetinfo.pos` 判空，跨分片目标本地无震动（[警告实现](../../../raw/current/scripts/components/sinkholespawner.lua#L91-L120)）。
- **攻击会合并**：与其他目标位置距离小于 `WAVE_MERGE_ATTACKS_DIST_SQ` 时跳过本次攻击不消耗波次（[合并](../../../raw/current/scripts/components/sinkholespawner.lua#L128-L133)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/sinkholespawner.lua` 全部 307 行、`prefabs/antlion.lua` 挂载与三处启停调用、`tuning.lua` ANTLION_SINKHOLE 块。未覆盖：`antlion_sinkhole` prefab 的塌陷表现、蚁狮 tribute 状态机全貌。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
