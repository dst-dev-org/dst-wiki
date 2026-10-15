---
title: "piratespawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级猴子海盗组件：需猴女王在场才按玩家出海距离分区加权掷骰刷海盗船，管战利品藏匿点（pirate_stash）与船员登记。"
aliases: ["海盗刷怪", "猴子海盗", "piratespawner"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/piratespawner.lua", "prefabs/stash_map.lua", "tuning.lua", "worldsettings_overrides.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# piratespawner

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界（[forest](../../../raw/current/scripts/prefabs/forest.lua#L646)）；世界设置开启海盗袭击且存在 `monkeyqueen` 时才启动更新（[门控](../../../raw/current/scripts/components/piratespawner.lua#L256-L268)）。
- 优先使用：代码触发刷船用 `SpawnPiratesForPlayer(player, nodelivery, forcedelivery)` 或 `SpawnPirates(pt)`（[L638-L647](../../../raw/current/scripts/components/piratespawner.lua#L638-L647)）；藏匿点查询 `GetCurrentStash`（stash_map 用它定位宝图目标，[消费](../../../raw/current/scripts/prefabs/stash_map.lua#L7-L11)）。
- 重要副作用：玩家 40 距离内有存活海盗时置 `player._piratemusicstate` 真值驱动战斗音乐（[L727-L747](../../../raw/current/scripts/components/piratespawner.lua#L727-L747)）；巨耀斑海上引爆后按运气延迟刷袭击（[L435-L467](../../../raw/current/scripts/components/piratespawner.lua#L435-L467)）。
- 修改边界：船与船员的归属绑定（boatcrew/crewmember/vanish_on_sleep）由组件装配，但 AI 行为归各 prefab；组件不管海盗船的战斗推进（boatcrew 组件职责）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `shipdatas` / `SaveShipData(shipdata)` / `RemoveShipData(ship)` | `{boat, captain, crew[]}` | 在场船队登记；onremove 监听自动清员（[L124-L155](../../../raw/current/scripts/components/piratespawner.lua#L124-L155)、[L758-L773](../../../raw/current/scripts/components/piratespawner.lua#L758-L773)）。 |
| `GetCurrentStash()` / `ClearCurrentStash()` | → `pirate_stash` 实体 | 懒生成：`FindStashLocation` 选拓扑节点外缘、距玩家 ≥40 的临水点，生成后按平均玩家天数填随机战利品（[L516-L630](../../../raw/current/scripts/components/piratespawner.lua#L516-L630)、[ loot 表](../../../raw/current/scripts/components/piratespawner.lua#L553-L618)）。 |
| `StashLoot(ent)` | — | 把实体（或其容器/背包内容）送进当前藏匿点：`irreplaceable` 只掉不收，`personal_possession`／`cursed`／不可拾取物直接删除（[L21-L73](../../../raw/current/scripts/components/piratespawner.lua#L21-L73)、[L549-L551](../../../raw/current/scripts/components/piratespawner.lua#L549-L551)）。 |
| `SpawnPiratesForPlayer(player, nodelivery, forcedelivery)` | → bool | 在玩家所乘平台 38 距离外找全水且无玩家/杂物的落点刷船；`forcedelivery` 或 35% 概率走"送货"变体（boatcrew status=delivery、丢漂流瓶宝、带船炮）（[L396-L431](../../../raw/current/scripts/components/piratespawner.lua#L396-L431)、[TUNING](../../../raw/current/scripts/tuning.lua#L8219)）。 |
| `OnUpdate(dt)`（含 LongUpdate） | — | 玩家出海满 10s 宽限（GRACETIME）后按其与猴女王距离分带（INNER≤300/MID≤600/OUTTER≤800），倒计时到 0 时按带权重抽一名玩家 `TryLuckRoll(chance * PIRATE_RAIDS_CHANCE_MODIFIER)` 决定是否刷（[L649-L715](../../../raw/current/scripts/components/piratespawner.lua#L649-L715)、[分带](../../../raw/current/scripts/components/piratespawner.lua#L202-L218)、[tuning](../../../raw/current/scripts/tuning.lua#L6771-L6790)）。 |
| OnSave/OnLoad/LoadPostPass | shipdatas GUID、nextpiratechance、currentstash | LoadPostPass 重建船组件/船长/船员绑定（[L775-L855](../../../raw/current/scripts/components/piratespawner.lua#L775-L855)）。 |

## 关键流程

### 船的构成与消失

1. `spawnpirateship(pt, player)`：生成 `boat_pirate`＋旗杆＋船长 `prime_mate`（带树果溶液×2、猴桨、中帽、stash_map，全部 personal_possession）＋ 2~5 名 `powder_monkey`（数量按平均玩家天数 15/30/60 分档，运气卷再各 +1，[L283-L368](../../../raw/current/scripts/components/piratespawner.lua#L283-L368)）；38 距离内可见玩家延迟播报 + 警告音效。
2. 船组件装配 `SetPirateBoat`：boatcrew、vanish_on_sleep（消失回调把平台上的物品/海盗送藏匿点、其余生物击杀或移除）、被炮击漏水时全船撤退宣告（[L81-L122](../../../raw/current/scripts/components/piratespawner.lua#L81-L122)）。
3. 刷船间隔 `getnextmonkeytime`：`PIRATESPAWNER_BASEPIRATECHANCE`（5 天）× 随平均玩家年龄递减的 mult（0.6→0.2）（[L229-L246](../../../raw/current/scripts/components/piratespawner.lua#L229-L246)）。

## 常见陷阱与限制

- **无猴女王即静默**：组件仍在更新循环但 `OnUpdate` 开头直接返回，世界设置关闭海盗时更不会启动（[L655-L657](../../../raw/current/scripts/components/piratespawner.lua#L655-L657)、[L265-L267](../../../raw/current/scripts/components/piratespawner.lua#L265-L267)）。
- **玩家必须真在海上**：`IsVisualGroundAtPoint` 为假才计时；回陆即清计数（[L659-L679](../../../raw/current/scripts/components/piratespawner.lua#L659-L679)）。
- **藏匿点不持久化位置逻辑**：pirate_stash 实体随 GUID 存档重绑，但 `FindStashLocation` 是无限循环找点（极端地图形态可能卡主端，源码无重试上限，[L521-L544](../../../raw/current/scripts/components/piratespawner.lua#L521-L544)）。
- **世界设置只改频率**：`pirate_raids` 难度经 worldsettings_overrides 只改 `PIRATE_RAIDS_CHANCE_MODIFIER`（0.5~5），门槛仍是猴女王（[overrides](../../../raw/current/scripts/worldsettings_overrides.lua#L1293-L1304)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、stash_map 消费、tuning 分带与概率段、worldsettings_overrides 段、forest 装配行；未覆盖 boatcrew/prime_mate/powder_monkey AI 细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
