---
title: "waterphysics"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "水面碰撞弹性数据：唯一字段 restitution 决定船对船碰撞的反弹推力倍率，默认 1，脚本侧唯一读取者是 boatphysics。"
aliases: ["水面物理弹性", "船碰撞反弹系数", "waterphysics"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/waterphysics.lua", "components/boatphysics.lua", "standardcomponents.lua", "prefabs/boat.lua", "prefabs/boatrace_seastack.lua", "prefabs/world.lua", "prefabs/yotd_boats.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# waterphysics

## 快速判断

- 职责与适用：宿主（船、比赛浮标、世界实体、海上障碍）的碰撞弹性数据，无方法、无更新逻辑；脚本侧唯一读取者是 [[reference/components/boatphysics]] 的碰撞回调，用对方 `restitution` 计算反弹推力 `push_back = restitution × max(speed, 0) × |hit_dot_velocity|`。[组件](../../../raw/current/scripts/components/waterphysics.lua#L1-L7)、[读取](../../../raw/current/scripts/components/boatphysics.lua#L120-L123)
- 优先使用：静态水面障碍用 `MakeWaterObstaclePhysics(inst, rad, height, restitution)` 一并建 Physics 与组件（[辅助函数](../../../raw/current/scripts/standardcomponents.lua#L664-L678)）；船类宿主在物理构建旁直接 AddComponent。
- 重要副作用：纯被动数据；碰撞对方没有本组件时按 1 处理（[回退](../../../raw/current/scripts/components/boatphysics.lua#L120)）；被撞毁的对方推力再乘 0.35（[折减](../../../raw/current/scripts/components/boatphysics.lua#L122-L123)）。
- 修改边界：原版宿主都在各自 mastersim 判定之前添加，双端存在（[boat](../../../raw/current/scripts/prefabs/boat.lua#L420-L421)、[判定位置](../../../raw/current/scripts/prefabs/boat.lua#L454)）；数值为无量纲倍率，不影响子弹物理的普通碰撞。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `restitution` | 数字，默认 1 | 仅被 boatphysics 碰撞回调读取；无读取者场景下字段无作用。[缺省](../../../raw/current/scripts/components/waterphysics.lua#L4) |

## 原版取值

- `boat` 0.75、`yotd_boat*` 0.75、world 实体 0.75（[boat](../../../raw/current/scripts/prefabs/boat.lua#L420-L421)、[yotd](../../../raw/current/scripts/prefabs/yotd_boats.lua#L178-L179)、[world](../../../raw/current/scripts/prefabs/world.lua#L565-L566)）。
- `boatrace_seastack*` 1.5：`1.0 + TUNING.BOATRACE_SEASTACK_EXTRA_RESTITUTION`（tuning.lua:8158 值为 0.5），比赛浮标故意弹得更高（[设置](../../../raw/current/scripts/prefabs/boatrace_seastack.lua#L95-L96)、[第二形态](../../../raw/current/scripts/prefabs/boatrace_seastack.lua#L188-L189)）。
- `MakeWaterObstaclePhysics` 调用者多取 0.75（sharkboi_ice_hazard、ocean_trawler、boatrace_checkpoint、crabking_cannontower、waterplant_baby），crabking 取 0.1 抑制反弹（[调用](../../../raw/current/scripts/prefabs/crabking.lua#L1589)）。

## 典型接入

### 给自定义海上障碍设定反弹

- 前提与选择：主机侧用辅助函数一次建好 Physics 与组件；只要船会撞上它，restitution 就体现在船被弹开的推力上。
- 生效与撤销：碰撞发生时读取，数值改大即弹得更远；组件无生命周期钩子，运行期可改字段即时生效。

```lua
AddPrefabPostInit("my_buoy", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    GLOBAL.MakeWaterObstaclePhysics(inst, 0.4, 2, 0.75)
end)
```

- 调整与验证：单位是无量纲倍率，0 相当于不反弹；超过 1 会放大推力，需自行限制。

## 常见陷阱与限制

- **只影响脚本侧船对船反弹**：读取点在 boatphysics 的碰撞回调（[回调内](../../../raw/current/scripts/components/boatphysics.lua#L120-L124)）；普通实体与障碍的子弹碰撞不经过它，改 restitution 不会改变穿模或阻挡表现。
- **world 实体也持有本组件**：世界实体挂在组件列表里（[world](../../../raw/current/scripts/prefabs/world.lua#L565-L566)），碰撞查找 `other.components.waterphysics` 时会命中它；不要假设只有船才有。
- **缺省回退为 1**：读取处对没有组件的对方硬编码回退 1（[回退](../../../raw/current/scripts/components/boatphysics.lua#L120)），不给宿主加组件也等效于默认弹性。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、boatphysics 读取与回退、MakeWaterObstaclePhysics 辅助函数、boat／yotd_boats／world／boatrace_seastack 宿主及 crabking 调用值。未覆盖：引擎侧是否有额外读取、boatphysics 碰撞回调全文、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
