---
title: "minigame"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "小游戏主机组件：周期脉冲把周围角色收编为观众、玩家收编为参与者，管理激活状态与热度窗口供奖励判定。"
aliases: ["小游戏主机", "minigame", "嘉年华游戏"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/minigame.lua", "prefabs/carnivalgame_common.lua", "prefabs/pigking.lua", "prefabs/yotr_fightring.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# minigame

## 快速判断

- 职责与适用：挂在游戏机本体（嘉年华游戏机、猪王、约特拳台），激活后每 0.75 秒扫描一圈，把非玩家角色挂 [[reference/components/minigame_spectator]]、玩家挂 [[reference/components/minigame_participator]]（[脉冲](../../../raw/current/scripts/components/minigame.lua#L91-L109)）。
- 优先使用：做一个可参与的小游戏时，注册激活/关闭回调 + `Activate` 即可，观众/参与者收编全自动。
- 重要副作用：关闭时推 `ms_minigamedeactivated`（[事件](../../../raw/current/scripts/components/minigame.lua#L67)）；组件移除会自动关闭（[清理](../../../raw/current/scripts/components/minigame.lua#L28-L30)）。
- 修改边界：观众收编要求目标没有跟随领袖；参与者一律玩家且带 `notimeout` 选项。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnActivatedFn(fn)` / `SetOnDeactivatedFn(fn)` | `fn(inst)` | 激活/关闭回调（[注册](../../../raw/current/scripts/components/minigame.lua#L32-L38)）。 |
| `Activate()` / `Deactivate()` | 无参数 | 启停脉冲任务；重复 Activate 先行关闭（[实现](../../../raw/current/scripts/components/minigame.lua#L44-L72)）。 |
| `IsActive()` | → `bool` | 当前激活态（[实现](../../../raw/current/scripts/components/minigame.lua#L40-L42)）。 |
| `AddSpectator(spectator)` / `AddParticipator(participator, notimeout)` | 实体 | 手动收编；自动脉冲内部走同一路径（[实现](../../../raw/current/scripts/components/minigame.lua#L74-L87)）。 |
| `SetIsIntro/Playing/Outro` + 对应 `GetIs*` | 状态串 `intro/playing/outro` | 小游戏阶段标记（[实现](../../../raw/current/scripts/components/minigame.lua#L111-L133)）。 |
| `RecordExcitement()` / `IsExciting()` | 时间戳 | 热度窗口：距上次兴奋不足 `excitement_delay`（默认 5 秒）为热（[实现](../../../raw/current/scripts/components/minigame.lua#L135-L145)）。 |
| `spectator_dist` / `participator_dist` | 数值，默认 20 | 脉冲扫描半径，可写改（[字段](../../../raw/current/scripts/components/minigame.lua#L10-L11)）。 |

## 典型接入

### 注册一台可参与的游戏机

- 前提与选择：服务器 master 段注册；观众/参与者组件由本组件按需 `AddComponent`，无需预制。
- 生效与撤销：`Activate` 后脉冲持续收编；`Deactivate` 停脉冲并推关闭事件，参与者组件会自行退出。
- 原版依据：嘉年华游戏机的通用装配流程（[装配](../../../raw/current/scripts/prefabs/carnivalgame_common.lua#L227-L232)）；猪王、拳台同样注册（[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L719)、[拳台](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L645)）。

```lua
inst:AddComponent("minigame")
inst.components.minigame.gametype = "mygame"
inst.components.minigame:SetOnActivatedFn(function(inst)
    inst.components.minigame:SetIsPlaying()
end)
inst.components.minigame:SetOnDeactivatedFn(function(inst)
    -- 结算与清理
end)
inst.components.minigame:Activate()
```

- 调整与验证：观众扫描排除 `monster`/`player` 标签、只收 `character`（[过滤](../../../raw/current/scripts/components/minigame.lua#L89-L95)）；围观距离默认取 `TUNING.MINIGAME_CROWD_DIST_*`（[字段](../../../raw/current/scripts/components/minigame.lua#L13-L15)）。

## 常见陷阱与限制

- 脉冲只"收编"不"清退"：参与者退出依赖 [[reference/components/minigame_participator]] 的超时与小游戏关闭事件，组件本身不维护成员列表。
- `gametype` 默认 `unknown`（[字段](../../../raw/current/scripts/components/minigame.lua#L17)），部分消费方按 gametype 分支，注册后务必赋值。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 carnivalgame_common、pigking、yotr_fightring 装配链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
