---
title: "rockybossspawner"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "洞穴世界宿主组件：按 worldsettingstimer 计时把满体型睡眠的 rocky 升级为 rocky_boss（ReplacePrefab 换体并保持群籍），登记 rocky 群、跟踪当前 Boss 的死亡/移除并在无群时补生 rocky。"
aliases: ["岩石Boss生成器", "岩石龙虾Boss升级", "rockybossspawner"]
tags: ["component", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/rockybossspawner.lua", "prefabs/cave.lua", "prefabs/rockyherd.lua", "prefabs/rocky.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# rockybossspawner

## 快速判断

- 职责与适用：挂在洞穴世界实体上的主端组件（cave.lua master_postinit 添加，构造即 `assert(TheWorld.ismastersim)`）：用 worldsettingstimer 计时器 `rockyboss_timetospawn` 周期性把一只「满体型＋睡眠中」的岩石龙虾升级为 rocky_boss；无任何 rocky 群时改为补生 rocky；全程跟踪当前 Boss。[components/rockybossspawner.lua:8](../../../raw/current/scripts/components/rockybossspawner.lua#L8 [prefabs/cave.lua:578](../../../raw/current/scripts/prefabs/cave.lua#L578
- 优先使用：外部只读用 `GetRockyBoss()`；主动升级用 `UpgradeRockyToBoss(rocky)`（须给 rocky 实体）；生成节奏由 `OnPostInit` 注册的计时器驱动，不要自行 StartTimer 重名计时器。
- 重要副作用：`UpgradeRockyToBoss` 用 `ReplacePrefab(rocky, "rocky_boss")` 原地换体（旧 rocky 实体消失），新 Boss 重新加入原群并挂上死亡/移除监听；Boss 死亡后计时器自动重启。[components/rockybossspawner.lua:70](../../../raw/current/scripts/components/rockybossspawner.lua#L70
- 修改边界：仅洞穴分片（地上 TheWorld 无此组件）；rocky 侧依赖 `herdmember`（herdprefab "rockyherd"）与导出的 `IsMaxSize`，见下文接口契约。

## 关键接口

| 接口 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `UpgradeRockyToBoss(rocky)` | rocky 实体 → 无返回 | 取 `rocky.components.herdmember:GetHerd()`（无该组件会报错）→ `ReplacePrefab(rocky, "rocky_boss")` → 群存在则 `herd:AddMember(rocky_boss)` → 登记为当前 Boss 并监听其 onremove/death。[components/rockybossspawner.lua:70](../../../raw/current/scripts/components/rockybossspawner.lua#L70 |
| `GetRockyBoss()` | 实体\|nil | 当前 Boss；存读档经 GUID 重连。[components/rockybossspawner.lua:79](../../../raw/current/scripts/components/rockybossspawner.lua#L79 |
| `OnPostInit()` | — | `worldsettingstimer:AddTimer("rockyboss_timetospawn", ROCKY_BOSS_SPAWNDELAY_BASE + ROCKY_BOSS_SPAWNDELAY_RANDOM, TUNING.SPAWN_ROCKY_BOSS, OnRockyBossTimerDone)` 后 `TryBeginningRockyBossSpawns()`。[components/rockybossspawner.lua:180](../../../raw/current/scripts/components/rockybossspawner.lua#L180 [tuning.lua:9639](../../../raw/current/scripts/tuning.lua#L9639 |
| `OnSave()` | {boss=GUID}, ents | Boss 存在时记录 GUID 并列入 ents 供 LoadPostPass 重连。[components/rockybossspawner.lua:189](../../../raw/current/scripts/components/rockybossspawner.lua#L189 |
| `LoadPostPass(newents, savedata)` | — | 按 `savedata.boss` 从 newents 找回实体并重新 `RegisterRockyBoss`（重挂监听）。[components/rockybossspawner.lua:202](../../../raw/current/scripts/components/rockybossspawner.lua#L202 |
| `GetDebugString()` | string | `Rocky exists: %s, Spawning in %.2f (%.2f days)`；读计时器剩余时间。[components/rockybossspawner.lua:213](../../../raw/current/scripts/components/rockybossspawner.lua#L213 |

世界事件契约：组件在自身（＝TheWorld，洞穴分片）监听 `ms_registerrockyherd`，由 rockyherd prefab 构造时 `TheWorld:PushEvent("ms_registerrockyherd", inst)` 推送；回调去重登记进 `_herds`、对群实体监听 onremove，且仅在非 POPULATING 时尝试启动计时（population 期计时器与 Boss 尚未装载）。[components/rockybossspawner.lua:174](../../../raw/current/scripts/components/rockybossspawner.lua#L174 [prefabs/rockyherd.lua:72](../../../raw/current/scripts/prefabs/rockyherd.lua#L72

## 关键流程

- 计时启动：`TryBeginningRockyBossSpawns` —— 无活动计时且无 Boss 时 `StartTimer(name, GetRandomWithVariance(15 天 ± 5 天))`；计时被暂停则恢复。[components/rockybossspawner.lua:32](../../../raw/current/scripts/components/rockybossspawner.lua#L32 [tuning.lua:9639](../../../raw/current/scripts/tuning.lua#L9639
- 计时到点 `OnRockyBossTimerDone`：[components/rockybossspawner.lua:138](../../../raw/current/scripts/components/rockybossspawner.lua#L138
  1. `_herds` 为空 → `DoRockyRegrowth()`：在「生成密度含 rocky」的地形随机取点（远离玩家 `PLAYER_CAMERA_SEE_DISTANCE` 之外），在该点附近刷 4 只 rocky 后返回。[components/rockybossspawner.lua:114](../../../raw/current/scripts/components/rockybossspawner.lua#L114
  2. 否则按群 `membercount` 加权抽群，遍历其成员找第一只 `prefab == "rocky"` 且 `IsAsleep()` 且 `IsMaxSize()` 的候选 → `UpgradeRockyToBoss(member)`；该群无候选换下一群。
  3. 所有权重群都无候选 → 以 `ROCKY_BOSS_SPAWNDELAY_RETRY_BASE/RANDOM`（1.5 天 ± 0.5 天）重开计时静默重试。[tuning.lua:9642](../../../raw/current/scripts/tuning.lua#L9642
- Boss 跟踪：death 回调清 `_boss`、重启计时并注销两组监听；onremove 回调清 `_boss`、重启计时但不注销监听（实体已移除，无后续事件，属可忽略残留）。[components/rockybossspawner.lua:48](../../../raw/current/scripts/components/rockybossspawner.lua#L48
- 暂停路径 `TryPauseRockyBossSpawns` 在当前源码中定义但从未被调用（死代码）：群全部移除不会暂停计时，到点后改走补生分支。

### 与 rocky／rockyherd 的接口契约（本页核验范围）

- rocky 导出 `inst.IsMaxSize`（`scaler.scale >= TUNING.ROCKY_MAX_SCALE`），升级筛选直接调它；`herdmember.herdprefab = "rockyherd"` 保证群籍可回查。[prefabs/rocky.lua:403](../../../raw/current/scripts/prefabs/rocky.lua#L403 [prefabs/rocky.lua:589](../../../raw/current/scripts/prefabs/rocky.lua#L589 [prefabs/rocky.lua:509](../../../raw/current/scripts/prefabs/rocky.lua#L509
- 体型供给链：rocky 的 `grow` 按 `TUNING.ROCKY_GROW_RATE` 增长 scale 至 `ROCKY_MAX_SCALE`；移速键消费者 `TUNING.ROCKY_WALK_SPEED`（walkspeed/runspeed 同键）。当前值 `ROCKY_WALK_SPEED = 2.7`、`ROCKY_GROW_RATE = (1.2-0.75)/(total_day_time*4.8)`（满生长约 4.8 天），与更新报告改值键 ROCKY_WALK_SPEED 2→2.7、ROCKY_GROW_RATE 40→4.8 天一致。[prefabs/rocky.lua:56](../../../raw/current/scripts/prefabs/rocky.lua#L56 [prefabs/rocky.lua:527](../../../raw/current/scripts/prefabs/rocky.lua#L527 [tuning.lua:2394](../../../raw/current/scripts/tuning.lua#L2394 [tuning.lua:2397](../../../raw/current/scripts/tuning.lua#L2397
- rockyherd 是非网络化实体，构造即向 TheWorld 注册自身并经 periodicspawner 维持 rocky 数量；Boss 计时器的群权重直接读 `herd.membercount`。[prefabs/rockyherd.lua:55](../../../raw/current/scripts/prefabs/rockyherd.lua#L55

## 典型接入

### 把一只满体型睡眠岩石龙虾直接升级为 Boss

- 前提与选择：洞穴分片服务端代码（组件仅存在于 cave 世界）；从事件回调或控制台拿到 rocky 实体后调用。
- 生效与撤销：升级即换体，原 rocky 实体消失、不可撤销；已有 Boss 在位时本例直接放弃，避免双 Boss。

```lua
local function TryUpgradeRocky(rocky)
    local spawner = GLOBAL.TheWorld.components.rockybossspawner
    if spawner == nil or spawner:GetRockyBoss() ~= nil then
        return false
    end
    if rocky.prefab ~= "rocky" or rocky.IsMaxSize == nil or not rocky:IsMaxSize() then
        return false
    end
    spawner:UpgradeRockyToBoss(rocky)
    return true
end
```

- 调整与验证：候选筛选（睡眠/满体型）是原版计时器路径的行为，`UpgradeRockyToBoss` 本身不做体型检查，调用方自检如上；生成节奏改值走 `TUNING.ROCKY_BOSS_SPAWNDELAY_*` 与 `SPAWN_ROCKY_BOSS`。

## 常见陷阱与限制

- **UpgradeRockyToBoss 硬依赖 herdmember**：第一行就调 `rocky.components.herdmember:GetHerd()`，对无该组件的实体调用会直接报错；群不存在时跳过 AddMember 但仍完成换体。[components/rockybossspawner.lua:71](../../../raw/current/scripts/components/rockybossspawner.lua#L71
- **计时器到点不一定出 Boss**：候选必须是满体型且睡眠的 rocky，全部群无候选时仅静默重试 1.5 天 ± 0.5 天；调试先用 `GetDebugString`/`GetRockyBoss` 确认状态。[components/rockybossspawner.lua:156](../../../raw/current/scripts/components/rockybossspawner.lua#L156
- **仅洞穴分片**：组件由 cave.lua 添加，地上世界读 `TheWorld.components.rockybossspawner` 为 nil。[prefabs/cave.lua:578](../../../raw/current/scripts/prefabs/cave.lua#L578
- **不要绕过 worldsettingstimer**：计时器名固定 `rockyboss_timetospawn` 且带 `TUNING.SPAWN_ROCKY_BOSS` 总开关；直接对 worldsettingstimer StartTimer 同名计时器会与组件内部状态竞争。[components/rockybossspawner.lua:181](../../../raw/current/scripts/components/rockybossspawner.lua#L181

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件 221 行全部逻辑（计时、群登记、换体、存读、调试串）、cave 挂载、rockyherd 注册事件、rocky 接口契约切片（IsMaxSize 导出、herdmember 群籍、scaler 增长与移速键消费者）及 tuning ROCKY_* 键当前值。更新报告 prefabs/rocky.lua 条目（+306/−43）此前为待审：本页仅在其组件接口范围内随建页核验转正（上述锚点），rocky 的 boulder 形态、acidlevel 等 prefab 级内容未核验，完整转正留待 prefab 页；tuning 改值键（ROCKY_WALK_SPEED 2→2.7、ROCKY_GROW_RATE 40→4.8 天）为报告已核验条目，本页已对照当前源码复用。未覆盖：rocky_boss prefab 内部、游戏运行。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/herdmember]]、[[reference/components/herd]]、[[reference/components/timer]]。
