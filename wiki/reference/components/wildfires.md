---
title: "wildfires"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "夏季白天高温干燥时按在线玩家逐个掷运点燃可燃物的世界宿主组件；全部逻辑为事件驱动的私有闭包，无公开方法，频率由世界设置 wildfires 难度档调节。"
aliases: ["wildfires", "野火", "夏季自燃", "森林野火管理器"]
tags: [component, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/wildfires.lua", "prefabs/forest.lua", "worldsettings_overrides.lua", "tuning.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wildfires

## 快速判断

- 职责与适用：挂在 forest 世界实体的服务端组件（构造即断言 mastersim）；夏季＋白天＋世界温度高于 `WILDFIRE_THRESHOLD`＋地面无湿无雪＋`_chance > 0` 时进入活跃态，给每个在线玩家排定时任务掷运点燃。[构造与状态](../../../raw/current/scripts/components/wildfires.lua#L7-L27) [激活条件](../../../raw/current/scripts/components/wildfires.lua#L33-L35)
- 优先使用：调频率走世界生成 `wildfires` 难度档（never／rare／often／always 覆盖 `TUNING.WILDFIRE_CHANCE`）；保护单个实体加 `wildfireprotected` 或 `fireimmune` tag。[难度档](../../../raw/current/scripts/worldsettings_overrides.lua#L2674-L2695)
- 重要副作用：活跃时每玩家每 `TUNING.WILDFIRE_RETRY_TIME`（默认 1.5 个时段）掷一次 `TryLuckRoll`，成功则在玩家 25 半径内随机点燃一个通过校验的 `burnable`；沙暴中的玩家被豁免。[点火](../../../raw/current/scripts/components/wildfires.lua#L74-L104)
- 修改边界：仅 forest 世界挂载（cave 无此组件）；`never` 档把 `_chance` 置 -1 使激活条件恒 false；无任何公开方法，模组只能经调谐、tag 与事件（`ms_lightwildfireforplayer`）介入。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L663)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)`，断言非客户端 | 私有状态：`_activeplayers`、`_scheduledtasks`、季节／白天／温度／潮湿标记、`_chance`（`WILDFIRE_CHANCE`）、`_radius = 25`、排除 tag 表。([wildfires.lua:7-27](../../../raw/current/scripts/components/wildfires.lua#L7-L27) |
| 世界事件监听 | 7 个事件 | `seasontick`／`weathertick`／`temperaturetick`／`phasechanged` 更新标记并 `ToggleUpdate` 启停任务；`ms_playerjoined`／`ms_playerleft` 维护玩家表；`ms_lightwildfireforplayer` 立即对指定玩家点火。[监听](../../../raw/current/scripts/components/wildfires.lua#L198-L204) [强制点火](../../../raw/current/scripts/components/wildfires.lua#L159-L164) |
| 逐玩家调度 | 闭包 `ScheduleSpawn`／`CancelSpawn` | 每玩家一个 `DoTaskInTime(WILDFIRE_RETRY_TIME)` 任务，点火后自续；玩家退出活跃态时统一取消。([wildfires.lua:106-117-L117](../../../raw/current/scripts/components/wildfires.lua#L106-L117) |
| `CheckValidWildfireStarter(obj)` | 选点校验闭包 | 拒绝：无效实体、`fireimmune`、树冠遮阴内、`witherable:IsProtected()`、点位世界温度 ≤ `WILDFIRE_THRESHOLD`(80)。候选：野生 `pickable:IsWildfireStarter()` 植物、无 `crop`/`growable` 的非植物、蔫掉的农作物、可 CHOP 的树。([wildfires.lua:51-72-L72](../../../raw/current/scripts/components/wildfires.lua#L51-L72) |
| 树冠遮阴检查 | 闭包 `checkforcanopyshade` | 点位 `SHADE_CANOPY_RANGE`(28) 内有 `shadecanopy` tag 或 `SHADE_CANOPY_RANGE_SMALL`(22) 内有 `shadecanopysmall` tag 即拒绝。([wildfires.lua:37-49-L49](../../../raw/current/scripts/components/wildfires.lua#L37-L49) [调谐](../../../raw/current/scripts/tuning.lua#L6334-L6335) |
| `TryLuckRoll` 掷运 | componentutil | `_chance` 经 `LuckFormulas.WildFireIgnition` 结合玩家幸运值修正。([wildfires.lua:77-79](../../../raw/current/scripts/components/wildfires.lua#L77-L79) [公式](../../../raw/current/scripts/componentutil.lua#L2178 |

## 典型接入

### 保护指定实体不被野火选中

- 前提与选择：在 prefab fn（服务端构造）加 tag；`wildfireprotected` 在 FindEntities 阶段整体排除，`fireimmune` 在点火校验阶段拒绝，按"是否还想让它被普通火焰波及"选择。
- 生效与撤销：tag 常驻生效；原版在土壤湿度 > 0 时给作物挂 `wildfireprotected`（[[reference/components/farmsoildrinker]] 与 farming_manager 源码）。
- 原版依据：排除表与校验分支见上表证据。

```lua
-- 目标实体 prefab fn 内，服务端即可（tag 不联网同步也能生效，因为选点在服务端）
inst:AddTag("wildfireprotected")    -- 搜索阶段直接排除（wildfires.lua:27）
-- 或保留可燃性、只免野火点名：
-- inst:AddTag("fireimmune")        -- CheckValidWildfireStarter 拒绝（wildfires.lua:54）
```

- 调整与验证：两种 tag 都不阻止雷击、玩家点火等其他着火途径；世界生成调整频率见世界设置 `wildfires` 难度档。

## 常见陷阱与限制

- 温度是世界值不是实体值：校验用 `GetTemperatureAtXZ(x, z)`（([wildfires.lua:57-58](../../../raw/current/scripts/components/wildfires.lua#L57-L58)），给实体加保温或降温组件不影响其被点燃资格。
- 农作物与树默认"耐火"：有 `crop` 或 `growable` 的实体只在蔫掉（`witherable:IsWithered()`）或可 CHOP（`workable` 的动作是 CHOP）时才成为候选（([wildfires.lua:60-72-L72](../../../raw/current/scripts/components/wildfires.lua#L60-L72)）；想让健康作物可燃需改 `pickable.wildfirestarter`（原版普通花朵即如此，[flower.lua:176](../../../raw/current/scripts/prefabs/flower.lua#L176）。
- `wildfirepriority` tag 的可燃物会整组优先于普通候选被随机选中（([wildfires.lua:83-90-L90](../../../raw/current/scripts/components/wildfires.lua#L83-L90)），复用前先确认搜索半径内没有不想烧的高优先实体。
- 幸运值影响点火：`LuckFormulas.WildFireIgnition` 只吃幸运不吃厄运（([componentutil.lua:2178](../../../raw/current/scripts/componentutil.lua#L2178），幸运叠高的队伍更易遭野火。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读 components/wildfires.lua 全文，核验 forest.lua 挂载、worldsettings_overrides 难度档、tuning 键值与 componentutil 掷运公式。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/burnable]]、[[reference/components/witherable]]、[[reference/components/pickable]]、[[reference/components/sandstorms]]
