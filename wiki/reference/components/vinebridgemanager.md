---
title: "vinebridgemanager：藤桥地皮管理与生命周期"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级临时地皮管理器：在海洋格上铺 CHARLIE_VINE 藤桥地皮并保留底层地皮，按耐久网格承伤、计时破坏，存档可续接中断的破坏。"
aliases: [藤桥管理器, 藤蔓桥, vinebridgemanager, 温妮莎藤桥]
tags: [component, worldgen, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/vinebridgemanager.lua", "prefabs/world.lua", "componentutil.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vinebridgemanager（藤桥地皮管理与生命周期）

## 快速判断

- 职责与适用：世界级服务端组件（[world.lua:586](../../../raw/current/scripts/prefabs/world.lua#L586)，断言 [mastersim](../../../raw/current/scripts/components/vinebridgemanager.lua#L5)）。把指定地皮格改为 `WORLD_TILES.CHARLIE_VINE` 藤桥并在格上放 `vine_bridge_fx` 表现；承载耐久（默认 200）与方向数据，损伤到 0 或到时后恢复底层地皮。原版消费者是温妮莎技能藤桥部署（RosePoint 配置 `requiredworldcomponent = "vinebridgemanager"`，([componentutil.lua:655-660](../../../raw/current/scripts/componentutil.lua#L655-L660)）。
- 优先使用：要走原版入口 → 在 RosePoint/VineBridge 技能路径调用 `QueueCreateVineBridgeAtPoint`/`QueueDestroyForVineBridgeAtPoint`（[调用方](../../../raw/current/scripts/componentutil.lua#L663-L680)）；直接铺格用 `CreateVineBridgeAtPoint`。
- 重要副作用：铺格时经 [[reference/components/undertile]] 保存并恢复底层地皮（农田耕作回调会在 SetTile 时清 undertile，源内注明绕过方式，[顺序](../../../raw/current/scripts/components/vinebridgemanager.lua#L80-L84)）；破坏恢复后调 `TempTile_HandleTileChange` 通知压在格上的实体。
- 修改边界：数据网格在世界 `worldmapsetsize` 后才初始化（[网格](../../../raw/current/scripts/components/vinebridgemanager.lua#L22-L34)）；damage_prefab 表现生成代码整段被注释，现版本只有移除路径（[注释块](../../../raw/current/scripts/components/vinebridgemanager.lua#L231-L249)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateVineBridgeAtPoint(x, y, z, direction)` | 世界坐标、{x,z} 方向 | 换算格坐标后铺格（[入口](../../../raw/current/scripts/components/vinebridgemanager.lua#L51-L54)）。 |
| `CreateVineBridgeAtTile(tx, ty, x, z, direction)` | 格坐标 | `SetTile(CHARLIE_VINE)`＋undertile 恢复→耐久格写 `{VINEBRIDGE_HEALTH, direction}`→修正漂浮物落点→生成桥面 FX（按方向转 0/90/180/-90）（[实现](../../../raw/current/scripts/components/vinebridgemanager.lua#L71-L107)、[FX](../../../raw/current/scripts/components/vinebridgemanager.lua#L251-L268)）。 |
| `QueueCreateVineBridgeAtPoint(x, y, z, data)` | data: base_time/random_time/direction | 该格无数据时才排队；默认 0.5+0.3×random 秒后真正铺格（[排队](../../../raw/current/scripts/components/vinebridgemanager.lua#L109-L123)）。 |
| `DamageVineBridgeAtPoint/Tile(x,y,z|tx,ty, damage)` | 数值 → 新耐久或 nil | 无数据或已为 0 返回 nil；否则扣耐久（上限钳回满值）、更新表现，归零即排队破坏（[伤害](../../../raw/current/scripts/components/vinebridgemanager.lua#L202-L229)）。 |
| `QueueDestroyForVineBridgeAtPoint(x, y, z, data)` | data: destroytime/fxtime/shaketime | 标记待删、默认 2+(70±10) 帧后执行；有 fxtime 时先 `ShakeIt` 再 `TempTile_HandleTileChange_Warn` 预警（[排队破坏](../../../raw/current/scripts/components/vinebridgemanager.lua#L164-L200)）。 |
| `DestroyVineBridgeAtPoint(x, y, z, data)` | 世界坐标 → 布尔 | 非 CHARLIE_VINE 返回 false；移除伤害表现、恢复 undertile 底地皮（无记录按洞穴 IMPASSABLE/海面 OCEAN_COASTAL 兜底）、清网格、KillFX、`TempTile_HandleTileChange`（[破坏](../../../raw/current/scripts/components/vinebridgemanager.lua#L125-L162)）。 |
| `OnSave/OnLoad` | 存读钩子 | `ZipAndEncodeSaveData` 压缩存 marked_for_delete＋duration 网格；读回重启中断的破坏任务并重建桥面 FX（[存读](../../../raw/current/scripts/components/vinebridgemanager.lua#L274-L320)）。 |

调谐：`VINEBRIDGE_HEALTH=200`、`VINEBRIDGE_DURATION=18`（秒，原版部署到自动破坏的总时长，([tuning.lua:3748-3749](../../../raw/current/scripts/tuning.lua#L3748-L3749)）。

## 典型接入

### 温妮莎藤桥原版路径

- 前提与选择：服务端、世界已挂组件；温妮莎技能部署经 `Bridge_DeployCheck_Helper` 校验 `IsValidTileForVineBridgeAtPoint` 后逐格排队创建，并按 `VINEBRIDGE_DURATION` 排队破坏（[部署实现](../../../raw/current/scripts/componentutil.lua#L663-L681)）。
- 生效与撤销：每格独立计时；破坏 data 携带 `fxtime = duration + 0.25×i`、`shaketime = fxtime - 1`、`destroytime = fxtime + 70 帧`，桥面先抖动后塌陷。

```lua
-- 服务端：在海洋格铺一段临时藤桥（对照 componentutil.lua 的 RosePoint 实现）
local vm = TheWorld.components.vinebridgemanager
vm:QueueCreateVineBridgeAtPoint(x, y, z, { base_time = 0.25, random_time = 0, direction = { x = 1, z = 0 } })
vm:QueueDestroyForVineBridgeAtPoint(x, y, z,
    { fxtime = TUNING.VINEBRIDGE_DURATION, shaketime = TUNING.VINEBRIDGE_DURATION - 1,
      destroytime = TUNING.VINEBRIDGE_DURATION + 70 * FRAMES })
```

- 调整与验证：`direction` 决定桥面 FX 旋转；直接 `CreateVineBridgeAtPoint` 跳过排队立即铺格。

## 常见陷阱与限制

- **SetTile 会清 undertile**：先读当前底层再铺格，铺完立刻 `SetTileUnderneath` 恢复，这是源内注明的时序 workaround（[绕过](../../../raw/current/scripts/components/vinebridgemanager.lua#L78-L84)）。
- **破坏只认 CHARLIE_VINE 格**：对已恢复/别的地皮调 `DestroyVineBridgeAtPoint` 返回 false 不产生副作用（[守卫](../../../raw/current/scripts/components/vinebridgemanager.lua#L127-L130)）。
- **伤害表现未启用**：`SpawnDamagePrefab` 的生成段被注释，掉耐久只有桥面 FX 状态可见，无独立损坏贴图实体（[注释](../../../raw/current/scripts/components/vinebridgemanager.lua#L236-L242)）。
- **中断破坏读档续接**：`marked_for_delete` 里有标记说明破坏任务被打断，读档后按 1~10 帧随机延迟重启（[续接](../../../raw/current/scripts/components/vinebridgemanager.lua#L293-L300)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/vinebridgemanager.lua` 全部 322 行、world.lua 挂载行、componentutil.lua RosePoint 部署段（655-687）、tuning VINEBRIDGE 常量。未覆盖：`vine_bridge_fx` prefab 表现、`TempTile_HandleTileChange` 全流程、地图端 `IsValidTileForVineBridgeAtPoint` 判定。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
