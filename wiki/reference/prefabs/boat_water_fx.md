---
title: "船尾航迹水花（boat_water_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "船与水上生物身后的水花航迹：非联网动画实体，由 boattrail 组件按里程生成、boattrailmover 组件反向漂移，动画播完或 80 帧兜底移除。"
aliases: ["boat_water_fx", "船尾水花", "船航迹", "尾流特效", "boat wake fx"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/boat_water_fx.lua", "components/boattrail.lua", "components/boattrailmover.lua", "prefabs/boat.lua", "prefabs/wave.lua", "stategraphs/SGspider_water.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 船尾航迹水花（boat_water_fx）

## 快速判断

- 职责与适用：船／浪／水生生物移动时留下的**水面航迹动画实体**：非联网（工厂无 AddNetwork，源码注释标明 Non-networked entity），只有 Transform＋AnimState＋boattrailmover，无任何玩法组件；地面层海洋着色显示。[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L6-L33)
- 优先使用：查"船尾水花／航迹／尾流"选本页；生成节流与缩放配置在 [[reference/components/boattrail]]，漂移运动在 [[reference/components/boattrailmover]]。
- 重要副作用：两条移除路径并存——`animover` 即删＋80 帧（约 2.67 秒）兜底任务，寿命取二者先到；主动画 `idle_loop_1` 未显式传循环参数，且生成方常再覆写为 `idle_loop_1..3` 随机一支。[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L21-L30)、[覆写](../../../raw/current/scripts/components/boattrail.lua#L36-L37)
- 修改边界：实体非联网、persists=false，由各生成方本地生成与清理；船与浪上的生成器只装在非专用机器（`TheNet:IsDedicated()` 为假）上。[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L12-L14)、[船](../../../raw/current/scripts/prefabs/boat.lua#L435-L448)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boat_water_fx` | fn：CreateEntity → FX／NOBLOCK／ignorewalkableplatforms 标签 → SetCanSleep(false)、persists=false → Transform＋AnimState（bank `boat_water_fx`、build `boat_water_fx2`、播 `idle_loop_1` 未显式传循环参数、BOAT_TRAIL 排序 OnGround 地下层、海洋着色）→ boattrailmover 组件 → animover Remove＋80 帧兜底 Remove | 单一注册；无 AddNetwork，生成方在本机直接 SpawnPrefab 即得本地实体。[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L6-L33)、[注册](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L35) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：boattrail（宿主侧） | 宿主每帧累计位移，每走 `effect_spawn_rate`（船默认 1）单位生成一个本实体：放到行进反方向 `radius` 处、按 `scale_x/scale_z` 缩放、`anim_idx` 在 3 支动画间随机 ±1 走位后覆写播放、boattrailmover:Setup(dir, 0.5, -0.125) | 主模拟船实体（boat，Wiki 待建）；组件随实体睡眠停更；[SpawnEffectPrefab](../../../raw/current/scripts/components/boattrail.lua#L30-L42)、[OnUpdate](../../../raw/current/scripts/components/boattrail.lua#L44-L68) | [[reference/components/boattrail]]；[挂载（非专用）](../../../raw/current/scripts/prefabs/boat.lua#L435-L448) |
| 组件：boattrailmover（本体） | Setup 记反向方向并转体（-舵角+90），逐帧以 velocity 累加 acceleration（船路径 0.5→-0.125，即向前漂并减速）位移自身 | 实体自身组件，随宿主生成时装好即开始更新 | [[reference/components/boattrailmover]]；[Setup](../../../raw/current/scripts/components/boattrailmover.lua#L7-L18)、[OnUpdate](../../../raw/current/scripts/components/boattrailmover.lua#L20-L30) |
| 生成者：浪（wave） | 浪实体挂 boattrail（rate 2.5、radius 1、scale 0.75）生成小号水花 | 仅非专用机器；组件与实体都在本机 | [浪挂载](../../../raw/current/scripts/prefabs/wave.lua#L122-L127) |
| 生成者：SGspider_water | 水蜘蛛游泳态 16 帧事件在身后 0.5 距离生成，SetScale(0.7)、转体反向-90 | 主模拟生物状态图；实体仍为本地非联网 | [尾流](../../../raw/current/scripts/stategraphs/SGspider_water.lua#L240-L249) |
| 资源 | anim/boat_water_fx2.zip（bank `boat_water_fx`） | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L1-L4) |

## 典型接入

### 复用"反向漂移的水花"

- 前提与选择：本地（非联网）实体，SpawnPrefab 后无需网络配置；要运动就调 `boattrailmover:Setup(dir_x, dir_z, velocity, acceleration)`，方向是行进方向，组件内部取反。
- 生效与撤销：寿命由动画播完或 80 帧兜底决定，无需手工清理；换更长动画或去掉 animover 监听仍受 80 帧兜底限制，超过该时长需在自有工厂中同时调整定时清理。
- 原版依据：船走 boattrail 里程节流＋随机动画走位，水蜘蛛按状态图帧事件直插；两条路径都不传循环参数。[船路径](../../../raw/current/scripts/components/boattrail.lua#L30-L42)、[蜘蛛路径](../../../raw/current/scripts/stategraphs/SGspider_water.lua#L240-L249)

```lua
local fx = SpawnPrefab("boat_water_fx")
fx.Transform:SetPosition(x - dir_x * 2, 0, z - dir_z * 2)
fx.AnimState:PlayAnimation("idle_loop_2")            -- 可选：覆写动画
fx.components.boattrailmover:Setup(dir_x, dir_z, 0.5, -0.125) -- 反向漂移减速
```

- 调整与验证：velocity 单位为世界单位/秒，acceleration 为其每秒增量；专用服务器上不生成（船的 boattrail 仅非专用挂载），验证入口见工厂移除双路径（[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L29-L30)）。

## 常见陷阱与限制

- **误区：动画名带 loop 就会循环**：`idle_loop_1` 经 `PlayAnimation` 单参播放，未显式传循环参数，实际是否重复未核验；实体寿命由 animover／80 帧兜底限制，不依赖动画循环。[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L21-L30)
- **边界：非联网实体不走存档与网络**：工厂无 AddNetwork 且 persists=false（[工厂](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L12-L14)），在专用服务器上由 boat/wave 挂载判定挡住而不生成，客户端各自本地生成自家看到的水花（[船](../../../raw/current/scripts/prefabs/boat.lua#L435-L448)）。
- **边界：boattrailmover 的 Setup 方向取反**：传入的 dir 是"行进方向"，组件内部存 `-dir` 并按 -舵角+90 转体；照抄时传错方向水花会往前跑。[Setup](../../../raw/current/scripts/components/boattrailmover.lua#L7-L18)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；36 行工厂全文、boattrail／boattrailmover 两组件全文、boat 与 wave 挂载分支、SGspider_water 尾流状态已核验；动画包内容、非专用机器上三生成源的实际表现未运行核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
