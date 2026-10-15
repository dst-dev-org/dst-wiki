---
title: "月焰消退烟团（lunarflame_puff_small）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表 12 个 lunarflame_puff_* 注册（尺寸×高度 4×3）：水晶冠秃鹫进食燃烧尸体时的灭火烟团，复用附身座狼喷焰资源；纯表现无伤害。"
aliases: ["lunarflame_puff_small", "lunarflame_puff_tiny", "lunarflame_puff_med", "lunarflame_puff_large", "lunarflame_puff_tiny_low", "lunarflame_puff_tiny_high", "lunarflame_puff_small_low", "lunarflame_puff_small_high", "lunarflame_puff_med_low", "lunarflame_puff_med_high", "lunarflame_puff_large_low", "lunarflame_puff_large_high", "月焰烟团", "月焰消退", "火焰熄灭烟", "秃鹫灭火烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "stategraphs/SGbuzzard.lua", "prefabs/buzzard.lua", "componentutil.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月焰消退烟团（lunarflame_puff_small）

## 快速判断

- 职责与适用：水晶冠秃鹫进食燃烧尸体（且尸体不会变异）时的灭火烟团，12 注册为尺寸×高度完整矩阵；复用 [[reference/prefabs/warg_mutated_breath_fx|附身座狼喷焰]] 的 bank／build 但只是表现，伤害在火焰实体上。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[循环](../../../raw/current/scripts/fx.lua#L4136-L4157)
- 优先使用：查"火灭了的烟／月焰消退"选本页；火墙伤害、续时与池化选 [[reference/prefabs/warg_mutated_breath_fx|warg_mutated_breath_fx]]。
- 重要副作用：无；全员 `FinalOffset 1`，音效按尺寸三档（S／M／L 熄灭火焰声）。[循环](../../../raw/current/scripts/fx.lua#L4140-L4146)
- 修改边界：尸体位置一次性落点；动画是否循环未核验。[生成](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L814-L815)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunarflame_puff_<size><height>`（12 名） | 尺寸 tiny／small／med／large × 高度 `_low`／无后缀／`_high` 双循环 `table.insert` → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank／build=`warg_mutated_breath_fx`，anim=`flame_puff_<size><height>`；音效按尺寸：tiny／small→`flame_extinguish_S`，med→`_M`，large→`_L`；无后缀即"med 高度"（源码注释）。[双循环](../../../raw/current/scripts/fx.lua#L4136-L4157) |

定向检索简中 .po 未发现这 12 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 双循环注册 12 名，差异仅 anim 后缀与尺寸音效 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[循环](../../../raw/current/scripts/fx.lua#L4136-L4157) |
| 生成者：水晶冠秃鹫进食 | `mutatedbuzzard_gestalt`（官方名"水晶冠秃鹫"）进食尸体时，若尸体在燃烧**且 `not corpse:WillMutate()`**：先借 `_skip_extinguish_fade` 技巧灭火，再按 `GetCombatFxSize(尸体)` 的尺寸／高度生成烟团于尸体位置 | 服务端；单调用点（corpse_eat_loop 态），会变异的尸体不生成 | [进食灭火](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L802-L816)、[译名](../../../raw/current/scripts/languages/chinese_s.po#L345445-L345447) |
| 尺寸／高度映射 | `GetCombatFxSize`：尺寸按覆盖字段或物理半径／生物级别分 tiny／small／med／large；高度=水中或无 electrocute 态的植物或尸体→`low`、飞行→`high`、否则 nil（拼名无后缀） | 服务端通用 helper；`override_combat_fx_size` 等字段可覆盖 | [取尺寸](../../../raw/current/scripts/componentutil.lua#L959-L990) |
| 依赖声明 | buzzard.lua 双循环把 12 名插入 `mutated_prefabs`，随 `Prefab("mutatedbuzzard_gestalt")` 声明 | 仅声明；声明不等于生成 | [依赖](../../../raw/current/scripts/prefabs/buzzard.lua#L195-L201)、[注册](../../../raw/current/scripts/prefabs/buzzard.lua#L527-L527) |
| 同资源火焰 | 秃鹫喷出的火墙本体是 `warg_mutated_breath_fx` 实体（带真实伤害 ConfigureDamage）；烟团是消退／进食表现 | 服务端；两者 bank 相同但注册与职责不同 | [布焰](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L52-L54)、[[reference/prefabs/warg_mutated_breath_fx|附身座狼喷焰]] |

## 典型接入

### 复用消退烟团

- 前提与选择：装饰"火被灭"场景直接 `SpawnPrefab` 对应尺寸；想按目标体型自动选名，复用 `GetCombatFxSize`＋`GetLunarFlamePuffAnim` 组合（原版即此搭配）。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[拼名](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L86-L88)。

```lua
local _, sz, ht = GetCombatFxSize(corpse)
local fx = SpawnPrefab(string.format(ht and "lunarflame_puff_%s_%s" or "lunarflame_puff_%s", sz or "small", ht))
fx.Transform:SetPosition(corpse.Transform:GetWorldPosition())
```

- 调整与验证：高度名拼接里 `ht` 为 nil 时无后缀（对应"med 高度"条目）；用非法后缀会落空注册名，保持三档取值。

## 常见陷阱与限制

- **误区：烟团有伤害**：12 名均为纯表现条目；火墙伤害挂在 `warg_mutated_breath_fx` 实体的 `ConfigureDamage` 上，与进食灭火是两条独立链路——熄灭被吃的尸体不会终止场上火墙，页内不把它们串成同一机制。[布焰伤害](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L77-L79)
- **边界：仅进食路径生成**：全库检索 `GetLunarFlamePuffAnim` 只有进食一个调用点；火墙被玩家扑灭等其他熄灭场景不生成烟团。[单调用点](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L814-L814)
- **边界："med 高度"无独立后缀**：高度枚举是 `_low`／无后缀／`_high`，源码注释明确 med 高度不带标识；把 `med` 当高度值拼名会落空。[注释](../../../raw/current/scripts/fx.lua#L4137-L4137)
- **误区：与附身座狼喷焰同协议**：bank／build 相同但那是独立实体工厂（自管池化／结算／淡出），本族走通用表代理协议；不要把池化行为套到烟团上。[两套](../../../raw/current/scripts/fx.lua#L4136-L4157)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。12 名双循环配置、进食灭火链（含 hack 灭火与拼名）、GetCombatFxSize 尺寸高度映射、依赖声明与水晶冠秃鹫译名逐条核验；.po 注册名键定向核验无结果。未覆盖：`flame_puff_*` 动画资源内容、尸体尺寸覆盖字段的全部原版使用者。
- 游戏运行：未运行；烟团尺寸与音效分档表现未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
