---
title: "wintertreegiftable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬季树收礼资格组件：记录玩家上次收礼天数，冬季树按距上次赠礼是否满 4 天决定给好礼物还是坏礼物。"
aliases: ["冬季树收礼", "wintertreegiftable", "冬日盛宴赠礼冷却"]
tags: [component, loot]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/wintertreegiftable.lua", "prefabs/player_common.lua", "prefabs/winter_tree.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wintertreegiftable

## 快速判断

- 职责与适用：挂在玩家实体上，维护 `previousgiftday`（上次收礼的世界天数）；冬季树（winter_tree）赠礼前查询 `GetDaysSinceLastGift() >= 4` 决定礼物质量，收礼即记录当天。[判定消费](../../../raw/current/scripts/prefabs/winter_tree.lua#L397-L402)
- 优先使用：不要手动挂载。原版在玩家构造时仅当冬日盛宴特殊活动激活才加入（([player_common.lua:2750-2752](../../../raw/current/scripts/prefabs/player_common.lua#L2750-L2752)）；活动关闭时组件不存在。
- 重要副作用：天数基准是 `TheWorld.state.cycles`（世界循环天数），非真实时间；初始 `previousgiftday = -100` 保证首个冬季必可收好礼。[初始值](../../../raw/current/scripts/components/wintertreegiftable.lua#L6)
- 修改边界：无事件、无网络变量；OnSave/OnLoad 持久化 previousgiftday，冷却跨会话有效。[存读](../../../raw/current/scripts/components/wintertreegiftable.lua#L17-L28)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `previousgiftday` | `number`（默认 -100） | 上次收礼的世界天数；OnLoad 时被存档值覆写。[字段](../../../raw/current/scripts/components/wintertreegiftable.lua#L6) |
| `GetDaysSinceLastGift()` | `-> number` | `TheWorld.state.cycles - previousgiftday`。[实现](../../../raw/current/scripts/components/wintertreegiftable.lua#L9-L11) |
| `OnGiftGiven()` | `-> nil` | 把 previousgiftday 置为当前 cycles；由冬季树在发放好礼时调用。[实现](../../../raw/current/scripts/components/wintertreegiftable.lua#L13-L15) |
| `OnSave()` / `OnLoad(data)` | `-> {previousgiftday}`；`data` 同构 | 存读冷却基准；组件缺失/存档无值时 OnLoad 保持 -100。[实现](../../../raw/current/scripts/components/wintertreegiftable.lua#L17-L28) |

## 典型接入

### 理解或扩展冬季树的礼物冷却（原版参照）

- 前提与选择：服务端；判定方是冬季树对装饰玩家发放礼物的循环（([winter_tree.lua:397-402](../../../raw/current/scripts/prefabs/winter_tree.lua#L397-L402)）：满 4 天给 `GetNiceWinterTreeGiftLoot` 并调 `OnGiftGiven`，否则给 `GetNaughtyWinterTreeGiftLoot` 且不重置计数。
- 生效与撤销：组件随活动加入；好礼当天计数即重置，4 天内再装饰只出坏礼，冷却继续按 cycles 计算。
- 原版依据：玩家实体构造路径按 `IsSpecialEventActive(SPECIAL_EVENTS.WINTERS_FEAST)` 决定挂载（([player_common.lua:2750-2752](../../../raw/current/scripts/prefabs/player_common.lua#L2750-L2752)），是特殊事件组件的典型模式。

```lua
-- 服务端：自定义"周期性资格"可复用同一模式（等价组件实现）
inst.previousgiftday = -100
local function GetDaysSinceLastGift(inst)
    return TheWorld.state.cycles - inst.previousgiftday
end
```

- 调整与验证：修改冷却长度应改判定方的 `>= 4` 而非组件；组件只提供天数差，阈值语义归 winter_tree。

## 常见陷阱与限制

- **以为 4 天按现实时间计算：** 天数差基于 `TheWorld.state.cycles`，跳过天数的世界（如 rollback、控制台跳天）会立刻推进冷却；世界未过夜则永远不满足条件。[天数来源](../../../raw/current/scripts/components/wintertreegiftable.lua#L9-L11)
- **活动关闭期间收礼：** 组件在非盛宴存档中不存在，winter_tree 的判定分支对无组件玩家统一走坏礼路径（`~= nil` 守卫）；活动中途开启时所有玩家冷却从 -100 起算，均可收好礼。[守卫](../../../raw/current/scripts/prefabs/winter_tree.lua#L397)
- **OnLoad 的空守卫冗余：** `data ~= nil and self.previousgiftday ~= nil` 的第二个条件恒真（构造已赋值），存档键缺失时不会恢复 -100 而是保留构造默认——实际效果一致，但自定义解析存档时注意无键即默认。[实现](../../../raw/current/scripts/components/wintertreegiftable.lua#L24-L28)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（30 行），核对 player_common.lua 活动挂载、winter_tree.lua 判定与礼物分支；grep 确认消费者仅此两处。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；赠礼冷却未实测。
