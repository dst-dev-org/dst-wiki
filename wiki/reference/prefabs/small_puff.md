---
title: "变身与出生烟（small_puff）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用烟团表现：变身 helper DoChangePrefab 的标准开场烟，以及狼犬月变、蘑菇树移除、宠物出生等十余处直接落点；无特效组件，声音为死亡扑扑声。"
aliases: ["small_puff", "变身烟", "出生烟", "小烟团", "扑扑烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "standardcomponents.lua", "prefabs/hound.lua", "prefabs/mushtree.lua", "prefabs/lavae_tooth.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 变身与出生烟（small_puff）

## 快速判断

- 职责与适用：通用小烟团表现，单注册；最常见用途是实体变身／更换 prefab 时的开场烟（DoChangePrefab helper），另有月变、移除、出生等十余处直接落点。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L193-L198)
- 优先使用：查"变身烟／出生烟／小烟团"选本页；与变身 prefab 替换的联动实现在 standardcomponents 内部（未导出，见典型接入的实现参照）。
- 重要副作用：无；配音效 `dontstarve/common/deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L197-L197)
- 修改边界：一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/standardcomponents.lua#L1222-L1222)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `small_puff` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank=`small_puff`、build=`smoke_puff_small`、anim=`puff`。[条目](../../../raw/current/scripts/fx.lua#L193-L198) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L193-L198) |
| 生成者：变身 helper | `DoChangePrefab(inst, newprefab, haunter, nofx)`：换 prefab 前在原位置生成（`nofx` 可关），新实体落在同一点 | 服务端；作祟变身（haunter）与普通替换共用 | [DoChangePrefab](../../../raw/current/scripts/standardcomponents.lua#L1219-L1226) |
| 生成者：直接落点（代表例） | 狼犬月变旧位置 [hound](../../../raw/current/scripts/prefabs/hound.lua#L725-L728)、蘑菇树移除路径 [mushtree](../../../raw/current/scripts/prefabs/mushtree.lua#L230-L231)、熔蜗牛蛋孵化宠物 [lavae_tooth](../../../raw/current/scripts/prefabs/lavae_tooth.lua#L15-L18) | 服务端；各条件在调用方，全库约 24 文件引用（代表例非穷举） | — |
| 资源 | bank=`small_puff`、build=`smoke_puff_small` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L193-L198) |

## 典型接入

### 变身开场烟（实现参照）

- 前提与选择：`DoChangePrefab` 是 standardcomponents.lua 内的 **local 函数，未导出**——原版经作祟变身等内部流程使用；普通模组不能直接调用，应自行组合"生成烟＋新实体同点替换"两步（或复用游戏暴露的变身入口）。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[DoChangePrefab](../../../raw/current/scripts/standardcomponents.lua#L1219-L1226)。

```lua
-- 实现参照（standardcomponents 内部行为）：烟与新实体同点
SpawnPrefab("small_puff").Transform:SetPosition(x, y, z)
local new = SpawnPrefab("newprefab")
new.Transform:SetPosition(x, y, z)
```

- 调整与验证：helper 的 `newprefab` 可传表（随机选一）、`nofx` 可关烟；自行组合时这两点要自己实现。

## 常见陷阱与限制

- **边界：与 dirt_puff 同 build 不同注册**：`smoke_puff_small` 资源同时用于 [[reference/prefabs/dirt_puff|dirt_puff]]（有 FinalOffset 1、无声）；资源相同不代表表现或语义一致，选名按消费者语义走。
- **边界：DoChangePrefab 即时替换**：烟与新实体同帧生成（无错位延迟）；需要"先烟后人"的自定义节奏要自行拆分。[实现](../../../raw/current/scripts/standardcomponents.lua#L1219-L1226)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、DoChangePrefab helper、hound／mushtree／lavae_tooth 三个代表例核验；全库约 24 文件引用仅列代表例。未覆盖：`smoke_puff_small` 动画资源内容、其余 20+ 调用点逐条审阅、晚加入时序。
- 游戏运行：未运行；烟团视觉未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
