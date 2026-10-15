---
title: "minigame_participator"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把实体登记为小游戏参与者：挂标签供 AI 识别避让，3 秒无心跳自动退出，随小游戏关闭自动移除。"
aliases: ["minigame_participator", "小游戏参与者", "嘉年华参与者"]
tags: [component, ai]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/minigame_participator.lua", "components/minigame.lua", "prefabs/pigelite.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# minigame_participator

## 快速判断

- 职责与适用：标记"正在参与某个小游戏"的实体状态。组件持有 `minigame` 引用并监听其 `onremove`/`ms_minigamedeactivated`，一旦小游戏关闭或消失，自动把自己移除。[构造](../../../raw/current/scripts/components/minigame_participator.lua#L11-L14)
- 优先使用：不要直接 AddComponent——用小游戏侧 `minigame:AddParticipator(ent, notimeout)`，它负责挂组件、写 `notimeout` 并调 `SetMinigame`。[登记入口](../../../raw/current/scripts/components/minigame.lua#L81-L87)
- 重要副作用：构造即加 `minigame_participator` 标签，是大量大脑（猪王护卫、Abigail、随从、乌鸦小孩等）避让与逻辑分支的依据；`SetMinigame` 还会让非 `keepleaderduringminigame` 的跟随者停止跟随。[标签避让](../../../raw/current/scripts/brains/carnival_hostbrain.lua#L112) [停止跟随](../../../raw/current/scripts/components/minigame_participator.lua#L60-L66)
- 修改边界：无心跳超时（3 秒未续期自动退出）仅对 `notimeout` 为假者生效；`SetMinigame` 只在首个小游戏上生效（重复设置只续期）。[超时逻辑](../../../raw/current/scripts/components/minigame_participator.lua#L44-L58)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetMinigame(minigame)` | minigame: entity | 建立关联：监听其 `onremove` 与 `ms_minigamedeactivated`；无 `notimeout` 时启动 0.9 秒周期检查，`expireytime`（GetTime()+3）过期即自我移除。[实现](../../../raw/current/scripts/components/minigame_participator.lua#L43-L58) |
| `notimeout` | boolean | 参与者不过期；原版猪王护卫设 true（`pigelite.lua:220`），`minigame:AddParticipator` 透传该参数。[透传](../../../raw/current/scripts/components/minigame.lua#L83-L86) |
| `GetMinigame()` | `-> entity 或 nil` | 返回当前关联小游戏，供大脑查询。[实现](../../../raw/current/scripts/components/minigame_participator.lua#L39-L41) |
| `CurrentMinigameType()` | `-> string 或 nil` | 读 `minigame.components.minigame.gametype`；如 `bunnyman_pillowfighting`。[实现](../../../raw/current/scripts/components/minigame_participator.lua#L70-L73) |
| `OnRemoveFromEntity` | 生命周期回调 | 移除标签、取消超时任务、断开事件，并把跟随者的 `keeptargettimeout` 置 0 强制重新验目标。[清理](../../../raw/current/scripts/components/minigame_participator.lua#L17-L37) |

## 典型接入

### 让随从参与自建小游戏（原版参照）

- 前提与选择：目标实体需有 `minigame.components.minigame`（gametype 供 AI 查询）；登记统一走 `Minigame:AddParticipator`，不要绕过它手写三步。[登记实现](../../../raw/current/scripts/components/minigame.lua#L81-L87)
- 生效与撤销：`ms_minigamedeactivated` 由 `Minigame:Deactivate` 推送（`components/minigame.lua:64`），参与者组件随之自动移除；实体下线同样触发清理。

```lua
-- 服务端：把 ent 登记为小游戏参与者（不过期）
if ent.components.minigame_participator == nil then
    ent:AddComponent("minigame_participator")
    ent.components.minigame_participator.notimeout = true
end
ent.components.minigame_participator:SetMinigame(mygame_inst)
```

- 调整与验证：自建 AI 需求措辞用 `HasTag("minigame_participator")` 做避让（`RunAway` 条件），用 `GetMinigame()`/`CurrentMinigameType()` 做玩法分支。

## 常见陷阱与限制

- **直接 AddComponent 而不 SetMinigame：** 组件只是挂了标签，`GetMinigame()` 恒为 nil，也不会随任何小游戏结束自动移除；心跳超时逻辑同样不启动（`SetMinigame` 才写 `expireytime`）。[超时写入](../../../raw/current/scripts/components/minigame_participator.lua#L44-L46)
- **notimeout 绕过超时但不绕过移除：** `notimeout` 只跳过 0.9 秒周期检查；小游戏 `onremove`/`ms_minigamedeactivated` 时参与者照样被移除。[onminigameover](../../../raw/current/scripts/components/minigame_participator.lua#L11-L14)
- **依赖读取 `gametype`：** `CurrentMinigameType` 深层访问 `minigame.components.minigame.gametype`，对没有 minigame 组件的对象返回 nil，不要假设必有类型。[实现](../../../raw/current/scripts/components/minigame_participator.lua#L70-L73)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（79 行），核对 `minigame:AddParticipator`、`pigelite.lua:219-220` 挂载、以及 carnival_hostbrain/abigailbrain/kitcoonbrain/crittersbrain/shadowwaxwellbrain/cozy_bunnymanbrain 对标签与 `GetMinigame` 的消费。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
