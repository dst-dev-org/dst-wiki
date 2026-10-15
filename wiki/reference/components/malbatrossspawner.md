---
title: "malbatrossspawner：海鸥 Boss 生成调度"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在森林世界上的 Malbatross（海鸥 Boss）生成器：登记鱼群、按 worldsettingstimer 计时，在有玩家靠近的鱼群旁海上生成唯一活体。"
aliases: [海鸥Boss生成器, Malbatross生成器, malbatrossspawner]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/malbatrossspawner.lua", "prefabs/forest.lua", "prefabs/malbatross.lua", "consolecommands.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# malbatrossspawner（海鸥 Boss 生成调度）

## 快速判断

- 职责与适用：服务端世界组件（构造即断言 [mastersim](../../../raw/current/scripts/components/malbatrossspawner.lua#L7)），挂在 `forest` 世界（[forest.lua:677](../../../raw/current/scripts/prefabs/forest.lua#L677）。监听世界 `ms_registerfishshoal` 等事件收集鱼群，用 [[reference/components/worldsettingstimer]] 计时后在鱼群附近海面生成 Malbatross。
- 优先使用：要调整 Malbatross 出现节奏 → 改 `MALBATROSS_SPAWNDELAY_*` 调谐值；要立即拉 Boss → 控制台 `c_summonmalbatross()`（[实现](../../../raw/current/scripts/consolecommands.lua#L1046-L1051)）或组件 `Summon(player)`。
- 重要副作用：全服同时只有一只活体（`_activemalbatross`）；Boss 被杀或移除（`malbatrosskilled`／`malbatrossremoved`）后重新开计时（[事件](../../../raw/current/scripts/components/malbatrossspawner.lua#L289-L290)）。
- 修改边界：组件不存计时值本体（由 worldsettingstimer 持久化）；洗牌后的鱼群顺序不存档（源内 FIXME 注明，存读档可改变选中鱼群，[FIXME](../../../raw/current/scripts/components/malbatrossspawner.lua#L189-L193)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate(dt)` | 组件更新 | 计时未到则停更新；否则按洗牌序列抽 `ceil(#鱼群 × 0.5)` 个测试，鱼群 `MALBATROSS_NOTICEPLAYER_DISTSQ`（20²）内有玩家即 `SummonMalbatross` 并停表（[更新](../../../raw/current/scripts/components/malbatrossspawner.lua#L133-L155)）。 |
| `Summon(target)` | debug／控制台入口 | 有鱼群时重建洗牌序列、把计时设为 5s 并启动更新；传实体时按距离排序（明确注明非正常玩法路径）（[Summon](../../../raw/current/scripts/components/malbatrossspawner.lua#L255-L278)）。 |
| `Relocate(target_malbatross)` | 传回活体实体或 nil | Boss 初始化时自调（([malbatross.lua:76-77](../../../raw/current/scripts/prefabs/malbatross.lua#L76-L77)）：把计时清零/重启，传实体时把其进食鱼群排到序列末尾并移除该 Boss，靠 onremove 重新开局。 |
| `OnSave/OnLoad/LoadPostPass` | 存读钩子 | 存 `_firstspawn`、`_timerfinished`、活体 GUID；读回后恢复计时完成态或倒计时（[存读](../../../raw/current/scripts/components/malbatrossspawner.lua#L194-L225)）。 |
| `GetDebugString()` | debug 文本 | 输出倒计时、洗牌鱼群数（[debug](../../../raw/current/scripts/components/malbatrossspawner.lua#L231-L253)）。 |

生成链：鱼群注册（`ms_registerfishshoal`）→ `TryBeginningMalbatrossSpawns` 无计时且未 pending 时开表：首次 0s，之后 `SPAWNDELAY_BASE(10 天) ± SPAWNDELAY_RANDOM(5 天)`（[开表](../../../raw/current/scripts/components/malbatrossspawner.lua#L65-L77)、[调谐](../../../raw/current/scripts/tuning.lua#L6061-L6064)）；`OnMalbatrossTimerDone` 置 pending 并启动更新（[计时回调](../../../raw/current/scripts/components/malbatrossspawner.lua#L122-L125)）。`SummonMalbatross` 复用现存实体或 `SpawnPrefab("malbatross")`，在鱼群外 10 单位可游泳偏移处落地，`RememberLocation("home")`＋`entitytracker` 记录进食鱼群，进入 `arrive` 状态（[召唤](../../../raw/current/scripts/components/malbatrossspawner.lua#L39-L63)）。

## 典型接入

### 钓鱼提运提前引出

- 前提与选择：玩家在鱼群上钩鱼会触发世界事件 `ms_shoalfishhooked`（旧）或 `ms_shoalfishhooked_redux`（带 fisher），组件在无活体且计时剩余小于 10s（或无计时）时以 `TryLuckRoll`（30% 基率，`LuckFormulas.MalbatrossSpawn`）把该鱼群插为唯一生成候选（[上钩引出](../../../raw/current/scripts/components/malbatrossspawner.lua#L108-L116)）。
- 生效与撤销：命运掷中后仍走同一条"计时到 0→玩家靠近→生成"链路；Boss 生成即占用唯一名额。

```lua
-- 服务端控制台/调试：在离玩家最近的鱼群立即进入生成流程
TheWorld.components.malbatrossspawner:Summon(ThePlayer)
```

- 调整与验证：玩家靠近判定为 20 单位（[调谐](../../../raw/current/scripts/tuning.lua#L5068)），每批最多测一半鱼群（[比例](../../../raw/current/scripts/tuning.lua#L6063)）；重生成等待约 10~15 天。

## 常见陷阱与限制

- **无鱼群则永不生成**：`_fishshoals` 为空时不开表、`Summon` 直接返回；鱼群全部移除同样停摆（[前置检查](../../../raw/current/scripts/components/malbatrossspawner.lua#L65-L66)）。
- **必须是主分片世界组件**：构造断言 mastersim，客户端与从分片不存在该组件；`c_summonmalbatross` 在其上调 `Summon`（[控制台](../../../raw/current/scripts/consolecommands.lua#L1050)）。
- **读档改变"哪群出 Boss"**：洗牌序列不持久化，存读档后随机重排（[FIXME](../../../raw/current/scripts/components/malbatrossspawner.lua#L189-L193)）；`_firstspawn` 保证首次必出。
- **等待期的鱼群增减**：pending 期间鱼群数量变化会整体重洗（[重洗](../../../raw/current/scripts/components/malbatrossspawner.lua#L83-L90)），不保证先测某群。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/malbatrossspawner.lua` 全部 293 行、`prefabs/forest.lua` 挂载行、`prefabs/malbatross.lua` Relocate 调用、`consolecommands.lua` c_summonmalbatross、`tuning.lua` 相关常量。未覆盖：malbatross prefab 的 arrive 行为、LuckFormulas.MalbatrossSpawn 公式细节。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
