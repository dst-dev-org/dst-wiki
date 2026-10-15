---
title: "yoth_knightmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "马年活动（YOTH）世界宿主组件：登记激活的骑士神殿与公主帽，按冷却与在场判定从公主帽 petleash 生成 shadow 剑士 knight_yoth，供神殿与舞台协作读取。"
aliases: ["yoth_knightmanager", "马年骑士管理器", "骑士神殿管理器", "YOTH knight manager"]
tags: [component, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/yoth_knightmanager.lua", "prefabs/forest.lua", "prefabs/hats.lua", "prefabs/yoth_knightshrine.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yoth_knightmanager

## 快速判断

- 职责与适用：挂在 forest 世界实体的服务端组件（构造断言 mastersim）；维护两张登记表 `shrines`（激活神殿）与 `princesses`（主人→公主帽，`hats` 为反向表），并提供"该玩家能否被公主记仇、生成 shadow 剑士"的判定与生成。[构造与状态](../../../raw/current/scripts/components/yoth_knightmanager.lua#L5-L12)
- 优先使用：查询活动期是否有激活神殿 → `IsKnightShrineActive()`／`GetActiveKnightShrines()`（[[reference/components/yoth_hecklermanager]]、charlie_stage、雕像竖琴等消费者都这样读）；让公主帽接管生成流程 → 向世界推 `ms_register_yoth_princess`。[查询接口](../../../raw/current/scripts/components/yoth_knightmanager.lua#L35-L41)
- 重要副作用：登记公主后按 0.5-0.75s 延迟首次尝试生成；条件满足时从帽子的 `petleash` 生成最多 `#YOTH_HORSE_NAMES` 只 `knight_yoth`，隐藏后错峰揭示。[注册](../../../raw/current/scripts/components/yoth_knightmanager.lua#L168-L180) [生成](../../../raw/current/scripts/components/yoth_knightmanager.lua#L81-L110)
- 修改边界：神殿登记由实体推世界事件 `ms_knightshrineactivated`／`ms_knightshrinedeactivated` 驱动；`IsKnightShrineActive` 额外要求 `IsSpecialEventActive(SPECIAL_EVENTS.YOTH)`，活动关闭时即使神殿健在也返回 false。[事件监听](../../../raw/current/scripts/components/yoth_knightmanager.lua#L32-L33) [活动门控](../../../raw/current/scripts/components/yoth_knightmanager.lua#L35-L37)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsKnightShrineActive()` | 返回 boolean | 有登记神殿且 YOTH 活动开启；statueharp_hedgespawner 与 charlie_stage 据此切换行为。([yoth_knightmanager.lua:35-37](../../../raw/current/scripts/components/yoth_knightmanager.lua#L35-L37) |
| `GetActiveKnightShrines()` | 返回 shrine 集合表 | 直接返回内部 `shrines` 表（键为实体）；[[reference/components/yoth_hecklermanager]] 洗牌后选落点。([yoth_knightmanager.lua:39-41](../../../raw/current/scripts/components/yoth_knightmanager.lua#L39-L41) |
| 神殿登记 | 世界事件驱动 | `ms_knightshrineactivated`／`ms_knightshrinedeactivated` 由 yoth_knightshrine 的仪式状态推送（[yoth_knightshrine.lua:79](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L79)、[131](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L131)）。([yoth_knightmanager.lua:18-33](../../../raw/current/scripts/components/yoth_knightmanager.lua#L18-L33) |
| `RegisterPrincess(owner, hat)` | 实体, 公主帽实体 | 经世界事件 `ms_register_yoth_princess`（由公主帽 onequip 侧推送，[hats.lua:6343](../../../raw/current/scripts/prefabs/hats.lua#L6343）；登记后短延迟尝试生成，冷却中且近旁无骑士则推 `yoth_oncooldown` 供 wisecracker 用。([yoth_knightmanager.lua:168-180](../../../raw/current/scripts/components/yoth_knightmanager.lua#L168-L180) |
| `IsOnCooldown(owner)` | 返回 boolean | 玩家看 `yoth_princesscooldown_buff` debuff；非实体主人按"9 距离内最近玩家"追责，对象（如箱子）一律视为冷却中。([yoth_knightmanager.lua:50-67](../../../raw/current/scripts/components/yoth_knightmanager.lua#L50-L67) |
| `TryToSpawnKnights(owner)` | 返回 boolean | 门禁：有公主帽、非冷却、32 距内无 `gilded_knight`、主人不在船上平台、帽下无现存 knight_yoth；成功则在 6-20 距离找落点调 `SpawnKnights`。([yoth_knightmanager.lua:128-166](../../../raw/current/scripts/components/yoth_knightmanager.lua#L128-L166) |
| `SpawnKnights(hat, pos)` | 帽实体, 位置 | 临时改写 `petleash` 的 onspawnfn 设骑士名（`SetHorsemanOfTheAporkalypse`），`SpawnPetAt("knight_yoth")` 后隐藏，按 0.75s＋序号间隔 `RevealKnight`；先播 `yothknightwarningsound`。([yoth_knightmanager.lua:81-110](../../../raw/current/scripts/components/yoth_knightmanager.lua#L81-L110) |
| `RescheduleSpawnKnights(owner, timetocheck)` | 实体, 秒(可选) | 取消旧任务后按默认 5-6s 重试；`TryToSpawnKnights` 无论成败都会自续。([yoth_knightmanager.lua:116-125](../../../raw/current/scripts/components/yoth_knightmanager.lua#L116-L125) |

## 典型接入

### 让自定义神殿参与 YOTH 骑士／靴子鸟体系

- 前提与选择：宿主为世界（组件在 forest 世界）；自定义实体需在服务端推世界事件，且保持"激活时推 activated、熄灭时推 deactivated"的配对，否则表会残留。
- 生效与撤销：激活即进入 `shrines`，成为靴子鸟落点候选；`IsKnightShrineActive` 还受活动开关约束，活动外不触发。
- 原版依据：神殿侧推送与配对见 [yoth_knightshrine.lua:79](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L79)、[131](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L131)、[291](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L291)。

```lua
-- 自定义神役实体（服务端）：
-- 进入激活状态时
TheWorld:PushEvent("ms_knightshrineactivated", inst)
-- 退出或移除时
TheWorld:PushEvent("ms_knightshrinedeactivated", inst)
-- 读取端（任何服务端逻辑）
if TheWorld.components.yoth_knightmanager
    and TheWorld.components.yoth_knightmanager:IsKnightShrineActive() then
    -- 活动期存在激活神殿
end
```

- 调整与验证：`ms_register_yoth_princess` 事件的 `data.owner`／`data.hat` 缺一不可（桥接函数直接丢弃不完整数据，([yoth_knightmanager.lua:181-185](../../../raw/current/scripts/components/yoth_knightmanager.lua#L181-L185)）。

## 常见陷阱与限制

- 组件不持有神殿实体的 `onremove` 监听：神殿被砸毁/烧毁若只推 activated 不推 deactivated，`shrines` 会留下悬挂键（神殿自己在 onremove 侧补救，见注释 [yoth_knightshrine.lua:373](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L373）；自定义实体需自行保证配对。
- 生成走公主帽的 `petleash`，未登记公主（没戴/脱帽）时 `TryToSpawnKnights` 直接返回 false（[L129-131](../../../raw/current/scripts/components/yoth_knightmanager.lua#L129-L131)）；想脱离帽子生成需自己写分支而非复用本组件。
- 骑士数量上限由 `#YOTH_HORSE_NAMES` 决定，单次全部生成、错峰揭示；32 距离内已有一只 `gilded_knight` 即不再生成（[L141-143](../../../raw/current/scripts/components/yoth_knightmanager.lua#L141-L143)）。
- 无存读钩子：`OnSave`／`OnLoad` 未实现，登记表由神殿／帽子的实体存档与事件重放自然恢复，本组件不持久化任何数据。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读 components/yoth_knightmanager.lua 全文，核验 forest.lua 挂载、hats.lua 公主帽事件源、yoth_knightshrine.lua 神殿事件配对及三个读取消费者。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/yoth_hecklermanager]]、[[reference/components/petleash]]、[[reference/components/prototyper]]
