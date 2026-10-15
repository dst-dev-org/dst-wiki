---
title: "messagebottlemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "漂流瓶寻宝调度：按地图边缘环形布点在远海生成宝藏，管理隐士蟹指引与活跃寻宝上限，按 userid 记录玩家进度。"
aliases: ["漂流瓶寻宝", "宝藏投放", "messagebottlemanager"]
tags: [component, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/messagebottlemanager.lua", "prefabs/forest.lua", "prefabs/messagebottle.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# messagebottlemanager

## 快速判断

- 职责与适用：挂在地表世界实体（[安装](../../../raw/current/scripts/prefabs/forest.lua#L682)，[断言](../../../raw/current/scripts/components/messagebottlemanager.lua#L9)），为使用漂流瓶的玩家挑一个远海宝藏点并生成寻宝；对象是 `messagebottletreasures` 生成的宝藏链路。
- 优先使用：唯一核心入口 `UseMessageBottle(bottle, doer, is_not_from_hermit)`，漂流瓶 prefab 使用时调用（[调用](../../../raw/current/scripts/prefabs/messagebottle.lua#L53)、[实现](../../../raw/current/scripts/components/messagebottlemanager.lua#L197-L240)）。
- 重要副作用：宝藏落水后经 `on_submerge` 给水下物体开 `treasuremarked` 小地图标记；活跃寻宝标记增删经世界事件回写本组件（[L148-L171](../../../raw/current/scripts/components/messagebottlemanager.lua#L148-L171)）。
- 修改边界：仅主机模拟；玩家进度表按 userid 记忆并存档，宝藏本身不在此组件存读（[L246-L274](../../../raw/current/scripts/components/messagebottlemanager.lua#L246-L274)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `UseMessageBottle(bottle, doer, is_not_from_hermit)` | → 位置或 `nil, reason` | 隐士蟹给的瓶且未找到隐士 → 返回隐士蟹位置；活跃寻宝 < `TUNING.MAX_ACTIVE_TREASURE_HUNTS(10)` 时新建宝藏，否则随机复用现有标记位置（[L197-L240](../../../raw/current/scripts/components/messagebottlemanager.lua#L197-L240)；[常量](../../../raw/current/scripts/tuning.lua#L5519)）。 |
| `SetPlayerHasFoundHermit` / `GetPlayerHasFoundHermit` | player 实体 | 按 userid 记"见过隐士蟹"；hermitcrab prefab 装配时把自己登记到 `self.hermitcrab`（[L181-L187](../../../raw/current/scripts/components/messagebottlemanager.lua#L181-L187)、[登记](../../../raw/current/scripts/prefabs/hermitcrab.lua#L527)）。 |
| `SetPlayerHasUsedABottle` / `GetPlayerHasUsedABottle` | player 实体 | 按 userid 记"用过瓶"，存档恢复（[L189-L195](../../../raw/current/scripts/components/messagebottlemanager.lua#L189-L195)）。 |
| `GetHermitCrab()` | → 实体或 nil | 校验有效性后返回登记的隐士蟹（[L177-L179](../../../raw/current/scripts/components/messagebottlemanager.lua#L177-L179)）。 |
| `active_treasure_hunt_markers` | 实体键表 | 经 `messagebottletreasure_marker_added/removed` 世界事件维护；满员时随机挑一个仍有效的返回（[L158-L171](../../../raw/current/scripts/components/messagebottlemanager.lua#L158-L171)）。 |

## 选点规则（关键流程）

1. 环形布点：地图尺寸 ×0.85 的内接方框每边 8 点共 32 个缓存点，按索引四点一组推算（[L15-L16](../../../raw/current/scripts/components/messagebottlemanager.lua#L15-L16)、[L51-L66](../../../raw/current/scripts/components/messagebottlemanager.lua#L51-L66)）。
2. 偏移筛选：随机角在 `地图尺寸×0.15-4` 半径内找可游泳点，12 次尝试；失败沿索引环找下一候选（[L108-L135](../../../raw/current/scripts/components/messagebottlemanager.lua#L108-L135)）。
3. 距离门槛：离使用者 ≥200、离任意玩家 ≥50、离岸 ≥2（再无可游泳邻点判定）、16 半径内无大漩涡门户、1 半径内无 blocker（[L19-L30](../../../raw/current/scripts/components/messagebottlemanager.lua#L19-L30)、[L80-L103](../../../raw/current/scripts/components/messagebottlemanager.lua#L80-L103)）。
4. 生成：`messagebottletreasures.GenerateTreasure(pos, nil, nil, nil, doer)` 落位并挂 `on_submerge` 监听开标记（[L209-L213](../../../raw/current/scripts/components/messagebottlemanager.lua#L209-L213)）。

## 常见陷阱与限制

- **失败原因要接住**：选点失败返回 `nil, "NO_VALID_SPAWN_POINT_FOUND"`，满员且标记全失效返回 `nil, "STALE_ACTIVE_HUNT_REFERENCES"`；调用方（原版漂流瓶）用 reason 决定玩家反馈，自定义调用必须处理 nil（[L133-L135](../../../raw/current/scripts/components/messagebottlemanager.lua#L133-L135)、[L237](../../../raw/current/scripts/components/messagebottlemanager.lua#L237)）。
- **进度表不清理离线玩家**：`hermit_has_been_found_by`／`player_has_used_a_bottle` 按 userid 只增不减，长期服务器会缓慢增长；这是设计内行为（[L249-L273](../../../raw/current/scripts/components/messagebottlemanager.lua#L249-L273)）。
- **依赖地图边缘可游泳**：布点圈贴地图边界，超小或环海地形异常的自定义世界可能全点失败；先用控制台验证 `UseMessageBottle` 返回值再调参数。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、forest 装配行、messagebottle 调用点、hermitcrab 登记行与 MAX_ACTIVE_TREASURE_HUNTS 常量；未覆盖 messagebottletreasures 内部宝藏生成与 treasuremarked 表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/treasuremarked]]（水下标记）、[[reference/components/bottler]]（瓶子打捞端）。
