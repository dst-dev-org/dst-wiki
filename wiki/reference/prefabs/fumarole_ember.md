---
title: "喷气孔与查理仪式圈特效（fumarole_ember 等）"
created: "2026-11-01"
updated: "2026-11-07"
type: prefab
summary: "喷气孔余烬、烹熟烟和查理仪式的双层圆环共四个通用表注册。"
aliases: ["fumarole_ember", "fumarole_cook_fx", "charlie_circle_spawn_fx", "charlie_circle_spawn_ground_fx", "喷气孔余烬", "喷气孔烹熟烟", "查理仪式圈"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/trap_fumarole.lua", "components/trapfumaroleburning.lua", "components/charliecutscene.lua", "prefabs/atrium_gate.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 喷气孔与查理仪式圈特效（fumarole_ember 等）

## 快速判断

- 四名是通用表表现；喷气孔热伤害、烹熟与仪式启用由各自生成者处理。代理、本地表现与清理见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 名称：四名在简中 `.po` 无显示名键；标题“查理”取角色官方名（[po](../../../raw/current/scripts/languages/chinese_s.po#L340789-L340792)），其余中文为按条目和调用语境核验的效果描述。

## 注册与构造

| 注册名 | bank／build；anim | 其他字段及证据 |
|---|---|---|
| `fumarole_ember` | `trap_fumarole`／同左；`fire_pre` | animqueue、bloom；fn 设光覆盖 0.3、世界背景层与排序 3，用伪随机追加 1～5 段 loop 后接 pst，随机缩放与水平翻转；源码种子表达式为 `z * w + z`。无 sound。[条目](../../../raw/current/scripts/fx.lua#L3966-L3994) |
| `fumarole_cook_fx` | `lavaarena_creature_teleport_smoke_fx`／同左；函数随机 `smoke_1`～`3` | fn 随机 0.35～0.60 缩放并可能水平翻转，无 sound。[条目](../../../raw/current/scripts/fx.lua#L3997-L4004) |
| `charlie_circle_spawn_fx` | `charlie_basic`／同左；`circle_spawn_fx` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L4007-L4010) |
| `charlie_circle_spawn_ground_fx` | `charlie_basic`／同左；`circle_spawn_fx_ground` | 无 sound；fn=GroundOrientation，贴地背景层。[条目](../../../raw/current/scripts/fx.lua#L4013-L4017)、[helper](../../../raw/current/scripts/fx.lua#L24-L27) |

## 关联机制导航

| 生成者／触发 | 条件、落点与证据 |
|---|---|
| 喷气孔余烬 | 陷阱已部署且温度达热档时每 0.5 秒更新；未休眠时温度档 2／3 分别以 5%／10% 概率在孔附近随机 x/z、y=0 生成余烬。热量与伤害在目标组件，不由余烬执行。[更新](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L54-L85)、[启动条件](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L88-L105) |
| 烹熟烟 | `trapfumaroleburning` 累计点燃时间超过阈值、目标有 cookable 时烹熟一件，将 FX 代理父挂新熟食；调用者播放烹饪声并移除原食材。[烹熟与生成](../../../raw/current/scripts/components/trapfumaroleburning.lua#L107-L127) |
| 查理仪式 | `StartSocketingGateWithDelay` 在启用仪式前 71 帧安排圆环；按仪式角度在半径 725/150 的三个位置各生成一对，世界 y=0、不父挂；声音由生成者播放。`atrium_gate` 的数组为依赖声明。[生成](../../../raw/current/scripts/components/charliecutscene.lua#L427-L440)、[调度](../../../raw/current/scripts/components/charliecutscene.lua#L447-L452)、[声明](../../../raw/current/scripts/prefabs/atrium_gate.lua#L27-L28) |

## 常见陷阱与限制

- 余烬 `animqueue` 是队列播放，不是五个实体；伪随机参数按本地表现创建时的位置计算，不据名称推断不同位置必然不同。
- 烹饪及灼伤由陷阱和组件执行；直接生成视觉烟不会烹熟物品。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。四项 bank/build/anim/sound/fn 与生成点已核；`raw/current/scripts` 全树检索完整名及 `fumarole_`、`charlie_circle` 前缀，区分依赖声明与生成。动画素材、游戏表现及模组动态调用未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
