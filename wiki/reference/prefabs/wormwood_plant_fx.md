---
title: "沃姆伍德开花植物特效（wormwood_plant_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "沃姆伍德满级开花时在身边可种植地面周期生成的小植物动画：开花 plantkin 玩家靠近时切 idle 驻留，否则 ungrow 收缩后自删；变体由宿主 plantpool 加权轮换。"
aliases: ["wormwood_plant_fx", "沃姆伍德植物特效", "开花植物特效", "开花小植物"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wormwood_plant_fx.lua", "prefabs/wormwood.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 沃姆伍德开花植物特效（wormwood_plant_fx）

## 快速判断

- 职责与适用：沃姆伍德满级开花（fullbloom）状态在身边地面周期生成的小植物动画实体：bank／build `wormwood_plant_fx`、初始 `grow_1`，变体经 `SetVariation` 换 `grow_N`；无组件、无玩法作用，不会长成真植物。[工厂](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L32-L45)
- 优先使用：查"开花植物／沃姆伍德身边小植物"选本页；生成节奏与落点规则在宿主 PlantTick。
- 重要副作用：每次动画播完重查一次环境——任一开花的 `plantkin` 玩家在 2 距离内就切 `idle_N` 驻留，否则播 `ungrow_N` 后自删。[OnAnimOver](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L6-L23)
- 修改边界：变体与位置完全由生成方决定（SetVariation＋SetPosition）；FX 自身不选变体默认 1。[构造](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L53-L57)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wormwood_plant_fx` | fn：Transform＋AnimState＋Network → 标签 FX／NOCLICK／wormwood_plant_fx → 播 `grow_1` → SetPristine → 客户端早退 → 主模拟 variation=1、暴露 SetVariation、挂 animover（OnAnimOver）、`persists=false` | 单一注册；grow／idle／ungrow 各 anim 均未显式传循环参数，实际是否重复未核验。[注册](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L62)、[主模拟](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L53-L57) |

`SetVariation` 只在变体不同才重播 `grow_N`。[定义](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L25-L30)

标题中文为已核验效果描述，非官方译名（NAMES／SKIN_NAMES 未发现该 FX 注册名条目）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：wormwood.lua PlantTick | 0.25 秒周期：按开花计时档位决定概率（春天必出），不在船／平台上，附近 1 距离内上限 18 株、彼此 0.5 内上限 3 株，落点需 CanPlantAtPoint＋DeployPointClear＋非地洞 | 主模拟任务；开花期启动 | [PlantTick](../../../raw/current/scripts/prefabs/wormwood.lua#L260-L303)、[落点校验](../../../raw/current/scripts/prefabs/wormwood.lua#L287-L291) |
| fullbloom 门控 | 宿主开花阶段回调 `EnableFullBloom(stage >= 3)` 置 `fullbloom` 并启动 planttask；退出开花取消 | stage 来自 [[reference/components/bloomness#关键接口]] | [EnableFullBloom](../../../raw/current/scripts/prefabs/wormwood.lua#L364-L367)、[任务启动](../../../raw/current/scripts/prefabs/wormwood.lua#L378-L384)、[阶段回调](../../../raw/current/scripts/prefabs/wormwood.lua#L455-L468) |
| 变体轮换 | 生成时从 `inst.plantpool` 加权随机取变体（rnd² 偏向队首）再插回队尾 | plantpool 属宿主字段 | [加权随机](../../../raw/current/scripts/prefabs/wormwood.lua#L296-L300) |
| 驻留条件 | OnAnimOver 查 AllPlayers：`fullbloom`＋`plantkin`＋非死亡／鬼魂＋可见＋距离²<4 → 播 `idle_N`；否则 `ungrow_N` | `plantkin` 是沃姆伍德身份标签；fullbloom 是宿主字段不是实体标签 | [OnAnimOver](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L10-L22) |
| 动画资源 | bank／build `wormwood_plant_fx`，anim 组 grow_1／idle_N／ungrow_N（N 为变体号） | 资源包内容与变体总数未解包 | [设置](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L43-L45) |

## 典型接入

### 复用为开花跟随装饰

- 前提与选择：主模拟侧生成后 `SetPosition`＋`SetVariation(n)`；想让它驻留，保证范围内有满足条件的 plantkin 玩家，否则播完 grow 就会 ungrow 消失。
- 生效与撤销：自删路径唯一（ungrow 播完），提前清理需自己持引用 Remove。

```lua
local fx = SpawnPrefab("wormwood_plant_fx")
fx.Transform:SetPosition(x, 0, z)
fx:SetVariation(2)
```

- 调整与验证：变体号必须与动画里的 N 对齐，超界 anim 未核验；驻留判定每轮动画结束才刷新，不是每帧。[判定](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L11-L19)

## 常见陷阱与限制

- **误区：fx 是真实植物**：无 growable／pickable／workable 组件，只有动画；不可采集或破坏。[构造](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L32-L41)
- **边界：驻留依赖"开花的 plantkin 玩家"而非宿主**：宿主死后或玩家走开，现有 fx 下一轮就 ungrow；`fullbloom` 判定读的是玩家字段 `v.fullbloom`。[驻留](../../../raw/current/scripts/prefabs/wormwood_plant_fx.lua#L12-L19)
- **边界：船上不开花植物**：PlantTick 对 `GetCurrentPlatform()` 直接跳过，船／平台周边不会生成；fx 自身无漂浮处理。[跳过](../../../raw/current/scripts/prefabs/wormwood.lua#L271-L274)
- **边界：宿主中断不删 fx**：PlantTick 停止（退出开花时取消任务 [取消点](../../../raw/current/scripts/prefabs/wormwood.lua#L402-L404)）后已生成的 fx 走完自身 ungrow 才消失，最短一轮动画时长未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；62 行工厂全文、PlantTick 生成规则与落点校验、fullbloom 门控与任务启动、变体轮换、译名检索已核验；动画资源内容、各变体动画时长、plantpool 初值填充未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
