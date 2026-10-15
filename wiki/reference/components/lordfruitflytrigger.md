---
title: "lordfruitflytrigger"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "果蝇王的范围触发器（未接线）：扫描 15 距离内带 _activatefn 的实体并触发一次，死亡暂停、复活恢复；当前版本无任何挂载者。"
aliases: ["lordfruitflytrigger", "果蝇王触发器"]
tags: [component, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/lordfruitflytrigger.lua", "prefabs/farm_plants.lua", "components/singingshelltrigger.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lordfruitflytrigger

## 快速判断

- 职责与适用：与 [[reference/components/singingshelltrigger]] 同构的"靠近即触发"组件：每帧扫描 `trigger_range`（15）内实体，新进入者调用其 `v._activatefn(v, self.inst)`，离开再进入会重复触发。[扫描循环](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L62-L79)
- 优先使用：**当前快照内无任何消费者**——全库 grep 仅组件自身引用，没有 `AddComponent("lordfruitflytrigger")`，也没有读取其标签或字段的代码；不能作为可靠接入点，仅作考古/预研参考。
- 重要副作用：构造即加 `lordfruitflytrigger` 标签、常驻更新（构造里 farming_manager 分支与无条件调用双重 `StartUpdating`，靠 `updating` 标志幂等）；监听世界事件 `ms_fruitflytimerfinished` 清空重叠表。[构造](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L19-L39)
- 修改边界：组件按"挂在果蝇王身上"设计——`death` 暂停扫描、`respawnfromghost` 恢复；但 `prefabs/fruitfly.lua` 的 lordfruitfly 构建并未挂它。[事件钩子](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L37-L38)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `trigger_range` | number（默认 15） | 扫描半径；`findentitiesfn` 默认按 `fruitflyspawner` 标签过滤目标。[默认值](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L22-L23) |
| `findentitiesfn` | `function(inst, range) -> 实体列表` | 可覆盖的目标筛选；默认只找带 `fruitflyspawner` 标签的实体。[默认实现](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L1-L5) |
| `StartUpdating` / `StopUpdating` | `-> nil` | 幂等启停；死亡/复活钩子内部调用。[实现](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L48-L60) |
| `OnUpdate` | 更新回调 | 首次进入触发 `_activatefn`；上一帧在、这一帧仍在的实体只保持标记不重复触发。[触发逻辑](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L71-L78) |
| `ms_fruitflytimerfinished` | 世界事件（监听） | 果蝇重生计时结束后清空 `overlapping`，使已触发过的目标可再次触发；事件由 `farming_manager.lua:67` 推送。[监听](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L35-L36) |

## 典型接入

### 设计意图还原（未接线，勿当现成能力）

- 前提与选择：目标实体需暴露 `inst._activatefn`；原版对应点是农场作物——`farm_plants.lua:839` 设 `inst._activatefn = PushFruitFlySpawnerEvent`，其实现推 `ms_fruitflyspawneractive` 事件（`farm_plants.lua:680-683`）。[作物侧钩子](../../../raw/current/scripts/prefabs/farm_plants.lua#L839)
- 生效与撤销：若要启用，需在 lordfruitfly（或自定义实体）master postinit 手动挂组件并保证目标带 `fruitflyspawner` 标签；无存读钩子，纯运行期状态。

```lua
-- 复刻设计意图（当前原版未接线；自行验证）
inst:AddComponent("lordfruitflytrigger")
inst.components.lordfruitflytrigger.trigger_range = 15
```

- 调整与验证：改 `findentitiesfn` 可换目标集；触发语义固定为"进入范围一次"，重复触发依赖 `ms_fruitflytimerfinished` 清表。

## 常见陷阱与限制

- **以为挂了就会生效：** 该组件在 steam-buildid:26230496 内是孤儿组件，无 AddComponent 调用、无读者；农场催果蝇的实际链路是 farming_manager 直接按 `ms_fruitflyspawneractive` 事件与计时器运作（`components/farming_manager.lua:220-228`）。[实际链路](../../../raw/current/scripts/components/farming_manager.lua#L220-L228)
- **目标缺 `_activatefn`：** 扫描到无钩子的 `fruitflyspawner` 实体会在调用处报错；沿用前确认目标已设钩子。[调用点](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L74)
- **与 singingshelltrigger 的差异：** 后者已接线（玩家挂载，`player_common.lua:2845`，触发唱石 `PlaySound`），本组件触发半径与目标集不同，二者不可混用。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（80 行）；全库 grep（大小写不敏感）确认 `lordfruitflytrigger` 仅组件文件自引用，无挂载者与读者；核对 farm_plants `_activatefn` 与 farming_manager 事件链、singingshelltrigger 对照。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
