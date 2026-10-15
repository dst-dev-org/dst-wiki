---
title: "mightiness"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Wolfgang 专属体格值系统：管理 wimpy／normal／mighty 三态切换及其皮肤、体型、伤害、保温、工作与划船等属性联动。"
aliases: ["mightiness", "体格值", "沃尔夫冈体格", "wimpy", "mighty", "强弱值"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/mightiness.lua", "prefabs/wolfgang.lua", "prefabs/skilltree_wolfgang.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mightiness

## 快速判断

- 职责与适用：Wolfgang 的体格计量条（0～100），驱动 wimpy/normal/mighty 三态；只有 wolfgang prefab 在 master 初始化安装，其它角色调用会因缺组件判空。[安装](../../../raw/current/scripts/prefabs/wolfgang.lua#L667-L670)
- 优先使用：改值用 `DoDelta(delta[, force_update, delay_skin, forcesound, fromgym])`；按比例用 `SetPercent`；直接设置用 `SetInspiration` 同型的 `SetMax`／`current` 赋值后 `DoDelta(0, true)`。[改值](../../../raw/current/scripts/components/mightiness.lua#L202-L237)
- 重要副作用：每次正值增量都会重置 4 秒的下降延迟（`DelayDrain`）；三态切换会改皮肤模式、体型缩放、外显伤害倍率（0.75/1/2）、温度保温、饥饿燃烧率、划船参数、工作倍率与 `mightiness_*` 标签，并推 `powerup`／`powerdown` 事件。[状态数据](../../../raw/current/scripts/components/mightiness.lua#L1-L65) [切换实现](../../../raw/current/scripts/components/mightiness.lua#L288-L374)
- 修改边界：变换受 `CanTransform` 门禁（nomorph 状态、ghost、死亡、同态）拦截；骑乘中只改值不缩放（mounted 事件恢复缩放）；体格值本身存档（OnSave 只存 current）。[门禁](../../../raw/current/scripts/components/mightiness.lua#L262-L267) [缩放边界](../../../raw/current/scripts/components/mightiness.lua#L111-L112) [存读](../../../raw/current/scripts/components/mightiness.lua#L125-L134)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `current`／`max` | number（默认 100） | 0≤current≤max；经 player_classified 同步为 0-255 百分比（`currentmightiness`）。[同步](../../../raw/current/scripts/components/mightiness.lua#L67-L73) [上限常量](../../../raw/current/scripts/tuning.lua#L6451-L6452) |
| `DoDelta(delta, force_update?, delay_skin?, forcesound?, fromgym?)` | — | 正增量上限 clamp 到 max，`fromgym=true` 时允许到 `max + GetOverMax()`（健身房超上限）；负增量下限 0；正增量顺带 DelayDrain(4s)。[实现](../../../raw/current/scripts/components/mightiness.lua#L202-L237) [延迟](../../../raw/current/scripts/tuning.lua#L6508 |
| `GetOverMax()`／`SetOverMax(amount)` | number | 超上限容量，由 Wolfgang 技能树 `OVERBUFF` 技能设置；仅健身房路径能用到。[接口](../../../raw/current/scripts/components/mightiness.lua#L194-L200) [技能树](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L156-L158) |
| `GetState()`／`GetScale()`／`GetSkinMode()` | string / number | 当前态与该态体型缩放（0.9/1/1.2）、皮肤模式名；`IsMighty/IsNormal/IsWimpy` 是便捷判断（IsMighty 对 "over" 也真）。[读取](../../../raw/current/scripts/components/mightiness.lua#L136-L154) [状态表](../../../raw/current/scripts/components/mightiness.lua#L1-L65) |
| `DoDec(dt, ignore_damage?)` | 内部 | 每秒 tick：非暂停且非无敌时按 `rate(0.2) × drain_multiplier × ratemodifiers:Get()` 下降；移动中持有 dumbbell 标签装备改为 DelayDrain(2)。[消耗](../../../raw/current/scripts/components/mightiness.lua#L248-L256) |
| `Pause()`／`Resume()`／`IsPaused()` | — | 停/恢复自然下降；wolfgang 变鬼时暂停、复活恢复，携带 heavy 装备期间也暂停。[开关](../../../raw/current/scripts/components/mightiness.lua#L160-L170) [消费](../../../raw/current/scripts/prefabs/wolfgang.lua#L113-L120) [heavy 装备](../../../raw/current/scripts/prefabs/wolfgang.lua#L142-L157) |
| `OnHungerDelta`（监听 hungerdelta） | 内部 | 按饥饿百分比切 drain_multiplier：≥0.75→0.5、≥0.5→1、≥0.25→2、>0→3、断粮→8。[饥饿联动的分档](../../../raw/current/scripts/components/mightiness.lua#L376-L391) [常量](../../../raw/current/scripts/tuning.lua#L6454-L6458) |
| `BecomeState(state, ...)` | 内部 | 三态切换总入口：换皮肤（可延迟）、同步健身房皮肤、播报/音效、combat.externaldamagemultipliers、temperature 保温、hunger.burnrate、expertsailor 参数、workmultiplier/efficientuser 的 CHOP/MINE/HAMMER 倍率、体型、标签、talk/hurt 音源，最后推 `mightiness_statechange {previous_state, state}`。[切换](../../../raw/current/scripts/components/mightiness.lua#L288-L374) |
| `CanTransform(state)` | → bool | nomorph 状态标签、playerghost、死亡或同态时拒绝。[门禁](../../../raw/current/scripts/components/mightiness.lua#L262-L267) |
| 阈值与速率 | TUNING | `MIGHTY_THRESHOLD=75`、`WIMPY_THRESHOLD=25`、`MIGHTINESS_DRAIN_RATE=0.2/s`、`WOLFGANG_MIGHTINESS_DRAIN_DELAY=4`。[常量](../../../raw/current/scripts/tuning.lua#L6451-L6461) |

### 三态状态表（STATE_DATA 摘要）

| 态 | 外显伤害倍率 | 缩放 | 其他关键项 |
|---|---|---|---|
| wimpy | 0.75 | 0.9 | 冬-夏+微量保温、工作低效（WIMPY_WORK_EFFECTIVENESS）、饥饿消耗 WIMPY_HUNGER_RATE_MULT、瘦皮肤 build |
| normal | 移除修正 | 1 | 无附加参数 |
| mighty | 2 | 1.2 | 冬+夏-微量保温、工作增效（MIGHTY_WORK_EFFECTIVENESS）、划船推力/锚速/帆力增益、壮皮肤 build |

事件流：进入 mighty 推 `powerup`，降到 normal/wimpy 推 `powerdown`；normal↔两侧的 announce 与音效按来源态分派。[状态数据](../../../raw/current/scripts/components/mightiness.lua#L3-L64) [播报分支](../../../raw/current/scripts/components/mightiness.lua#L303-L311)

## 典型接入

### 从模组装备／事件给沃尔夫冈临时体格增减

- 前提与选择：目标实体必须装有 mightiness（判 `inst.components.mightiness`）；装备类增益通常在 equip/unequip 或工作回调里 `DoDelta`，原版撸铁（dumbbell）与工作收益都走这条路径。[工作收益](../../../raw/current/scripts/prefabs/wolfgang.lua#L186-L240)
- 生效与撤销：正增量会自动延迟下降 4 秒；想让状态立即刷新（不换态也重放皮肤/标签）传 force_update=true；延迟换肤（delay_skin）用于过度动画期间。
- 原版依据：技能树 OVERBUFF 用 `SetOverMax` 扩容后，只能通过健身房（`fromgym=true`）超出 100，普通增量仍被 max clamp。[技能树](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L156-L158) [clamp](../../../raw/current/scripts/components/mightiness.lua#L211-L217)

```lua
-- 服务器端：装备持有期间每 10 秒 +5 体格（示例：装备监听内）
local function OnEquip(inst, owner)
    inst._mighttask = owner:DoPeriodicTask(10, function()
        if owner.components.mightiness ~= nil then
            owner.components.mightiness:DoDelta(5)
        end
    end)
end
local function OnUnequip(inst, owner)
    if inst._mighttask ~= nil then
        inst._mighttask:Cancel()
        inst._mighttask = nil
    end
end
```

- 调整与验证：非沃尔夫冈调用 DoDelta 无效果也无害（组件不存在，需判空）；监控变化用 `mightinessdelta`（oldpercent/newpercent/delta）与 `mightiness_statechange` 事件。[事件](../../../raw/current/scripts/components/mightiness.lua#L222 [状态事件](../../../raw/current/scripts/components/mightiness.lua#L370-L373)

## 常见陷阱与限制

- **直接赋值 `current` 不同步网络：** `current` 的 oncurrent 监听只对 setter 生效；从存档外改值后要 `DoDelta(0, true)` 强制刷新 classified 与状态。[同步监听](../../../raw/current/scripts/components/mightiness.lua#L67-L73) [OnLoad 范式](../../../raw/current/scripts/components/mightiness.lua#L129-L134)
- **在健身房里换态的皮肤同步：** BecomeState 会找 `strongman.gym` 并调 `SetSkinModeOnGym` 同步器材皮肤；模组自建"健身房"实体必须复用 mightygym 组件，否则健身者换态时皮肤错位。[同步](../../../raw/current/scripts/components/mightiness.lua#L298-L301)
- **骑乘与换态的缩放竞争：** 进健身房或骑乘时 ApplyAnimScale 被覆盖，mightiness 的 mounted/dismounted 监听负责恢复；直接改 AnimState 缩放会被后续换态覆盖。[缩放监听](../../../raw/current/scripts/components/mightiness.lua#L111-L112) [健身房缩放](../../../raw/current/scripts/components/mightygym.lua#L347
- **"over" 态不是可设置状态：** `IsMighty` 对 "over" 为真但 STATE_DATA 无该键，`GetScale/GetSkinMode` 不会遇到它；不要尝试 `BecomeState("over")`。[判断](../../../raw/current/scripts/components/mightiness.lua#L144-L146)
- **多倍率来源：** ratemodifiers（SourceModifierList）与 drain_multiplier（饥饿分档）相乘；改饥饿分档常量不改变 ratemodifiers 来源，调试时用 GetDebugString 看三项。[乘链](../../../raw/current/scripts/components/mightiness.lua#L253 [调试串](../../../raw/current/scripts/components/mightiness.lua#L176-L179)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 mightiness.lua，核验 STATE_DATA 三态参数、DoDelta/DoDec/ BecomeState 全链路、饥饿分档、健身房超上限路径、classified 同步与存读；核对 wolfgang 安装、skilltree_wolfgang OVERBUFF 与 mightygym 皮肤同步消费。未逐项实测各 TUNING 数值在游戏内的表现。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；换态动画、健身房超上限与骑乘缩放未实测。

## 相关链接

- [[reference/components/mightygym]]：唯一能超 max 增长体格的路径。
- [[reference/components/strongman]]：健身房会话与本组件的状态联动。
- [[reference/components/dumbbelllifter]]：举铁动作的体格收益。
- [[reference/components/coach]]：换态时被禁用的教练组件。
