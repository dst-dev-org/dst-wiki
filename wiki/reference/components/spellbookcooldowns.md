---
title: "spellbookcooldowns"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "人物按技能键维护并同步多个冷却，效果入口需主动检查，冷却实体不存档。"
aliases: ["spellbookcooldowns","人物技能冷却","多技能共享CD","技能冷却百分比"]
tags: ["component","network","callback","server-client"]
dst_build: "steam-buildid:26230496"
sources: ["components/spellbookcooldowns.lua","prefabs/spellbookcooldown.lua","prefabs/player_common.lua","prefabs/ghostcommand_defs.lua","prefabs/abigail.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spellbookcooldowns

## 快速判断

- 职责与适用：人物按技能名／哈希管理多个冷却实体，服务端计时并同步到该人物客户端；区别物品 [[reference/components/rechargeable]] 和单实体 [[reference/components/cooldown]]。[components/spellbookcooldowns.lua:1](../../../raw/current/scripts/components/spellbookcooldowns.lua#L1)、[components/spellbookcooldowns.lua:49](../../../raw/current/scripts/components/spellbookcooldowns.lua#L49
- 优先使用：效果入口检查 IsInCooldown，成功后 RestartSpellCooldown；UI 用 GetSpellCooldownPercent。组件不自动阻止技能，已有名字不要借作无关模组的通用键。[prefabs/ghostcommand_defs.lua:125](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L125
- 重要副作用：同名 Restart 重置整段冷却，不叠加剩余时间；Stop 删除条目实体。实例无存档，不能承诺跨退出保留。[components/spellbookcooldowns.lua:49](../../../raw/current/scripts/components/spellbookcooldowns.lua#L49)、[components/spellbookcooldowns.lua:66](../../../raw/current/scripts/components/spellbookcooldowns.lua#L66)、[prefabs/spellbookcooldown.lua:120](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L120
- 修改边界：原版 player_common 已提供宿主，服务端才能启动／停止；本地只读同步状态，延迟下 UI 与服务端判断可能不同。[prefabs/player_common.lua:2523](../../../raw/current/scripts/prefabs/player_common.lua#L2523)、[components/spellbookcooldowns.lua:49](../../../raw/current/scripts/components/spellbookcooldowns.lua#L49

## 关键接口

| 接口 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `IsInCooldown(spellname)` | string 或 hash → bool | 字符串先 hash，只判断表里实体是否存在；不是百分比大于零判定。[components/spellbookcooldowns.lua:18](../../../raw/current/scripts/components/spellbookcooldowns.lua#L18)、[components/spellbookcooldowns.lua:25](../../../raw/current/scripts/components/spellbookcooldowns.lua#L25 |
| `GetSpellCooldownPercent(spellname)` | 剩余比例或 nil | 新条目为 1，随时间降至 0；无条目 nil，UI 可用 or 0，不当已恢复比例。[components/spellbookcooldowns.lua:29](../../../raw/current/scripts/components/spellbookcooldowns.lua#L29)、[prefabs/spellbookcooldown.lua:21](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L21 |
| `RestartSpellCooldown(spellname,duration)` | 秒；服务端，无显式返回 | 已有条目重置；否则 SpawnPrefab、父级与 ClassifiedTarget 设人物，再初始化并注册。[components/spellbookcooldowns.lua:49](../../../raw/current/scripts/components/spellbookcooldowns.lua#L49 |
| `StopSpellCooldown(spellname)` | 服务端，无显式返回 | 有则 Remove，onremove 监听从 cooldowns 表移除；无则不做操作。[components/spellbookcooldowns.lua:6](../../../raw/current/scripts/components/spellbookcooldowns.lua#L6)、[components/spellbookcooldowns.lua:66](../../../raw/current/scripts/components/spellbookcooldowns.lua#L66 |
| `RegisterSpellbookCooldown(cd)` | 已有条目实体 | 内部同步入口，名字 0／重复报诊断，不应用普通表代替协议实体。[components/spellbookcooldowns.lua:34](../../../raw/current/scripts/components/spellbookcooldowns.lua#L34)、[prefabs/spellbookcooldown.lua:1](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L1 |

duration 在冷却 Prefab 中按 0.1 秒四舍五入并夹至 ushort 0～65535，即最多 6553.5 秒；零值待更新时清理，不当作同步立即 Stop 的替代。冷却按 wall-update 的 dt 减比例，支持 LongUpdate；不是本组件自动序列化的持久计时。[prefabs/spellbookcooldown.lua:52](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L52)、[prefabs/spellbookcooldown.lua:65](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L65)、[prefabs/spellbookcooldown.lua:120](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L120

## 典型接入

### 新增原版人物入场后的两秒自定义技能锁

目标模组 `modmain.lua`，在玩家初始化后注册零延迟任务，服务端启动命名冷却。此例只交付人物冷却创建，不附带新技能／UI；新技能执行入口必须用同一个 key 检查 IsInCooldown 再做效果。原版幽灵命令展示了检查→效果／命令→共享键冷却的消费模式。[prefabs/ghostcommand_defs.lua:125](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L125

```lua
local SPELL_KEY = "my_mod:spawn_skill_lock"

AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, function(player)
        local cooldowns = player.components.spellbookcooldowns
        if cooldowns ~= nil then
            cooldowns:RestartSpellCooldown(SPELL_KEY, 2)
        end
    end)
end)
```

替换 my_mod 为模组命名空间；原版人物宿主已具备组件，不重复 AddComponent。两秒后条目删除；每次人物重新初始化都可能重新启动，不是仅新世界一次；停用模组不再注册，已启动两秒自然结束。不提供跨存档防刷保证。[components/spellbookcooldowns.lua:49](../../../raw/current/scripts/components/spellbookcooldowns.lua#L49)、[prefabs/spellbookcooldown.lua:21](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L21)、[prefabs/player_common.lua:2523](../../../raw/current/scripts/prefabs/player_common.lua#L2523

## 常见陷阱与限制

- **显示冷却等于服务端禁用**：只有消费者 IsInCooldown 才阻止操作；直接调效果能绕过 UI。原版命令同时检查共享 ghostcommand 和独立技能键。[prefabs/ghostcommand_defs.lua:125](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L125
- **Restart 往剩余时间上加秒**：它重设 duration 和 percent=1，适合刷新；延长需明确自己的算法。[prefabs/spellbookcooldown.lua:70](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L70
- **GetPercent 是已充好比例**：初始 1、倒数到 0；无条目 nil。客户端在服务端 Remove 到达前可有短暂边界差异。[prefabs/spellbookcooldown.lua:21](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L21)、[components/spellbookcooldowns.lua:29](../../../raw/current/scripts/components/spellbookcooldowns.lua#L29
- **跨存档会自动恢复**：冷却 Prefab persists=false，组件无 OnSave／OnLoad；重要长冷却另设计保存，不擅自升级此能力。[prefabs/spellbookcooldown.lua:120](../../../raw/current/scripts/prefabs/spellbookcooldown.lua#L120

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；组件／冷却 Prefab／幽灵命令／Abigail 均无变化，仅 player_common 宿主锚点 2533→2521（P-83 删除段在前）；nomagiccast（R-01/R-04）作用于动作/施法层不涉冷却管理，结论维持。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整管理组件／冷却 Prefab、player_common 挂载、幽灵命令检查／UI 与 Abigail 触发独立键；未审校全部玩家生命周期、Classified 底层或其他技能消费者。
- 结构检查：发布时核对格式、来源、链接／行号、大小与登记；完整示例作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；入场时点、客户端延迟、断线重连、零时长、同名冲突和技能锁实际消费待测。

相关：[[reference/components/spellbook]]；[[reference/components/aoespell]]；[[reference/components/rechargeable]]。
