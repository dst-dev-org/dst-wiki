---
title: "船炮瞄准范围圈特效（cannon_aoe_range_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "非联网的地面范围指示圈（OnGround 背景层 SortOrder 3，播 idle）：船炮瞄准时由本地玩家 boatcannonuser 生成，高尔夫球杆充能复用；同文件注册 cannon_reticule_fx（船炮 reticule 准星 prefab）。"
aliases: ["cannon_aoe_range_fx", "cannon_reticule_fx", "船炮范围圈", "瞄准范围", "船炮准星"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/cannon_aoe_range_fx.lua", "components/boatcannonuser.lua", "components/golfclub_reticule.lua", "prefabs/boat_cannon.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 船炮瞄准范围圈特效（cannon_aoe_range_fx）

## 快速判断

- 职责与适用：**非联网**的地面范围指示圈：OnGround 朝向＋世界背景层 SortOrder 3，播 `idle`（单参，未显式传循环参数）；同文件注册 `cannon_reticule_fx`（同构造换 bank/build，作船炮 reticule 组件的准星 prefab）。无声音、无玩法组件、无自删逻辑。[工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L11-L31)、[准星工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L33-L53)
- 优先使用：查"船炮瞄准范围圈／高尔夫球杆充能落点圈"选本页；瞄准流程与准星机制见 boatcannonuser／golfclub_reticule 组件页。
- 重要副作用：谁生成谁可见——实体无 Network 组件，专服与其他客户端均无此实体；生命周期完全由生成方 Remove 持有引用管理。[非联网注释](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L16-L18)
- 修改边界：两种注册名只差 bank/build 与 anim 资源（cannon_aoe_range.zip／cannon_reticule.zip）；位置、旋转、缩放由消费者设置。[注册](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L55-L56)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cannon_aoe_range_fx` | fn：Transform＋AnimState（无 Network）→ FX/NOCLICK → SetCanSleep(false)＋persists=false → bank/build `cannon_aoe_range` 播 `idle` → OnGround＋LAYER_WORLD_BACKGROUND＋SortOrder 3 | 船炮瞄准圈（原尺寸）与球杆充能圈（scale 0.4）共用。[工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L11-L31) |
| `cannon_reticule_fx` | fn_reticule：同构造，bank/build `cannon_reticule` 播 `idle` | 作为 boat_cannon 的 reticuleprefab。[工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L33-L53)、[接线](../../../raw/current/scripts/prefabs/boat_cannon.lua#L293) |

两注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：boatcannonuser | 本地玩家（`inst == ThePlayer`）切到船炮后经 0 帧任务生成于炮位、随炮旋转，`AddPlatformFollower` 跟随炮所在平台；切炮／收炮时 Remove | 组件注释明确 aim_range_fx 是"本地瞄准变量，勿在服务端代码使用"；专服与其他玩家不可见 | [生成](../../../raw/current/scripts/components/boatcannonuser.lua#L66-L84)、[本地变量注释](../../../raw/current/scripts/components/boatcannonuser.lua#L9-L13)、[ThePlayer 限定与清理](../../../raw/current/scripts/components/boatcannonuser.lua#L86-L115)、[[reference/components/boatcannonuser]] |
| 消费者：golfclub_reticule | 客户端充能时生成于目标点、按持有者朝向旋转、`SetScale(.4)`、挂平台跟随者；收杆清理 | 复用船炮瞄准圈资源做高尔夫充能显示 | [生成](../../../raw/current/scripts/components/golfclub_reticule.lua#L209-L217)、[[reference/components/golfclub_reticule]] |
| 关联 Prefab：cannon_reticule_fx | boat_cannon 声明依赖并把 reticuleprefab 设为它（准星由 reticule 组件管理） | 准星移动／有效性机制在 reticule 组件侧 | [依赖](../../../raw/current/scripts/prefabs/boat_cannon.lua#L14-L15)、[接线](../../../raw/current/scripts/prefabs/boat_cannon.lua#L293) |
| 资源 | anim/cannon_aoe_range.zip、anim/cannon_reticule.zip | 声明与使用已核验，包内容未解包 | [声明](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L1-L9) |

## 典型接入

### 复用"地面范围圈"

- 前提与选择：仅用于本地表现（本地玩家上下文或 HUD 侧）；想全服可见需换联网 prefab。
- 生效与撤销：SpawnPrefab→摆位→（可选）AddPlatformFollower；用完自己 Remove，本体不会自灭。
- 原版依据：boatcannonuser（原尺寸随炮）与 golfclub_reticule（0.4 缩放随目标）即两种官方用法（[组件实现](../../../raw/current/scripts/components/boatcannonuser.lua#L66-L84)）。

```lua
local fx = SpawnPrefab("cannon_aoe_range_fx")
fx.Transform:SetPosition(x, y, z)
fx.Transform:SetRotation(rot)
fx.AnimState:SetScale(.4, .4)
-- 可选：platform:AddPlatformFollower(fx) 跟随船移动
```

- 调整与验证：客户端本地生成即可（无网络同步需求）；贴地用背景层＋SortOrder 3（[工厂](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L26-L28)）。

## 常见陷阱与限制

- **误区：生成后全队可见**：实体无 Network 组件，专服不参与、其他客户端无此实体；只有生成端本地显示。[非联网注释](../../../raw/current/scripts/prefabs/cannon_aoe_range_fx.lua#L16-L18)
- **边界：无自删逻辑**：与 collapse／superjump 系不同，animover 不 Remove；消费者忘 Remove 会常驻（原版消费者都持引用清理）。
- **边界：动画是否重复未核验**：`idle` 单参播放；原版当静态圈用，别依赖循环。
- **边界：本页不含炮击结算**：船炮伤害、瞄准数据与 StartAiming/StopAiming 在 boatcannon／boatcannonuser 组件，见 [[reference/components/boatcannonuser]]。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；57 行工厂全文（两工厂、非联网边界）、boatcannonuser 生成与清理链（含 ThePlayer 限定与本地变量注释）、golfclub_reticule 充能圈、boat_cannon reticule 接线、NAMES 检索（无命中）已核验；anim zip 内容、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
